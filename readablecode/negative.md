### 부정어를 대하는 자세

```java
if (!isLeftDirection()) {
    doSomething();    
}
```

* isLeftDirection()을 먼저 이해한 다음에 부정 연산자를 대입해야함
  * 사고 과정이 두 번 일어남

```java
if (isRightDirection()) {
    doSomething();
}

if (isNotLeftDirection()) {
    doSomething();    
}
```

* 부정어구를 쓰지 않아도 되는 상황인지 체크하기
* 부정의 의미를 담은 다른 단어가 존재하는지 고민하기 or 부정어구로 메서드명 구성
  * 부정 연사자(!)의 가독성 ⬇️