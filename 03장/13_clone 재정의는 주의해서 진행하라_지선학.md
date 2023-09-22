## clone 재정의는 주의해서 진행하라

믹스인 인터페이스(mixin interface)  
객체지향언어에서 다른 클래스에서 '사용'할 목적으로 만들어진 클래스이다.  
- '포함'으로 설명된다.
- '상속'과 주로 비교되는 개념이다. (is-a vs has-a)  

Composition 혹은 Aggregation 이라고 불리기도 한다.  
코드 재사용성을 높여주고, 상속의 단점을 해결할 수도 있다.  
자바코드에서는 다중 상속의 제한이 없는 인터페이스로 구현하기 용이하다.  
대상 타입의 주된 기능에 선택적 기능을 '혼합(mixed in)'한다고 해서 믹스인이라 불린다.

```java
class Sample(){

    public interface Singer {
        AudioClip sing(Song s);
    }

    public interface Songwriter {
        Song compose(int chartPosition);
    }

    public interface SingerSongwriter extends Singer, Songwriter {
        AudioClip strum();
        void actSensitive();
    }
    
}
```
Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스이지만, 아쉽게도 의도한 목족을 제대로 이루지 못했음  
clone 메서드가 선언된 곳이 Cloneable이 아닌 Object이고, 그마저도 protected라는 데 있음  
리플렉션을 사용하면 가능하지만, 100% 성공하는 것도 아니다.  

Cloneable 인터페이스는 놀랍게도 Object의 protected 메서드인 clone의 동작 방식을 결정함
Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며,  
그렇지 않은 클래스의 인스턴스에 호출하면 CloneNotSupportedException을 던짐  
  
실무에서는 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제데로 이뤄지리라 기대함

clone 메서드의 일반 규약은 허술하다.

```shell
x.clone() != x  

x.clone().getClass() == x.getClass()

x.clone().equals(x)
이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야한다.  
이 클래스와 (Object를 제외한) 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.

x.clone().getClass == x.getClass()
반횐된 객체와 원본 객체는 독립적이어야 한다.  
이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.
```

제대로 동작하는 clone 메서드를 가진 상위 클래스를 상송해 Cloneable을 구현하고 싶자고하자!
super.clone을 호출한다. 그렇게 얻은 객체는 원본의 완벽한 복제본일 것임

```java
class example(){
    
    @Override public PhoneNumber clone(){
        try{
            return (PhomeNumber) super.clone();
        }catch (CloneNotSupportedException e){
            throw new AssertionError();
        }
    }
    
}
```
자바가 공변 반환 타이핑을 지원하닌 이렇게 하는 것이 가능하고 권정하는 방식이다.
*공변 반환 타이핑 : 부모 클래스의 메소드를 오버라이딩하는 경우, 부모 클래스의 반환 타입은 자식 클래스의 타입으로 변경이 가능하다.
```java
public class Main {
    public static void main(String[] args) {
        Parent parent = new Parent();
        Child child = new Child();
        Parent pc = new Child();

        System.out.println(parent.createNewOne().getClass());
        System.out.println(child.createNewOne().getClass());
        System.out.println(pc.createNewOne().getClass());
    }
}

class Parent {
    protected Parent createNewOne() {
        return new Parent();
    }
}

class Child extends Parent {
    // 부모 클래스로부터 재정의하였으나 반환형을 자식 클래스로 변경할 수 있다.
    @Override public Child createNewOne() {
        return new Child();
    }
}
```

```java
public class Stack {
    private Object[] elements; 
    private int size =0; 
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack(){
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e){
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop(){
        if(size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }
    
    private void ensureCapacity(){
        if(elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
    
}
```
위 클래스에서 반환된 Stack 인스턴서의 size필드는 올바른 값을 갖겠지만,  
elements 필드는 원본 Stack 인스턴스와 똑같은 배열ㅇ르 참조할 것임  
원본이나 복제본 중 하나를 수정하면 다른 하난도 수정되어 불변식을 해친다는 이야기임  

elements 배열의 clone을 재귀적으로 호출해서 이를 해결해줌
```java
class Sample(){
    @Override public Stack clone(){
        try{
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e){
            throw new AssertionError();
        }
    }
}
```
위 코드에서 elements.clone의 결과를 Object[]로 형변활할 필요는 없음  
배열의 clone은 런타임 타입과 컴파일타임 타입 모두가 원본 배열과 똑같은 배열을 반환함
배열은 clone 기능을 제대로 사용하는 유일한 예임

Cloneable 아키텍처는 '가변 객체를 참조하는 필드는 final로 선언하라'는 일반 용법과 충돌함

#### 하지만, clone을 재귀적으로 호출하는 것만으로는 충분하지 않을 때도 있음

