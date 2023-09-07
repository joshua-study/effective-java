## 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라.

객체 인스턴스화 막기
java.lang.Math, java.util.Arrays, java.util.Collections 등이 있다.

이 객체들은 인스턴스화를 막았기 때문에 아래와 같이 코드를 작성할 수 없다.
```java
Math mathObject = new Math();
Arrays arraysObject = new Arrays();
Collections collectionsObject = new Collection();
```

추상클래스를 통해 인스턴스화를 막을 수 없다.  
추상클래스는 상속을 위한 클래스이기 때문에 상속하라는 뜻으로 받아들여지기 쉽다.  
그래서 추상클래스를 통해 인스턴스화를 막더라도 단순히 하위 클래스를 만들어 인스턴스화가 가능하기 때문에 크게 의미가 없다.
```java
abstract class AbstractUtils{
...
}

class InstanceUtils extends AbstractUtils{
...
}

AbstractUtils abstractUtils = new AbstractUtils(); // X
InstanceUtils instanceUtils = new InstanceUtils(); // O

```
간단하게 public 생성자를 private 생성자로 설정해주어 막을 수 있다.
```java
public class UtilityClass{
// 기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용).
private UtilityClass(){
throw new AssertionError();
}
...
}
```
꼭 AssertionError를 발생시킬 필요는 없지만, 클래스 안에서 실수로라도 생성자를 호출하는 것을 방지해줄 수 있다.


### 마무리   
인스턴스화를 막기위해 생성자를 private으로 작성해주면 된다. 하지만 생성자가 private이기 때문에 상속이 불가능하다.
