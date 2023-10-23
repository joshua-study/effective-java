# 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라.

---

## (1) 싱글턴
- 오직 하나만 생성할 수 있는 클래스
- 무상태 객체나 설계상 유일해야 하는 시스템 컴포넌트

※ 클래스를 싱글턴으로 만들면 테스트하기가 어려워질 수 있다.

---

## (2) 싱글턴을 만드는 방법

### (2-1) 생성자는 private으로 감춰두고, public static final 필드 추가
- public이나 protected가 없으므로 클래스가 초기화될 때 만들어진 인스턴스가 단 하나뿐임을 보증
- 해당 클래스는 싱글턴임이 API에 명백히 드러나고, 코드가 간결하다.
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
} 
```

### (2-2) 정적 팩토리 메서드 제공
- API를 바꾸지 않고도 싱글턴이 아니게 변경 가능하다.
- 정적 팩토리를 제네릭 싱글턴 팩토리로 만들 수 있다. 
- 정적 팩토리의 메서드 참조를 공급자로 사용할 수 있다.
```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() {
        return INSTANCE;
    }
} 
```

---

### (2-3) 원소가 하나인 열거타입을 선언.
- 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법.
  - 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.
```java
public enum Elvis {
    INSTANCE;
}
```
