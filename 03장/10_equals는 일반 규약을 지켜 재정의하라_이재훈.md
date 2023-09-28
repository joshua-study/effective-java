# 아이템 10. equals는 일반 규약을 지켜 재정의하라.

## 1. 핵심정리
- 언제 equals를 구현하고 언제 구현 하지말아야 하는지 구분이 필요

### (1) equals를 재정의 하지 않는 것이 최선
- 다음의 경우에 해당한다면 equals를 재정의 할 필요가 없다.
- 각 인스턴스가 본질적으로 고유하다. (`ex: 싱글톤 인스턴스, enum ...`)
- 인스턴스의 `논리적 동치성`을 검사할 필요가 없다.
  - 논리적 동치성 : 만원짜리 2개의 값은 같다. (`동등성`)
  - 동등성(값 비교)를 할 필요가 없는 경우
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 적절하다.
  - List, Map, Set 등을 상속받는 경우 (이미 구현이 되어있음)
- 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.
  - public은 equals가 호출될 수 있음, 제한적인 상황에서만 equals가 필요없을 수 있음.

### (2) equals 규약
- 반사성 : A.equals(A) == true
- 대칭성 : A.equals(B) == B.equals(A)
    - CaseInsensitiveString
- 추이성 : A.equals(B) && B.equals(C), A.equals(C)
  - Point, ColorPoint(inherit), CounterPointer, ColorPoint(comp)
- 일관성 : A.equals(B) == A.equals(B)
- null-아님 : A.equals(null) == false

###
- 대칭성이 맞지 않는 경우
```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    // 대칭성이 위배
    @Override
    public boolean equals(Object o) {
        // 자기 자신과 같은 타입일 때 비교
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);

        // 비교 하는 타입이 자기 자신의 타입의 인스턴스변수와 같을 때 비교(String)
        // 대칭성을 지키려면 제거해야함.
        if (o instanceof String)  // 한 방향으로만 작동하게 됨
            return s.equalsIgnoreCase((String) o);
        return false;
    }

    public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        String polish = "polish";
        System.out.println(cis.equals(polish)); // true
        System.out.println(polish.equals(cis)); // false

        List<CaseInsensitiveString> list = new ArrayList<>();
        list.add(cis);

        // polish가 들어있지 않다고 나올 것
        System.out.println(list.contains(polish));
    }
}
```

- 추이성이 맞지 않는 경우
- 아래와 같은 예제는 대칭성과 추이성을 모두 만족하는 equals를 만들수 없다.
- p59, 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재 하지 않는다.
```java
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    // 코드 10-2 잘못된 코드 - 대칭성 위배! (57쪽)
    // Color와 ColorPoint를 비교하면 대칭성이 깨짐
    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }

    // 코드 10-3 잘못된 코드 - 추이성 위배! (57쪽)
    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;

        // 타입이 Point인 경우
        // o가 일반 Point면 색상을 무시하고 비교
        if (!(o instanceof ColorPoint))
            return o.equals(this);

        // o가 ColorPoint면 색상까지 비교
        return super.equals(o) && ((ColorPoint) o).color == color;
    }

    public static void main(String[] args) {
        // 첫 번째 equals 메서드(코드 10-2)는 대칭성을 위배
        Point p = new Point(1, 2);
        ColorPoint cp = new ColorPoint(1, 2, Color.RED);
        System.out.println(p.equals(cp)); // true
        System.out.println(cp.equals(p)); // false (p는 ColorPoint 타입이 아님)

        // 두 번째 equals 메서드(코드 10-3)는 추이성을 위배
        ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
        Point p2 = new Point(1, 2);
        ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
        System.out.println(p1.equals(p2)); // true
        System.out.println(p2.equals(p3)); // true
        System.out.println(p1.equals(p3)); // false
    }
}
```

- 구체 클래스를 확장할 때 상속을 하지말고 컴포지션으로 구현해야 한다. (위임을 사용)
```java
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    /**
     * 이 ColorPoint의 Point 뷰를 반환한다.
     */
    public Point asPoint() {
        return point;
    }

  @Override
  public boolean equals(Object o) {
    if (!(o instanceof ColorPoint))
      return false;
    ColorPoint cp = (ColorPoint) o;
    return cp.point.equals(point) && cp.color.equals(color);
  }

  @Override
  public int hashCode() {
    return 31 * point.hashCode() + color.hashCode();
  }
}
```

