# 아이템 13. clone 재정의는 주의해서 진행하라.

## 1. 핵심정리
애매모호한 clone 규약
- clone 규약
  - x.clone() != x 반드시 ture
  - x.clone().getClass() == x.getClass() 반드시 ture
  - x.clone().equals(x) true가 아닐 수도 있다.
    - 객체를 식별하는 id와 같은 값이 다르게 가지고 있다면 false
- 불변 객체라면 다음으로 충분하다.
  - Cloenable 인터페이스를 구현하고
  - clone 메서드를 재정의한다. 이때 super.clone()을 사용해야 한다. (super.clone()을 사용하지 않으면 TypeCasting Error)

```java
public final class PhoneNumber implements Cloneable {
  
  private final short areaCode, prefix, lineNum;

  public PhoneNumber(int areaCode, int prefix, int lineNum) {
    this.areaCode = rangeCheck(areaCode, 999, "지역코드");
    this.prefix = rangeCheck(prefix, 999, "프리픽스");
    this.lineNum = rangeCheck(lineNum, 9999, "가입자 번호");
    System.out.println("constructor is called");
  }

  public PhoneNumber(PhoneNumber phoneNumber) {
    this(phoneNumber.areaCode, phoneNumber.prefix, phoneNumber.lineNum);
  }

  private static short rangeCheck(int val, int max, String arg) {
    if (val < 0 || val > max)
      throw new IllegalArgumentException(arg + ": " + val);
    return (short) val;
  }

  // 코드 13-1 가변 상태를 참조하지 않는 클래스용 clone 메서드 (79쪽)
  @Override
  public PhoneNumber clone() {
    try {
      return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
      throw new AssertionError();  // 일어날 수 없는 일이다.
    }
  }

  public static void main(String[] args) {
    PhoneNumber pn = new PhoneNumber(707, 867, 5309);
    Map<PhoneNumber, String> m = new HashMap<>();
    m.put(pn, "제니");
    PhoneNumber clone = pn.clone(); // 생성자를 통해서 만들어지지 않음
    System.out.println(m.get(clone));

    System.out.println(clone != pn); // 반드시 true
    System.out.println(clone.getClass() == pn.getClass()); // 반드시 true
    System.out.println(clone.equals(pn)); // true가 아닐 수도 있다.
  }
}
```

### (1) 가변 객체의 clone 구현하는 방법
- 접근 제어자는 public, 반환 타입은 자신의 클래스로 변경한다.
- super.clone을 호출한 뒤 필요한 필드를 적절히 수정한다.
  - 배열을 복제할 때는 배열의 clone 메서드를 사용하라.
  - 경우에 따라 final을 사용할 수 없을지도 모른다.
  - 필요한 경우 deep copy를 해야한다.
  - super.clone으로 객체를 만든 뒤, 고수준 메서드를 호출하는 방법도 있다.
  - 오버라이딩 할 수 있는 메서드는 참조하지 않도록 조심해야 한다.
  - 상속용 클래스는 Cloneable을 구현하지 않는 것이 좋다.
  - Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 동기화를 해야 한다.

```java
public class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    ... push, pop, isEmpty

    // 코드 13-2 가변 상태를 참조하는 클래스용 clone 메서드
    // TODO stack -> elementsS[0, 1]
    // TODO copy -> elementsC[0, 1]
    // TODO elementsS[0] == elementsC[0]

    @Override public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
    
    // clone이 동작하는 모습을 보려면 명령줄 인수를 몇 개 덧붙여서 호출해야 한다.
    public static void main(String[] args) {
        Object[] values = new Object[2];
        values[0] = new PhoneNumber(123, 456, 7890);
        values[1] = new PhoneNumber(321, 764, 2341);

        Stack stack = new Stack();
        for (Object arg : values)
            stack.push(arg);

        Stack copy = stack.clone();

        System.out.println("pop from stack");
        while (!stack.isEmpty())
            System.out.println(stack.pop() + " ");

        System.out.println("pop from copy");
        while (!copy.isEmpty())
            System.out.println(copy.pop() + " ");

        // 같은 인스턴스를 가지게 된다.
        System.out.println(stack.elements[0] == copy.elements[0]);
    }
}
```

#### shallow copy
```java
/**
 * TODO hasTable -> entryH[],
 * TODO copy -> entryC[]
 * TODO entryH[0] == entryC[0]
 *
 * @return
 */
@Override
   public HashTable clone() {
       HashTable result = null;
       try {
           result = (HashTable)super.clone();
           result.buckets = this.buckets.clone(); // p82, shallow copy 라서 위험하다.
           return result;
       } catch (CloneNotSupportedException e) {
           throw  new AssertionError();
       }
   }
```

