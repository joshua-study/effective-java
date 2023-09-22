# 아이템 10. equals는 일반 규약을 지켜 재정의하라

## equals를 재정의 하지 않는 것이 최선인 경우
1. 각 인스턴스가 본질적으로 고유할 때</br>
값(Integer나 String을 표현하는 클래스)이 아닌 동작하는 개체를 표현하는 클래스 ex) Thread
2. 인스턴스의 '논리적 동치성'을 검사할 일이 없을 때 - 기본 equals로 가능</br>
   ex) java.util.regax.Pattern은 equals를 재정의해 두 Pattern의 정규표현식을 검사
3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는 경우</br>
   ex) Set은 AbstractSet이 구현한 equals를 상속, List는 AbstractList, Map은 AbstractMap
4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없는 경우
```java
@Override public boolean equals(Object o) {
	throw new AssertionError(); // 호출 금지!
}
```

## equals를 제정의 해야하는 경우
객체 식별성(두 객체가 물리적으로 같은가)이 아니라 논리적 동치성을 확인해야 하는데,
상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때이다.

주로 값 클래스들이 해당된다. 두 값 객체를 equals로 비교하는 경우 객체가 같은지가 아니라 값이 같은지가 궁금하다.
논리적 동치성을 확인하도록 재정의해두면, 값 비교는 물론 Map의 키와 Set의 원소로 사용 가능하다.
그러나, 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면 재정의하지 않아도 된다.</br>
ex) Enum - 논리적으로 같은 인스턴스가 2개 이상 만들어지지 않으니, 논리적 동치성 = 객체 식별성

## equals 메서드 재정의 일반 규약 : 동치관계
- 반사성(reflexivity)
: null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.</br>
객체는 자기 자신과 같아야 한다.
- 대칭성(symmetry)
: null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.</br>
두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다. equals는 대소문자를 무시한다!(대칭성 위배)
- 추이성(transitivity)
: null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고, y.equals(z)도 true면 x.equals(z)도 true다.</br>
첫 번째 객체 = 두 번째 객체, 두 번째 객체 = 세 번째 객체라면? 첫 번째 객체 = 세번째 객체 </br>
  - 리스코프 치환 원칙 : 부모 객체와 이를 상속한 자식 객체가 있을 때 부모 객체를 호출하는 동작에서 자식 객체가 부모 객체를 완전히 대체할 수 있다.</br>
  해결 1. 상속(is-a) 대신 컴포지션(has-a)을 사용하라(아이템 18)
  해결 2. 추상 클래스의 하위 클래스 사용하기
- 일관성(consistency)
: null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true이거나 false다.
두 객체가 같다면 앞으로도 영원히 같아야 한다.
- **null-아님**
: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.
모든 객체가 null과 같지 않아야 한다.

동치관계란, 집합을 서로 같은 원소들로 이뤄진 부분집합으로 나누는 연산이다. equals 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환할 수 있어야 한다.

## equals 메서드 구현 방법
1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
자기자신이면 true 반환
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
그렇지 않다면 false 반환
3. 입력을 올바른 타입으로 형변환한다.
instanceof 검사를 했다면 통과.
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.

- 기본 타입 필드는 '=='연산자로 비교
- 참조 타입 필드는 equals 메서드로 비교
- float, double 필드는 각각 정적 메서드인 Float.compare(float, float)와 Double.compare(double, double)로 비교 : 부동소수값 때문에
- null도 정상 값으로 취급하는 참조 필드인 경우는 Objects.equals(Object, Object)로 비교하여 NullPointerException 예방 필요
- CaseInsensitiveString 처럼 복잡한 필드의 클래스 경우에는 필드의 표준형끼리 비교

## 정리
꼭 필요한 경우가 아니라면 equals를 재정의하지 말자. 재정의해야한다면 다섯가지 규약을 지키자! </br>
AutoValue 프레임워크를 추가하면 equals 테스트 메서드들을 알아서 작성해줄 것임.
