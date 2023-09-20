# 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라.

## 1. 핵심정리
hashCode 규약
- equals 비교에 사용하는 정보가 변경되지 않았다면 hashCode는 매번 같은 값을 리턴해야한다.
    - 변경되거나, 애플리케이션을 다시 실행했다면 달라질 수 있다.
- `두 객체에 대한 equals가 같다면, hashCode의 값도 같아야 한다.`
- 두 객체에 대한 equals가 다르더라도, hashCode의 값은 같을 수 있지만 `해시 테이블 성능을 고려해 다른 값을 리턴하는 것이 좋다.`

### (1) hashCode
- `hashCode`는 객체를 식별하고 검색하는 데 사용되는 해시 코드를 반환하는 java의 메서드. 객체의 `equals` 메서드와 함께 정의하여 사용
- 어떠한 공간에 객체를 넣을 때 hashCode의 값을 가지고 어느 버킷에 넣을지 정한다. 시간복잡도 : O(1)
- 객체를 꺼낼 때도 hashCode의 값을 먼저 가져와서, hash에 해당하는 버킷에 있는 객체를 꺼낸다.

### (2) hashCode 구현방법 
1. 핵심 필드 하나의 값의 해쉬값을 계산해서 result 값을 초기화 한다.
2. 기본 타입은 Type.hashCode 참조 타입은 해당 필드의 hashCode 배열은 모든 원소를 재귀적으로 위의 로직을 적용하거나, Arrays.hashCode result = 31 * result + 해당 필드의 hashCode 계산값
3. result를 리턴.

- 짧은 한 줄짜리 hashCode 메서드 (성능이 살짝 아쉬움)
```java
// 코드 11-3 한 줄짜리 hashCode 메서드 - 성능이 살짝 아쉽다. (71쪽)
  @Override public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
  }
```

- guava hashCode 메서드 (굳이 사용할까?)
```java
  @Override
  public int hashCode() {
    return Hashing.goodFastHash(32)
                  .hashObject(this, PhoneNumberFunnel.INSTANCE)
                  .hashCode();
  }
```
### (3) hashCode 구현대안
- 구글 구아바의 com.google.common.hash.Hashing
- Objects 클래스의 hash 메서드
- 캐싱을 사용해 불변 클래스의 해시 코드 계산 비용을 줄일 수 있다.

### (4) 주의 할 것
- 지연 초기화 기법을 사용할 때 스레드 안전성을 신경써야 한다. (아이템 83)
- 성능 때문에 핵심 필드를 해시코드 계산할 때 빼면 안 된다.
- 해시코드 계산 규칙을 API에 노출하지 않는 것이 좋다.

###
### (5) 해시코드를 지연 초기화하는 hashCode 메서드
- 생성자가 아니라 처음 생성될 때 hashCode 계산
- 불변 인스턴스 변수를 활용해서, 재사용을 할 수 있다.(캐싱)
- 스레드 안정성까지 고려헤야 한다. (여러개의 쓰레드가 접근할 수 있음)

```java
  @Override
  public int hashCode() {
    if (this.hashCode != 0) {
      return hashCode;
    }

    synchronized (this) {
      int result = hashCode;
      if (result == 0) {
        
        // 특정 필드를 해시값으로 변환
        result = Short.hashCode(areaCode);
        
        // 소수 31 = 홀수를 써야함,해시 충돌이 가장 적음
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        this.hashCode = result;
      }
      return result;
    }
  }
```

###
## 2. 추가 지식
- p68, 연결 리스트 
- p70, `해시 충돌`이 더욱 적은 방법을 꼭 써야 한다면... 
- p71, 클래스를 `스레드 안전`하게 만들도록 신경 써야 한다.


### (1) 해시맵 내부의 연결 리스트
- 자바 8에서 해시 충돌시 성능 개선을 위해 내부적으로 동일한 버켓에 일정 개수 이상의 엔트리가 추가되면, 연결 리스트 대신 이진 트리를 사용하도록 바뀌었다.
- https://dzone.com/articles/hashmap-performance
- 연결 리스트에서 어떤 값을 찾는데 걸리는 시간은?
- 이진 트리에서 어떤 값을 찾는데 걸리는 시간은?

### (2) 스레드 안전
- 가장 안전한 방법은 여러 스레드 간에 공유하는 데이터가 없는 것!
- 공유하는 데이터가 있다면:
  - Synchronization
  - ThreadLocal
  - 불변 객체 사용
  - Synchronized 데이터 사용
  - Concurrent 데이터 사용
  - ...