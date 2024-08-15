### record 등장 배경
* DTO를 구현하기 위해서는 getter, setter, equals, hashCode, toString 같은 데이터 처리 혹은 특정 연산을 수행하기 위해 오버라이드된 메소드를 반복해서 작성

```java
public class Address {
    private String city;
    private String street;
    private String zipCode;

    public Address() {
    }
    
    public Address(String city, String street, String zipCode) {
        this.city = city;
        this.street = street;
        this.zipCode = zipCode;
    }

    public String getCity() {
        return city;
    }

    public String getStreet() {
        return street;
    }

    public String getZipCode() {
        return zipCode;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public void setStreet(String street) {
        this.street = street;
    }

    public void setZipCode(String zipCode) {
        this.zipCode = zipCode;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Address)) return false;
        Address address = (Address) o;
        return Objects.equals(city, address.city) &&
                Objects.equals(street, address.street) &&
                Objects.equals(zipCode, address.zipCode);
    }

    @Override
    public int hashCode() {
        return Objects.hash(city, street, zipCode);
    }
}
```

### 문제점
보일러 플레이트 코드가 불필요하게 크다.  

**보일러 플레이트 코드**
* 최소한의 변경(인자, 혹은 결과 타입)으로 여러 곳에서 재사용 되면 반복적으로 비슷한 형태를 가지고 있는 코드 → getter, setter, equals, hashCode, toString 등이 여기에 해당

- - -

### Record의 등장

**record의 목표**
* 객체 지향의 사상에 맞게 데이터를 간결하게 표현하기 위한 방법을 제공 
* 개발자가 동작을 확장하는 것보다 불변 데이터를 모델링하는데 집중 
* 데이터 지향 메소드를 자동으로 구현

**record의 특징**
* record는 보일러플레이트 코드(생성자, getter, equals, hashCode, toString 등)를 자동으로 생성
* 불변 객체 
  * record는 기본적으로 불변 객체(immutable)입니다. 생성 시점에 필드가 설정되면, 해당 필드는 변경할 수 없다. 즉, record의 필드는 final로 선언된다. 불변 객체이기 때문에 record는 스레드 안전성을 기본적으로 제공한다.

**제약 사항**
* record는 상속이 불가능하다. 이는 불변성과 간결함을 보장하기 위함이다.
* 모든 필드는 반드시 생성자에서 설정되어야 하며, 필드에 기본값을 설정하거나 나중에 설정할 수 없다.

**활용**  
record는 불변성, 간결한 구문, 자동 생성된 메서드를 통해 간단한 데이터 저장용 클래스를 쉽고 빠르게 정의할 수 있도록 도와주기 때문에 특히 DTO, VO(Value Object)와 같은 용도로 사용하기 적합하며, 데이터 캡슐화와 데이터의 불변성을 보장하는 데 유용하다.

**Record는 Entity가 아닌 DTO로 사용하자**
* hibernate와 같은 jpa는 프록시 생성을 위해 인수 생성자, non-final 필드, setter 및 non-final 클래스가 없는 엔티티에 의존한다. 즉, 프록시를 생성하기 위해서 entity는 불변이면 안된다.
* 쿼리 결과를 매핑할 때 객체를 인스턴스화 할 수 있도록 매개변수가 없는 생성자가 필요하다. → record는 매개변수가 없는 생성자를 제공하지 않는다

참고 - https://velog.io/@pp8817/record