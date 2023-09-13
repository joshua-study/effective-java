# 아이템 8. finalizer와 cleaner 사용을 피하라

## 1. 핵심정리
- try-finally는 더이상 최선의 방법이 아니다. (자바7부터)
- try-with-resources를 사용하면 코드가 더 짧고 분명하다.
- 만들어지는 예외 정보도 훨씬 유용하다.

### (1) try-finally
- 자원이 둘 이상이면 try-finally 방식은 너무 지저분하다.
- 2개 이상의 자원을 처리하려면 중첩 try-finally를 사용해야 한다.
- 또한, 예외가 2개 발생하는 경우는 마지막 예외만 보이는 문제가 생길 수 있다.

###
- 2개 이상의 자원을 처리의 잘못된 예
```java
public class Copy {
  private static final int BUFFER_SIZE = 8 * 1024;

  // 코드 9-2 자원이 둘 이상이면 try-finally 방식은 너무 지저분하다! (47쪽)
  static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
      OutputStream out = new FileOutputStream(dst);
      byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while ((n = in.read(buf)) >= 0)
        out.write(buf, 0, n);
    } finally {
      out.close(); // 여기에서 에러가 생기면? in은 닫히지 않음
      in.close();
    }
  }
}
```

- 2개 이상의 자원을 처리하기 위한 중첩 try-finally
```java
public class Copy {
  private static final int BUFFER_SIZE = 8 * 1024;

  // 코드 9-2 자원이 둘 이상이면 try-finally 방식은 너무 지저분하다! (47쪽)
  static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
      OutputStream out = new FileOutputStream(dst);
      try {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
          out.write(buf, 0, n);
      } finally {
        out.close();
      }
    } finally {
      in.close();
    }
  }
}
```
```java
public class TopLine {
  // 코드 9-1 try-finally - 더 이상 자원을 회수하는 최선의 방책이 아니다! (47쪽)
  static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
      return br.readLine();
    } finally {
      br.close();
    }
  }

  public static void main(String[] args) throws IOException {
    String path = args[0];
    System.out.println(firstLineOfFile(path));
  }
}

```

### (2) try-with-resources
- 복수의 자원을 처리하는 방법은 try-with-resources를 사용해야한다.
- 예외가 2개 발생하는 경우는 모든 예외를 볼 수 있다.
- try-resources를 사용하려면 자원을 닫는 클래스가 Closeable을 구현하고 있는지 확인해야한다.

```java
public class Copy {
  private static final int BUFFER_SIZE = 8 * 1024;

  // 코드 9-4 복수의 자원을 처리하는 try-with-resources - 짧고 매혹적이다! (49쪽)
  static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
        OutputStream out = new FileOutputStream(dst)) {
      byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while ((n = in.read(buf)) >= 0)
        out.write(buf, 0, n);
    }
  }

  public static void main(String[] args) throws IOException {
    String src = args[0];
    String dst = args[1];
    copy(src, dst);
  }
}
```

```java
public class TopLine {
  // 코드 9-3 try-with-resources - 자원을 회수하는 최선책! (48쪽)
  static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
      return br.readLine();
    }
  }

  public static void main(String[] args) throws IOException {
    String path = args[0];
    System.out.println(firstLineOfFile(path));
  }
}
```

## 2. 추가 지식
- p48, 자바 퍼즐러 예외 처리 코드의 실수
- p49, try-with-resources 바이트코드

### (1) 자바 퍼즐러
- 자바 퍼즐러에서의 코드 실수
```java
public String request() {
        HttpsURLConnection conn;
        OutputStreamWriter writer = null;
        BufferedReader reader = null;
        InputStreamReader isr = null;

        try {
          ...

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // 안전할까?
            // 만약 IOException이 아니라, RuntimeException이 발생하면 문제가 생김
            if (writer != null) try { writer.close(); } catch (IOException e) { }
            if (reader != null) try { reader.close(); } catch (IOException e) { }
            if (isr != null) try { isr.close(); } catch (IOException e) { }
        }

        return null;
}
```

### (2) try-with-resources가 어떻게 구현되어 있는지?
```java
public class TopLine {
  static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
      return br.readLine();
    }
  }

  public static void main(String[] args) throws IOException {
    String path = args[0];
    System.out.println(firstLineOfFile(path));
  }
}
```
- build가 된 .class을 try-with-resources를 보게되면?
- Show byte viewer
```java
public class TopLine {
  public TopLine() {
  }

  static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));

    String var2;
    try {
      var2 = br.readLine();
    } catch (Throwable var5) {
      try {
        br.close();
      } catch (Throwable var4) {
        // 후속 예외를 쌓아준다.
        var5.addSuppressed(var4);
      }

      throw var5;
    }

    br.close();
    return var2;
  }

  public static void main(String[] args) throws IOException {
    String path = args[0];
    System.out.println(firstLineOfFile(path));
  }
}
```