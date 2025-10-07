### Value Object
* 도메인의 어떤 개념을 추상화하여 표현한 값 객체 
* 값으로 취급하기 위해서, 불변성, 동등성, 유효성 검증 등을 보장해야 한다. 
  * 불변상 : final 필드, setter 금지 
  * 동등성 : 서로 다른 인스턴스여도(동일성이 달라도), 내부의 값이 같으면 같은 값 객체로 취급한다. equals() & hashCode() 재정의 필요. 
  * 유효성 검증 : 객체가 생성되는 시점에 값에 대한 유효성을 보장하기

### Vo vs Entity
* Entity는 식별자가 존재한다. 식별자가 아닌 필드의 값이 달라도, 식별자가 같으면 동등한 객체로 취급한다. 
  * equals() & hashCode()도 식별자 필드만 가지고 재정의할 수있다.  
  * 식별자가 같은데 식별자가 아닌 필드의 값이 서로 다른 두 인스턴스가 있다면, 같은 Entity가 시간이 지남에 따라 변화한 것으로 이해할 수도 있다. 
* VO는 식별자 없이, 내부의 모든 값이 다 같아야 동등한 객체로 취급한다. 
  * 개념적으로, 전체 필드가 다같이 식별자 역할을 한다고 생각해도 된다.

```java
// Entity
class userAccount {
    private String userId; // 식별자
    private String 이름;
    private String 생년월일;
    private Address 집주소;
}

// VO
class Address {
    private String 시도;
    ...
}
```
