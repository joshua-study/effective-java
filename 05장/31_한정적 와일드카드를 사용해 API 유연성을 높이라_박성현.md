# 31. 한정적 와일드카드를 사용해 API 유연성을 높이라.

---

## PECS - Producer-Extends, Consumer-Super
- Get and Put Principle
- 와일드카드(?)를 사용할 때 생길 수 있는 혼란을 줄이기 위한 규칙
  - 생산자(producer)인 경우 상한 경계 와일드 카드 사용 `<? extends E>`
  - 소비자(consumer)인 경우 하한 경계 와일드 카드 사용 `<? super E>`

### (1) Producer-Extends
- 데이터 생산, 저장
```java
// 와일드카드 타입을 사용하지 않은 pushAll 메서드
// 컴파일은 가능하지만 main 실행시 오류.
// Integer는 Number의 하위 타입이지만
// 제네릭 매개변수화 타입은 불공변이기 때문
public void pushAll(Iterable<E> src) {
   for (E e : src)
       push(e);
}

// E 생산자(producer) 매개변수에 와일드카드 타입 적용
// E의 하위타입의 Iterable -> Number의 하위타입(Integer)의 Iterable
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}

public static void main(String[] args) {
    Stack<Number> numberStack = new Stack<>();
    // 데이터를 저장하는 생산자 객체
    Iterable<Integer> integers = Arrays.asList(3, 1, 4, 1, 5, 9);
    numberStack.pushAll(integers);
    // ...
}
```

### (2) Consumer-Super
- 데이터 조회
```java
// 와일드카드 타입을 사용하지 않은 popAll 메서드
// Producer-Extends와 같은 오류 발생.
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}

// E 소비자(consumer) 매개변수에 와일드카드 타입 적용
// E의 상위 타입의 Collection -> Number의 상위 타입(Object)의 Collection
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}

public static void main(String[]args){
    // ...
        
    // 데이터를 조회하는, 꺼내오는 소비자 객체
    Collection<Object> objects = new ArrayList<>();
    numberStack.popAll(objects);
}
```

<blockquote>
유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라. <br>
한편, 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 사용하지 말아야 한다.
</blockquote>

---

## 와일드 카드

비한정적 와일드 카드, 불특정 타입
- `<?>`로 선언한 매개변수를 사용하여 데이터를 조회할 수는 있지만 데이터를 저장하지는 못한다.
- `<?>`은 null 외에는 어떤 값도 넣을 수 없기 때문
- 와일드카드 타입의 실제 타입을 알려주는 메서드를 private 도우미 메서드로 작성하여 활용하는 방법으로 해결 가능.
```java
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

// 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
public static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

---

## 타입 추론
- 타입을 추론하는 컴파일러 기능
- 자바 7까지는 타입 추론 범위가 좁아 문맥에 맞는 반환 타입을 명시해야 했다.
  - 컴파일러가 올바튼 타입을 추론하지 못할 때면 명시적 타입 인수(`explicit type argument` 또는 `type witness`)를 사용 
- 제네릭 클래스 생성자를 호출할 때 다이아몬드 연산자 `<>`를 사용하여 타입 추론
- 자바 컴파일러는 `타겟 타입`을 기반으로 호출하는 제네릭 메서드의 타입 매개변수를 추론
  - 자바 8에서 `타겟 타입`이 `메서드 인자`까지 확장되면서 이전에 비해 타입 추론이 강화
```java
// 자바 7까지는 명시적 타입 인수를 사용해야 한다.
Set<Number> numbers = Union.<Number>union(integers, doubles);
// 자바 8 이후 타입 추론 기능 강화로 생략 가능
Set<Number> numbers = Union.union(integers, doubles);
```

---

## 핵심 정리
```
와일드 카드 타입을 적용하면 API가 훨씬 유연해진다.
많이 쓰일 라이브러리를 작성한다면 반드시 와일드 카드 타입을 적절히 사용해줘야 한다.

PECS 공식을 기억하자.
생산자는 extends를, 소비자는 super를 사용한다.
Comparable과 Comparator는 모두 소비자라는 사실을 잊지 말자.
```