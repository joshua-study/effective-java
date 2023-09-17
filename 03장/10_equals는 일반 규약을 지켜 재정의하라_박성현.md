# 10. equals는 일반 규약을 지켜 재정의하라.

---

## 1. equals를 재정의 하지 않는 것이 최선
### 1-1. equals를 재정의하지 않아도 될 경우
- 각 인스턴스가 본질적으로 고유할 때
  - 싱글톤
  - Enum
- 논리적인 동치성을 검사할 필요가 없는 경우
  - 논리적 동치성 비교(equals)
    - 동일한 객체는 아니지만 객체의 상태 값이 서로 같은 경우를 논리적으로 동등하다 함.
- 상위 클래스에 정의한 equals를 하위클래스에 재사용할 때
- 클래스가 private이거나 package-private일 때 equals를 호출할 필요가 없다.
  - public일 경우 어디서든 참조가 가능하기 때문에 equals가 호출되지 않을 것이란 보장이 없음. 
  - List나 Map에, 구현한 public 클래스를 넣는다면 equals가 호출됨.

---


## 2. equals 규약

###
### 2-1. 반사성 
- A.eqauls(A) == true


### 2-2. 대칭성
- A.eqauls(B) == B.eqauls(A)
```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    // 대칭성 위배!
    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        if (o instanceof String)  // 한 방향으로만 작동한다!
            return s.equalsIgnoreCase((String) o);
        return false;
    }

  // CaseInsensitiveString에서 정의한 equals는
  // String을 알고 있지만
  // String 클래스는 CaseInsensitiveString의 존재를 알지 못함.
  public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        String polish = "polish";
        System.out.println(cis.equals(polish)); // true
        System.out.println(polish.equals(cis)); // false
    
        List<CaseInsensitiveString> list = new ArrayList<>();
        list.add(cis);
        System.out.println(list.contains(polish)); // false
    }
    
    // 수정한 equals 메서드
    @Override public boolean equals(Object o) {
        return o instanceof CaseInsensitiveString &&
                ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
    }
}
```

### 2-3. 추이성
- A.eqauls(B) && B.eqauls(C), A.eqauls(C)
- 구체 클래스를 확장해 새로운 값(필드)을 추가하면 equals 규약을 만족 시킬 수 없다.  
```java
public class Point {
    private final int x;
    private final int y;
    
    // ...
    
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }

    // 잘못된 코드 - 리스코프 치환 원칙 위배!
    @Override public boolean equals(Object o) {
      if (o == null || o.getClass() != getClass())
        return false;
      Point p = (Point) o;
      return p.x == x && p.y == y;
    }
}

public class ColorPoint extends Point { 
    private final Color color;
    
    // ...

    // 대칭성 위배
    @Override public boolean equals(Object o) {
      if (!(o instanceof ColorPoint))
        return false;
      
      // ColorPoint는 Point 타입이지만
      // Point는 ColorPoint 타입이 아니다.
      return super.equals(o) && ((ColorPoint) o).color == color;
    }

    // 추이성 위배
    @Override 
    public boolean equals(Object o) {
      if (!(o instanceof Point))
          return false;

      // o가 일반 Point면 색상을 무시하고 비교한다.
      // ColorPoint 레벨의 다른 클래스에 같은 방식으로 equals를 구현했다면
      // equals 메서드를 계속 호출하기 때문에 StackOverflow가 발생.
      if (!(o instanceof ColorPoint))
          return o.equals(this);

      // o가 ColorPoint면 색상까지 비교한다.
      return super.equals(o) && ((ColorPoint) o).color == color;
    }
}

public class CounterPoint extends Point {
    // 필드를 추가하지 않았으므로 상위 클래스의 equals를 사용해도 됨.
    // 상위 클래스에 정의한 equals를 하위클래스에 재사용할 때
    // equals를 재정의하지 않아도 된다고 했음.
    // ...
}

public class CounterPointTest {
    private static final Set<Point> unitCircle = Set.of(
          new Point( 1,  0), new Point( 0,  1),
          new Point(-1,  0), new Point( 0, -1));

    public static boolean onUnitCircle(Point p) {
        return unitCircle.contains(p);
    }

    public static void main(String[] args) {
        Point p1 = new Point(1,  0);
        Point p2 = new CounterPoint(1,  0);
    
        // true
        System.out.println(onUnitCircle(p1));
    
        // true를 출력해야 하지만, 
        // 리스코프 치환 원칙으로 상위 클래스로 동작하고 있는 코드에 하위 클래스의 인스턴스가 들어가도 동일하게 동작해야 함.
        // but, Point의 equals가 getClass를 사용해 작성되었다면 그렇지 않다.
        // CounterPoint는 Point를 상속받고 있지만 
        // getClass를 사용하여 비교한다면 구체적 클래스를 비교하므로 false
        System.out.println(onUnitCircle(p2));
    }
}
```
- 필드를 추가하고 싶을 경우 상속을 하지않고 컴포지션(조합)을 추가하여 필드 추가
  - 상속받을 클래스를 하나의 필드로 추가
```java
public class ColorPoint {
    private final Point point;
    private final Color color;
    
    // ...

    /**
     * 이 ColorPoint의 Point 뷰를 반환한다.
     */
    public Point asPoint() {
        return point;
    }
    
    // ...
}
public class CounterPointTest {
  private static final Set<Point> unitCircle = Set.of(
          new Point( 1,  0), new Point( 0,  1),
          new Point(-1,  0), new Point( 0, -1));

  public static boolean onUnitCircle(Point p) {
    return unitCircle.contains(p);
  }

  public static void main(String[] args) {
    Point p1 = new Point(1,  0);
    // ColorPoint이지만 asPoint()를 사용해서 Point로 볼 수 있게 만듦. 
    Point p2 = new me.whiteship.chapter01.item10.composition.ColorPoint(1, 0, Color.RED).asPoint();

    // ...
  }
}
```

### 2-4. 일관성
- A.equals(B) == A.equals(B)
- 두 객체가 같다면 앞으로도 영원히 같아야 한다. (불변 객체)
- 가변 객체라면 일관성은 깨질 수 있다.
- equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해선 안된다. ex)url

### 2-5. null-아님 
- A.equals(null) == false

---

## 3. equals 구현과 주의 사항

###
### 3-1. 구현
- == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
- instanceof 연산자로 입력이 올바른 타입인지 확인다.
- 입력을 올바른 타입으로 형변환한다.
- 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 검사
- 필드타입별 비교
  - 참조 타입: eqauls
  - 기본 타입: ==
  - float, double: compare()
```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    @Override
    public boolean equals(Object o) {
      if (this == o) {
          return true;
      }
      
      if (!(o instanceof Point)) 
          return false;
      
      Point p = (Point) o;
      
      return p.x == x && p.y ==y; 
    }
    
}
```
- 하지만 직접 구현하기엔 어렵다.
- AutoValue: @AutoValue가 있는 클래스에 equals,ToString,hashcode 메서드 생성
- Lombok: @EqualsAndHashCode, @ToString
- Java11 -> Lombok, Java17 -> Recode

###
### 3-2. 주의 사항
- 반드시 hashCode를 재정의해줘야 함.
- 너무 복잡하게 해결하지 말자.
- Object가 아닌 타입의 매개변수를 받는 equals 메서드는 선언하지 말자. -> 오버로딩이 아닌 오버라이딩.