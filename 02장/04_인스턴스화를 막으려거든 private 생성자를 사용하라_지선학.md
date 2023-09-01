정적 메서드와 정적 필드만을 담은 클래스
1) java.lang,Math, java.util.Arrays과 같이 기본 타입 값이나 배열 관련 메서드들을 모아놓음
2) java.util.Collections처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드를 모아놓을 수 있음
3) final 클래스와 관련된 메서드들

추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.

private 생성자를 추가하면 클래스의 인스턴스활르 막을 수 있음

### 인스턴스를 만들 수 없는 유틸리티 클래스
```java
public class UtilityClass {
    
    //기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용)
    private UtilityClass(){
        throw new AssertionError();
    }
}
```
- 위 코드는 어떤 환경에서도 클래스가 인스턴스화되는 것을 막아줌
- 상속을 불가능하게 하는 효과도 존재함
- 하위 클래스가 상위 클래스의 생성자에 접근할 길이 막힘
