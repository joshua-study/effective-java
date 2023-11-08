# 24. 멤버 클래스는 되도록 static으로 만들라.

---

## (1) 중첩 클래스

### 비정적 멤버 클래스
- 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결
- 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this를 사용해 바깥 인스턴스의 메서드 호출, 참조를 가져올 수 있다.
  - 정규화된 this : 클래스명.this 형태로 바깥 클래스의 이름을 명시하는 용법
- 비정적 멤버 클래스는 바깥 인스턴스 없이는 생성할 수 없다.

```java
public class OutterClass {

    private int number = 10;

    void printNumber() {
        InnerClass ex = new InnerClass();
    }

    private class InnerClass {
        void doSomething() {
            System.out.println(number);
            OutterClass.this.printNumber();
        }
    }

    // 비정적 멤버 클래스인 경우
    // OutterClass의 인스턴스를 먼저 생성한 후 접근
    public static void main(String[] args) {
        InnerClass innerClass = new OutterClass().new InnerClass();
        innerClass.doSomething();
    }
}
```
```
※ 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 static을 붙여 정적 멤버 클래스로 만들자.
- static 생략 시 바깥 인스턴스로부터 숨은 외부 참조를 갖게 된다.
- GC가 바깥 클래스의 인스턴스를 수거하지 못해 메모리 누수가 생길 수 있다.
```


### 정적 멤버 클래스
- 다른 클래스 안에 선언
- 바깥 클래스의 인스턴스를 필요로 하지 않고 독립적으로 존재
- 바깥 클래스(outer class)의 private 멤버에도 접근 가능
- 다른 정적 멤버와 똑같은 접근 규칙을 적용받는다.
  - private으로 선언하면 바깥 클래스에서만 접근 가능
```java
public class OutterClass {

    static private class InnerClass {
        private int number = 10;
        
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

### 익명 클래스
- 쓰이는 시점에 선언과 동시에 인스턴스가 생성
- 하지만 선언한 지점에서만 인스턴스를 만들 수 있고
- instanceof 검사나 클래스의 이름이 필요한 작업은 수행할 수 없다.
- 자바8에서 람다 등장 이후로 잘 쓰이지 않는다.
```java
public class IntArrays {
    static List<Integer> intArrayAsList(int[] a) {
        Objects.requireNonNull(a);
        
        // 선언과 동시에 인스턴스 생성
        return new AbstractList<Integer>() {
            @Override
            public Integer get(int i) {
                return a[i];
            }

            @Override
            public Integer set(int i, Integer val) {
                int oldVal = a[i];
                a[i] = val;
                return oldVal; 
            }

            @Override
            public int size() {
                return a.length;
            }
        };
    }
}
```

### 지역 클래스
- 가장 드물게 사용
- 지역변수를 선언할 수 있는 곳이라면 어디서든 선언 가능
- 멤버 클래스처럼 이름이 있고 반복해서 사용할 수 있다.

---

## (2) 어댑터
기존 코드를 클라이언트가 사용하는 인터페이스의 구현체로 바꿔주는 패턴

<img src="https://velog.velcdn.com/images/weekbelt/post/995ad03c-7b5f-4ea6-9a12-3149012df661/image.png">

- 클라이언트가 사용하는 인터페이스를 따르지 않는 기존 코드를 재사용할 수 있게 해준다.
- 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용.
```java
class MySet<E> extends AbstractSet<E> {

    @Override
    public Iterator<E> iterator() {
        return new MyIterator();
    }

    @Override
    public int size() {
        return 0;
    }

    // 새로 만든 어댑터
    private class MyIterator implements Iterator<E> {

        @Override
        public boolean hasNext() {
            return false;
        }

        @Override
        public E next() {
            return null;
        }
    }
}
```

---

## 핵심 정리
```
메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기엔 너무 길다면 멤버 클래스로 만든다.

멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면 비정적으로, 그렇지 않으면 정적으로 만들자.

중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳이고
해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 있다면 익명 클래스로 만들고,
그렇지 않으면 지역 클래스로 만들자.
```