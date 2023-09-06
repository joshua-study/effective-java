# 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라.

## 1. 목표
- 인스턴스를 만들지 않는 경우를 권장하는 경우에는 private 생성자를 사용하자.

## 2. 핵심정리
- 정적 메서드만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 클래스가 아니다.
- 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.
- private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.
- 생성자에 주석으로 인스턴스화 불가한 이유를 설명하는 것이 좋다.
- 상속을 방지할 때도 같은 방법을 사용할 수 있다.

## 인스턴스화를 막는 방법
### (1) private 생성자
- 기본 생성자를 private으로 선언 (클래스 밖에서는 접근 X)
```java
public class UtilityClass {
  /**
   * 이 클래스는 인스턴스를 만들 수 없습니다.
   */
  private UtilityClass() {
    throw new AssertionError();
  }
}
```

### (1) 추상클래스 선언
- abstract를 선언하면 기본생성자가 없음 (인스턴스 직접 생성 불가)
- 그러나, 상속받은 하위 객체는 인스턴스 생성이 가능해지고, 또한 상속해서 사용하라는 의미처럼 보이기도 한다.
- 결국 좋은 해결책이 될 수는 없다.

```java
public abstract class UtilityClass {
  public UtilityClass() {
    System.out.println("constructor");
    throw new AssertionError();
  }
}
```

```java
// abstract이더라도 상속받은 서브객체는 인스턴스 생성이 가능하다.
public class DefaultUtilityClass extends UtilityClass{

  public static void main(String[] args) {
    DefaultUtilityClass utilityClass = new DefaultUtilityClass();
    utilityClass.hello();
  }
}
```