- 일관성은 객체가 변화하면 깨질 수 있다.
- 불변 객체라면 equals의 결과는 항상 같아야 한다.
```java
public class EqualsInJava extends Object {

    public static void main(String[] args) throws MalformedURLException {
        long time = System.currentTimeMillis();
        Timestamp timestamp = new Timestamp(time); // Date를 상속받은 클래스
        Date date = new Date(time);

        // 대칭성 위배! P60 -> 컴포지션 권장
        System.out.println(date.equals(timestamp)); // true
        System.out.println(timestamp.equals(date)); // false

        // 일관성 위배 가능성 있음. P61
        URL google1 = new URL("https", "about.google", "/products/");
        URL google2 = new URL("https", "about.google", "/products/");
        System.out.println(google1.equals(google2));
    }
}
```

### (3) equals 구현 방법
- == 연산자를 사용해 자기 자신의 참조인지 확인한다.
- instanceof 연산자로 올바른 타입인지 확인한다.
- 입력된 값을 올바른 타입으로 형변환 한다.
- 입력 객체와 자기 자신의 대응되는 핵심 필드가 일치하는지 확인한다.

```java
@Override
    public boolean equals(Object o) {
        // 반사성
        if (this == o) {
            return true;
        }

        if (!(o instanceof Point)) {
            return false;
        }

        // 핵심적인 필드들만 비교
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
```

###
- 그러나, 직접 구현하는 것은 어렵고 실수가 나올 수 있음
- 그래서, 구글의 AutoValue 또는 Lombok을 사용한다.
- IDE의 코드 생성 기능을 사용한다.
- Java 14부터 지원하는 Record 타입을 사용한다.

```java
// Lombok 사용
@EqualsAndHashCode
@ToString
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

}
```

### (4) 주의사항
- equals를 재정의 할 때 hashCode도 반드시 재정의하자.
- 값들을 비교할 때 너무 복잡하게 하지 말자.
- Object가 아닌 타입의 매개변수를 받는 equals 메서드는 선언하지말자.

```java
@Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }

        if (!(o instanceof Point)) {
            return false;
        }

        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
```

## 2. 추가 지식
- p53, 값 클래스
- p58, StackOverflowError
- p59, 리스코프 치환 원칙 (LSP)
- p59, 상속 대신 컴포지션을 사용하라. (아이템 18)


### (1) Value 기반의 클래스
클래스처럼 생겼지만 int처럼 동작하는 클래스
- 식별자가 없고 불변이다.
- 식별자가 아니라 인스턴스가 가지고 있는 상태를 기반으로 equals, hashCode, toString을
구현한다.
- == 오퍼레이션이 아니라 equals를 사용해서 동등성을 비교한다. 
- 동일한(equals) 객체는 상호교환 가능한다.
- https://docs.oracle.com/javase/8/docs/api/java/lang/doc-files/ValueBased.html
- https://cr.openjdk.java.net/~jrose/values/values-0.html

### (2) StackOverflowError
로컬 변수와 객체가 저장되는 공간의 이름은?
- 스택(stack)과 힙(heap)
- 메서드 호출시, 스택에 스택 프레임이 쌓인다.
  - 스택 프레임에 들어있는 정보 : 메서드에 전달하는 매개변수, 메서드 실행 끝내고 돌아갈 곳, 힙에 들어있는 객체에 대한 레퍼런스 ...
  - 그런데 더이상 스택 프레임을 쌓을 수 없다며? StackOverflowError!
- 스택의 사이즈를 조정하고 싶다면? -Xss1M
  - https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/jrdocs/refman/optionX.html

### (3) 객체 지향 5대 원칙 SOLID 중에 하나.
- 1994년, 바바라 리스코프의 논문, “A Behavioral Notion of Subtyping”에서
  기원한 객체 지향 원칙.
- `하위 클래스의 객체`가 `상위 클래스 객체`를 대체하더라 도 소프트웨어의 기능을 깨트리지 않아야 한다. (semantic over syntacic, 구문 보다는 의미!)