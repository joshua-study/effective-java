# 아이템 13. clone 재정의는 주의해서 진행하라

Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스 인터페이스이지만,</br>
clone 메서드가 선언된 곳은 Cloneable가 아닌 Object이며, protected이다. 그래서 Cloneabled을 구현하는 것만으로 외부 객체에서 clone 메서드를 호출 할 수 없다.

## clone 메서드
객체의 모든 필드를 복사하여 새로운 객체에 넣어 반환하는 동작을 수행한다. 즉, 필드의 값이 같은 객체를 새로 만드는 것이다.

## Cloneable 인터페이스가 하는 일
Object의 protected 메서드인 clone의 동작 방식을 결정한다.
Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며,
그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던진다.

## clone 메서드 일반 규약
(매우 허술하다.)
1. x.clone() != x
이 식은 참이다.
2. x.clone().getClass() == x.getClass()
이 식도 참이다. 하지만 반드시 만족해야 하는 것은 아니다.
3. x.clone().getClass().equals(x)
이 식은 참이지만 필수는 아니다. 관례상, 이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야 한다.
4. x.clone().getClass() == x.getClass()
만약, super.clone() 을 호출해 얻은 객체를 clone()이 반환한다면 이 식은 참이다. 또한, 관례상 반환된 객체와 원본객체는 독립적이어야 한다. 이를 만족하려면 super.clone()으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다

super.clone을 연쇄적으로 호출하도록 구현해두면 clone이 처음 호출된 하위 클래스의 객체가 만들어진다.

```java
    @Override
    public PhoneNumber clone(){
        try {
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(); //일어날 수 없는 일이다.
        }
    }
```
1) 재정의한 clone()은 다른 패키지에서 접근할 수 있게 접근 제어자를 protected에서 public 으로 구현했다.
2) PhoneNumber의 clone 메서드는 PhoneNumber를 반호나하게 했는데 공변 반환 타입으로 인해
재정의한 메서드의 반환 타입은 상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있다.
    * 공변 반환 타입 : 메서드를 재정의 할 때 재정의된 메서드의 반환 유형이ㅣ 상위 클래스의 메서드가 반환하는 하위 유형이 될 수 있음을 말하는 것.
3) 호출을 try-catch으로 감싼 이유는 Object의 clone 메서드가 검사 예외인 CloneNotSupportedException을 던지도록 선언되었기 때문이다. 

## 가변 객체를 참조하는 clone 메서드 재정의
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack(final Object[] elements) {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object obj) {
        ensureCapacity();
        elements[size++] = obj;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }

    // 원소를 위한 공간을 적어도 하나 이상 확보한다.
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

    @Override
    protected Stack clone() {
        try {
            return (Stack) super.clone();
        } catch (CloneNotSupportedException cloneNotSupportedException) {
            throw new AssertionError();
        }
    }
}
```
clone 메서드가 단순히 super.clone 결과를 그대로 반환한다면? 반환된 Stack 인스턴스의 size는 올바른 값을 갖겠지만, elements는 원본 Stack 인스턴스와 똑같은 배열을 참조할 것이다.</br>
원본이나 복제본 중 하나를 수정하게 된다면 다른 하나도 수정되어 불변식을 해치게 되고 프로그램은 NullPointerException을 던지게 된다.

**Stack 클래스의 하나뿐인 생성자를 호출하면 이런 문제는 발생하지 않는다. 생성자와 사실상 같은 효과를 내는 clone은 원본객체에 아무런 해를 끼치지 않으며 복제된 객체의 불변식을 보장해야 한다.**

## 가변 상태를 참조하는 클래스용 clone 메서드 
```java
@Override
protected Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException cloneNotSupportedException) {
            throw new AssertionError();
        }
}
```

elements.clone() 을 굳이 Object[] 로 형변환 할필요는 없다. 
clone은 런타임 타입과 컴파일타임 타입 모두가 원본 배열과 똑같은 배열을 반환한다. 
배열을 복제할때는 clone 사용이 권장되는데 배열 복제는 clone 기능이 제대로 사용되는 유일한 예라 할 수 있다.

하지만, elements 필드가 final 이었다면 앞서의 방식은 사용할 수 없다. 
이는 Cloneable 아키텍처는 "가변 객체를 참조하는 필드는 final로 선언하라" 는 일반 용법과 충돌한다. 
따라서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final을 제거해야할 수도 있다.

## 해시테이블용 clone 메서드
```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;
    
    private static class Entry {
        final Object key;
        Object value;
        Entry next;
        
        Entry(Object key, Object value, Entry next){
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }
    ...//나머지 코드 생략
}
```
**잘못된 clone 메서드 - 가변 상태를 공유한다!**
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

복제본은 자신만의 버킷 배열을 갖지만, 이 배열은 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 가능성이 있다.
이를 해결하려면 각 버킷을 구성하는 연결 리스트를 복사해야 한다.

## 복잡한 가변 상태를 갖는 클래스용 재귀적 clone 메서드
```java
public class HashTable implements Cloneable{
    private Entry[] buckets = ...;

