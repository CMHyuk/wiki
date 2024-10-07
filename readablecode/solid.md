### SRP: Single Responsibility Principle
* 하나의 클래스는 단 한가지의 변경 이유만을 가져야 한다.
  * 변경 이유 = 책임
* 객체가 가진 공개 메서드, 필드, 상수 등은 해당 객체의 단일 책임에 의해서만 변경 되는가?
* 관심사 분리
* 높은 응집도, 낮은 결합도

### OCP: Open-Closed Principle
* 확장에는 열려 있고, 수정에는 닫혀 있어야 한다.
  * 기존 코드의 변경 없이, 시스템의 기능을 확장할 수 있어야 한다.
* 추상화와 다형성을 활용해서 OCP를 지킬 수 있다.

### LSP: Liskov Substitution Principle
* 상속 구조에서, 부모 클래스의 인스턴스를 자식 클래스의 인스턴스로 치환할 수 있어야 한다.
  * 자식 클래스는 부모 클래스의 책임을 준수하며, 부모 클래스의 행동을 변경하지 않아야 한다.
* LSP를 위반하면, 상속 클래스를 사용할 때 오동작, 예상 밖의 예외가 발생하거나, 이를 방지하기 위한 불필요한 타입 체크가 동반될 수 있다.

```java
class Parent {
    public Result doSomething() { ... }
}

class Child extends Parent {}

Result r = new Parent().doSomething();
Result r = new Child().doSomething();
```

### ISP: Interface Segregation Principle
* 클라이언트는 자신이 사용하지 않는 인터페이스에 의존하면 안 된다.
  * 인터페이스를 잘게 쪼개라!
* ISP를 위반하면, 불필요한 의존성으로 인해 결합도가 높아지고, 특정 기능의 변경이 여러 클래스에 영향을 미칠 수 있다.

### DIP: Dependency Inversion Principle
* 상위 수준의 모듈은 하위 수준의 모듈에 의존해서는 안 된다. 둘 모두 추상화에 의존해야 한다. 
* 의존성의 순방향: 고수준 모듈(추상화 레벨이 높은)이 저수준 모듈(추상화 레벨이 낮은)을 참조하는 것 
* 의존성의 역방향: 고수준, 저수준 모듈이 모두 추상화에 의존하는 것 
  * 저수준 모듈이 변경되어도, 고수준 모듈에는 영향이 가지 않는다.
* DI (Dependency Injection) - "3"
  * A가 B가 필요해서 의존하고 싶을 때 제 3자(스프링 컨텍스트)가 의존성 주입
  * 만약 의존성 주입을 인터페이스가 아닌 구체 클래스로 한다면 DI는 맞지만 DIP는 위반
* IoC (Inversion of Control)
  * 프로그램 주도권을 개발자가 아닌 프레임워크에 넘기는 것