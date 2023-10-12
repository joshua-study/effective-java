# 아이템 14. Comparable을 구현할지 고려하라.

## compareTo 메서드의 일반 규약
compareTo는 단순 동치성 비교(Object.equals)에 더해 순서까지 비교할 수 있으며, Generic을 지원한다.

1. 첫 번째 객체 < 두 번째 객체 -> 두 번째 >첫 번째, (대칭성)
첫 번째 = 두 번째 -> 두 번째 = 첫 번째
첫 번째 > 두 번째 -> 두 번째 < 첫 번쩨 
2. 첫 번째 > 두 번째 > 세  -> 첫 번째 > 세 번째 (추이성)
3. 크기가 같은 객체들끼리는 어떤 객체와 비교해도 항상 같아야 한다. (반사성)
4. 네 번째 = 두 번째 -> 네 번째 > 첫번째 라면, 두 번째 > 첫 번째 (일관성)
5. compareTo가 0이라면 equals는 true이여야 한다. (안지켜질 수도 있음)
```java
BigDecimal oneZero = new BigDecimal(val:"1.0");
BigDecimal oneZeroZero = new BigDecimal(val:"1.00");
System.out.println(oneZero.compareTo(oneZeroZero)); //Tree, TreeMap
System.out.println(oneZero.equals(oneZeroZero)); //순서가 없는 콜렉션 (버젼까지 같아야 한다!)
```

## compareTo 구현 방법 1
자연적인 순서를 제공할 클래스에 implements Comparable<T>을 선언한다. </br>
compareTo 메서드를 재정의한다. </br>
compareTo 메서드 안에서 기본 타입은 박싱된 기본 타입의 compare을 사용해 비교한다. </br>
핵심 필드가 여러 개라면 비교 순서가 중요하다. 순서를 결정하는데 있어서 가장 중요한 필드를 비교하고 그 값이 0이라면 다음 필드를 비교한다.</br>
기존 클래스를 확장하고 필드를 추가하는 경우 Composition을 활용할 것





## 정리
순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여, 그 인스턴스들을 쉽게 정렬/검색/비교 기능을 제공하는 컬렉션과 어우러지도록 해야 한다.
compareTo 메서드에서 필드의 값을 비교할 때 <와> 연산자는 쓰지 말아야 한다.
그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.

