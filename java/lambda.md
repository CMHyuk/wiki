### 함수형 인터페이스

| 인터페이스            | 메서드 시그니처           | 입력     | 출력     | 대표 사용 예시         |
|------------------|--------------------|--------|--------|------------------|
| `Function<T, R>` | `R apply(T t)`     | 1개 (T) | 1개 (R) | 데이터 변환, 필드 추출 등  |
| `Consumer<T>`    | `void accept(T t)` | 1개 (T) | 없음     | 로그 출력, DB 저장 등   |
| `Supplier<T>`    | `T get()`          | 없음     | 1개 (T) | 객체 생성, 값 반환 등    |
| `Runnable`       | `void run()`       | 없음     | 없음     | 스레드 실행 (멀티스레드 등) |

---

### 특화 함수형 인터페이스
특화 함수형 인터페이스는 의도를 명확하게 만든 조금 특별한 함수형 인터페이스다.  

- `Predicate` : 입력O, 반환 `boolean`
  - 조건 검사, 필터링 용도
- `Operator` : (`UnaryOperator` ,`BinaryOperator` ): 입력O, 반환O 
  - 동일한 타입의 연산 수행, 입력과 같은 타입을 반환하는 연산 용도

### Predicate   

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```

**Predicate가 꼭 필요할까?**  
`Predicate` 는 입력이 `T` , 반환이 `boolean` 이기 때문에 결과적으로 `Function<T, Boolean>`으로 대체할 수 있다. 그럼에도 불구하고 `Predicate` 를 별도로 만든 이유는 다음과 같다.  

1. **의미의 명확성**  
  `Predicate<T>` 를 사용하면 "이 함수는 조건을 검사하거나 필터링 용도로 쓰인다"라는 의도가 더 분명해진다.
   `Function<T, Boolean>` 을 쓰면 "이 함수는 무언가를 계산해 `Boolean` 을 반환한다"라고 볼 수도 있지만, "조건 검사"라는 목적이 분명히 드러나지 않을 수 있다.  
2. **가독성 및 유지보수성**  
   여러 사람과 협업하는 프로젝트에서, "조건을 판단하는 함수"는 `Predicate<T>` 라는 패턴을 사용함으로
   써 의미 전달이 명확해진다.
   `boolean` 판단 로직이 들어가는 부분에서 `Predicate<T>` 를 사용하면 코드 가독성과 유지보수성이 향상된다.
   이름도 명시적이고, 제네릭에 `<Boolean>` 을 적지 않아도 된다.

### Operator

Operator는 `UnaryOperator` , `BinaryOperator` 2가지 종류가 제공된다.

**UnaryOperator(단항 연산)**  

```java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {
    T apply(T t); // 실제 코드가 있지는 않음
}
```

**BinaryOperator(이항 연산)**  

```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T,T,T> {
    T apply(T t1, T t2); // 실제 코드가 있지는 않음
}
```

"단항 연산(입력 하나)"이고 **타입이 동일**하다면 `UnaryOperator<T>` 를,  
"이항 연산(입력 두 개)"이고 **타입이 동일**하다면 `BinaryOperator<T>` 를 쓰는 것이 개발자의 **의도**와 **로직**을
더 명확히 표현하고, **가독성**을 높일 수 있는 장점이 있다.

---