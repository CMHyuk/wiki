## 제네릭 사용 주의사항
1. 제네릭 타입의 객체는 생성이 불가
제네릭 타입 자체로 타입을 지정하여 객체를 생성하는 것은 불가능 한다. 즉, new 연산자 뒤에 제네릭 타입 파라미터가 올수는 없다.

```java
class Sample<T> {
public void someMethod() {
        // Type parameter 'T' cannot be instantiated directly
        T t = new T();
    }
}
```

2. static 멤버에 제네릭 타입이 올 수 없음
아래처럼 static 변수의 데이터 타입으로 제네릭 타입 파라미터가 올 수 없다. 왜냐하면 static 멤버는 클래스가 동일하게 공유하는 변수로서 제네릭 객체가 생성되기도 전에 이미 자료 타입이 정해져 있어야 하기 때문이다. 즉, 논리적인 오류인 것이다.

```java
class Student<T> {
    private String name;
    private int age = 0;

    // static 메서드의 반환 타입으로 사용 불가
    public static T addAge(int n) {

    }
}
```

3. 제네릭으로 배열 선언 주의점
기본적으로 제네릭 클래스 자체를 배열로 만들 수는 없다.

#### 제네릭 배열을 만들 수 없는 이유
Java에서 제네릭 배열을 만들 수 없는 이유는 Java의 타입 시스템과 배열의 타입 안전성 문제 때문이다. 제네릭 클래스는 런타임에 타입 정보가 사라지는 **타입 소거(type erasure)** 라는 특성을 가지고 있다. 이로 인해 배열의 타입 안정성을 보장할 수 없게 된다.

#### 타입 소거(Type Erasure)
Java의 제네릭은 컴파일 시에만 타입을 체크하고, 런타임에는 그 타입 정보를 지우는 방식으로 작동합니다. 예를 들어, `List<String>`은 컴파일 후 `List`로 변환되며, `String`의 타입 정보는 제거된다.

이로 인해, 제네릭 배열을 만들면 다음과 같은 문제가 발생할 수 있다.

**타입 안전성 문제** 
* 만약 제네릭 배열을 허용하면, 배열에 잘못된 타입의 객체가 삽입될 수 있다. 예를 들어, `ArrayList<String>[]` 배열을 만들고 여기에 `ArrayList<Integer>`를 추가할 수 있어야 하는데, 이는 타입 불일치를 초래할 수 있다.  

**런타임 에러**
* 제네릭 배열을 생성하면 런타임에 `ClassCastException`이 발생할 가능성이 높다. 배열의 런타임 타입은 제네릭 타입 정보가 사라진 후 Object로 변환되기 때문이다.

```java
class Sample<T> { 
}

public class Main {
    public static void main(String[] args) {
        Sample<Integer>[] arr1 = new Sample<>[10];
    }
}
```

하지만 제네릭 타입의 배열 선언은 허용된다.
위의 식과 차이점은 배열에 저장할 Sample 객체의 타입 파라미터를 Integer로 지정한다는 뜻이다. 즉, `new Sample<Integer>()` 인스턴스는 저장이 가능하며, `new Sample<String>()` 인스턴스는 저장이 불가능하다는 소리이다.

```java
class Sample<T> { 
}

public class Main {
    public static void main(String[] args) {
    	// new Sample<Integer>() 인스턴스만 저장하는 배열을 나타냄
        Sample<Integer>[] arr2 = new Sample[10]; 
        
        // 제네릭 타입을 생략해도 위에서 이미 정의했기 때문에 Integer 가 자동으로 추론됨
        arr2[0] = new Sample<Integer>(); 
        arr2[1] = new Sample<>();
        
        // ! Integer가 아닌 타입은 저장 불가능
        arr2[2] = new Sample<String>();
    }
}
```

**특정 범위 내로 좁혀서 제한**
* `extends` 와 `super`, 그리고 `?`(물음표)다. `?`는 와일드 카드라고 해서 쉽게 말해 '알 수 없는 타입'이라는 의미다.

```java
<K extends T>	// T와 T의 자손 타입만 가능 (K는 들어오는 타입으로 지정 됨)
<K super T>	// T와 T의 부모(조상) 타입만 가능 (K는 들어오는 타입으로 지정 됨)
 
<? extends T>	// T와 T의 자손 타입만 가능
<? super T>	// T와 T의 부모(조상) 타입만 가능
<?>		// 모든 타입 가능. <? extends Object>랑 같은 의미
```

* 보통 이해하기 쉽게 다음과 같이 부른다. 
  * extends T : 상한 경계 
  * ? super T : 하한 경계
*  K extends T와 ? extends T는 비슷한 구조지만 차이점이 있다.
  * '유형 경계를 지정'하는 것은 같으나 경계가 지정되고 K는 특정 타입으로 지정이 되지만, ?는 타입이 지정되지 않는다는 의미다.

```java
/*
 * Number와 이를 상속하는 Integer, Short, Double, Long 등의
 * 타입이 지정될 수 있으며, 객체 혹은 메소드를 호출 할 경우 K는
 * 지정된 타입으로 변환이 된다.
 */
<K extends Number>
 
 
/*
 * Number와 이를 상속하는 Integer, Short, Double, Long 등의
 * 타입이 지정될 수 있으며, 객체 혹은 메소드를 호출 할 경우 지정 되는 타입이 없어
 * 타입 참조를 할 수는 없다.
 */
<? extends T>	// T와 T의 자손 타입만 가능
```

```java
<K super B>	// B와 A타입만 올 수 있음
<K super E>	// E, D, A타입만 올 수 있음
<K super A>	// A타입만 올 수 있음
 
<? super B>	// B와 A타입만 올 수 있음
<? super E>	// E, D, A타입만 올 수 있음
<? super A>	// A타입만 올 수 있음
```

출처
* https://inpa.tistory.com/entry/JAVA-☕-제네릭Generics-개념-문법-정복하기
* https://st-lab.tistory.com/153