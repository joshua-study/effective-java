## equals를 재정의하려거든 hashCode도 재정의하라

hashCode 일반 규약을 어기에 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제

논리적으로 같은 객체는 같은 해시코드를 반환해야함

```java
class example() {

    public static void main(String[] args) {
        Map<PhoneNumber, String> m = new HashMap<>();
        m.put(new PhoneNumber(707, 867, 5309), "제니");    
    }
    
}
```
위 코드에서 m.get(new PhoneNumber(707, 867, 5309))를 실행하면 "제니"가 나와야함
하지만, null을 반환함

PhoneNumber 클래스는 hashCode를 재정의하지 않았기 때문에 논리적 동치인 두 객체가
서로 다른 해시코드를 반환하여 두번째 규약을 지키지 못함

#### 최악의 (하지만 적법한) hashCode 구헌 - 사용 금지!
```java
class sample(){
    
    @Override public int hashCode(){return 42;}
}
```

위와 같이 정의하면 모든 객체에서 똑같은 해시코드를 반환
해시테이블의 버킷 하나에 담겨 마치 연결 리스트 처럼 동작한다.

그 결과 평균 수행 시간이 O(1)인 해시테이블이 O(n)으로 느려져서 객체가 많아지면 도저기 쓸수 없게 된다.

https://zippy-garlic-b9b.notion.site/JAVA-HashSet-HashMap-ef9b85aec6bb40b8a7a34da9a1d4a59b?pvs=4


곱한 숫자를 31로 정한 이유는 31이 홀수이면서 소수(prime)이기 때문임
소수를 곱하는 이유는 명확하지 않지만 정통적으로 그리해왔다.
결과적으로 31을 이용하면, 이 곱셈을 시프트 연산과 뺄셈으로 대체해 최적화할 수 있음
요즘 VM들은 이런 최적화를 자동으로 해줌

### 전형적인 hashCode 메서드
```java
class sample(){
    @Override public int hashCode(){
        int result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        return result;
    }
}
```

### 한 줄짜리 hashCode 메서드 - 성능이 살짝 아쉽다.
```java
class sample(){
    @Override public int hashCode(){
        return Object.hash(lineNum, prefix, areaCode);
    }
}
```
hash 메서드는 성능에 민감하지 않은 상황에서만 사용하자

클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기보다는 캐싱하는 방식으로 교려해야함

해시의 키로 사용되지 않는 경우라면 hashCode가 처음 불릴 때 계산하는 지연 초기화(lazy initialization) 전략을 고려

### 해시코드를 지연 초기화하는 hashCode 메서드 - 스레드 안정성까지 고려해야 함
```java
class sample(){
    private int hashCode; // 자동으로 0으로 초기화됨
    
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
    
}
```
성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안됨
특히 어떤 필드는 특정 영역에 몰른 인스턴스들의 해시코드를 넓은 범위로 고르게 퍼트려주는 효과가 있을 수 있음 
핵심 필드가 없다면 수많은 인스턴스가 단 몇 개의 해시코드로 집중되어 해시테이블의 속도가 선형으로 느려짐

### 정리
equals를 재정의할 때는 hashCode도 반드시 재정의 해야함

hashCode는 Object의 API 문서에 기술된 일반 규약을 따라야 하며, 
서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 규현해야함

AutoValue 프레임워크를 사용하면 멋진 equals와 hashCode를 자동으로 만들어줌


