# 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라.

## 1. 목표
- 어떠한 인스턴스가 애플리케이션 내에서 여러 인스턴스가 필요하지 않은 경우 (1개만 유지해야 하는 경우) 
- 싱글턴을 보장하는 3가지 방법을 확인하자.(1. private 생성자, 2. static 팩터리, 3. 열거 타입(enum))
- 스프링을 사용한다면, 현실적으로 스프링 컨테이너를 활용하자.

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
   * 필드를 만들면 javadoc을 만들 때 주석을 활용할 수 있다.
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
  - 코드가 무거워지면 싱글톤 객체를 사용하기 때문에, 인터페이스를 활용하여 가짜 객체를 활용하자.
```java
class ConcertTest {
  @Test
  void perform() {
    // 테스트마다 Elvis가 노래를 부르게 할 수는 없다.
    // 그래서, Mocking된 Elvis(가짜 객체)를 추가 할 수 있다.
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
      System.out.println(elvis3 == Elvis.INSTANCE); // true
    } catch (IOException | ClassNotFoundException e) {
      e.printStackTrace();
    }
  }

```

###
### (2) private 생성자 + 정적 팩터리 메서드
- 단점은 인스턴스 필드에 직접 접근하는 방법과 동일함 (테스트, 리플렉션, 역직렬화)
- 장점 1 : API를 바꾸지 않고도 싱글톤이 아니게 변경할 수 있다. (정적팩터리 메서드를 수정하여 행위를 바꿀수 있음)
- 장점 2 : 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
- 장점 3 : 정적 팩터리의 메서드 참조를 공급자(Supplier)로 사용할 수 있다.
###

- 정적팩터리 메서드 접근
```java
  // 정적팩터리 메서드 (메서드를 통해서 접근)
  // 필드에 직접 접근하는 것 처럼 1. 테스트 2. 리플렉션 3. 직렬화,역직렬화의 3가지 단점 동일
  public static Elvis getInstance() {
    // return INSTANCE;
    // getInstance 메서드의 행위 변경이 가능 (예시와 같이 새로운 객체 반환으로 변경)
    return new Elvis();
  }
```
- 제네릭 싱글톤 팩터리
```java
// 제네릭 싱글톤 팩터리
public class MetaElvis<T> {

  private static final MetaElvis<Object> INSTANCE = new MetaElvis<>();

  private MetaElvis() {
  }

  @SuppressWarnings("unchecked")
  // 앞의 <T>는 클래스에 선언한 <T>가 아닌 static의 다른 스코프에 있는 제네릭
  public static <E> MetaElvis<E> getInstance() {
    return (MetaElvis<E>) INSTANCE;
  }

  public void say(T t) {
    System.out.println(t);
  }

  public void leaveTheBuilding() {
    System.out.println("Whoa baby, I'm outta here!");
  }

  public static void main(String[] args) {
    // 제네릭 싱글톤 팩터리를 사용하면 원하는 타입으로 바꿔서 사용할 수 있다.
    MetaElvis<String> elvis1 = MetaElvis.getInstance();
    MetaElvis<Integer> elvis2 = MetaElvis.getInstance();
    // 같은 객체이지만 타입이 다르기때문에, == 동일성 비교는 불가
    System.out.println(elvis1.equals(elvis2));
  }
}
```

- @FunctionalInterface (`Supplier<T>`)

```java
public class Elvis implements Singer {

  public static final Elvis INSTANCE = new Elvis();

  // Serializable를 구현하면 기본 생성자가 필요함
  private Elvis() {
  }

  public static Elvis getInstance() {
    return INSTANCE;
  }

  public void leaveTheBuilding() {
    System.out.println("I'm outta here!");
  }

  @Override
  public void sing() {
    System.out.println("my way~~~");
  }
}
```
```java
public class Concert {

  // supplier (매개변수를 받지 않고, 리턴값이 있는)
  public void start(Supplier<Singer> singerSupplier) {
    Singer singer = singerSupplier.get();
    singer.sing();
  }

  public static void main(String[] args) {
    Concert concert = new Concert();
    // Supplier에 준하는 메서드 참조를 추가
    concert.start(Elvis::getInstance);
  }
}
```

