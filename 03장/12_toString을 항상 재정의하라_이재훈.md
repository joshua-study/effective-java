# 아이템 12. toString을 항상 재정의하라

## 1. 핵심정리
- toString은 `간결하면서 사람이 읽히 쉬운 형태의 유익한 정보`를 반환해야 한다.
- Object의 toString은 `클래스이름@16진수로 표시한 해시 코드` (default 값)
- 객체가 가진 모든 정보를 보여주는 것이 좋다.
  - 외부에 공개할 수 있는 정보를 보여주는 것이 더 맞는 말, 탈취될 수 있음 (ex: password)
- 값 클래스라면 포맷을 문서에 명시하는 것이 좋으며 해당 포맷으로 객체를 생성할 수 있는 정적 팩터리나 생성자를 제공하는 것이 좋다.
- toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하는 것이 좋다.
  - getter로 제공할 수 있음 (클라이언트에 편리하게 제공)
- 경우에 따라 AutoValue, Lombok 또는 IDE를 사용하지 않는게 적절할 수 있다.
  - 특별한 포맷이 필요한 경우 직접 만듬

### (1) 값 클래스 java doc 주석
```java
public final class PhoneNumber {
  private final short areaCode, prefix, lineNum;

  /**
   * 이 전화번호의 문자열 표현을 반환한다.
   * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
   * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
   * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
   *
   * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
   * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
   * 전화번호의 마지막 네 문자는 "0123"이 된다.
   */
  @Override
  public String toString() {
    return String.format("%03d-%03d-%04d",
                         areaCode, prefix, lineNum);
  }
}
```

### (2) 정적 팩터리 메서드 or 생성자를 제공
```java
public final class PhoneNumber {
  private final short areaCode, prefix, lineNum;

  // 생성자
  public PhoneNumber(int areaCode, int prefix, int lineNum) {
    this.areaCode = rangeCheck(areaCode, 999, "지역코드");
    this.prefix = rangeCheck(prefix, 999, "프리픽스");
    this.lineNum = rangeCheck(lineNum, 9999, "가입자 번호");
  }

  private static short rangeCheck(int val, int max, String arg) {
    if (val < 0 || val > max)
      throw new IllegalArgumentException(arg + ": " + val);
    return (short) val;
  }

  // 정적 팩터리 메서드
  public static PhoneNumber of(String phoneNumberString) {
    String[] split = phoneNumberString.split("-");
    PhoneNumber phoneNumber = new PhoneNumber(
        Short.parseShort(split[0]),
        Short.parseShort(split[1]),
        Short.parseShort(split[2]));
    return phoneNumber;
  }

  public static void main(String[] args) {
    PhoneNumber jenny = new PhoneNumber(707, 867, 5309);
    System.out.println("제니의 번호: " + jenny);

    PhoneNumber phoneNumber = PhoneNumber.of("707-867-5309"); // 정적 팩터리 메서드를 통해 객체 생성
    System.out.println(phoneNumber); // 707-867-5309
  }
}
```

### (3) 나는 어떻게 쓰고 있는가?
- apache commons:commons-lang을 사용하고 있음 (리플렉션 기반)
- [참고링크](http://www.java2s.com/example/java-api/org/apache/commons/lang3/builder/tostringstyle/json_style-0.html)

```java
// gradle
// https://mvnrepository.com/artifact/org.apache.commons/commons-lang3
implementation 'org.apache.commons:commons-lang3:3.12.0'
```

- IDE에서 toString() 자동 생성
```java
// user = User{userId=1, email='alsltnpf1209@gmail.com', name='이재훈'}
  @Override
  public String toString() {
    return "User{" +
        "userId=" + userId +
        ", email='" + email + '\'' +
        ", name='" + name + '\'' +
        '}';
  }
```

- new ToStringBuilder 사용
  - JSON 형태로 출력 
  - 출력 순서를 보장 
  - 오류가 발생하기 쉬운 코드 감소
```java
// 유니코드기 이스케이프 문자열로 대체됨
// user = {"userId":1,"email":"alsltnpf1209@gmail.com","name":"\uC774\uC7AC\uD6C8"}
  @Override
  public String toString() {
    return new ToStringBuilder(this, ToStringStyle.JSON_STYLE)
        .append("userId", userId)
        .append("email", email)
        .append("name", name)
        .toString();
  }
```

- 출력 순서를 보장하지 않음
- JSON 형태로 출력
```java
// user = {"email":"alsltnpf1209@gmail.com","name":"\uC774\uC7AC\uD6C8","userId":1}
@Override
  public String toString() {
    return ToStringBuilder.reflectionToString(this, ToStringStyle.JSON_STYLE);
  }
```