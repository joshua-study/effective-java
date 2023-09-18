### try-finally - 더 이상 자원을 회수하는 최선의 방책이 아니다.!
```java
class sampleCode() {
    
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

### 자원이 둘 이상이면 try-finally 방식은 너무 지저분하다!
```java
static void copy(String src, String dst) throws IOException{
    InputStream in = new FileInputStream(src);
    try{
        OutputStream out = new FileOutputStream(dst);
        try{
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while((n=in.read(buf))) >=0 )
                out.write(buf, 0, n);
        
        }finally{
            out.close();
        }
        
        }finally{
        in.close();
        }
    
}
```
물리적인 문제가 생긴다면 firstLineOfFile 메서드 안의 readLine 메서드가 예외를 던지고, 
같은 이유로 close 메서드도 실패할 것이다.
이러한 상황이라면 두번째 예외가 첫 번째 예외를 완전히 삼겨 버림

### try-with-resources - 자원을 회수하는 최선책
```java
static String firstLineOfFile(String path) throws IOException {
    try(BufferedReader br = new BufferedReader( new FileReader(path))){
        return br.readLine();
    }
        }
```

### 복수의 자원을 처리하는 try-with-resources 짧고 매혹적이다.
```java
static void copy(String src, String dst) throws IOException{
    try(InputStream in = new FileInputStream(src);
        OutputStream out = new FileOutputStream(dst)
        ){
        byte[] buf = new byte[BUFFER_SIZE];
        int n; 
        while((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
        }
}
```
readLine과 close 호출 약쪽에서 예외가 발생하면, close에서 발생한 예외는 숨겨지고
readLine에서 발생한 예외가 기록됨

### try-with-resouces를 catch 절과 함께 쓰는 모습
```java
static String firstLineOfFile(String path,String defaultVal){
        try(BufferedReader br=new BufferedReader(
        new FileReader(path))){
            return br.readLine();
        }catch(IOException e){
            return defaultVal;
        }
}
```

## 정리
꼭 회수해야 하는 자원을 다룰 때는 try~finll 말고, try-with-resources를 사용하자
코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용함