### (3)  열거 타입(enum) 방식의 싱글톤
- 가장 간결한 방법이며 직렬화와 리플렉션에도 안전하다.
- 또한 인터페이스를 implements 할 수 있기 때문에 테스트에도 용이
- enum은 내부에서 리플렉션을 막아놨기 때문에 생성자를 가져오지 못한다고 나옴
- 대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.
###
```java
// 열거타입으로 싱글톤을 보장
public enum Elvis {
  // 실제 private 기본생성자가 존재하지만, enum은 new 객체를 생성할 수 없게 막아둠
  INSTANCE;

  public void leaveTheBuilding() {
    System.out.println("기다려, 지금 나갈께!");
  }

  public static void main(String[] args) {
    Elvis elvis = Elvis.INSTANCE;
    elvis.leaveTheBuilding();
  }
}
```

- Class.getConstructor (생성자를 가져오지 못함)
```java
public class EnumElvisReflection {
  public static void main(String[] args) {
    try{
      Constructor<Elvis> declaredConstructor = Elvis.class.getDeclaredConstructor();
      System.out.println(declaredConstructor);
    } catch (NoSuchMethodException e) {
      e.printStackTrace();
    }
  }
}
```

- enum은 역질렬화하면 동일한 인스턴스를 반환
```java
public class EnumElvisSerialization {

  public static void main(String[] args) {
    try (ObjectOutput out = new ObjectOutputStream(new FileOutputStream("elvis.obj"))) {
      out.writeObject(Elvis.INSTANCE);
    } catch (IOException e) {
      throw new RuntimeException(e);
    }

    try (ObjectInput in = new ObjectInputStream(new FileInputStream("elvis.obj"))) {
      Elvis elvis3 = (Elvis) in.readObject();
      // enum은 역직렬화하면 동일한 인스턴스를 반환
      System.out.println(elvis3 == Elvis.INSTANCE);
    } catch (IOException | ClassNotFoundException e) {
      e.printStackTrace();
    }
  }
}
```

## 5. 추가 지식
- p23, `리플렉션 API`로 private 생성자 호출하기
- p24, `메서드 참조`를 공급자로 사용할 수 있다.
- p24, `Supplier<T>, 함수형 인터페이스`
- p24, `직렬화`, 역직렬화, Serializable, transient

### (1) 메서드 참조
- 메서드 하나만 호출하는 람다 표현식을 줄여쓰는 방법
  - 스태틱 메서드 레퍼런스
  - 인스턴스 메서드 레퍼런스
  - 임의 객체의 인스턴스 메서드 레퍼런스
  - 생성자 레퍼런스
  - https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html
### (2) 함수형 인터페이스
- 자바가 제공하는 기본 함수형 인터페이스
  - 함수형 인터페이스는 람다 표현식과 메서드 참조에 대한 "타겟 타입"을 제공한다.
  - 타겟 타입은 변수 할당, 메서드 호출, 타입 변환에 활용할 수 있다.
  - 자바에서 제공하는 기본 함수형 인터페이스 익혀 둘 것. [java.util.function 패키지](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)
  - 함수형 인터페이스를 만드는 방법
    - 심화 1 : [Understanding java method invocation with](https://blogs.oracle.com/javamagazine/post/understanding-java-method-invocation-with-invokedynamic)
    - 심화 2 : [LambdaMetaFactory](https://docs.oracle.com/javase/8/docs/api/java/lang/invoke/LambdaMetafactory.html) 
###

- 기본 함수형인터페이스로 타입을 지정할 수 없는 경우 직접 정의할 수 있음

```java
// 기본 함수형인터페이스
public class DefaultFunctions {
  // 매개변수를 받아서 리턴함
  Function<Integer, String> intToString;

  // 매개변수를 받지 않고 리턴함
  Supplier<Integer> integerSupplier;

  // 매개변수를 받고 리턴하지 않음
  Consumer<Integer> integerConsumer;

  // 매개변수를 받고 boolean 값을 리턴함
  Predicate<Integer> predicate;
}
```

```java
@FunctionalInterface
public interface MyFunction {

  // 메서드 선언은 1개만 있어야 함
  // 기본 함수형인터페이스로 타입을 지정할 수 없는 경우 직접 정의할 수 있음
  String valueOf(Integer integer);

  static String hello() {
    return "hello";
  }
}

```

### (3) 객체 직렬화
- 객체를 바이트스트림으로 상호 변환하는 기술
  - 바이트스트림으로 변환한 객체를 파일로 저장하거나 네트워크를 통해 다른 시스템으로 전송할 수 있다.
  - Serializable 인터페이스 구현
  - transient를 사용해서 직렬화 하지 않을 필드 선언하기
  - serialVersionUID는 언제 왜 사용하는가?
    - 클래스의 필드가 변경되더라도 역직렬화를 사용할 때 사용
  - 심화 1 : [객체 직렬화 스펙](https://docs.oracle.com/javase/8/docs/platform/serialization/spec/serialTOC.html)