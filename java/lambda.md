## 함수형 인터페이스

### Function

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

- 하나의 매개변수를 받고, 결과를 반환하는 함수형 인터페이스이다.
- 입력값(`T` )을 받아서 다른 타입의 출력값(`R` )을 반환하는 연산을 표현할 때 사용한다. 물론 같은 타입의 출력 값도 가능하다. 
- 일반적인 함수(Function)의 개념에 가장 가깝다. 
  - 예: 문자열을 받아서 정수로 변환, 객체를 받아서 특정 필드 추출 등


### Consumer

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

- 입력 값(T)만 받고, 결과를 반환하지 않는(`void` ) 연산을 수행하는 함수형 인터페이스이다. 
- 입력값(T)을 받아서 처리하지만 결과를 반환하지 않는 연산을 표현할 때 사용한다. 
- 입력 받은 데이터를 기반으로 내부적으로 처리만 하는 경우에 유용하다. 
  - 예) 컬렉션에 값 추가, 콘솔 출력, 로그 작성, DB 저장 등

### Supplier

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

- 입력을 받지 않고(`()` ) 어떤 데이터를 공급(supply)해주는 함수형 인터페이스이다. 
- 객체나 값 생성, 지연 초기화 등에 주로 사용된다.

| 인터페이스            | 메서드 시그니처           | 입력     | 출력     | 대표 사용 예시         |
|------------------|--------------------|--------|--------|------------------|
| `Function<T, R>` | `R apply(T t)`     | 1개 (T) | 1개 (R) | 데이터 변환, 필드 추출 등  |
| `Consumer<T>`    | `void accept(T t)` | 1개 (T) | 없음     | 로그 출력, DB 저장 등   |
| `Supplier<T>`    | `T get()`          | 없음     | 1개 (T) | 객체 생성, 값 반환 등    |
| `Runnable`       | `void run()`       | 없음     | 없음     | 스레드 실행 (멀티스레드 등) |

---

