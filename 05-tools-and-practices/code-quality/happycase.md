### 해피 케이스
* 사람은 해피 케이스에 몰두하는 경향이 있다.

### 해피 케이스와 예외 처리
* 예외가 발생할 가능성 낮추기 
* 어떤 값의 검증이 필요한 부분은 주로 외부 세계와의 접점 
* 사용자 입력, 객체 생성자, 외부 서버의 요청 등 
* 의도한 예외와 예상하지 못한 예외를 구분하기 
* 사용자에게 보여줄 예외와, 개발자가 보고 처리해야 할 예외 구분

### Null을 대하는 자세
* 항상 NullPointException을 방지하는 방향으로 경각심 가지기 
* 메서드 설계 시 return null을 자제한다. 
  * 만약 어렵다면, Optional 사용을 고민해 본다.

### Optional에 관하여
* Optional은 비싼 객체다. 꼭 필요한 상황에서 반환 타입에 사용한다. 
  * Optional을 반환받았다면 최대한 빠르게 해소한다.
* Optional을 파라미터로 받지 않도록 한다. 분기 케이스가 3개나 된다. 
  * Optional을 가진 데이터가 null인지 아닌지 + Optional 그 자체가 null

### Optional을 해소하는 방법
* 분기문을 만드는 isPresent()-get() 대신 풍부한 API 사용 
* orElseGet(), orElseThrow(), ifPresent(), ifPresentOrElse()
* orElse(), orElseGet(), orElseThrow()의 차이 숙지 
  * orElse(): 항상 실행, 확정된 값일 때 사용 
  * orElseGet(): null인 경우 실행, 값을 제공하는 동작(Supplier) 정의

```java
public T orElse(T other) {
    return value != null ? value : other;
}

public T orElseGet(Supplier<? extends T> supplier) {
    return value != null ? value : supplier.get();
}

// 호출할 필요가 없는 경우에도 항상 실행
Integer result = somethingOptional
        .orElse(performanceHeavy());

// null인 경우에만 실행
Integer result2 = somethingOptional
        .orElseGet(() -> performanceHeavy());
```
