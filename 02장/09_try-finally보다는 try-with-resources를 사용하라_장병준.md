# 아이템 9. try-finally보다는 try-with-resources를 사용하라.

### 자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야하는 자원이 많다.
#### 방법
1. try-finally (전통적인 방법)
2. try-with-resources (자바 7 이후)
---
### 1) try-finally
#### 예제1) try-finally 는 더 이상 자원을 회수하는 최선의 방책이 아니다!
```java
public class item9 {
    static String firstLineOfFile(String path) throws IOException {
            BufferedReader br = new BufferedReader(new FileReader(path));
            try {
                return br.readLine();
            } finally {
                br.close();
            }
        }
}
```
#### 예제2) 자원 두개
```java
public class Item9 {
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
                out.close();
            }
        } finally {
            in.close();
        }
    }
}
```
예외는 try 블록과 finally 블록 모두에서 발생할 수 있는데, 예컨대 기기에 물리적인 문제가 생긴다면 firstLineOfFile 메서드 안의 readLine 메서드가 예외를 던지고, 같은 이유로
close 메서드도 실패할 것이다.  
이런 상황이라면 두 번째 예외가 첫 번째 예외를 완전히 집어삼켜 버린다.  
그러면 스택 추적 내역에 첫 번째 예외에 관한 정보는 남지 않게 되어, 실제 시스템에서의 디버깅을 몹시 어렵게 한다(일반적으로 문제를 진단하려면 처음 발생한 예외를 보고 싶을 것이다).
---
### 2) try-with-resources
```java
public class Item9 {
    static String firstLineOfFile(String path) throws IOException {
      try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
      }
    }
}
```
```java
public class Item9 {
    static void copy(String src, String dst) throws IOException {
      try (InputStream   in = new FileInputStream(src);
           OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
          out.write(buf, 0, n);
      }
    }
}
```
#### 장점
1. 편리함과 실수 방지도 있지만, 누락없이 모든 자원을 반납할 수 있다는 점도 있다. 
2. 코드를 간결하게 만들 수 있다.
3. 번거로운 자원 반납 작업을 하지 않아도 된다 
4. 실수로 자원을 반납하지 못하는 경우를 방지할 수 있다. 
5. 에러로 자원을 반납하지 못하는 경우를 방지할 수 있다.
6. 모든 에러에 대한 스택 트레이스를 남길 수 있다.
#### 주의할 점
try-with-resources 구조를 사용하려면 해당 자원이 AutoCloseable 인터페이스를 구현해야 한다.
닫아야 하는 자원을 뜻하는 클래스를 작성한다면 AutoCloseable을 반드시 구현하도록 하자.
```java
public abstract class InputStream extends Object implements Closeable {
    ...
}

public interface Closeable extends AutoCloseable {
    void close() throws IOException;
}
```
---
### 핵심정리
꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고 try-with-resources를 사용하자.   
예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다.   
try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resources로는 정확하고 쉽게 자원을 회수할 수 있다. **