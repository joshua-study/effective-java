# 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라.

### equals를 재정의한 클래스 모두에서 hashCode도 재정의 재정의해야 한다.
- 그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap, HashSet 등 컬렉션의 원소로 사용할 때 문제가 발생한다.

### hashCode의 규약
- equals에 비교되는 정보가 변경되지 않았다면, hashCode 메서드는 일관되게 항상 같은 값을 반환해야 한다.(단, 애플리케이션 재실행 시 값이 달라져도 상관없다.)
- equals가 두 객체를 같다고 판단했다면, 두 객체의 hashCode도 동일해야 한다.
- equals가 두 객체를 다르다고 판단하더라도, 두 객체의 hashCode가 다른 값을 반환할 필요는 없다.(단, 다른 값을 반환해주어야 해시테이블의 성능이 향상된다.)
---

### hashCode 재정의 잘못된 경우
hashCode 재정의 시 hashCode의 두번째 규약을 어기기 쉽다. 즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.
```java
   Map<PhoneNumber, String> m = new HashMap<>();
        m.put(new PhoneNumber(707, 867, 5309), "제니");
        m.get(new PhoneNumber(707, 867, 5309)); // "제니" (X), null (O)
```
위의 코드를 보면 get(PhoneNumber)을 통해 "제니"가 나올 것으로 기대했지만, 
실제로는 hashCode가 재정의되지 않아 원하는 데이터를 찾지 못하고 null을 반환한다. 
이는 hashCode를 재정의 해주면 된다.

---

### 좋은 HashCode 작성하는 법
1. int 변수 result를 선언한 후 값 c로 초기화한다.
   1. 이때 c는 해당 객체의 첫번째 핵심 필드를 계산한다 - 방식은 2-1)번에서 설명한다
      (핵심 필드란 equals 비교에 사용되는 필드)
2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
   1. 해당 필드의 해시코드 c를 계산한다.
      1. 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본 타입의 박싱 클래스이다. 
      2. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다 .계산이 복잡해질 것 같으면, 이 필드의 표준형을 만들어 그 표준형의 hashCode를 호출한다 .필드의 값이 null이면 0을 사용한다
      3. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다.
이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음 갱신한다.
배열에 핵심 원소가 하나도 없다면 단순히 상수 0을 사용한다.
모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.
   2. 위에서 계산한 해시코드 c로 result를 갱신한다.  result = 31 * result + C;
3. result를 반환한다
---
### 전형적인 hashcode 메서드
```java
@Override public int hashCode(){
	int result = Short.hashCode(areaCode);
	result = 31 * result + Short.hashCode(prefix);
	result = 31 * result + Short.hashCode(lineNum);
	return result;
}
```
1. 동치인 PhoneNumber 인스턴스들은 같은 해시코드를 가질 것이다.

### Objects.hash() 메서드
```java
@Override public int hashCode(){
        return Objects.hash(lineNum, prefix, areaCode);
        }
```
1. 입력 변수를 담기 위한 배열이 만들어지고, 입력 중 박싱 언박싱 과정도 거쳐야 하기 때문에 성능상 조금 느리다.
2. 장점 : 단 한줄로 작성할 수 있다.
3. 단점: 속도가 더 느리다.

### 지연 초기화 하는 hashCode 메서드
```java
private int hashCode;

@Override public int hashCode(){
        int result = hashCode;
        if(result == 0){
        int result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
        }
        return result;
        }
```
1. 성능을 높이기 위해 해시코드를 계산할 때 핵심 필드를 생략해서는 안됨 
   1. 속도는 빨라지지만, 해시 품질이 나빠져 해시테이블의 성능을 떨어뜨릴 수 있음
2. hashCode 가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말 것
   1. 클라이언트가 이 값에 의지하지 않고, 추후에 계산 방식을 바꿀 수 있도록 해야함
---
## 핵심정리
equals를 재정의할 때는 hashCode도 반드시 재정의해야 한다. 그렇지 않으면 프로그램이 제대로 동작하지 않을 수 있다.   
hashCode는 Object의 API문서에 기술 된 일반 규약을 따라야 하며, 서로 다은 인스턴스라면 되도록 해시코드도 서로 다르게 구현하는 것이 좋다.   
 AutoValue 프레임워크를 사용하면 equals와 hashCode를 자동으로 만들어준다.