# 13. clone 재정의는 주의해서 진행하라.

---

##  clone 규약
- x.clone() != x → 반드시 true
  - 객체가 참조하는 레퍼런스가 달라야한다.
  - clone으로 만들어진 인스턴스는 생성자를 통해 만들어지지 않는다.
  - clone 메서드를 사용해서 만들어짐
- x.clone().getClass() == x.getClass()  → 반드시 true
- x.clone().equals(x) → true일 수도 아닐수도 

### 1. 불변 객체
Clonenable 인터페이스를 구현하고 
clone 메서드를 재정의한다. 이때 <u>super.clone()을 사용</u>해야 한다.
- 만약 상위클래스에서 super.clone()을 호출하지 않고
- 하위 클래스에서 super.clone()을 호출한다면 잘못된 클래스의 객체가 만들어져
- 하위 클래스의 clone 메서드가 제대로 동작하지 않는다.
```java
public class Item implements Cloneable{
    private String name;
    
    @Override
    public Item clone() {
        // 오류 코드
        // 자기 자신을 리턴 
        Item item = new Item();
        item.name = this.name;
        return item;
        
        // 정상 작동 코드
        Item result = null;
        try {
            result = (Item) super.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
public class SubItem extends Item implements Cloneable {
    private String name;
    
    // 구체적인 타입은 상위 타입으로 변할 수 있지만
    // 상위 타입은 구체적 타입으로 변환되지 못함
    @Override
    public SubItem clone() {
        return (SubItem) super.clone();
    }

    public static void main(String[] args) {
        SubItem item = new SubItem();
        SubItem clone = item.clone();

        System.out.println(clone != null);
        System.out.println(clone.getClass() == item.getClass());
    }
}
```
---

### 2. 가변 객체의 clone 구현 방법
- 접근 제한자는 public, 반환 타입은 자신의 클래스로 변경.
- super.clone()을 호출한 뒤 필요한 필드를 적절히 수정한다.

```java
public class Stack implements Cloneable {
    private Object[] elements;
    
    @Override
    public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            // 아래 코드 주석 시 stack, Object를 clone한 result 가 동일한 elements를 참조하게 됨
            // 아래 코드를 사용한다 해도 역시 문제.
            // 서로 다른 인스턴스를 갖고 있지만 복제한 객체는 배열의 형태만 갖췄을 뿐 배열 안의 인스턴스는 같은 값을 참조 -> 얕은 복제
            // stack의 값을 변경하면 복제한 result의 값도 달라짐. -> 깊은 복제 필요
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
    
    public static void main(String[] args) {
        Object[] values = new Object[2];
        values[0] = "123-4567-8900";
        values[1] = "010-4289-7891";

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
    }
}
```
- 얕은 복제 테스트
```java
System.out.println("stack의 해시코드");
System.out.println(stack.elements[0].hashCode()); // -356295471
System.out.println(stack.elements[1].hashCode()); // 1802683081

System.out.println("copy의 해시코드");
System.out.println(copy.elements[0].hashCode()); // -356295471 
System.out.println(copy.elements[1].hashCode()); // 1802683081

System.out.println("배열 인스턴스");
System.out.println(stack == copy); // false

System.out.println("배열 안 인스턴스");
System.out.println(stack.elements[0] == copy.elements[0]); // true
System.out.println(stack.elements[1] == copy.elements[1]); // true
```

- 깊은 복제
```java
public class HashTable implements Cloneable {
    private Entry[] buckets = new Entry[10];

    private static class Entry {
        final Object key;
        Oject value;
        Entry next;
        
        // 생성자 생략..
        
        // 재귀 : 자신을 정의할 때 자기 자신을 재참조하는 방법
        public Entry deepCopy() {
            return new Entry(key, value, next == null ? null : next.deepCopy);
        }

        // 반복자 사용
        public Entry deepCopy() {
            Entry result = new Entry(key, value, next);
            for (Entry p = result; p.next != null; p = p.next) {
                p.next = new Entry(p.next.key, p.next.value, p.next.next);
            }
            return result;
        }
    }

    @Override
    public HashTable clone() {
        HashTable result = null;
        try {
            HashTable result = (HashTable) super.clone();
            // 새로 배열을 만든 후 인덱스마다 깊은 복제 작업 진행
            result.buckets = new Entry[buckets.length];
            for (int i = 0; i < this.buckets.length; i++) {
                if (buckets[i] != null) {
                    // 연결리스트가 길고 deepCopy를 재귀적으로 호출한다면 
                    // StackOverflow 발생 가능성 ↑
                    result.buckets[i] = this.buckets[i].deepCopy();
                }
            }
            return result;
        } catch (CloneNotSupportedException cloneNotSupportedException) {
            throw new AssertionError();
        }
    }

    public static void main(String[] args) {
        HashTable hashTable = new HashTable();
        Entry entry = new Entry(new Object(), new Object(), null);
        hashTable.buckets[0] = entry;
        HashTable clone = hashTable.clone();
        System.out.println(hashTable.buckets[0] == entry);
        System.out.println(hashTable.buckets[0] == clone.buckets[0]);
    }
}
```

- 상속용 클래스에 Cloneable을 구현하지 않도록 하자.

- 고수준의 api를 활용.
  - 기존의 배열을 새로운 배열로 초기화하고
  - 원본 배열의 값 각각을 put,get 등을 사용해 복제 배열에 넣어주는 방법도 있다.
- clone에서 하위클래스에서 재정의될 수 있는 메서드를 호출하지 않아야 한다.
- Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone 메서드 동기화 필요. 

### 복제 기능은 생성자와 팩터리를 이용하는 것이 좋다.
- 복사 생성자와 복사 팩터리
  - 복사 생성자
    - 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자
  - 복사 팩터리
    - 복사 생성자를 모방한 정적 팩터리
- 생성자를 쓰지 않으며, 모호한 규약, 불필요한 검사 예외, final 용법 방해 등에서 벗어날 수 있다.
- 인터페이스 타입의 인스턴스를 리턴할 수있다.
  - 클라이언트가 복제본의 타입을 결정할 수 있다.

---

## 핵심 정리
```
새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서는 안 되며,
새로운 클래스도 이를 구현해서는 안 된다.
복제 기능은 생성자와 팩터리를 이용하는 것이 가장 좋다.
단, 배열만은 clone 메서드 방식이 가장 깔끔하다.
```