## 특화 함수형 인터페이스

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
`Predicate` 는 입력이 `T` , 반환이 `boolean` 이기 때문에 결과적으로 `Function<T, Boolean>`으로 대체할 수 있다. 그럼에도 불구하고 `Predicate` 를 별도로 만든 이유는
다음과 같다.

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
public interface BinaryOperator<T> extends BiFunction<T, T, T> {
    T apply(T t1, T t2); // 실제 코드가 있지는 않음
}
```

"단항 연산(입력 하나)"이고 **타입이 동일**하다면 `UnaryOperator<T>` 를,  
"이항 연산(입력 두 개)"이고 **타입이 동일**하다면 `BinaryOperator<T>` 를 쓰는 것이 개발자의 **의도**와 **로직**을
더 명확히 표현하고, **가독성**을 높일 수 있는 장점이 있다.

| 인터페이스               | 메서드 시그니처              | 입력        | 출력      | 대표 사용 예시                  |
|---------------------|-----------------------|-----------|---------|---------------------------|
| `Predicate<T>`      | `boolean test(T t)`   | 1개 (T)    | boolean | 조건 검사, 필터링                |
| `UnaryOperator<T>`  | `T apply(T t)`        | 1개 (T)    | 1개 (T)  | 단항 연산 (예: 문자열 변환, 단항 계산)  |
| `BinaryOperator<T>` | `T apply(T t1, T t2)` | 2개 (T, T) | 1개 (T)  | 이항 연산 (예: 두 수의 합, 최댓값 반환) |

---

## 람다 vs 익명 클래스

### 1. 문법 차이
- **람다**
  - 람다 표현식은 함수를 간결하게 표현할 수 있는 방식이다. 
  - 함수형 인터페이스(메서드가 하나인 인터페이스)를 간단히 구현할 때 주로 사용한다. 
  - 람다는 `->` 연산자를 사용하여 표현하며, 매개변수와 실행할 내용을 간결하게 작성할 수 있다. 
  - 물론 람다도 인스턴스가 생성된다.

- **익명 클래스**
  - 익명 클래스는 클래스를 선언하고 즉시 인스턴스를 생성하는 방식이다. 
  - 반드시 `new 인터페이스명() { ... }` 형태로 작성해야 하며, 메서드를 오버라이드해서 구현한다. 
  - 익명 클래스도 하나의 클래스이다.

### 2. 코드의 간결함
- 익명 클래스는 문법적으로 더 복잡하고 장황하다. `new 인터페이스명()` 같은 형태와 함께 메서드를 오버라이드해야 하므로 코드의 양이 상대적으로 많다.
- 람다 표현식은 간결하며, 불필요한 코드를 최소화한다. 또한 많은 생략 기능을 지원해서 핵심 코드만 작성할 수 있다.

### 3. 상속 관계
- 익명 클래스는 일반적인 클래스처럼 다양한 인터페이스와 클래스를 구현하거나 상속할 수 있다. 즉, 여러 메서드를 가진 인터페이스를 구현할 때도 사용할 수 있다.
- 람다 표현식은 메서드를 딱 하나만 가지는 함수형 인터페이스만을 구현할 수 있다. 
  - 람다 표현식은 클래스를 상속할 수 없다. 오직 함수형 인터페이스만 구현할 수 있으며, 상태(필드, 멤버 변수)나 추가적인 메서드 오버라이딩은 불가능하다. 
  - 람다는 단순히 함수를 정의하는 것으로, 상태나 추가적인 상속 관계를 필요로 하지 않는 상황에서만 사용할 수 있다.

### 4. 호환성
- 익명 클래스는 자바의 오래된 버전에서도 사용할 수 있다.
- 람다 표현식은 자바 8부터 도입되었기 때문에 그 이전 버전에서는 사용할 수 없다.

### 5. this 키워드 의미
- 익명 클래스 내부에서 `this`는 익명 클래스 자신을 가리킨다. 외부 클래스와 별도의 컨텍스트를 가진다.
- 람다 표현식에서 `this`는 람다를 선언한 클래스의 인스턴스를 가리킨다. 즉, 람다 표현식은 별도의 컨텍스트를 가지는 것이 아니라, 람다를 선언한 클래스의 컨텍스트를 유지한다.
  - 쉽게 말해, 람다 내부의 `this`는 **람다가 선언된 외부 클래스의 `this`와 동일하다.

### 6. 캡처링(capturing)
- 익명 클래스는 외부 변수에 접근할 수 있지만, 지역 변수는 반드시 `final` 혹은 사실상 final인 변수만 캡처할 수 있다.
- 람다도 익명 클래스와 같이 캡처링을 지원한다. 지역 변수는 반드시 `final` 혹은 사실상 final인 변수만 캡처할 수 있다.

> 용어 - 사실상 final  
영어로 effectively final이라 한다. 사실상 `final` 지역 변수는 지역 변수에 `final` 키워드를 사용하지는 않았지만, 값을 변경하지 않는 지역 변수를 뜻한다. `final` 키워드를 넣지 않았을 뿐이지, 실제로는 `final` 키워드
를 넣은 것 처럼 중간에 값을 변경하지 않은 지역 변수이다. 따라서 사실상 `final` 지역 변수는 `final` 키워드를 넣어도 동일하게 작동해야 한다.

### 7. 용도 구분
- 익명 클래스
  - 상태를 유지하거나 다중 메서드를 구현할 필요가 있는 경우 
  - 기존 클래스 또는 인터페이스를 상속하거나 구현할 때 
  - 복잡한 인터페이스 구현이 필요할 때

- 람다
  - 상태를 유지할 필요가 없고, 간결함이 중요한 경우 
  - 단일 메서드만 필요한 간단한 함수형 인터페이스 구현 시 
  - 더 나은 성능(이 부분은 미미함)과 간결한 코드가 필요한 경우

### 정리
자바 8 이후 **람다**가 등장하면서 **익명 클래스**가 대부분 람다로 대체되었지만, 상황에 따라(여러 메서드 구현이 필요하거
나 상태 유지가 필요한 경우 등) 여전히 **익명 클래스**를 사용해야 할 때가 있다.
선택 시 **가독성**, **유지보수성**, **필요 기능**(상태 관리 여부, 인터페이스 메서드 수 등)을 기준으로 판단하면 된다.
물론 둘 다 선택할 수 있다면 람다를 선택하는 것이 대부분 더 나은 선택이다.

