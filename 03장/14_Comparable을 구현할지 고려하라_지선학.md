## Comparable을 구현할지 고려하라

### Object.equals에 더해서 순서까지 비교할 수 있으며 Generic을 지원한다.

### 자기 자신이 (this)이 compareTo에 전달된 객체보다 작으면 음수, 같으면 0, 크다면 양수를 리턴한다.

### 반사성, 대칭성, 추이성을 만족해야 한다. 

```java
public class CompareToConvention {

    public static void main(String[] args) {
        BigDecimal n1 = BigDecimal.valueOf(23134134);
        BigDecimal n2 = BigDecimal.valueOf(11231230);
        BigDecimal n3 = BigDecimal.valueOf(53534552);
        BigDecimal n4 = BigDecimal.valueOf(11231230);

        // p88, 반사성
        System.out.println(n1.compareTo(n1));

        // p88, 대칭성
        System.out.println(n1.compareTo(n2));
        System.out.println(n2.compareTo(n1));

        // p89, 추이성
        System.out.println(n3.compareTo(n1) > 0);
        System.out.println(n1.compareTo(n2) > 0);
        System.out.println(n3.compareTo(n2) > 0);

        // p89, 일관성
        System.out.println(n4.compareTo(n2));
        System.out.println(n2.compareTo(n1));
        System.out.println(n4.compareTo(n1));
        
    }
}
```


### 반드시 따라야 하는 것은 아니지만 x.compareTo(y) == 0이라면 x.equals(y)가 true여야 한다.
### hashCode 규약을 지키지 못하면 해시를 사용하는 클래스와 어울리지 못하듯, compareTo 규약을 지키지 못하면 비교를 활용하는 클래스와 어울리지 못함
### 비교를 활용하는 클래스의 예로는 정렬된 컬렉션인 TreeSet과 TreeMap, 검색과 정렬 알고리즘을 활용하는 유틸리티 클래스인 Collections와 Arrays가 있음

```java
public class CompareToConvention {

    public static void main(String[] args) {
        BigDecimal n1 = BigDecimal.valueOf(23134134);
        BigDecimal n2 = BigDecimal.valueOf(11231230);
        BigDecimal n3 = BigDecimal.valueOf(53534552);
        BigDecimal n4 = BigDecimal.valueOf(11231230);

        
        // p89, compareTo가 0이라면 equals는 true여야 한다. (아닐 수도 있고..)
        BigDecimal oneZero = new BigDecimal("1.0");
        BigDecimal oneZeroZero = new BigDecimal("1.00");
        System.out.println(oneZero.compareTo(oneZeroZero)); // Tree, TreeMap
        System.out.println(oneZero.equals(oneZeroZero)); // 순서가 없는 콜렉션 
        // 두 값을 비교 했을 때 같지 않으면 같지 않은 이유를 설명해야한다 
        // 2.0을 3으로 나누는 것과 2.00을 3으로 나누는 것이 같지 않을 수있다는 주석이 있음
    }
}
```

## 기본 타입 필드가 여럿일 때의 비교자
```java
public final class PhoneNumber implements Cloneable, Comparable<PhoneNumber> {
    
    private final short areaCode, prefix, lineNum;

    @Override
    public int compareTo(PhoneNumber pn) {
        int result = Short.compare(areaCode, pn.areaCode);
        if (result == 0) {
            result = Short.compare(prefix, pn.prefix);
            if (result == 0)
                result = Short.compare(lineNum, pn.lineNum);
        }
        return result;
    }

}
```

### 특정 객체를 비교하는데 필드를 추가 하고 싶으면 composition를 사용해라

```java
public class Point implements Comparable<Point>{

    final int x, y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

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
```java
public class NamedPoint implements Comparable<NamedPoint> {

    private final Point point;
    private final String name;

    public NamedPoint(Point point, String name) {
        this.point = point;
        this.name = name;
    }

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

### java8을 이용해서 개선하는 방법


### 비교자 생성 메서드를 활용한 비교자
```java
    private static final Comparator<PhoneNumber> COMPARATOR =
            comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.getPrefix())
                    .thenComparingInt(pn -> pn.lineNum);

   public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
    }
```
### 장점: 가독성, 단점: 성능 10% 떨어짐
성능이 진짜 문제면 직접 구현해서 써봐라 

제네릭을 사용하기 때문에 컴파일 타임에 문제를 찾을 수 있다는 장점 있음 

### 해시코드 값의 차를 기준으로 하는 비교자 - 추이성을 위배한다.
```java
public class IntOverflow {

    public static void main(String[] args) {
        System.out.println(-2147483648 - 10); //언더오버플로 (자바 자료형의 최대값이 넘어서 양수로 넘어옴)
        System.out.println(Integer.compare(-2147483648, 10)); // 내부적으로 부등호로 비교
    }
}
```

```java
public class DecimalIsNotCorrect {

    public static void main(String[] args) {
        int i = 1;
        double d = 0.1;
        System.out.println(i - d * 9);

        BigDecimal bd = BigDecimal.valueOf(0.1);
        System.out.println(BigDecimal.valueOf(1).min(bd.multiply(BigDecimal.valueOf(9))));
    }
}
```
### 정확한 값을 얻기 위해서는 BigDecimal를 사용해야함!!!

### BigDecimal를 사용하는 이유
- java에서 숫자를 정밀하게 표현하기 위해 사용이 됨 
- 큰 소수점이나 큰 액수를 다룰때 선택이 아닌 필수로 사용해야 되는걸 기억하자
- 내부 코드를 보면 계산을 하기 위해 직접 만든 수식들을 사용함
- 정수* 10-scale로 표현함
scale : 주수 소수점의 자리수를 표현

## 정리 
compareTo 메서드에서 필드의 값을 비교할 때 <와 > 연사자는 쓰지 말아야 한다.
그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.



