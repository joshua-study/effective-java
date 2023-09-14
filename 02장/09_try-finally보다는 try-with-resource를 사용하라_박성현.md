# 아이템 9. try-finally보다는 try-with-resource를 사용하라.

- try-finally는 더이상 최선의 방법이 아니다.
- try-with-resources를 사용하면 코드가 더 짧고 분명하다.
```java
public class Copy {
    private static final int BUFFER_SIZE = 8 * 1024;

    // 중첩 try-finally
    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0) {
                    out.write(buf, 0, n);
                }
            } finally {
                in.close();
            }
        } finally {
            out.close();
        }
    }
    
    public static void main (String[]args) throws IOException {
        String src = args[0];
        String dst = args[1];
        copy(src, dst);
    }
}
```
```java
public class TopLine {
    static void copy(String src, String dst) throws IOException {
        
        // try-with-resource
        try (InputStream in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst);) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0) 
                out.write(buf, 0, n);
        }
    }

    public static void main (String[]args) throws IOException {
        String src = args[0];
        String dst = args[1];
        copy(src, dst);
    }
}
```

- try-with-resource는 예외를 잡아먹지 않는다.
- 2개의 예외처리가 되어야한다 가정해보자.
- try-finally를 사용했을 때 가장 나중에 실행되는 예외'만' 처리되고
- try-with-resource를 사용했을 때 던진 예외가 모두 stacktrace에 출력.
- 코드가 간결해질뿐만아니라 예외처리에도 용이

```java
public class BadBufferedReader extends BufferedReader {
    public BadBufferedReader(Reader in, int sz) {
        super(in, sz);
    }

    public BadBufferedReader(Reader in) {
        super(in);
    }
    
    @Override
    public String readLine() throws IOException {
        throw new CharConversionException();
    }
    @Override
    public void close() throws IOException {
        throw new StreamCorruptedException();
    }
} 
```
```java
public class TopLine {
    static String firstLineOfFile(String path) throws IOException {
        // 1.
        BufferedReader br = new BadBufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            // 가장 나중에 처리되는 예외'만' stack에 출력
            br.close();
        }

        // 2.
        // try-with-resource 사용으로 2가지 예외 모두 처리
        try(BufferedReader br = new BadBufferedReader(new FileReader(path))) {
            return br.readLine();
        }
    }

    public static void main (String[]args) throws IOException {
        System.out.println(firstLineOfFile("pom.xml"));
    }
}
```

##
## 그래서 try-with-resource는 어떻게 구현되는 걸까.

- 실제 작성한 TopLine.java
```java
public class TopLine {
    static String firstLineOfFile(String path) throws IOException {
        try (BufferedReader br = new BadBufferedReader(new FileReader(path))) {
            return br.readLine();
        }
    }

    public static void main (String[]args) throws IOException {
        String path = args[0];
        System.out.println(firstLineOfFile(path));
    }
}
```
- target 파일의 TopLine.class
```java
public class TopLine {
    public TopLine () {
    }
    
    static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BadBufferedReader(new FileReader(path));
        
        // 실행될 코드
        String var2;
        try {
            var2 = br.readLine();
        } catch (Throwable var5) { // readLine()에서 발생될 예외
            try {
                br.close();
            } catch (Throwable var4) { // close()할 때 후속으로 발생하는 예외가 있다면
                var5.addSuppressed(var4); // var5 예외를 먼저 출력한 후 Suppressed로 stack에 출력
            }
            
            throw var5;
        }
        
        br.close();
        return var2;
    }

    public static void main (String[]args) throws IOException {
        String path = args[0];
        System.out.println(firstLineOfFile(path));
    }
}
```