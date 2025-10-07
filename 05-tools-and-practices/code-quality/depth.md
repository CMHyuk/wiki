### 사고의 depth 줄이기 - 중첩 분기문, 중첩 반복문
```java
for (int i = 0; i< 20; i++) {
    for (int j = 20; j < 30; j++) {
        if (i >= 10 && j < 25) {
            doSomething();
        }
    }
}
```

```java
for (int i = 0; i < 20; i++) {
    doSomethingWithI(i);
}

private void doSomethingWithI(int i) {
    for(int j = 20; j < 30; j++) {
        doSomethingWithIJ(i, j);
    }
}

private void doSomethingWithIJ(int i, int j) {
    if (i >= 10 && j < 25) {
        doSomething();
    }
}
```

* **무조건 1 depth로 만들어라**가 아니다.
  * 보이는 depth를 줄이는 데에 급급한 것이 아니라, 추상화를 통한 **사고 과정의 depth**를 줄이는 것이 중요
  * 2중 중첩 구조로 표현하는 것이 사고하는 데에 더 도움이 된다고 판단한다면, 메서드 분리보다 그대로 놔두는 것이 더 나은 선택일 수 있다.
  * 때로는 메서드를 분리하는 것이 더 혼선을 줄 수도 있다.

### 사고의 depth 줄이기 - 사용할 변수는 가깝게 선언하기
```java
int i = 10;
// 코드 20줄

int j = i + 30;
```

```java
// 코드 20줄
int i = 10;
int j = i + 30;
```

### 공백 라인을 대하는 자세
* 공백 라인도 의미를 가진다.
  * 복잡한 로직의 의미 단위를 나누어 보여줌으로써 읽는 사람에게 추가적인 정보를 전달할 수 있다.