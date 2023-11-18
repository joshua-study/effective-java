# 아이템 31. 한정적 와일드카드를 사용해 API 유연성을 높이라

##  핵심정리
### (1) Chooser와 Union API 개선
- PECS: Producer-Extends, Consumer-Super
  - Producer-Extends
    - Object의 컬렉션 Number나 Integer를 넣을 수 있다.
    - Number의 컬렉션에 Integer를 넣을 수 있다.
  - Consumer-Super
    - Integer의 컬렉션의 객체를 꺼내서 Number의 컬렉션에 담을 수 있다.
    - Number나 Integer의 컬렉션의 객체를 꺼내서 Object의 컬렉션에 담을 수 있다.

- 뭔가를 저장하는 쪽은 extends, 뭔가를 빼는 쪽은 super -> 유연한 API설계가 가능해진다.
###
[PECS 스택오버플로우](https://stackoverflow.com/questions/4343202/difference-between-super-t-and-extends-t-in-java/4343547#4343547)
###

### 한정적 와일드카드를 사용하는 예 
```java
public static void main(String[] args) {
    // Integer pushAll
    Stack<Number> numberStack = new Stack<>();
    Iterable<Integer> integers = Arrays.asList(3, 1, 4, 1, 5, 9);
    numberStack.pushAll(integers);

    // Double pushAll
    Iterable<Double> doubles = Arrays.asList(3.1, 1.0, 4.0, 1.0, 5.0, 9.0);
    numberStack.pushAll(doubles);
    
    // Object popAll
    Collection<Object> objects = new ArrayList<>();
    numberStack.popAll(objects);
  }
```

### Producer-Extends의 예
```java
// 코드 31-1 와일드카드 타입을 사용하지 않으면 Number의 하위 Integer를 넣을 수 없다.
  public void pushAll(Iterable<E> src) {
    for (E e : src)
      push(e);
  }

// 코드 31-2 E 생산자(producer) 매개변수에 와일드카드 타입 적용 (182쪽)
  public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
    push(e);
  }
  
```

### Consumer-Super의 예
```java
// 코드 31-3 와일드카드 타입을 사용하지 않으면 Number의 상위 Object는 꺼낼 수 없다.
// Collection<E>의 Number와 Object는 다름
  public void popAll(Collection<E> dst) {
    while (!isEmpty())
      dst.add(pop());
  }

  // 코드 31-4 E 소비자(consumer) 매개변수에 와일드카드 타입 적용 (183쪽)
  public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
      dst.add(pop());
  }
```

### (2) Comparator와 Comparable은 소비자
- Comparable을 직접 구현하지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원하려면 와일드카드가 필요하다.
  - ScheduledFuture는 Comparable을 직접 구현하지 않았지만, 그 상위 타입`(Delayed)`이 구현하고 있다.

```java
// T 생산자 매개변수에 와일드카드 타입 적용 (184쪽)
public class Chooser<T> {
  private final List<T> choiceList;
  private final Random rnd = new Random();

  // 코드 31-5 T 생산자 매개변수에 와일드카드 타입 적용 (184쪽)
  public Chooser(Collection<? extends T> choices) {
    choiceList = new ArrayList<>(choices);
  }

  public T choose() {
    return choiceList.get(rnd.nextInt(choiceList.size()));
  }

  public static void main(String[] args) {
    List<Integer> intList = List.of(1, 2, 3, 4, 5, 6);
    Chooser<Number> chooser = new Chooser<>(intList);
    for (int i = 0; i < 10; i++) {
      Number choice = chooser.choose();
      System.out.println(choice);
    }
  }
}
```
```java
// 코드 30-2의 제네릭 union 메서드에 와일드카드 타입을 적용해 유연성을 높였다. (185-186쪽)
public class Union {
  public static <E> Set<E> union(Set<? extends E> s1,
                                 Set<? extends E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
  }

  // 향상된 유연성을 확인해주는 맛보기 프로그램 (185쪽)
  public static void main(String[] args) {
    Set<Integer> integers = new HashSet<>();
    integers.add(1);
    integers.add(3);
    integers.add(5);

    Set<Double> doubles = new HashSet<>();
    doubles.add(2.0);
    doubles.add(4.0);
    doubles.add(6.0);

    Set<Number> numbers = union(integers, doubles);

    //      // 코드 31-6 자바 7까지는 명시적 타입 인수를 사용해야 한다. (186쪽)
    //      Set<Number> numbers = Union.<Number>union(integers, doubles);

    System.out.println(numbers);
  }
}
```

### Comparable은 안에 있는 값을 꺼내서 비교하기 때문에 super를 사용하면 더 유연해짐
```java
public class Box<T extends Comparable<T>> implements Comparable<Box<T>> {

  protected T value;

  public Box(T value) {
    this.value = value;
  }

  public void change(T value) {
    this.value = value;
  }

  @SuppressWarnings("unchecked")
  @Override
  public int compareTo(Box anotherBox) {
    return this.value.compareTo((T) anotherBox.value);
  }

  @Override
  public String toString() {
    return "Box{" +
        "value=" + value +
        '}';
  }
}
```
```java
public class IntegerBox extends Box<Integer> {

  private final String message;

  public IntegerBox(int value, String message) {
    super(value);
    this.message = message;
  }

  @Override
  public String toString() {
    return "IntegerBox{" +
        "message='" + message + '\'' +
        ", value=" + value +
        '}';
  }
}

```

```java
public interface Comparable<T> {
  public int compareTo(T o);
}

// 와일드카드 타입을 사용해 재귀적 타입 한정을 다듬었다. (187쪽)
public class RecursiveTypeBound {
  
  public static <E extends Comparable<? super E>> E max(List<? extends E> list) {
    if (list.isEmpty())
      throw new IllegalArgumentException("빈 리스트");

    E result = null;
    for (E e : list)
      if (result == null || e.compareTo(result) > 0)
        result = e;

    return result;
  }

  public static void main(String[] args) {
    List<IntegerBox> list = new ArrayList<>();
    list.add(new IntegerBox(10, "jaehoon"));
    list.add(new IntegerBox(2, "effective"));

    // IntegerBox{message='jaehoon', value=10}
    System.out.println(max(list));
  }
}
```

### (3) 와일드카드 활용 팁
- 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라.
  - 한정적 타입이라면 한정적 와일드카드로
  - 비한정적 타입이라면 비한정적 와일드카드로
- 주의할 점
  - 비한정적 와일드카드`(?)` 로 정의한 타입에는 Null을 제외한 아무것도 넣을 수 없다.
  - 무엇인가를 넣어야 한다면, 그냥 한정적타입을 사용하자.
  - 꺼내는 용도라면, 비한정적타입은 유용할 수 있다. (PECS 원칙으로 사용)

### 한정적 타입은 무슨 타입인지 안다는 의미
```java
public class Swap {

  public static <E> void swap(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
  }

  public static void main(String[] args) {
    // 첫 번째와 마지막 인수를 스왑한 후 결과 리스트를 출력한다.
    List<String> argList = Arrays.asList(args);
    swap(argList, 0, argList.size() - 1);
    System.out.println(argList);
  }
}
```

### 비한정적 타입은 타입을 모른다라는 의미
- 넣는것은 불가능, 꺼내는 것은 가능
```java
public class Swap {

  public static void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i))); // set을 할 수 없음 (? 가 들어와야함, null만 들어올 수 있음)
  }

  public static void main(String[] args) {
    // 첫 번째와 마지막 인수를 스왑한 후 결과 리스트를 출력한다.
    List<String> argList = Arrays.asList(args);
    swap(argList, 0, argList.size() - 1);
    System.out.println(argList);
  }
}
```

###
## 2. 추가 지식

### (1) [타입 추론](https://docs.oracle.com/javase/tutorial/java/generics/genTypeInference.html)
- 타입을 추론하는 컴파일러의 기능
- 모든 인자의 가장 구체적인 공통 타입 (most specific type)
- 제네릭 메서드와 타입 추론 : 메서드 매개변수를 기반으로 타입 매개변수를 추론할 수 있다.
- 제네릭 클래스 생성자를 호출할 때 다이아몬드 연산자 `<>` 를 사용하면 타입을 추론한다.
- 자바 컴파일러는 "타겟 타입"을 기반으로 호출하는 제네릭 메서드의 타입 매개변수를 추론한다.
  - 자바 8에서 "타겟 타입"이 "메서드의 인자"까지 확장되면서 이전에 비해 타입 추론이 강화되었다.

```java
public class BoxExample {

  private static <U> void addBox(U u, List<Box<U>> boxes) {
    Box<U> box = new Box<>();
    box.set(u);
    boxes.add(box);
  }

  private static <U> void outputBoxes(List<Box<U>> boxes) {
    int counter = 0;
    for (Box<U> box : boxes) {
      U boxContents = box.get();
      System.out.println("Box #" + counter + " contains [" +
                             boxContents.toString() + "]");
      counter++;
    }
  }

  // 매세드의 인자까지 추론 List<String>
  private static void processStringList(List<String> stringList) {
  }
  
  public static void main(String[] args) {
    ArrayList<Box<Integer>> listOfIntegerBoxes = new ArrayList<>();
    BoxExample.addBox(10, listOfIntegerBoxes);
    BoxExample.addBox(20, listOfIntegerBoxes);
    BoxExample.addBox(30, listOfIntegerBoxes);
    BoxExample.outputBoxes(listOfIntegerBoxes);

    // 타겟 타입의 확장
    List<String> stringlist = Collections.emptyList();
    List<Integer> integerlist = Collections.emptyList();
    BoxExample.processStringList(Collections.emptyList());
    BoxExample.processStringList(Collections.<String>emptyList()); // 자바 8 이전에는 이렇게 사용
  }
}

```