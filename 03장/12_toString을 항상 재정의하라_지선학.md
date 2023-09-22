## toString을 항상 재정의하라

Object의 기본 toString 메서드가 우리가 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없음

toString을 잘 구현한 클래스는 사용하기에 훨씬 즐겁고, 그 클래스를 사용한 시스템은 디버깅하기 쉬음

실전에서 toString은 그 객체가 가진 주요 정보 모두를 반환하는 게 좋음

포맷을 명시하든 아니든 여러분의 의도는 명확히 밝혀야 함

```java
class sample(){
    
    @Override public String toString(){
        return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
    }
    
}
```

```java
class sample(){
    
    @Override public String toString(){
        ...
    }
    
}

```
포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자  
PhoneNumber 클래스는 지역 코드, 프리픽스, 가입자 번호용 접근자를 제공해야함  
그렇지 않으면 이 정보가 필요한 프로그래머는 toString의 반환값을 파싱할 수 밖에 없음

정적 유틸리티 클래스, 대부분의 열거 타입 이미 toString을 제공함

하지만 하위 클래스들이 공유해야 할 문자열 표현이 있는 추상 클래스라면 toString을 재정의해줘야 함  
대다수의 컬렉션 구현체는 추상 컬렉션 클래스들의 toString 메서드를 상속해 씀


구글의 AutoValue 프레임워크는 toString도 생성해줌

## 정리
모든 구체 클래스에서 Object의 toString을 재정의하자.

toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야함


