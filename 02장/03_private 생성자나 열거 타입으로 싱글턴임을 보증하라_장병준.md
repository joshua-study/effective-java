## 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라.

싱글톤(singleton)이란  
인스턴스를 오직 하나만 생성할 수 있는 클래스

ex) 함수와 같은 무상태(stateless) 객체, 설계상 유일해야 하는 시스템 컴포넌트

#### 장점  
한번의 객체 생성으로 재사용이 가능하다.
1. 메모리 낭비를 방지할 수 있다.
2. 전역성을 갖기 때문에 다른 객체와 공유가 용이하다.

#### 단점  
클래스가 싱글톤이면 이를 사용하는 클라이언트를 테스트하기가 어려워 질 수 있다.
1. 타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글톤이 아니라면 싱글톤 인스턴스를 mock 구현으로 대체할수 없기 때문.

## 싱글턴을 만드는 방식
### 1. public static final 필드 방식
```java
public class Elvis {
public static final Elvis INSTANCE = new Elvis();
private Elvis() { ... }

public void leaveThebuilding() { ... }
}
```
생성자는 private 으로 감추기  
유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 마련  
#### 특징  
1. 해당 클래스가 싱글턴임이 API 에 명백히 드러난다.  
2. 간결함
### 2. 정적 팩토리 방식
```java
public class Elvis {
private static final Elvis INSTANCE = new Elvis();
private Elvis() { ... }
public static Elvis getInstance() { return INSTANCE; }

public void leaveThebuilding() { ... }
}
```
생성자는 private 으로 감추기  
정적 팩토리메소드를 public static 멤버로 제공
#### 특징
1. API를 바꾸지 않고도 싱글턴이 아니게 변경이 가능하다.
2. 정적 팩토리를 제네릭 싱글톤 팩토리로 만들 수 있다
3. 정적 팩토리의 메소드 참조자를 공급자로 사용이 가능하다.

### 3. 열거 타입 방식
```java
public enum Elvis {
INSTANCE;
public void leaveTheBuilding() { ... }
}
```
원소가 하나인 열거타입을 선언하는 것
#### 특징
1. 대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법.  
2. 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.

