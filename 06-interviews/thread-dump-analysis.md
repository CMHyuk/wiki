# Thread Dump는 보통 어떤 상황에서 분석하나요?

> 2026-08-07 · 데일리 면접 질문

## 답변

Thread Dump는 특정 시점에 JVM 안의 **모든 Java 스레드의 상태와 stack trace를 스냅샷으로 찍은 자료**입니다. `jstack <pid>`, `jcmd <pid> Thread.print -l`, Unix 계열이라면 `kill -3 <pid>`(SIGQUIT), 또는 Spring Boot Actuator의 `/actuator/threaddump` 같은 방법으로 뜰 수 있습니다.

주로 분석하는 상황은 네 가지입니다. 첫째, **API 응답이 갑자기 느려지거나 멈춘 것처럼 보일 때** — 로그에는 에러가 없는데 요청이 처리되지 않는 경우, 스레드들이 어디서 멈춰 있는지 확인합니다. 둘째, **CPU 사용률이 비정상적으로 높을 때** — 어떤 스레드가 CPU를 태우고 있는지 찾습니다. 셋째, **thread pool 고갈이 의심될 때** — Tomcat worker나 커스텀 풀의 스레드가 전부 어딘가에 붙잡혀 있어 새 요청을 못 받는 상황입니다. 넷째, **deadlock이 의심될 때**입니다.

덤프를 열면 각 스레드의 상태가 보이는데, 실무에서 주로 보는 상태는 `RUNNABLE`, `WAITING`, `TIMED_WAITING`, `BLOCKED` 네 가지입니다(`Thread.State`에는 `NEW`, `TERMINATED`도 있습니다). 주의할 점은 `RUNNABLE`이 "CPU를 쓰는 중"만 의미하지 않는다는 것입니다. socket read 같은 **native I/O 대기도 JVM 관점에서는 `RUNNABLE`로 표시**되기 때문에, 상태만 보고 CPU 문제로 단정하면 오진합니다.

분석의 핵심은 개별 스레드보다 **패턴**입니다. 같은 stack trace에 많은 스레드가 몰려 있으면 병목 후보입니다. 예를 들어 수십 개의 스레드가 DB 커넥션 획득(`HikariPool.getConnection`)에서 대기 중이면 커넥션 풀 고갈(풀 크기 부족, 커넥션 leak, 느린 쿼리, DB 측 lock 등 원인은 더 나눠봐야 합니다), 외부 API의 socket read에 몰려 있으면 외부 응답 지연(타임아웃 미설정이거나 과도하게 긴 경우 특히 위험), 특정 lock의 `BLOCKED`에 몰려 있으면 동기화 병목을 의심할 수 있습니다. 다만 idle 상태의 thread pool worker가 작업 큐 대기에 몰려 있는 것처럼 **정상적으로 같은 스택에 몰리는 경우도 많으므로**, 요청량·응답 시간·풀 active count 같은 지표와 함께 봐야 합니다.

Deadlock의 경우 JVM이 덤프 하단에 `Found one Java-level deadlock` 섹션으로 **어떤 스레드가 어떤 lock을 잡은 채 어떤 lock을 기다리는지** 순환 관계를 직접 보여주기 때문에 비교적 명확하게 확인됩니다. 다만 이는 JVM 수준의 lock(monitor, ownable synchronizer)에 한정된 이야기이고, DB lock·분산 락·커넥션 풀 상호 대기 같은 **논리적 교착은 이 섹션에 잡히지 않으므로** 스레드들이 무엇을 기다리는지 직접 읽어야 합니다.

마지막으로, 한 번의 덤프는 "그 순간의 사진"일 뿐이라 정상적인 대기와 진짜 멈춤을 구분할 수 없습니다. 그래서 **5~10초 간격으로 3~5회 찍어서 비교**하고, 여러 덤프에 걸쳐 같은 지점에 계속 머물러 있는 스레드가 있는지 봅니다. 같은 작업 스택에 계속 머물러 있으면 멈춤·심한 지연을 의심하되 정상 대기(idle worker)는 걸러내야 하고, 반대로 매번 다른 지점이라고 정상인 것도 아닙니다 — busy loop나 retry 폭주처럼 "계속 움직이지만 장애"인 경우도 있습니다. 운영 환경에서 덤프 자체는 대체로 안전하지만 순간적인 safepoint 정지가 있으므로 과도하게 반복해서 뜨지는 않습니다.

