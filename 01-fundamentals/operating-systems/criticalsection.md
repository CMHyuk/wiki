## 하나의 객체를 두 개의 스레드가 접근할 때 생긴 일

```java
public void increment() {
    state++;
}

LOAD state to R1
R1 = R1 + 1
STORE R1 to state
```

### race condition(경쟁 조건)

* 여러 프로세스/스레드가 동시에 같은 데이터를 조작할 때 타이밍이나 접근 순서에 따라 결과가 달라질 수 있는 상황

### 동기화

* 여러 프로세스/스레드를 동시에 실행해도 공유 데이터의 일관성을 유지하는 것

### critical section (임계 영역)

* 공유 데이터의 일관성을 보장하기 위해 하나의 프로세스/스레드만 진입해서 실행 가능한 영역

### critical section problem의 해결책이 되기 위한 조건
1. mutual exclustion (상호 배제)
    1. 한 번에 하나의 프로세스/스레드만 실행 가능
2. progress (진행)
    1. 만약 임계 영역이 비어있고 어떤 프로세스/스레드가 임계 영역에 들어가길 원하면 실행될 수 있어야 함
3. bounded waiting (한정된 대기)
    1. 어떤 프로세스/스레드가 무한정 대기하면 안됨