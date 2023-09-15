# 아이템 8. finalizer와 cleaner 사용을 피하라

자바의 두 가지 객체 소멸자 중 하나이다. **finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다. **
'쓰지 말아야'하기에 자바 9에서는 대신 cleaner를 대안으로 소개했다.
**cleaner는 finalizer보다 덜 위험하지만, 여전히 예측할 수 없고, 느리고 일반적으로 불필요하다.**

## finalizer와 cleaner의 문제점
1. (finalizer와 cleaner)수행 시점이 보장되지 않는다.
2. (finalizer)자원 회수가 제멋대로 지연될 수 있다. <br>인스턴스 반납을 지연 시킬 수 있으며, 심지어 실행이 되지 않을 수도 있다.
3. (finalizer와 cleaner)수행 여부가 보장되지 않는다.
4. (finalizer)  동작 중 발생한 예외는 무시된다. 
<br>finalizer은 처리할 작업이 남았더라도 그 순간 종료된다. cleaner는 자신의 스레드를 통제하기 때문에 위의 문제는 발생하지 않는다.
5. (finalizer와 cleaner) 성능 문제
6. (finalizer) 보안 문제 <br>공격으로부터 방어하려면 아무 일도 하지 않는 finalize를 만들고 final로 선언해야한다.

## AutoCloseable
AutoCloseable을 구현해주고, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하면 된다. (예외 발생해도 제대로 종료되도록 try-with-resources 사용해야 한다.)
close 메서드에서 객체가 더 이상 유효하지 않음을 필드에 기록하고, 다른 메서드는 필드를 검사해 객체가 닫힌 후에 불렸
다면 IllegalStateException을 던지는 것이다.

## finalizer와 cleaner의 쓰임
1. 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할
2. 네이티브 피어와 연결된 객체
네이티브 피어란, 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다.
GC는 이 존재를 알지 못해 자바 피어를 회수 할 때 네이티브 객체까지 회수하지 못한다.
성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때만 사용해야해며, 자원을 즉시 회수해야 한다면  close 메서드를 사용해야 한다.
```java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    //청소가 필요한 자원. 절대 Room을 참조해서는 안 된다!
    private static class State implements Runnable {
        int numJunkPiles; // 방 안의 쓰레기 수 (수거할 자원)

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        //close 메서드나 cleaner가 호출한다.
        @Override public void run() {
            System.out.println("Cleaning room");
            numJunkPiles = 0;
        }
    }

    //방의 상태. cleanable과 공유한다.
    private final State state;

    //cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override public void close() {
        cleanable.clean();
    }
}
```
위는 cleaner를 안정망으로 활용하는 AutoClosable 클래스이다.<br>
State가 정적 중첩 클래스인 이유? State 인스턴스는 절대로 Room 인스턴스를 참조해서는 안된다. 참조할 경우 순환참조가 생겨 GC가 Room 인스턴스를 회수할 기회가 오지 않는다.

```java
public class Adult {
    public static void main(STring[] args) {
        try (Room myRoom = new Room(7)) {
            System.out.println("안녕~");
        }
    }
}
```
try-with-resources 블록으로 감쌌다면 자동 청소를 전혀 필요하지 않다.

## 정리
cleaner는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자. 이런 경우라도 불확실성과 성능 저하에 주의해야 한다.