## 꼬리 질문

### Thread Dump에서 BLOCKED와 WAITING은 어떻게 다른가요?

- **`BLOCKED`**: `synchronized` 블록/메서드에 진입하려는데 **다른 스레드가 monitor lock을 잡고 있어서** 못 들어가는 상태입니다. 덤프에 `waiting to lock <0x...>`로 표시되고, 누가 그 lock을 `locked <0x...>`로 잡고 있는지 추적할 수 있습니다. BLOCKED가 다수면 lock 경합 문제입니다.
- **`WAITING`**: 스레드가 **스스로** `Object.wait()`, `LockSupport.park()`, `Thread.join()` 등을 호출해 다른 스레드의 신호를 기한 없이 기다리는 상태입니다. thread pool의 유휴 스레드가 작업 큐에서 대기하는 것이 전형적인 예로, WAITING 자체는 정상인 경우가 많습니다.
- `TIMED_WAITING`은 WAITING과 같지만 `sleep(n)`, `wait(timeout)`처럼 기한이 있는 대기입니다.
- 덤프에서 구분할 때는 표시도 다릅니다: `synchronized` 계열은 `waiting to lock` / `waiting on <monitor>`, `java.util.concurrent` 계열은 `parking to wait for <ownable synchronizer>`로 나타납니다.
- 요약하면 BLOCKED는 "lock을 못 잡아서 밀려남(수동적)", WAITING은 "신호를 기다리려고 스스로 멈춤(능동적)"입니다.

### Deadlock 상황에서는 thread state가 어떻게 나타나나요?

- `synchronized` 기반 deadlock이면 관련 스레드들이 서로의 monitor lock을 기다리며 **`BLOCKED`** 상태로 나타나고, JVM이 덤프 끝에 `Found one Java-level deadlock` 섹션으로 순환 관계(A가 lock1을 잡고 lock2 대기, B가 lock2를 잡고 lock1 대기)를 출력해 줍니다.
- `ReentrantLock` 같은 `java.util.concurrent` lock 기반 deadlock이면 내부적으로 `LockSupport.park()`를 쓰기 때문에 **`WAITING`** 상태로 보입니다. 이 경우 `jstack -l`(또는 `jcmd Thread.print -l`)로 ownable synchronizer 정보까지 떠야 감지가 잘 되고, BLOCKED만 찾으면 놓칠 수 있습니다. 또 `Condition.await()`, `CountDownLatch`, semaphore 대기처럼 lock 순환이 아닌 대기는 자동 감지에 안 잡히는 경우가 많습니다.
- 여러 번 덤프를 떠도 해당 스레드들의 stack trace가 전혀 변하지 않는 것은 deadlock의 특징이지만, 그것만으로는 충분하지 않습니다 — 긴 I/O 대기나 느린 쿼리도 같은 스택이 반복될 수 있으므로 lock 소유/대기 관계까지 확인해야 합니다.
- 참고로 **starvation**(스레드가 자원을 얻지 못해 계속 일을 못 하는 상태, pool 고갈 포함)과는 구분해야 합니다. deadlock은 서로가 서로를 기다리는 순환 대기라 영원히 안 풀리고, starvation은 자원 부족·불공정 배분이 원인이라 자원이 풀리면 진행됩니다.

### CPU 사용률이 높을 때 thread dump와 CPU profiling은 어떻게 함께 보나요?

