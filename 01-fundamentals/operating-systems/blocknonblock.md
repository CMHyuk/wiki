**블로킹과 논블로킹은 A 함수가 B 함수를 호출했을 때,제어권을 어떻게 처리하느냐에 따라 달라짐**

### 블로킹

* 블로킹은 A 함수가 B 함수를 호출하면,제어권을 A가 호출한 B 함수에 넘김

### 논블로킹

* 논블로킹은 A함수가 B함수를 호출해도 제어권은 넘기지 않음 
* 동기와 비동기의 차이는 호출되는 함수의 작업 완료 여부를 신경쓰는지의 여부의 차이

### 동기
* 함수 A가 함수 B를 호출한 뒤, 함수 B의 리턴값을 계속 확인하면서 신경쓰는 것이 동기

### 비동기
* 함수 A가 함수 B를 호출할 때 콜백 함수를 함께 전달해서, 함수 B의 작업이 완료되면 함께 보낸 콜백 함수를 실행

### 동기 프로그래밍

* 여러 작업들을 순차적으로 실행하도록 개발

### 비동기 프로그래밍

* 여러 작업들을 독립적으로 실행하도록 개발

### synchronous하게 음식 준비하기

* 사람 1 : 김치 썰기 → 스팸 굽기 → 국 끓이기  → 햇반 데우기

### asynchronous하게 음식 준비하기

* 사람 1 : 햇반 데우기 → 김치썰기 
* 사람 2 : 국 끓이기 → 스팸 굽기
* **사람 = 스레드**

**asynchronous programming ≠ multithreading**

### asynchronous programming

- 여러 작업을 동시에 실행하는 프로그래밍 방법론

### multithreading

- asynchronous programming의 한 종류 
- asynchronous programming 을 가능하게 하는 것
- multi-threads
    - 장점
        - 멀티 코어를 사용할 수 있음
    - 단점
        - 컨텍스트 스위칭 비용 증가
        - 레이스 컨디션 발생 가능성
- non-block I/O
    - 싱글 스레드여도 여러 가지일을 가능

**백엔드 프로그래밍의 추세는 스레드를 적게 쓰면서도 non-block I/O를 통해 전체 처리량을 늘리는 방향으로 발전 중**

### 정리
* synchronous I/O = block I 
* asynchronous I/O = non-block I/O 
* synchronous I/O : 요청자가 I/O 완료까지 챙겨야 할 때 
* asynchronous I/O : 완료를 noti 주거나 callback으로 처리, block I/O를 다른 thread에서 실행


