### 상속과 조합
* 상속보다 조합을 사용하자 
* 상속은 시멘트처럼 굳어지는 구조다. 수정이 어렵다. 
  * 부모와 자식의 결합도가 높다.
* 조합과 인터페이스를 활용하는 것이 유연한 구조 
* 상속을 통한 코드의 중복 제거가 주는 이점보다, 중복이 생기더라도 유연한 구조 설계가 주는 이점이 더 크다.

### 상속  
```java
public class Animal {
    public void makeSound() {
        System.out.println("Animal sound");
    }
}

public class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Bark");
    }
}
```

```java
// 새로운 행동 추가
public class Lion extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Roar");
    }
}
```

**문제점**
* 새로운 동물 종류마다 새로운 자식 클래스를 계속 만들어야 한다.
* Roar 소리를 여러 클래스에서 재사용하려면 동일한 코드를 중복해서 작성하게 된다. 
* 이미 존재하는 동물이 다른 소리를 내게 하려면 해당 클래스를 수정해야한다.


### 조합  

```java
public interface SoundBehavior {
    void makeSound();
}

public class BarkSound implements SoundBehavior {
    @Override
    public void makeSound() {
        System.out.println("Bark");
    }
}

public class Dog {
    private SoundBehavior soundBehavior;

    public Dog(SoundBehavior soundBehavior) {
        this.soundBehavior = soundBehavior;
    }

    public void performSound() {
        soundBehavior.makeSound();
    }
}

public class RoarSound implements SoundBehavior {
  @Override
  public void makeSound() {
    System.out.println("Roar");
  }
}

public class Lion {
  private SoundBehavior soundBehavior;

  public Lion(SoundBehavior soundBehavior) {
    this.soundBehavior = soundBehavior;
  }

  public void performSound() {
    soundBehavior.makeSound();
  }
}
```

**장점**
* RoarSound 클래스를 만들어서 바로 다양한 동물 클래스에 적용할 수 있다.  
* 동일한 RoarSound 객체를 여러 동물에 재사용할 수 있다.
* 소리 행동을 필요에 따라 쉽게 변경할 수 있다.