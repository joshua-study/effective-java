# 아이템 8. finalizer와 cleaner 사용을 피하라

- 객체 소멸자인 finalizer와 cleaner.
- finalizer와 cleaner는 즉시 수행된다는 보장이 없다.
- 실행 자체가 되지 않을 수도 있다. == 리소스가 반납되지 않는다.
- finalizer 동작 중에 예외가 발생하면 정리 작업이 처리되지 않을 수도 있다.
- 이것은 곧 심각한 성능 문제를 유발시킴.
- finalizer는 보안상의 문제가 있다.
- **반납할 자원이 있는 클래스는 AutoCloseable을 구현하고 클라이언트에서 close()를 호출하거나 try-with-resource를 사용해야 한다.**


## 1. finalizer
- Ojbect에 정의 되어 있는 finalizer 메서드를 통해 자원 정리 
- 하지만 자바 9부터는 finalizer보다 PhantomReference, Cleaner 사용을 권장하고
- 더 나아가 AutoCloseable, try-with-resource 사용을 권장
```java
public class FinalizerIsBad {

  @Override
  protected void finalize() throws Throwable{
    System.out.print("");
  }
}
```

```java
public class App {

    //실행해보니 리소스가 몇 백만개씩 쌓임..
    //큐를 처리하는 스레드의 우선순위가 더 낮기 때문에 처리가 안되는 것.
    //그래서 finalize를 사용하지 않음.
    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException, IllegalAccessException {
        int i = 0;
        while(true){
            i++;
            new FinalizerIsBad();

            if ((i % 1_000_000) == 0) {
                Class<?> finalizerClass = Class.forName("java.lang.ref.Finalizer");
                Field queueStaticField = finalizerClass.getDeclaredField("queue");
                queueStaticField.setAccessible(true);
                ReferenceQueue<Object> referenceQueue = (ReferenceQueue) queueStaticField.get(null);

                Field queueLengthField = ReferenceQueue.class.getDeclaredField("queueLength");
                queueLengthField.setAccessible(true);
                long queueLength = (long) queueLengthField.get(referenceQueue);
                System.out.format("There are %d references in the queue%n", queueLength);
            }
        }
    }
}
```

##
## 2. Cleaner
- finalizer가 하던 작업을 Runnable의 구현체를 정의하여 자원 정리
- **Runnable 구현체에 정리하려는 자원을 참조해선 안됨.**
- GC가 될 때 수행되는 Runnable 구현체가 자원을 참조하게 되면 정리하려던 자원이 다시 생성될 수 있음.
```java
public class BigObject {
    
    private List<Object> resource;
    public BigObject(List<Object> resource) {
        this.resource = resource;
    }
    
    public static class ResourceCleaner implements Runnable {
        private List<Object> resourceToClean;

        public ResourceCleaner(List<Object> resourceToClean) {
            this.resourceToClean = resourceToClean;
        }

        @Override
        public void run() {
            resourceToClean = null;
            System.out.println("cleaned up");
        }
    }
}
```

```java
public class CleanerIsNotGood {

  public static void main(String[] args) throws InterruptedException {
    Cleaner cleaner = Cleaner.create();

    // 사용하고 싶은 객체
    List<Object> resourceToCleanUp = new ArrayList<>();
    BigObject bigObject = new BigObject(resourceToCleanUp);
    
    // cleaner에 등록
    // BigObject에 구현한 task를 사용해서 자원 정리
    cleaner.register(bigObject, new BigObject.ResourceCleaner(resourceToCleanUp));

    bigObject = null;
    System.gc();
    Thread.sleep(3000L);
  }
}
```
###
### 2-1. Cleaner 사용 시점
- try-with-resource를 사용하지 않았을 때에 <u>`자원 반납의 안전망`</u>으로 사용할 수 있다.
- **네이티브 피어 자원을 해제할 수 있는 기회를 준다.**
  - 네이티브 피어란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다.
  - 네이티브 객체는 자바 객체가 아니므로 GC가 네이티브 객치를 회수하지 못한다. (존재 자체를 알지 못함.)
  - 이런 상황을 방지하기 위해 cleaner가 사용되는 것.
  - 단, 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때에만 해당
   
