# 아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

## 1. 핵심정리
- 클라이언트 코드가 필드를 직접 사용하면 캡슐화의 장점을 제공하지 못한다.
- 필드를 변경하려면 API를 변경해야 한다.
- 필드에 접근할 때 부수 작업을 할 수 없다.
- package-private(default) 클래스 또는 private 중첩 클래스라면 데이터 필드를 노출해도 문제가 없다.

### (1) 캡슐화의 장점을 제공하지 못하는 클래스 (인스턴스를 모아 놓는 일 외에는 목적이 없는 클래스)
- API를 수정하지 않고는 내부 표현을 바꿀수 없다.
- 불변식을 보장할 수 없다.
- 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다.

```java
public class Point {
    public double x;
    public double y;

    public static void main(String[] args) {
        Point point = new Point();
        point.x = 10;
        point.y = 20; // 필드에 바로 접근함, (필드명이 변경되면 모두 바꿔야함)

        System.out.println(point.x);
        System.out.println(point.y);
    }
}
```

### (2) 접근자와 변경자 메서드를 활용한 클래스 (필드를 private으로 바꾸고, public 접근자(getter)를 추가)
- public 클래스인 경우 (package-private 인 경우는 데이터필드를 노출해도 문제가 없음)
- 그러나 package-private 클래스라고 하더라도 데이터필드를 노출하는 경우는 불필요하다.
- 데이터 필드는 노출하지 말고, 접근자와 변경자 메서드를 활용한 클래스를 사용하자.

```java
public class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() {
      // 부가 작업 추가 가능
      return x;
    }
    public double getY() { return y; }

    public void setX(double x) {
        // 부가 작업 추가 가능
        this.x = x;
    }
    public void setY(double y) { this.y = y; }
}
```

### (3) 불변 필드를 노출한 public 클래스
- public 클래스에서 public final 필드를 노출하면, 사이드 이펙트가 줄어들 수는 있다.
- 그러나, (1)번과 같이 API를 변경하지 않고는 표현방식을 변경할 수 없고,
- 필드를 읽을 때 부수 작업을 수행할 수 없다.

```java
public final class Time {
  private static final int HOURS_PER_DAY = 24;
  private static final int MINUTES_PER_HOUR = 60;

  public final int hour;
  public final int minute;

  public Time(int hour, int minute) {
    if (hour < 0 || hour >= HOURS_PER_DAY)
      throw new IllegalArgumentException("Hour: " + hour);
    if (minute < 0 || minute >= MINUTES_PER_HOUR)
      throw new IllegalArgumentException("Min: " + minute);
    this.hour = hour;
    this.minute = minute;
  }
}
```

###
## 2. 추가 지식
- p103, 아이템 67에서 설명하듯, 내부를 노출한 Dimesion 클래스의 심각한 성능 문제는 오늘날까지도 해결되지 못했다.

### (1) 심각한 성능문제를 야기하는 Dimesion 클래스 예시
- height, width 내부 필드가 노출되어 있음
- 내부 필드가 노출되어 있어서, 안전하게 사용하려면 복사를해서 사용해야 한다. 
- Dimesion을 재사용할 때는 값을 copy하고 새로운 객체를 생성하게 됨

```java
public class DimensionExample {

  public static void main(String[] args) {
    Button button = new Button("hello button");
    button.setBounds(0, 0, 20, 10);

    Dimension dimension = button.getSize();
    System.out.println(dimension.height);
    System.out.println(dimension.width);

    Dimension copyDimension = dimensionCopy(dimension);
  }

  // 재사용시 copy (불필요한 성능문제를 야기)
  private static Dimension dimensionCopy(Dimension size) {
    Dimension dimension = new Dimension();
    dimension.height = size.height;
    dimension.width = size.width;
    return dimension;
  }
}
```

### (2)