- 먼저 OS 레벨에서 `top -H -p <pid>`로 **CPU를 많이 쓰는 스레드의 TID(스레드 ID)** 를 찾고, 이를 16진수로 변환해 thread dump의 `nid=0x...`와 매칭하면 어떤 자바 스레드인지 특정할 수 있습니다.
- 이때 CPU를 태우는 스레드는 `RUNNABLE` 상태입니다(역은 성립하지 않습니다 — native I/O 대기도 `RUNNABLE`로 보입니다). 무한 루프, 비효율적인 정규식/직렬화 같은 원인을 stack trace로 짐작할 수 있고, GC 스레드가 상위에 오면 GC가 원인일 수 있는데 이 경우 판단은 GC 로그와 `jstat`이 더 직접적입니다.
- 다만 thread dump는 스냅샷이라 "누적으로 어디에 시간을 쓰는지"는 못 보여줍니다. 그래서 원인 지점을 정확히 잡으려면 async-profiler, JFR(Java Flight Recorder) 같은 **샘플링 프로파일러로 flame graph를 떠서** hot path를 확인하는 것으로 이어갑니다. 덤프는 빠른 1차 스크리닝, 프로파일링은 정밀 분석이라는 관계입니다.

### Thread pool 고갈이 의심되면 어떤 stack trace를 확인하나요?

- 해당 풀 이름(예: Tomcat의 `http-nio-8080-exec-*`)으로 스레드를 필터링해서, **몇 개가 있고 각각 무엇을 하고 있는지** 셉니다.
- 유휴 여력이 있다면 일부는 작업 큐 대기(`WAITING` at `getTask`/`park`)로 보이는 게 보통인데, 고갈 상태라면 대부분이 실제 작업의 stack trace를 들고 있습니다. 그때 **다수가 공통으로 멈춰 있는 지점** — DB 커넥션 획득 대기, 외부 API socket read, 특정 lock — 이 고갈의 원인일 가능성이 높습니다. (풀 구조에 따라 보이는 패턴은 다릅니다 — Netty 같은 event loop 기반은 worker 수 자체가 적어 해석 방식이 다릅니다.)
- 즉 "풀이 작아서"가 아니라 대부분 "스레드가 무언가에 붙잡혀 반납이 안 돼서"이므로, 붙잡고 있는 자원 쪽(타임아웃 미설정, 커넥션 풀 크기, 느린 쿼리)을 함께 봐야 합니다.

### Thread Dump만으로 알기 어려운 정보는 무엇이고 어떤 지표를 함께 봐야 할까요?

- **시간 축 정보**: 덤프는 순간 스냅샷이라 각 구간에 얼마나 오래 머물렀는지, 누적 소요 시간은 알 수 없습니다 → APM 트레이스, JFR/async-profiler로 보완합니다.
- **메모리/GC 상황**: heap 사용량, GC pause는 안 보입니다. GC pause 중에는 애플리케이션 스레드가 전부 멈추므로 "느림"의 원인이 스레드가 아니라 GC일 수 있습니다 → GC 로그, heap dump, `jstat`을 함께 봅니다.
- **시스템 자원**: CPU, 디스크 I/O, 네트워크, 컨테이너 CPU throttling 여부 → OS 지표(top, iostat)와 인프라 모니터링을 봅니다.
- **애플리케이션 지표**: 요청량, 응답 시간 분포, 커넥션 풀 사용률(HikariCP active/idle/pending), 스레드 풀 큐 길이 같은 메트릭과 대조해야 "덤프에서 본 패턴"이 실제 장애와 연결되는지 확인할 수 있습니다.
- 결국 thread dump는 "지금 스레드들이 어디에 있는가"를 보여주는 도구이고, "왜 그렇게 됐는가"는 메트릭·로그·프로파일링과 조합해서 판단합니다.

## 한 줄 요약

Thread Dump는 특정 시점 JVM 전체 스레드의 상태와 stack trace 스냅샷으로, 응답 지연·CPU 급등·thread pool 고갈·deadlock 상황에서 분석한다. 개별 스레드보다 같은 지점에 몰린 패턴을 보는 것이 핵심이고, 한 번이 아니라 간격을 두고 여러 번 찍어 "계속 같은 자리에 있는 스레드"를 찾아야 하며, 시간 축·메모리·시스템 지표는 다른 도구로 보완한다.
