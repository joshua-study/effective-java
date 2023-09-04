# 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라.

## 1. 목표
- 어떠한 인스턴스가 애플리케이션 내에서 여러 인스턴스가 필요하지 않은 경우 (1개만 유지) 
- 싱글턴을 보장하는 3가지 방법을 확인하자.(1. private 생성자, 2. static 팩터리, 3. 열거 타입(enum))

##
## 2. 인스턴스가 싱글턴을 보장하는 방법

### (1) private 생성자 + public static final 필드
- 장점 : 간결하게 싱글턴임을 API에 들어낼 수 있다.
- 단점 1 : 싱글톤을 사용하는 클라이언트는 테스트하기 어려워진다.
- 단점 2 : 리플렉션으로 private 생성자를 호출할 수 있다.
- 단점 3 : 역직렬화 할 때 새로운 인스턴스가 생길 수 있다.
###

```java
public interface IElvis {

  void leaveTheBuilding();

  void sing();
}

```

```java
public class Elvis {
  /**
   * 싱글톤 오브젝트
   * javadoc을 만들 때 주석을 활용할 수 있다.
   */
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() { }

    public void leaveTheBuilding() {
        System.out.println("I'm outta here!");
    }
    
    public void sing(){
      System.out.println("I'll have a blue. christmas without you");
    }
}
```

```java
public class Concert {

  private boolean lightsOn;

  private boolean mainStateOpen;

  // 인터페이스 기반으로
  private IElvis elvis;

  public Concert(IElvis elvis) {
    this.elvis = elvis;
  }

  public void perform() {
    mainStateOpen = true;
    lightsOn = true;
    elvis.sing();
  }

  public boolean isLightsOn() {
    return lightsOn;
  }

  public boolean isMainStateOpen() {
    return mainStateOpen;
  }
}
```

- 싱글톤을 사용하는 클라이언트는 테스트하기 어려워짐
```java
class ConcertTest {
  @Test
  void perform() {
    // 테스트마다 Elvis가 노래를 부르게 할 수는 없다.
    // 그래서, Mocking된 Elvis를 추가 할 수 있다.
    Concert concert = new Concert(new MockElvis());
    concert.perform();

    assertTrue(concert.isLightsOn());
    assertTrue(concert.isMainStateOpen());
  }
}
```

- 리플렉션으로 private 생성자 호출이 가능 (싱글톤 X)
  - 새로운 인스턴스가 만들어질 수 있음
```java
public class ElvisReflection {
  public static void main(String[] args) {
    try{
      // getDeclaredConstructor는 접근제어자에 상관없이 private 생성자에도 접근가능
      Constructor<Elvis> defaultConstructor = Elvis.class.getDeclaredConstructor();
      defaultConstructor.setAccessible(true);

      Elvis elvis1 = defaultConstructor.newInstance();
      Elvis elvis2 = defaultConstructor.newInstance();
      // 다른 인스턴스가 생성됨
      System.out.println(elvis1 == elvis2);
      System.out.println(elvis1 == Elvis.INSTANCE);
    } catch (NoSuchMethodException | InstantiationException | IllegalAccessException | InvocationTargetException e) {
      e.printStackTrace();
    }
  }
```

- 같은 인스턴스를 반환하려면?
  - 기본생성자에 검증로직을 추가 
  - 리플렉션으로 더이상 새로운 인스턴스를 만들 수 없게 됨
```java
// Elvis 클래스의 생성자에 검증 로직 추가 
private static boolean created;

private Elvis() {
    if (created) {
      throw new UnsupportedOperationException("can't be created by constructor");
    }

    created = true;
  }
```

- 역직렬화 할 때 새로운 인스턴스가 생길 수 있음. (싱글톤 X)
  - 싱글톤 클래스를 직렬화, 역직렬화 하려면?
  - readResolve 메서드를 제공해야함
  - 이렇게 하지 않으면 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어짐
```java
public class Elvis implements IElvis, Serializable{
  // Serializable를 구현하면 기본 생성자가 필요함
  private Elvis() {
    if (created) {
      throw new UnsupportedOperationException("can't be created by constructor");
    }
    created = true;
  }

  // 역직렬화를 할 때 사용됨 (override가 아니지만 똑같이 동작)
  // 같은 인스턴스를 리턴하게 함
  private Object readResolve() {
    return INSTANCE;
  }
}
```
```java
public class ElvisSerialization {

  public static void main(String[] args) {
    try (ObjectOutput out = new ObjectOutputStream(new FileOutputStream("elvis.obj"))) {
      out.writeObject(Elvis.INSTANCE);
    } catch (IOException e) {
      throw new RuntimeException(e);
    }

    try (ObjectInput in = new ObjectInputStream(new FileInputStream("elvis.obj"))) {
      Elvis elvis3 = (Elvis) in.readObject();
      System.out.println(elvis3 == Elvis.INSTANCE);
    } catch (IOException | ClassNotFoundException e) {
      e.printStackTrace();
    }
  }

```

###
### (2) static 팩토리 방식의 싱글톤

### (3)  열거 타입(enum) 방식의 싱글톤