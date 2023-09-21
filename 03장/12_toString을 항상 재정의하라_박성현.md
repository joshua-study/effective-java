# 12. toString을 항상 재정의하라.
- `모든 하위 클래스에서 toString을 재정의하라.`

---

### 1. toString은 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 함.
- Object의 toString은 '클래스 이름@16진수로 표시한 해시코드'로 표현 
  - PhoneNumber@adbbd
- 직관적으로 보여주는 것이 더 유익한 정보를 담고 있다.
  - {PhoneNumber=707-867-5309}

### 2. 객체가 가진 모든 정보를 보여주는 것이 좋다(?)
  - 객체가 거대하거나 객체의 상태가 문자열로 표현하기에 적합하지 않다면 요약정보만 담아야 한다.
  - (?) → 회원의 주문 상세 내역을 가진 객체가 있다면 보여주는 것이 맞는 걸까?
    - 사용자의 정보를 중요시하는 서비스라면 log도 남겨서는 안됨.

### 3. 값 클래스라면 포맷을 문서에 명시하고 정적팩터리나 생성자를 제공하자
- 값 클래스
  - 객체의 값을 나타내는 클래스로서 데이터를 캡슐화하고 불변성을 가지며, 객체의 상태를 표현하는 데 사용

```java
public class PhoneNumber {
    // equals, hashCode 생략
    
    private final short areaCode, prefix, lineNum;
    
    @Override
    public String toString() {
        return String.format("%03d-%03d-%04d",
                areaCode, prefix, lineNum);
    }

    // 정적 팩터리 메서드
    public static PhoneNumber of(String phoneNumberString) {
        String[] split = phoneNumberString.split("-");
        PhoneNumber phoneNumber = new PhoneNumber(
                Integer.parseInt(split[0]),
                Integer.parseInt(split[1]),
                Integer.parseInt(split[2]));
        return phoneNumber;
    }

    public static void main(String[] args) {
        PhoneNumber jenny = new PhoneNumber(707, 867, 5309);
        System.out.println("제니의 번호: " + jenny);

        PhoneNumber phoneNumber = PhoneNumber.of("707-867-5309");
        System.out.println(phoneNumber);

      System.out.println(jenny.equals(phoneNumber)); // true
      System.out.println(jenny.hashCode(phoneNumber)); // true
    }
}
```

### 4. toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API(접근자)를 제공하는 것이 좋다.
- toString에 사용되는 데이터의 Getter 제공하자.
- toString으로 반환한 값에 포함된 일부 정보(필드)가 필요하다면 접근자(getter)를 통해 가져올 수 있지만,
- 접근자가 없다면 toString의 반환값을 파싱할 수 밖에 없다. → 불필요 작업, 성능저하.

### 5. 경우에 따라 AutoValue, Lombok, IDE를 사용하지 않는 것이 적절할 수도 있다.
- 사용하는 객체의 특성에 맞게 toString을 제공해야하는 경우가 있으므로 직접 재정의하는 것이 좋음.

---
## 핵심 정리
```
모든 구체(하위) 클래스에서 toString을 재정의하자.
상위 클래스에서 이미 알맞게 재정의한 경우는 예외다. 
toString을 재정의한 클래스는 그 클래스를 사용한 시스템을 디버깅하기 쉽게 해준다.
toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.
```