## 
## 3. 완벽 공략
- 정적이 아닌 중첩 클래스는 자동으로 바깥 객체의 참조를 갖는다.
  - 중첩 클래스에서 Runnable 구현체를 정의한다면 static 선언 필요.
- 람다 역시 바깥 객체의 참조를 갖기 쉽다.

###
### 3-1. finalizer 공격
- 만들다 만 객체를 finalize 메서드에서 사용하는 방법
```java
public class Account {

  private String accountId;

  public Account(String accountId) {
    this.accountId = accountId;

    if (accountId.equals("대표")) {
      throw new IllegalArgumentException("대표 계정을 막습니다.");
    }
  }

  public void transfer(BigDecimal amount, String to) {
    System.out.printf("transfer %f from %s to %s\n", amount, accountId, to);
  }
}
```

```java
public class BrokenAccount extends Account {

  public BrokenAccount(String accountId) {
    super(accountId);
  }

  // finalize 재정의하고 transfer()를 다시 호출
  @Override
  protected void finalize() throws Throwable {
    this.transfer(BigDecimal.valueOf(7000), "SH");
  }
}
```

```java
class AccountTest {

    @Test
    void 푸틴_계정() throws InterruptedException {
        // 일반
        Account account = new Account("대표");
        account.transfer(BigDecimal.valueOf(10.4), "hello");

        // 공격
        Account account = null;
        try {
            account = new BrokenAccount("대표");
        } catch (RuntimeException e) {
            System.out.println("이러면???");
        }

        // gc 수행 시 finalize 메서드 실행
        System.gc();
        Thread.sleep(3000L);
    }
}
```
### 3-1-1. finalizer 공격 방어
- 상속을 받을 수 없게 Account에 final 선언.
- 상속이 필요한 경우, Account에 아무 일도 하지 않는 finalize 메서드를 만들고 final 선언.
  - 상속은 가능하지만 finalize 메서드를 재정의할 수 없음.
```java
public class Account {
  // 기존 코드
  ...
  
  // final 선언으로 오버라이드 허용x
  @Override
  protected final void finalize() throws Throwable {
  }
}
```

##
## 4. AutoCloseable
try-with-resource를 지원하는 인터페이스

- void close() throws Exception
  - 인터페이스에 정의된 메서드에서 Exception 타입으로 예외를 던지지만
  - 실제 구현체에서는 구체적인 예외를 던지는 것을 추천하며
  - 가능하다면 예외를 던지지 않는 것도 권장한다.
- Closeable 클래스와 차이점
  - 예외를 던지지 않거나
  - 던진다면 IOException을 던지며
  - AutoCloseable은 idempotent 하면 좋지만 Closeable 반드시 idempotent 해야한다.
    - idempotent(멱등) : 여러 번 호출될 때마다 같은 결과가 발생해야 함.

```java
public class AutoClosableIsGood implements AutoCloseable{

  private BufferedReader reader;

  public AutoClosableIsGood(String path) {
    try {
      this.reader = new BufferedReader(new FileReader(path));
    } catch (FileNotFoundException e) {
      throw new IllegalArgumentException(path);
    }
  }
  
  // 1.
  // BufferedReader 사용시 IOException 발생
  // close()에서 예외를 던지는데 BufferedReader를 사용하는 client가 예외를 잡아줘야 함 (책임 전가)
  // 예외를 던질 때 구체적 예외 던지기.
  @Override
  public void close() throws IOException {
    reader.close();
  }

  // 2.
  // close() 자체적으로 예외 잡아줄 수도 있음.
  // client에게 책임전가x
  @Override
  public void close() {
    try {
      reader.close();
    } catch (IOException e) {
      // 예외를 던져 스레드를 종료시키거나
      // logging을 할 수도
      throw new RuntimeException(e);
    }
  }
}
```

```java
public class App {
    
    public static void main(String[] args) {
        // close()가 호출될 때 예외가 발생하니 
        try (AutoClosableIsGood good = new AutoClosableIsGood("")) {
            // TODO 자원 반납 처리가 됨.
        // client에서 잡아야 함.
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```