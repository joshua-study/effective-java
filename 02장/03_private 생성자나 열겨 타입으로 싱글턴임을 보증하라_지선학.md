싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말함

클래스를 싱글턴으로 만들면 이를 사용하는 클라이언튼를 테스트하기가 어려워 질 수 있음
타입이 인터페이스이면 그 인터페이스를 구현해서 만든 싱글턴이 아니라면 싱글턴 인스턴스를 가자 구현으로 대체할 수 없음

### public static final 필드 방식의 싱글턴
```java
public class Elvis{
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    
    public void leaveTheBuilding() { ... }
}
```
위와 같은 방식으 싱글턴 생성은 리플렉션 API를 사용할 경우 싱글톤임을 보장하지 못함
이러한 공격(?)을 방어하려면 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 하면됨

장점
1) 해당 클래스가 싱글텀임이 API에 명백히 드러남, public static 필드가 final이니 절대로 다른 객체를 참조할 수 없음
2) 간결함

### 정적 팩터리 방식의 싱글턴
```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() {return INSTANCE;}

    public void leaveTheBuilding() { ... }
}
```

Elvis.getInstance는 리플렉션을 통한 예외에서도 제2의 Elvis 인스턴스를 만들지 않음

장점
1) API를 바꾸지 않고도 싱글턴이 아니게 변경 할 수 있다. -> 스레드별 다른 인스턴스
2) 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있음
3) 원한다면 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있음

싱글톤 클래스를 직렬화하는 경우 
- 모든 인스턴스 필드를 일시적이라고 선언하고, readResolve 메서드를 제공해야함
- 이렇게 하지 않으면 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어짐
- 
### 싱글턴임을 보장해주는 readResolve 메서드
```java
private Object readResolve(){
    return INSTANCE;
}
```

### 열거 타입 방식의 싱글턴 - 바람직한 방법
```java
public enum Elvis {
    INSTANCE; 
    
    public void leaveTheBuilding(){ ... }
}
```

- 복잡한 직렬화 상황이나 리플렉션 공경에서도 제2의 인스턴스가 생기는 일을 완벽히 막아줌
- 대부분 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법임

