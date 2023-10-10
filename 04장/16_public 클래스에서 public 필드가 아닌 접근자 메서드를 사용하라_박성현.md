# 16. public 클래스에서 public 필드가 아닌 접근자 메서드를 사용하라.

---

## (1) 필드를 public으로 선언했을 경우.
- 클라이언트 코드가 필드를 직접 사용하면 캡슐화의 장점을 제공하지 못한다.
  - 개발속도, 개발비용, 성능 최적화, 재사용성, 개발난이도
- 필드를 변경하려면 API를 변경해야 한다.
- 필드에 접근할 때 부가 작업을 할 수 없다.
  - 필드값에 조건이 있어(양수o, 음수x ...) 검증하는 작업이 필요하다면 처리할 수 없음.

### ※ 필드를 private으로 변경해주고 public 접근자 메서드를 사용하자(Getter, Setter)
- 클래스가 package-private, private 중첩 클래스라면 데이터 필드를 노출해도 문제는 없다.

```java
class Point {
    private double x;
    private double y;
    
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() {
        // 부가 작업
        return x;
    }
    public double getY() {
        return y;
    }

    public void setX(double x) {
        // 부가 작업
        this.x = x;
    }
    public void setY(double y) {
        this.y = y;
    }
}
```

---

## (2) public 클래스의 필드가 불변인 경우.

- public 필드를 직접 노출해도 변경의 문제가 없지만
- 여전히 API를 변경하지 않고는 표현방식을 바꿀 수 없고,
- 필드를 읽을 때 부가 작업을 수행할 수 없다.
- 하지만, 불변식은 보장할 수 있게 된다.
```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;
    
    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY) {
          throw new IllegalArgumentException("시간: " + hour);
        }
        if (minute < 0 || minute >= MINUTES_PER_HOUR) {
          throw new IllegalArgumentException("분: " + minute);
        }
        this.hour = hour;
        this.minute = minute;
    }
}
```

---

## 핵심 정리

```
public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다.
불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다.
하지만 package-private 클래스나 private 중첩 클래스에서는 종종 필드를 노출하는 편이 나을 때도 있다.
```