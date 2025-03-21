### 뮤텍스, 세마포어, 모니터

**경쟁 상태를 해결하는 법**
- **상호 배제**
    - 한 프로세스가 임계 영역에 들어갔을 때 다른 프로세스는 들어갈 수 없음
- **한정 대기**
    - 특정 프로세스가 임계영역 진입을 요청한 후 해당 요청이 승인되기 전까지 다른 프로세스가 임계영역에 진입하는 횟수를 제한하는 것을 말하며 이를 통해 특정 프로세스가 영원히 임계 영역에 들어가지 못하게 하는 것을 방지
- **진행의 융통성**
    - 만약 어떠한 프로세스도 임계영역을 사용하지 않는다면 임계영역 외부의 어떠한 프로세스도 들어갈 수 있으며 이 때 프로세스끼리 서로 방해하지 않는 것을 말함

**뮤텍스**
- 공유 자원을 lock()을 통해 잠금설정하고 사용한 후에 unlock()을 통해 잠금해제가 되는 객체 lock을 기반으로 경쟁상태를 해결한다. 잠금이 설정되면 다른 프로세스나 스레드는 잠긴 코드 영역에 접근할 수 없고 해제는 그와 반대가 된다. 한번에 하나의 프로세스만 임계영역에 있을 수 있다.

**세마포어**
- 일반화된 뮤텍스를 말한다. 간단한 정수 S와 두 가지 함수 wait() 및 signal()로 공유 자원에 대한 접근을 처리한다. 이를 통해 여러 프로세스가 동시에 임계영역에 접근할 수 있다.
- S는 현재 쓸 수 있는 공유자원의 수
- wait()은 S를 1씩 감소시킨다. 감소시키다 음수가 되면 공유자원을 쓸 수 없기 때문에 프로세스는 차단되며 대기열에 프로세스를 집어넣는다.
- signal()은 S를 1씩 증가시킨다. 공유자원을 프로세스가 다 쓴 상태를 말한다. 이때 만약 S가 0이하라면 대기열에 있던 프로세스가 동작하게 된다.

**바이너리 세마포어**
- 0과 1 두 가지 값만 가질 수 있는 세마포어

**카운팅 세마포어**
- 여러 개의 값을 가질 수 있는 세마포어

**뮤텍스 vs 세마포어**
- '잠금 메커니즘과 '신호 메커니즘'을 사용한 것이 차이

**모니터**
- 둘 이상의 스레드나 프로세스가 공유 자원에 안전하게 접근할 수 있도록 해당 접근에 대해 인터페이스만 제공하는 객체, 이를 통해 공유자원에 대한 작업들을 순차적으로 처리한다.
