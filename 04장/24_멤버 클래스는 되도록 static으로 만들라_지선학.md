## 멤버 클래스는 되도록 static으로 만들라

각각의 중첩 클래스를 언제 그리고 왜 사용해야 하는지 이야기한다.

### 정적 멤버 클래스
- 다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에도 접근할 수 있다는 점만 제외하고는 일반 클래스와 똑같다. 
- 정적 멤버 클래스와 비정적 멤버 클래스의 구문상 차이는 단지 static이 부터 있고 없고 뿐이지만, 의미상 차이는 의외로 꽤 크다.
- 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다. 
- 정규화된 this란 클래스명.this 형태로 바깥 클래스의 이름을 명시하는 용법을 말한다.
- 바깥 클래스의 인스턴스 메서드에서 비정적 멤버 클래스의 생성자를 호출할 때 자동으로 만들어지는 게 보통이지만, 드물게는 직접 바깥 인스턴스의 클래스.new Member class(args)를 호출해 수동으로 만들기도 한다. 
- 이 관계 정보는 비정적 멤버 클래스의 인스턴스 안에 만들어져 메모리 공간을 차지하며, 생성 시간도 더 걸린다. 
- 비정적 멤버 클래스는 어댑터를 정의할 때 자주 쓰인다. 

* 어댑터 : 호환되지 않는 인터페이스를 가진 객체들이 협업할 수 있도록 하는 구조적 디자인 패턴입니다.
  https://refactoring.guru/ko/design-patterns/adapter

### 비정적 멤버 클래스의 흔한 쓰임 - 자신의 반복자 구현
```java
public class MySet<E> extends AbstractSet<E> {
    
    @Override public Iterator<E> iterator(){
        return new MyIterator();
    }
    
    private class MyIterator implements Iterator<E>{
        ...
    }
}
```
### 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자 
메모리 누수가 생길 수 있다

### Map 인스턴스
- 엔트리를 선언할 때 실수로 static을 빠뜨려도 맵은 여전히 동작하겠지만, 모든 엔트리가 바깥 맵으로 참조를 갖게 되어 공간과 시간을 낭비할 것이다. 
- 멤버 클래스가 공개된 클래스의 public이나 protected 멤버라면 정적이냐 아니냐는 두배로 중요해진다.
- 멤버 클래스 역시 공개 API가 되니, 혹시라도 향후 릴리스에서 static을 붙이면 하위 호환성이 깨진다. 

### 익명 클래스 
- 익명 클래스는 응용하는 데 제약이 많은 편이다. 
- 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없다.
- 익명 클래스는 표현식 중간에 등장하므로 짧지 않으면 가독성이 떨어진다.
- 자바가 람다를 지원하기 전에는 즉석에서 작은 함수 객체나 처리 객체를 만드는 데 익명 클래스를 주로 사용했다. 
- 이제는 람다에게 그 자리를 물려줬다.
- 익명 클래스의 또 다른 주 씨임은 정적 팩터리 메서드를 구현할 때다
- 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있으며, 정적 멤버를 가질 수 없으며, 가독성을 위해 짧게 작성해야 한다. 

## 정리 
- 중첩 클래스에는 네 가지가 있으며, 각각의 쓰임이 다르다. 
- 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳이고 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있다면
익명 클래스로 만들고, 그렇지 않으면 지역 클래스로 만들자.


