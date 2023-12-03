# 아이템 24. 멤버 클래스는 되도록 static으로 만들라. 

##  핵심정리
네 종류의 중첩 클래스와 각각의 쓰임

### (1) 정적 멤버 클래스
- 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미 클래스. 예) Calculator.Operation.PLUS

```java
public class OutterClass {

  private static int number = 10;

  // 바깥 클래스에 있는 static 변수에 접근할 수 있다. 
  static private class InnerClass {
    void doSomething() {
      System.out.println(number);
    }
  }

  public static void main(String[] args) {
    InnerClass innerClass = new InnerClass();
    innerClass.doSomething();

  }
}
```

### (2) 비정적 멤버 클래스
- 바깥 클래스의 인스턴스와 암묵적으로 연결된다.
- 감싸고 있는 바깥 클래스 없이는 자기 자신을 생성할 수 없다.
- 어댑터를 정의할 때 자주 쓰인다.
- `멤버 클래스에서 바깥 인스턴스를 참조할 필요가 없다면 무조건 정적 멤버 클래스로 만들자.`

```java
public class OutterClass {

  private int number = 10;

  void printNumber() {
    InnerClass innerClass = new InnerClass();
  }

  // 비정적 멤버 클래스
  private class InnerClass {
    void doSomething() {
      System.out.println(number);
      OutterClass.this.printNumber();
    }
  }

  public static void main(String[] args) {
    // 바깥 클래스 없이는 자기 자신을 생성할 수 없다.
    InnerClass innerClass = new OutterClass().new InnerClass();
    innerClass.doSomething();
  }

}
```

### (3) 익명 클래스
- 바깥 클래스의 멤버가 아니며, 쓰이는 시점과 동시에 인스턴스가 만들어진다.
- 비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조할 수 있다.
- 자바에서 람다를 지원하기 전에 즉석에서 작은 함수 객체나 처리 객체를 만들 때 사용했다.
- 정적 팩터리 메서드를 만들 때 사용할 수도 있다.

```java
// 코드 20-1 골격 구현을 사용해 완성한 구체 클래스 (133쪽)
public class IntArrays {
  static List<Integer> intArrayAsList(int[] a) {
    Objects.requireNonNull(a);

    // 익명 클래스
    // 다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능하다.
    // 더 낮은 버전을 사용한다면 <Integer>로 수정하자.
    return new AbstractList<>() {
      @Override
      public Integer get(int i) {
        return a[i];  // 오토박싱(아이템 6)
      }

      @Override
      public Integer set(int i, Integer val) {
        int oldVal = a[i];
        a[i] = val;     // 오토언박싱
        return oldVal;  // 오토박싱
      }

      @Override
      public int size() {
        return a.length;
      }
    };
  }

  public static void main(String[] args) {
    int[] a = new int[10];
    for (int i = 0; i < a.length; i++)
      a[i] = i;

    List<Integer> list = intArrayAsList(a);
    Collections.shuffle(list);
    System.out.println(list);
  }
}
```

### (4) 지역 클래스
- 가장 드물게 사용된다.
- 지역 변수를 선언하는 곳이면 어디든 지역 클래스를 정의해 사용할 수 있다.
- 가독성을 위해 짧게 작성해야 한다.

```java
public class MyClass {

  private int number = 10;

  void doSomething() {
    class LocalClass {
      private void printNumber() {
        System.out.println(number);
      }
    }

    LocalClass localClass = new LocalClass();
    localClass.printNumber();
  }

  public static void main(String[] args) {
    MyClass myClass = new MyClass();
    myClass.doSomething();
  }
}
```

###
## 2. 추가 지식
- p147, 어댑터

### (1) 어댑터 패턴
기존 코드를 클라이언트가 사용하는 인터페이스의 구현체로 바꿔주는 패턴
- 클라이언트가 사용하는 인터페이스를 따르지 않는 기존 코드를 재사용할 수 있게 해준다.
![어댑터 패턴](https://velog.velcdn.com/images/weekbelt/post/995ad03c-7b5f-4ea6-9a12-3149012df661/image.png)