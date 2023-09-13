# 아이템 8. finalizer와 cleaner 사용을 피하라

## 1. 핵심정리
- finalizer와 cleaner는 즉시 수행된다는 보장이 없다.
- finalizer와 cleaner는 실행되지 않을 수도 있다.
- finalizer 동작 중에 예외가 발생하면 정리 작업이 처리되지 않을 수도 있다.
- finalizer와 cleaner는 심각한 성능 문제가 있다.
- finalizer는 보안 문제가 있다.
- `반남할 자원이 있는 클래스는 AutoCloseable을 구현하고 클라이언트에서 close()를 호출하거나 try-with-resource를 사용해야 한다.`

## 
### (1) finalizer 사용
- finalize 메서드를 통해 자원을 반납할 수 있지만 사용을 권장하지 않음
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
  /**
   * 코드 참고 https://www.baeldung.com/java-finalize
   */
  public static void main(String[] args) throws InterruptedException, ClassNotFoundException, NoSuchFieldException, IllegalAccessException {
    int i = 0;
    while(true) {
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

### (2) cleaner 사용
- `자원 반납용 안전망`으로 사용할 수 있다.
  - PhantomReference를 사용한다.
  - 호출되리라는 보장이 없지만 없는 것 보다는 나을 수 있다.
- `네이티브 피어` 자원 회수
  - 단, 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때에만 해당한다.
  - 네이티브 피어가 사용하는 자원을 즉시 회수해야 한다면 close() 메서드를 호출해야 한다.

```java
public class BigObject {

  private List<Object> resource;

  public BigObject(List<Object> resource) {
    this.resource = resource;
  }

  public static class ResourceCleaner implements Runnable {

    // BigObject를 참조하면 X
    private List<Object> resourceToClean;

    public ResourceCleaner(List<Object> resourceToClean) {
      this.resourceToClean = resourceToClean;
    }

    @Override
    public void run() {
      resourceToClean = null;
      System.out.println("cleaned up.");
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

    // cleaner 큐에 등록
    // BigObject의 Runable을 사용해서 청소
    cleaner.register(bigObject, new BigObject.ResourceCleaner(resourceToCleanUp));

    bigObject = null;
    System.gc();
    Thread.sleep(3000L);
  }
}
```

## 2. 추가 지식
- p42, Finalizer 공격
- p43, AutoClosable
- p45, 정적이 아닌 중첩 클래스는 자동으로 바깥 객체의 참조를 갖는다.
```java
public class BigObject {

  private List<Object> resource;
  ...
  
  // 내부에 클래스를 만든다면 static 클래스로 만들어라 (static이 아니면 내부클래스는 외부클래스를 참조하게 됨)
  public static class ResourceCleaner implements Runnable {
    ...
  }
}
```
- p45, 람다 역시 바깥 객체의 참조를 갖기 쉽다.

### (1) Finalizer 공격
- 만들다 만 객체를 finalize 메서드에서 사용하는 방법
  - 객체를 상속하여 finalize() 메서드를 재정의하여 Finalizer 공격이 가능
```java
public class BrokenAccount extends Account {

  public BrokenAccount(String accountId) {
    super(accountId);
  }

  @Override
  protected void finalize() throws Throwable {
    this.transfer(BigDecimal.valueOf(100000), "jaehoon");
  }
}

```

```java
public class Account {

  private String accountId;

  public Account(String accountId) {
    this.accountId = accountId;

    if (accountId.equals("푸틴")) {
      throw new IllegalArgumentException("푸틴은 계정을 막습니다.");
    }
  }

  public void transfer(BigDecimal amount, String to) {
    System.out.printf("transfer %f from %s to %s\n", amount, accountId, to);
  }
}
```

```java
@Test
  void 푸틴_계정() throws InterruptedException {
    // Account account = new Account("푸틴");
    // account.transfer(BigDecimal.valueOf(10.4), "hello");

    Account account = null;
    try {
      account = new BrokenAccount("푸틴");
    } catch (RuntimeException e) {
      System.out.println("이러면???");
    }

    // gc를 수행하면 finalize 메서드가 실행된다.
    // finalize 메서드로 공격이 가능해짐
    System.gc();
    Thread.sleep(3000L);
  }
```

- finalizer 공격을 방어하려면 아무 일도 하지 않는 finalize 메서드를 만들고 final로 선언하자.
```java
public class Account {
  ...
  @Override
  protected final void finalize() throws Throwable {
  }
}

public class BrokenAccount extends Account {

  public BrokenAccount(String accountId) {
    super(accountId);
  }

  // final로 선언했기 때문에 finalize메서드를 오버라이딩 할 수 없음
  // @Override
  // protected void finalize() throws Throwable {
  //   this.transfer(BigDecimal.valueOf(100000), "jaehoon");
  // }
}
```

### (2) AutoClosable
try-with-resource를 지원하는 인터페이스
- void close() throws Exception
  - 인터페이스에 정의된 메서드에서 Exception 타입으로 예외를 던지지만
  - 실제 구현체에서는 구체적인 예외를 던지는 것을 추천하며
  - 가능하다면 예외를 던지지 않는 것도 권장한다.
  - 가급적 idempotent(멱등) 해야한다.
- Closeable 클래스와 차이점
  - void close() throws IOException
  - IOException을 던지며 반드시 idempotent(멱등) 해야한다.
  - (몇번이 실행되던지 항상 같은 결과가 발생해야 한다.)

```java
// AutoCloseable을 사용하면 try with resource를 통해 자원 반납을 자동으로 해줌
public class AutoClosableIsGood implements AutoCloseable {

  private BufferedInputStream inputStream;

  @Override
  public void close() throws Exception {
    try {
      inputStream.close();
    } catch (IOException e) {
      throw new RuntimeException("failed to close" + inputStream);
    }
  }
}

```

```java
public class App {

  public static void main(String[] args) {
    try (AutoClosableIsGood good = new AutoClosableIsGood()) {
      // TODO 자원 반납 처리기
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
}

```