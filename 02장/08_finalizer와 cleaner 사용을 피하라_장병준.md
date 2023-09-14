# 아이템 8. finalizer와 cleaner 사용을 피하라

### 자바의 두가지 객체 소멸자 finalizer, cleaner

#### finalizer (자바9 deprecated)   
finalize 메서드는 Object 클래스에서 제공하는 기본 메서드이다. 그래서 어느 클래스에서든지 finalize 메서드를 오버라이드 할 수 있다. 가비지 컬렉터에 의해 객체가 회수 될 때, finalize 메서드가 호출된다.   
특징으로는 예측할 수 없고, 상황에 따라 위험할 수 있어서 일반적으로 불필요하다. 기본적으로 쓰지말아야 한다.

#### cleaner
Cleaner는 자바 9부터 지원하는 소멸자이다. cleaner를 사용할지는 내부 구현 방식에서 선택의 문제이다. 이 뜻은 Object 클래스에서 API로 제공했던 finalizer처럼 오버라이드 하는게 아닌 구성을 통해 cleaner를 사용해야 한다.   
특징으로는 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.

---
### finalizer와 cleaner의 문제점
#### 1. finalizer와 cleaner는 즉시 수행된다는 보장이 없다.
    객체에 접근을 할 수 없게 된 후 finalizer나 cleaner가 실행되기까지 얼머나 걸리는지 알수 없다.
    즉, finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없다.
#### 2. finalizer는 인스턴스의 자원 회수가 제멋대로 지연될 수 있다.
    finalizer쓰레드는 다른 애플리케이션 쓰레드보다 우선순위가 낮아서 제대로 실행될 기회를 얻지 못한다.
    이는 곧 OOM을 발생시키는 원인이 될 수도 있다.
    cleaner은 자신을 수행할 쓰레드를 제어할 수 있다는 점이 조금 나을순 있으나, 여전히 백그라운드에서 수행되며 GC의 통제하에 있으니 즉각수행되리라 보장은 없다.
#### 3. 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다.
    DB같은 공유 자원의 영구 락 해제를 finalizer나 cleaner에 맡겨 놓으면 분산시스템 전체가 서서히 멈출 것이다.
#### 4. finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다.
    잡지 못한 예외 때문에 해당 객체는 자칫 마무리가 덜 된 상태로 남을 수 있다.
    다른 쓰레드가 이 훼손된 객체를 사용하려 한다면 무슨 일이 일어날지 알 수 없다.
#### 5. finalizer와 cleaner는 심각한 성능 문제도 동반한다.
    AutoCloseable 객체를 생성하고 GC가 수거하기까지 12ns가 걸린 반면(try-with-resources로 자신을 닫도록 했다), finalizer를 사용하면 550ns가 걸렸다.
    finalizer가 gc의 효율을 떨어뜨리기 때문이다.
#### 6. finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다.
    final이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무 일도 하지 않는 finalize 메소드를 만들고 final로 선언하면 된다.
---
### 자원 반납하는 방법(대체 방법)
#### 자원 반납이 필요한 클래스 AutoCloseable 인터페이스를 구현하고 try-with-resource를 사용하거나, 클라이언트가 close 메서드를 호출하면 된다.
1. cleaner를 안정망으로 활용하는 autocloseable 클래스 예시
```java
public class Room implements AutoCloseable {

    private static final Cleaner cleaner = Cleaner.create();
    
    private static class State implements Runnable {
        int numJunkPiles;

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        @Override
        public void run() {
            System.out.println("Room Clean");
            numJunkPiles = 0;
        }
    }

    private final State state;

    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override
    public void close() {
        cleanable.clean();
    }
}
```
State 클래스는 자원을 뜻하며, Room 클래스가 반드시 자원을 수거해야 하는 대상이다.   
State 클래스의 run 메서드는 cleanable에 의해 호출된다.   
run 메서드가 호출되는 상황은 두 가지이다.
1. Room 클래스의 close 메서드를 직접 호출할 때.
2. 가비지 컬렉터가 Room을 회수할 때 close 메서드를 호출하는 경우.

#### try-with-resources 예시
```java
public class Adult {
    public static void main(String[] args) {
        try (Room myRoom = new Room(7)) {
        System.out.println("안녕~");
        }
    }
}
```
---
### 핵심정리
finalizer 또는 cleaner는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용해야 한다. 안전망 용도이니 자원을 사용한 경우라면 즉시 해제 하는 것이 좋다. 자원은 한정적이기 때문이다.