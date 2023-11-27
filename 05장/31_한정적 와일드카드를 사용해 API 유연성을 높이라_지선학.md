## 한정적 와일드카드를 사용해 API 유연성을 높이라

- List<Object>에는 어떤 객체든 넣을 수 있지만, List<String>에는 문자열만 넣을 수 있다. 
- List<String>은 List<Object>가 하는 일을 제대로 수행하지 못하니 하위 타입이 될 수 없다(리스코드 치환 원칙에 어긋난다.)  

### 와일드카드 타입을 사용하지 않은 pushAll 메서드 - 결함이 있다!
```java
class Sample(){
    public void pushAll(Iterable<E> src){
        for(E e: src)
            push(e);
    }
}
```
- Integer는 Number의 하위 타입이나 잘 동작한다. 아니, 논리적으로 잘 동작해야 할 거 같다. 
```java
class Sample(){
    public static void main(String[] args) {
        Stack<Number> numberStack = new Stack<>();
        Iterable<Integer> integers =  ... ;
        numberStack.pushAll(integers);
    }
}
```
- 하지만 실제로는 다음의 오류 메시지가 뜬다. 매개변수화 타입이 불공변이기 때문이다. 
```java
StackTest.java:7: error: incompatible types: Iterable<Integer>
cannot be converted to Iterable<Number>
    numberStack.pushAll(integers);
```
- 자바는 이런 상황에 대처할 수 있는 한정적 와일드카드 타입이라는 특별한 매개변수화 타입을 지원한다. 
- pushAll의 입력 매개변수 타입은 'E의 Iterable'이 아니라 'E의 하위 타입의 Iterable'이어야 하며, 와이들 카드 타입 Iterable<? extends E>가 정확히 이런 뜻이다. 

### E 생산자(producer) 매개변수에 와일드카드 타입 적용
```java
class Sample(){
    public void pushAll(Iterable<? extends E> src){
        for(E e: src)
            push(e);
    }
}
```
- 이번 수정으로 Stack은 물론 이를 사용하는 클라이언트 코드도 말끔히 컴파일된다. 
- 모든 것이 타입 안전하다는 뜻이다.  

### 와일드카드 타입을 사용하지 않은 popAll 메서드 - 결함이 있다!
```java
class Sample(){
    public void popAll(Collection<E> dst){
        while (!isEmpty()) {
            dst.add(pop());
        }
    }
}
```
- Stack<Number>의 원소를 Object용 컬렉션으로 옮기려 한다고 해보자.
```java
Stack<Number> numberStack = new Stack<>();
Collection<Object> objects = ...;
numberStack.popAll(objects);
```
- popAll 코드와 함께 컴파일하면 "Collection<Object>는 Collection<Number>의 하위 타입이 아니다"라는 코드 31-1의 pushAll을 사용했을 때와 비슷한 오류가 발생한다. 
- 이번에도 와일드카드 타입으로 해결 할 수 있다. 
- popAll의 입력 매개변수의 타입이 'E의 Collection'이 아니라 'E의 상위 타입이 Collection'이어야 한다.(모든 타입은 자기 자산의 상위 타입이다.)
- 와일드카드 타입을 사용한 Collection<? super E>가 정확히 이런 의미다.  

### E 소비자(consumer) 매개변수에 와일드카드 타입 적용
```java
class Sample(){
    public void popAll(Collection<? super E> dst){
        while (!isEmpty()) {
            dst.add(pop());
        }
    }
}
```
- Stack과 클라이언트 코드 모두 말끔히 컴파일된다.
- 유연성을 극대화하려면 원소의 생산자나 소비자용 입력매개변수에 와일드카드 타입을 사용하라.
- 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을 게 없다. 

### 펙시(PECS) : producer-extends, consumer-super  

- 매개변수화 타입 T가 생산자라면 <? extends T>를 사용하고, 소비자라면 <? super T>를 사용하라.

```java
public Chooser(Collection<T> choices)
```
- 이 생성자로 넘겨진 choices 컬렉션은 T 타입의 값을 생성하기만 하니, T를 확장하는 와일드카드 타입을 사용해 선언해야한다.

### T 생성자 매개변수에 와일드카드 타입 적용
```java
class Sample(){
    public Chooser(Collection<? extends T> choices)
}
```
- 수정 전 생성자로는 컴파일조차 되지 않겠지만, 한정적 와일드카드 타입으로 선언한 수정 후 생성자에서는 문제가 사라진다. 

## union 메서드 
```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2)
```
### PECS 공식에 따라 다음처럼 선언
```java
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
```
- 반환 타입은 여전히 Set<E>임에 주목하자.
- 반환 타입에는 한정적 와일드카드 타입을 사용하면 안 된다. 
- 유연성을 높여주기는커녕 클라이언트 코드에서도 와일드카드 타입을 써야 하기 때문이다.

```java
Set<Integer> integers = Set.of(1, 3, 5);
Set<Double> doubles = Set.of(2.0, 4.0, 6.0);
Set<Number> numbers = union(integers, doubles);
```
- 클래스 사용자가 와일드카드 타입을 신경 써야 한다면 그 API에 무슨 문제가 있을 가능성이 크다. 
- 앞의 코드는 자바 8부터 제대로 컴파일 된다. 
- 컴파일러가 올바른 타입을 추론하지 못할 때면 언제든 명시적 타입 인수를 사용해서 타입을 알려주면 된다. 

### 자바 7까지는 명시적 타입 인수를 사용해야 한다. 
```java
Set<Number> numbers = Union.<Number>union(integers, doubles);
```









