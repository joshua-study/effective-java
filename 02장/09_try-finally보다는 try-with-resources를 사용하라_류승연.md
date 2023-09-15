# 아이템 9. try-finally보다는 try-with-resources를 사용하라

자원이 둘 이상이면 try-finally도 close의 호출을 보장하지만, 코드의 가독성이 떨어진다.
이런 문제들을 자바 7의 try-with-resources로 해결되었다.\

해당 자원이 AutoClosable 인터페이스를 구현해야 한다.
```java
public static String firstLineOfFile(String path) throw IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (Exception e) {
        return defaultVal;
    }
}
```
```java
static void copy(String src, String dst) throws IOException {
	try (InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n = in.read(buf)) >= 0)
		out.write(buf, 0, n);
	}
}
```

첫 번째 코드는 자원을 회수하는 최선책이며, 두 번째 코드는 복수의 자원을 처리하는 try-with-resources 이다. 이 버전이 짧고 읽기 수월할 뿐 아니라 문제를 진단하기도 좋다.
try-with-resources 에서도 catch절을 쓸 수 있으며, 오히려 덕분에 try 문을 더 중첩하지 않고도 다수의 예외를 처리할 수 있다.

## 정리
꼭 회수해야 하는 자원을 다룰 때는 try-finall말고 try-with-resources를 사용하자. <br>
예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다. <br>
try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resources로는 정확하고 쉽게 자원을 회수할 수 있다.