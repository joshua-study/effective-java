# 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라.

싱글턴이란? 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.

## 방식 1. public static final 필드 방식의 싱글톤

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() { }

    public void leaveTheBuilding() {
        System.out.println("public static final 필드 방식의 싱글톤");
    }
}
```

private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화할 때 한 번만 호출된다. 
Elvis 클래스는 생성자가 private로 되어있기 때문에 외부에서 객체를 생성할 수 없다. 그러면 INSTANCE 객체가 Elvis 클래스의 유일한 객체가 된다. 

**장점은 이런 API 사용이 static 팩토리 메소드를 사용하는 방법에 비해 더 명확하고 더 간단해진다.**
다만 추후 해당 클래스를 싱글톤이 아니게 바꾸거나, 상황에 따라 다른 인스턴스를 만들거나 하는 유연성이 떨어진다.

예외로 리플렉션이 있는데,, 리플렉션이 뭔지 알아보도록 하자..!

## 방식 2. static 팩토리 방식의 싱글톤

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    
    private Elvis() { }
    
    public static Elvis getInstance() { return INSTANCE; } //private static을 리턴한다.

}
```

getInstance 메소드를 통해서 하나의 객체만을 사용하게 된다.

**장점**

1. API(=public static 메소드)를 변경하지 않고도 싱글톤으로 쓸지 안쓸지 변경할 수 있다. (클라이언트 코드를 변경하지 않아도 된다는 것이다! 구현체 코드만 변경하면 된다.)
2. static 팩토리를 제네릭 싱글턴 팩토리로 만들 수 있다. (아이템30)
3. static 팩토리의 메서드 참조를 공급자로 사용할 수 있다. (아이템 43,44)

## 직렬화

위의 두 방식 모두 직렬화를 사용한다면 역직렬화할 때 같은 타입의 인스턴스가 여러개 생길 수 있다.

문제를 해결하려면 모든 인스턴스 필드를 일시적(transient)이라고 선언하고 readResolve 메서드를 제공해야 한다. (아이템 89)
```java
private Object readResolve(){
    //'진짜' Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
        return INSTANCE;
}
```


## 방식 3. 열거 타입(enum) 방식의 싱글톤

```java
public enum Elvis{
    INSTANCE;
}
```

직렬화/역직렬화할 때와 리플렉션으로 호출될 때의 고민을 모두 할 필요 없다.

코드는 부자연스러워보일 순 있으나 **대부분 상황에서는 원소가 하나뿐인 열거 타입의 싱글턴을 만드는 것이 가장 좋은 방법이다.**

단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 사용할 수 없다. (열거 타입이 다른 인터페이스를 구현하도록 선언할 수는 있다.)

[참고1] (https://nankisu.tistory.com/89)