해시테이블용 clone 메서드
```java
public class HashTable implements Cloneable{
    private Entry[] buckets = ...;
    
    private static class Entry{
        final Object key; 
        Object value; 
        Entry next;
        
        Entry(Object key, Object value, Entry next){
            this.key = key; 
            this.value = value;
            this.next = next;
        }
    }
}
```

### 잘못된 clone 메서드 - 가변 상태를 공유한다.
```java
   @Override
    protected HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];

            return result;
        }catch (CloneNotSupportedException cloneNotSupportedException){
            throw new AssertionError();
        }
    }
```
복제본은 자신만의 버킷 배열을 갖지만, 이 배열은 원본과 같은 연결 리스트를 참조하여  
원본과 복제본 모두 예기치 않게 동작할 가능성이 생김  
이를 해결하려면 각 버킷을 구성하는 연결 리스트를 복사해야함

### 복잡한 가변 상태를 갖는 클래스용 재귀적 clone 메서드
```java
public class HashTable implements Cloneable{
    private Entry[] buckets = new Entry[50];
    private int size = 0;

    public void put(Entry entry){
        buckets[size++] = entry;
    }

    public void printAll(){
        for (int i=0;i<size;i++){
            System.out.println(buckets[i].toString());
        }
    }

    static class Entry{
        final Object key;
        Object value;
        Entry next;

        public Entry(final Object key, final Object value, final Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        Entry deepCopy(){
            return new Entry(key,value, next==null ? null : next.deepCopy());
        }
    }

    @Override
    protected HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];

            for (int i =0; i < buckets.length; i++){
                if(buckets[i] != null){
                    result.buckets[i] = buckets[i].deepCopy();
                }
            }

            return result;
        }catch (CloneNotSupportedException cloneNotSupportedException){
            throw new AssertionError();
        }
    }
}
```
private 클래스인 HashTable.Entry는 깊은복사(deep copy)를 지원하도록 보강됨  
  
연결 리스트를 복제하는 방법으로는 그다지 좋지 않음
재귀 호출 떄문에 리스트의 원소 수만큼 스택 프레임을 소비하여, 리스트가 길면 스택 오버플로를 일으킬 위험이 있음  
이 문제를 피하려면 deepCopy를 재귀 호출 대신 반복자를 써서 순회하는 방향으로 수정해야함

```java
Entry deepCopy() {
            Entry result = new Entry(key, value, next);
            for (Entry p = result; p.next != null; p = p.next) {
                p.next = new Entry(p.next.key, p.next.value, p.next.next);
            }

            return result;
}
```

HashTable 예에서라면, buckets필드를 새로운 버킷 배열로 초기화한 다음 원본 테이블에 담긴 
모든 키-값 쌍 각각에 대해 복제본 테이블 put(key, value) 메서드를 호출해 둘의 내용이 똑같게 해주면 됨  
고수준 API를 활용해 복제하면 보통은 간단하고 제번 우아한 코드를 얻게됨
하지만, 저수준에서 바로 처리할 때보다는 느림

clone이 하위 클래스에서 재정의한 메서드를 호출하면, 하위클래스는 복제 과정에서 자신의 상태를 교정할 기회를  
잃게 되어 원본과 복제본의 상태가 달라질 가능성이 큼  
  
put(key, value) 메서드는 final이거나 private이어야 함

방법 -> clone을 동작하지 않게 구현해놓고 하위 클래스에서 재정의하지 못하게 할 수 있음 
```java
@Override
protected final Object clone() throws CloneNotSupportedException{
    throw new CloneNotSupportedException();
        }
```

방법 -> 복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공  

### 복사 생성자
```java
public Yum(Yum yum){
	...
}
```

### 복사 팩터리
```java
public static Yum newInstance(Yum yum){
	...
}
```
복사 생성자와 그 변형인 복사 팩터리는 Cloneable/clone 방식보다 나은 면이 많음  
객체 생성 메커니즘을 사용하지 않으며, 엉성하게 문서되된 규약에 기대지 않음, 정성적인 final필드 용법과 충돌하지 않으며,  
불필요한 검사 예외를 던지지 않음, 형변환도 필요치 않음  
  
해당 클래스가 구현한 '인터페이스'타입의 인스턴스를 인수로 받을 수 있음

## 정리
- 인터페이스르를 만들때는 절대 Cloneable을 활정해서는 안 되며, 새로운 클래스도 이를 구현해서는 안됨
- final 클래스라면 Cloneable을 성능 최적화 관점에서 허용하자
- Cloneable/clone 방식보다 생성자와 팩터리를 이용하는게 최고임
- 단, 배열만은 clone메서드 방식이 가장 깔끔함



