finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다. 

cleaner는 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.

finalizer와 cleanser는 즉시 수행된다는 보장이 없다. 객체에 접근 할 수 없게 된 후 finalizer나 cleaner가 실행되기까지
얼마나 걸릴지 알 수 없다.
finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없다.

#### finalizer와 cleaner는 심각한 성능 문제도 동반한다. 
finalize가 가비지 컬렉터의 효율을 떨어뜨리기 때문이다.

#### 객체 생성을 막으려면 생성자에서 예외를 던지는 것만드로 충분하지만, finalize가 있다면 그렇지 않다.
final 클래스들은 그 누구도 하위 클래스를 만들 수 없으니 이 공격에서 안전함 

#### final이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무 일도 하지 않는 finalize 메서드를 만들고 final로 선언하자.
AutoCloaseable을 구현해주고, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하면 됨

#### cleaner와 finalizer를 적절히 활용하는 두 번째 예는 네이티브 피어(native peer)와 연결된 객체에서다.

### cleaner를 안정망으로 활용하는 AutoCloseable 클래스 
```java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner= Cleaner.create();
    
    private static class State implements Runnable {
        int numJunkPiles;
        
        State(int numJunkPiles){
            this.numJunkPiles = numJunkPiles;
        }
        
        @Override public void run(){
            System.out.println("방 청소");
            numJunkPiles = 0;
        }
    }
    
    private final State state;
    
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles){
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }
    
    @Override public void close(){
        cleanable.clean();
    }
    
}
```

### 모든 Room 생성을 try-with-resources 블록으로 감싼다면 자동 청소는 전혀 필요하지 않음
```java
public class Adult{
    public static void main(String[] args) {
        try(Room myRoom = new Room(7)){
            System.out.println("안녕~");
        }
    }
}
```

### 방 청소를 하지 않는 프로그램
```java
public class Teenager {
    public static void main(String[] args) {
        new Room(99);
        System.out.println("아무렴");
    }
}
```

## 정리
cleaner는 안정망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자,
물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다.
