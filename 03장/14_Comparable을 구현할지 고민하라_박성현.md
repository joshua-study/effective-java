# 14. Comparable을 구현할지 고민하라.

---

## (1) Comparable, Comparator?
- Comparator와 Comparable은 자바에서 객체 정렬에 사용되는 인터페이스
- 서로 다른 방식으로 객체를 정렬하고 비교하는 데 사용

### Comparable 인터페이스
- Comparable 인터페이스는 정렬 가능한 클래스(즉, 정렬을 지원하는 클래스)가 구현해야 하는 인터페이스
- compareTo 메서드를 정의하여 객체를 다른 객체와 비교하고 정렬 기준을 제공
- compareTo 메서드는 현재 객체가 다른 객체보다 작으면 음수 값을, 크면 양수 값을, 같으면 0을 반환
- Comparable을 구현한 클래스는 자연 순서(기본 정렬 순서)에 따라 정렬
```java
public class MyClass implements Comparable<MyClass> {
   private int value;

   public MyClass(int value) {
       this.value = value;
   }

   @Override
   public int compareTo(MyClass other) {
       // value 값을 기준으로 비교
       return this.value - other.value;
   }
}
```

### Comparator 인터페이스

- Comparator 인터페이스는 객체의 정렬 기준을 제공하는 `별도의 클래스`를 생성할 때 사용
- compare 메서드를 정의하여 두 객체를 비교하고 정렬 기준을 제공
- Comparator를 사용하면 기본 정렬 순서 외에도 다양한 정렬 방식을 지원

```java
public class MyComparator implements Comparator<MyClass> {
   @Override
   public int compare(MyClass obj1, MyClass obj2) {
       // value 값을 기준으로 비교
       return obj1.getValue() - obj2.getValue();
   }
}
```

`Comparable`은 클래스 자체에 정렬 방식을 포함시키는 반면, `Comparator`는 정렬 방식을 별도의 클래스로 분리하여 정의

정렬을 수행하는 메서드들(예: `Collections.sort()` 또는 `Arrays.sort()`)은 `Comparable` 또는 `Comparator`를 사용하여 정렬 방식을 결정하고, 사용자는 이 인터페이스들을 구현하여 원하는 정렬 동작을 정의할 수 있음.

---

## (2) Comparable 규약
- Object.equals에 더해서 순서까지 비교할 수 있으며, 제네릭을 지원한다.
- 자기 자신이 compareTo에 전달된 객체보다 작으면 음수, 같으면 0, 크다면 양수를 리턴한다.
  - 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.
- 반사성, 대칭성, 추이성을 만족해야 한다.
- 반드시 따라야 하는 것은 아니지만 x.compareTo(y) == 0 이라면 x.equals(y)가 true여야 한다.
```java
public class CompareToConvention {
    public static void main(String[] args) {
        BigDecimal n1 = BigDecimal.valueOf(23134134);
        BigDecimal n2 = BigDecimal.valueOf(11231230);
        BigDecimal n3 = BigDecimal.valueOf(53534552);
        BigDecimal n4 = BigDecimal.valueOf(11231230);

        // 반사성
        System.out.println(n1.compareTo(n1)); // 0

        // 대칭성
        System.out.println(n1.compareTo(n2)); // 1
        System.out.println(n2.compareTo(n1)); // -1

        // 추이성
        System.out.println(n3.compareTo(n1) > 0); // true
        System.out.println(n1.compareTo(n2) > 0); // true
        System.out.println(n3.compareTo(n2) > 0); // true

        // compareTo가 0이라면 equals는 true여야 한다. (아닐 수도 있도)
        BigDecimal oneZero = new BigDecimal("1.0");
        BigDecimal oneZeroZero = new BigDecimal("1.00");
        System.out.println(oneZero.compareTo(oneZeroZero)); // Tree, TreeMap
        System.out.println(oneZero.equals(oneZeroZero)); // 순서가 없는 콜렉션
    }
}
```

---

## (3) Comparable 구현 방법 1
- 자연적인 순서를 제공할 클래스에 'implement Comparable<T>' 을 선언한다.
- compareTo 메서드를 재정의한다.
- compareTo 메서들 안에서 기본 타입은 박싱된 기본 타입의 compare을 사용해 비교한다.
  - 관계연산자를 <, > 사용 --> 오류 유발 ↑
```java
public class Point implements Comparable<Point> {

    final int x, y;
    
    // ...

    @Override
    public int compareTo(Point point) {
        int result = Integer.compare(this.x, point.x);
        if (result == 0) {
            result = Integer.compare(this.y, point.y);
        }
        return result;
    }
}
```
- 핵심 필드가 여러 개라면 비교 순서가 중요. 
- 순서를 결정하는데 있어서 가장 중요한 필드를 비교하고 그 값이 0이라면 다음 필드를 비교한다.
```java
@Override
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if (result == 0)  {
        result = Short.compare(prefix, pn.prefix);
        if (result == 0)
            result = Short.compare(lineNum, pn.lineNum);
    }
    return result;
}
```
- 기존 클래스를 확장(상속)하고 필드를 추가하는 경우 compareTo 규약을 지킬 방법이 없다.
  - Composition을 활용할 것.
    - Composition: 상속에서의 상위 클래스의 인스턴스 변수를 하위 클래스가 가지는 것
```java
public class Parent { 
    private int x, y;
    
    // ...
}
public class Child {
    private Parent parent;
    private int z;
    
    // ...
}
```

---

## (4) Comparable 구현 방법 2
- Comparator가 제공하는 static 메서드를 사용해서 Comparable 인스턴스 생성
  - static 인스턴스를 생성한 후 Comparator의 default 메서드를 체이닝으로 사용 가능
```java
private static final Comparator<PhoneNumber> COMPARATOR =
        comparingInt((PhoneNumber pn) -> pn.areaCode)
                .thenComparingInt(pn -> pn.getPrefix())
                .thenComparingInt(pn -> pn.lineNum);
@Override
public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
}
```

---

# 핵심정리
```
순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여,
그 인스턴스를 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야 한다.
compareTo 메서드에서 필드의 값을 비교할 때 <와 > 연산자를 쓰지 말아야 한다.
그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나
Comparator 인스턴스가 제공하는 비교자 생성 메서드를 사용하자.
```