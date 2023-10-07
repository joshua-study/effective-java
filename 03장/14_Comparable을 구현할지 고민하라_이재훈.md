# 아이템 14. Comparable을 구현할지 고려하라

## 1. 핵심정리
compareTo 규약
- Object.equals에 더해서 순서까지 비교할 수 있으며 Generic을 지원한다.
- 자기 자신이(this) compareTo에 전달된 객체보다 작으면 음수, 같으면 0, 크다면 양수를 리턴한다.
- 반사성, 대칭성, 추이성을 만족해야 한다.
- 반드시 따라야 하는 것은 아니지만 x.compareTo(y) == 0이라면 x.equals(y)가 true여야 한다.
```java
public static void main(String[] args) {
        BigDecimal n1 = BigDecimal.valueOf(23134134);
        BigDecimal n2 = BigDecimal.valueOf(11231230);
        BigDecimal n3 = BigDecimal.valueOf(53534552);
        BigDecimal n4 = BigDecimal.valueOf(11231230);

        // p88, 반사성
        // A.equals(A) = A.equals(A)
        System.out.println(n1.compareTo(n1)); // n1 == n1 = 0

        // p88, 대칭성
        // A.equals(B) = B.equals(A)
        System.out.println(n1.compareTo(n2)); // n1 > n2 = 양수 (+1)
        System.out.println(n2.compareTo(n1)); // n2 < n1 = 음수 (-1)

        // p89, 추이성
        // A.equals(B), B.equals(C) = A.equals(C)
        System.out.println(n3.compareTo(n1) > 0); // true
        System.out.println(n1.compareTo(n2) > 0); // true
        System.out.println(n3.compareTo(n2) > 0); // true

        // p89, 일관성
        // A, B가 같을때 A > C 라면, B > C
        System.out.println(n4.compareTo(n2)); // 0
        System.out.println(n2.compareTo(n1)); // -1
        System.out.println(n4.compareTo(n1)); // -1

        // p89, compareTo가 0이라면 equals는 true여야 한다. (아닐 수도 있고..)
        // equals는 .0과 .00을 같다고 보지 않음
        BigDecimal oneZero = new BigDecimal("1.0");
        BigDecimal oneZeroZero = new BigDecimal("1.00");
        System.out.println(oneZero.compareTo(oneZeroZero)); // ture, Tree, TreeMap
        System.out.println(oneZero.equals(oneZeroZero)); // false, 순서가 없는 콜렉션
    }
```

### (1) compareTo 구현 방법 1
- 자연적인 순서를 제공할 클래스에 implements Compratable<T> 을 선언한다.
- compareTo 메서드를 재정의한다.
- compareTo 메서드 안에서 기본 타입은 박싱된 기본 타입의 compare을 사용해 비교한다.
- 핵심 필드가 여러개라면 비교순서가 중요하다. 순서를 결정하는데 있어서 가장 중요한 필드를 비교하고 그값이 0이라면 다음 필드를 비교한다.
- 기존 클래스를 확장하고 필드를 추가하는 경우 compareTo 규약을 지킬 수 없다.
  - Composition을 활용할 것.

```java
public class Point implements Comparable<Point>{

    final int x, y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

  @Override
  public int compareTo(Point point) {
    // 정렬 순서는 x, y
    int result = Integer.compare(this.x, point.x); // x - y
    if (result == 0) {
      result = Integer.compare(this.y, point.y);
    }
    return result;
  }
}
```
#### 상속을 받는 하위 클래스에서 compareTo를 재정의 하려면?
- 컴포지션을 활용해야 한다.
```java
public class NamedPoint implements Comparable<NamedPoint> {

  // equals 규약이 깨지지 않도록 컴포지션을 사용하는 것처럼 compareTo도 컴포지션을 통해 재정의
  private final Point point;
  private final String name;

  public NamedPoint(Point point, String name) {
    this.point = point;
    this.name = name;
  }

  // Point view를 제공
  public Point getPoint() {
    return this.point;
  }

  @Override
  public int compareTo(NamedPoint namedPoint) {
    int result = this.point.compareTo(namedPoint.point);
    if (result == 0) {
      result = this.name.compareTo(namedPoint.name);
    }
    return result;
  }
}
```


### (2) compareTo 구현 방법 2
- 자바 8부터 함수형 인터페이스, 람다, 메서드 레퍼런스와 Comprator가 제공하는 기본 메서드와 static 메서드를 사용해서 Comprator를 구현할 수 있다.
- Comparator가 제공하는 메서드 사용하는 방법
  - Comparator의 static 메서드를 사용해서 Comparator 인스턴스 만들기
  - 인스턴스를 만들었다면 default 메서드를 사용해서 메서드 호출 이어가기 (체이닝)
  - static 메서드와 default 메서드의 매개변수로는 람다 표현식 또는 메서드 레퍼런스를 사용할 수 있다.

```java
// PhoneNumber를 비교할 수 있게 만든다. (91-92쪽)
public final class PhoneNumber implements Comparable<PhoneNumber> {
    
  private final short areaCode, prefix, lineNum;

  ...

    // 코드 14-3 비교자 생성 메서드를 활용한 비교자 (92쪽)
    private static final Comparator<PhoneNumber> COMPARATOR =
            comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.getPrefix())
                    .thenComparingInt(pn -> pn.lineNum);

   @Override
   public int compareTo(PhoneNumber pn) {
       return COMPARATOR.compare(this, pn);
   }

}
```

###
## 2. 추가 지식
- p90, `제네릭 인터페이스`이므로 compareTo 메서드의 인수 타입은 `컴파일 타임`에 정해진다.
- p92, 자바의 `타입 추론 능력`이 이 상황에서 타입을 알아낼 만큼 ...
- p93, 이 방식은 `정수 오버플로`를 일으키거나
- p93, IEEE 754 `부동소수점` 계산 방식에 따른 오류를 낼 수 있다.

자바 타입 추론
```java
private static final Comparator<PhoneNumber> COMPARATOR =
            // comparingInt를 통해서 생성되는 객체는 PhoneNumber 인 것을 알수 있었기 때문에
            // thenComparingInt는 타입을 명시할 필요가 없어짐
            comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.getPrefix())
                    .thenComparingInt(pn -> pn.lineNum);

```
```java
    @Override
public int compareTo(PhoneNumber pn) {
        // 지역 변수로 var 타입 추론 사용
        var result = Short.compare(areaCode, pn.areaCode); // x - y
        if (result == 0)  {
        result = Short.compare(prefix, pn.prefix);
        return result;
        }
```

정수 오버플로우
- 비교할때는 정수 오버플로우를 조심해야함 (Integer.compare 써라.)
```java
public class IntOverflow {
    public static void main(String[] args) {
        System.out.println(-2147483648 - 10); // 2147483638
        System.out.println(Integer.compare(-2147483648, 10)); // -1
    }
}

```

부동소수점
- 정확한 계산이 필요하다면? BigDecimal 써야함
```java
public class DecimalIsNotCorrect {

    public static void main(String[] args) {
        int i = 1;
        double d = 0.1;
        // double 계산법
        System.out.println(i - d * 9); // 0.1이 나올 것 같지만 -> 0.09999999999999998

        // BigDecimal 계산법
        BigDecimal bd = BigDecimal.valueOf(0.1);
        System.out.println(BigDecimal.valueOf(1).min(bd.multiply(BigDecimal.valueOf(9)))); // 0.9
    }
}

```