    private static class Entry{
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, final Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
        //이 엔트리가 가리키는 연결 리스트를 재귀적으로 복사
        Entry deepCopy(){
            return new Entry(key,value, 
                    next == null ? null : next.deepCopy());
        }
    }

    @Override
    public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];

            for (int i =0; i < buckets.length; i++){
                if(buckets[i] != null){
                    result.buckets[i] = buckets[i].deepCopy();
                }
            }
            return result;
        }catch (CloneNotSupportedException e){
            throw new AssertionError();
        }
    }
}
```
HashTable.Entry 는 깊은복사(deep copy) 를 지원하도록 deepCopy에서 값만 복사해 만들어주고있다. 
HashTable의 clone은 적절한 크기의 새로운 버킷 배열을 할당한 다음 원래의 버킷 배열을 순회하며 비어있지 않은 각 버킷에 대해 깊은 복사를 수행한다.

하지만 연결리스트를 복제하는 방법으로 재귀적 호출을 선택하는것이 좋은 방법은 아니다. 
재귀 호출 떄문에 리스트의 원소 수만큼 스텍 프레임을 소비하여 리스트가 길면 스택 오버플로우를 일으킬 수 있다.

이 문제를 해결하려면 deepCopy를 재귀 호출대신 반복자를 사용하여 순회하는 방향으로 수정해야한다.

## 엔트리 자신이 가리키는 연결 리스트를 반복적으로 복사한다.
```java
Entry deepCopy() {
    Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p = p.next) {
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    }
    return result;
}
```

**가변 객체를 복제하는 방법**
1. super.clone()을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정한다.
2. 원본 객체의 상태를 다시 생성하는 고수준 메서드들을 호출한다.

고수준 API를 활용해 복제하면 우아한 코드를 얻게 되지만, 저수준에서는 느리다.

또한, clone 메서드는 생성자와 마찬가지로 재정의될 수 있는 메서드를 호출하지 않아야 한다.
만약 하위 클래스에서 재정의한 메서드를 호출하면, 하위 클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃게 되어 원본과 복제본의 상태가 달라질 가능성이 크다.
따라서, put(key, value)는 final이거나 private이어야 한다.

## clone 메서드 주의사항
1. 재정의한 clone메서드는 throws 절을 없애야 한다.
   사용의 편의성 때문.(아이템 71)
2. 상속용 클래스에서는 Cloneable을 구현해서는 안된다.
clone 메서드를 재정의해 CloneNotSupportedException()을 던지게하자.
3. Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone 메서드 역시 적절히 동기화해줘야 한다. (아이템 78)
   
Cloneable로 구현하는 모든 클래스는 clone을 재정의해줘야 한다. 접근제한자는 public으로, 반환 타입은 클래스 자신으로 변경해야 한다.</br>
이 메서드는 가장 먼저 super.clone을 호출한 후 필요한 필드를 전부 적절히 수정한다.</br>
'깊은 구조'에 숨어있는 모든 가변 객체를 복사하고, 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 한다.

## 복사 생성자와 복사 팩터리
복사 생성자란, 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다.
```java
public Yum(Yum yum){
	...
}
```
복사 팩터리란, 복사 생성자를 정적 팩터리 형식으로 정의한 것이다.
```java
public static Yum newInstance(Yum yum){
	...
}
```

1. Cloneable/clone 방식보다 나은 면이 많다.</br>
   final 필드 용법과 충돌하지 않으며, 불필요한 검사예외(Exception) 처리를 하지 않아도 되고 
   형변환도 필요하지 않으며 객체 생성 메커니즘(생성자를 쓰지 않는 방식)을 사용하지도 않는다.
2. 해당 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있어 이들을 이용하면 복제본 타입을 선택하는데 있어 유연성이 향상될 수 있다.


## 정리
새로운 인터페이스를 만들 때는 절대 Cloneable를 확장해서는 안 되며, 새로운 클래스도 이를 구현해서는 안된다. </br>
final 클래스도 성능 최적화 관점에서 검토한 후 별다른 문제가 없을 때만 Cloneable를 구현하는 것을 허용해야 한다. (아이템 67)

기본 원칙은 '복제 기능'은 생성자와 팩터리를 이용하는 게 '최고'이다.
단. 배열만은 clone 메서드 방식이 가장 깔끔한 예외라는 것이다.