#### deep copy
```java
/**
 * TODO hasTable -> entryH[],
 * TODO copy -> entryC[]
 * TODO entryH[0] != entryC[0]
 *
 * @return
 */
    @Override
    public HashTable clone() {
        HashTable result = null;
        try {
            result = (HashTable)super.clone();
            // 객체를 생성할 때 메서드로 빼면 안됨
            // 
            result.buckets = new Entry[this.buckets.length];

            for (int i = 0 ; i < this.buckets.length; i++) {
                if (buckets[i] != null) {
                    result.buckets[i] = this.buckets[i].deepCopy(); // p83, deep copy
                }
            }
            return result;
        } catch (CloneNotSupportedException e) {
            throw  new AssertionError();
        }
    }
```

### (2) clone 대신 권장하는 방법
- `복사 생성자` 또는 변환 생성자, `복사 팩터리` 또는 변환 팩터리
- 생성자를 쓰지 않으며, 모호한 규약, 불필요한 검사 예외, final 용법 방해 등에서 벗어날 수 있다.
- 또 다른 큰 장점 중 하나로 인터페이스 타입의 인스턴스를 인수로 받을 수 있다.
  - 클라이언트가 복제본의 타입을 결정할 수 있다. (원하는 하위타입으로 변환이 가능)
- 읽어볼 것) [Josh Bloch on Design](https://www.artima.com/articles/josh-bloch-on-design), "Copy Constructor versus Cloning"
```java
public final class PhoneNumber implements Cloneable {
  private final short areaCode, prefix, lineNum;

  // 생성자
  public PhoneNumber(int areaCode, int prefix, int lineNum) {
    ...
  }

  // 생성자를 이용해서 copy (clone을 대신)
  // 인터페이스 타입을 받아서 원하는 하위타입으로 변환이 가능해집
  public PhoneNumber(PhoneNumber phoneNumber) {
    this(phoneNumber.areaCode, phoneNumber.prefix, phoneNumber.lineNum);
  }
}
```

#### 대표적인 예로 TreeSet이 있음
```java
public static void main(String[] args) {
        Set<String> hashSet = new HashSet<>();
        hashSet.add("jaehoon");
        hashSet.add("noah");
        System.out.println("HashSet: " + hashSet);

        // Collection을 받아서 하위타입으로 copy
        Set<String> treeSet = new TreeSet<>(hashSet);

        System.out.println("TreeSet: " + treeSet);
    }
```


###
## 2. 추가 지식
- p80, 비검사 예외(UnChecked Exception)였어야 했다는 신호다.
- p81, HashTable과 LinkedList
- p83, 깊은 복사 (deep copy)
- p83, 리스트가 길면 `스택 오버플로`를 일으킬 위험이 있기 때문이다.
- p85, clone 메서드 역시 적절히 `동기화`해줘야 한다.
- p86, TreeSet


### (1) UncheckedException
왜 우리는 비검사 예외를 선호하는가?
- 컴파일 에러를 신경쓰지 않아도 되며,
- try-catch로 감싸거나
- 메서드 선언부에 선언하지 않아도 된다.
- 그렇다면 우리는 비검사 예외만 쓰면 되는걸까? 검사 예외는 왜 있는 것일까?

<br></br>

그렇다면 우리는 비검사 예외만 쓰면 되는걸까?
- 왜 잡지 않은 예외를 메서드에 선언해야 하는가?
  - 메서드에 선언한 예외는 `프로그래밍 인터페이스의 일부`다.
  - 즉, 해당 메서드를 사용하는 코드가 반드시 알아야하는 정보다. 그래야 해당 예외가 발생했을 상황에 대처하는 코드를 작성할 수 있을테니까.
- 비검사 예외는 그럼 왜 메서드에 선언하지 않아도 되는가?
  - 비검사 예외는 어떤식으로든 처리하거나 복구할 수 없는 경우에 사용하는 예외다.
  - 가령, 숫자를 0으로 나누거나, null 레퍼런스에 메서드를 호출하는 등...
  - 이런 예외는 프로그램 전반에 걸쳐 어디서든 발생할 수 있기 때문에 이 모든 비검사 예외를 메서드에 선언하도록 강제한다면 프로그램의 명확도가 떨어진다.

<br></br>

언제 어떤 예외를 써야 하는가?
- 단순히 처리하기 쉽고 편하다는 이유만으로 RuntimeException을 선택하지는 말자.
- 가이드라인 : 클라이언트가 해당 예외 상황을 복구할 수 있다면 검사 예외를 사용하고, 해당 예외가 발생했을 때 아무것도 할 수 없다면, 비검사 예외로 만든다.
- https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html

### (2) TreeSet
AbstactSet을 확장한 `정렬된 컬렉션`
- 엘리먼트를 추가한 순서는 중요하지 않다.
- 엘리먼트가 지닌 자연적인 순서(natural order)에 따라 정렬한다.
- 오름차순으로 정렬한다.
- 스레드 안전하지 않다.
- 과제) 이진 검색트리, 레드 블랙 트리에 대해 학습