## 🤔 정적 메서드와 정적 필드만을 담은 클래스(유틸리티 클래스)는 언제 만드는 걸까? 
-	어플리케이션에서 **전역으로 쓰여질** 변수, 메서드를 모아놓을때
-	상태가 없이 행위만 가질때
-	final 클래스와 관련된 메서드를 만들고 모아두고 싶을때
	-	final 클래스는 상속과 오버라이딩 금지
	-	상속한 후 하위 클래스에 메서드를 넣는 것이 불가능
	- final 클래스 예시 : String
   	<img width="552" alt="String" src="https://github.com/kim0527/goorm_Clone_Coding/assets/143387515/5985fe03-ef40-4cf8-9cb5-45cffbbd5080">


### 유릴리티 클래스 예시
-	```javajava.lang.Math``` : 계산 관련 메서드 및 상수의 집합
-	```java util.Arrays``` : 배열 관련 메서드의 집합
-	```java.util.Collections``` : 생성, 정렬, 병합, 검색등의 객체 관련 메서드의 집합
	
### 유틸리티 클래스 제작 의도
- 인스턴스화 없이 적은 코드로 편리하게 사용하기 위해서

유틸리티 클래스는 인스턴스화 없이 쓰려고 설계한 것이기 때문에 **생성자가 필요가 없다.**<br>
하지만 생성자를 명시하지 않는다면 컴파일러가 자동으로 public 생성자를 만들어 사용자를 헷갈리게 할 수 있다. 

***
## 🙅‍♂️ 추상 클래스로도 인스턴스화를 막을 수 없다.
```java
abstract class AbstractClass{
	public AbstractClass() {}
	public static Method_A(){...}
    ...
}

class SubClass extends AbstractClass{
	...
}

public static void main(String[] args){
	// 추상클래스를 직접 인스턴스화는 불가능
	// AbstractClass abstractClass = new AbstractClass();
    
	// 추상클래스를 상속받은 하위 클래스는 인스턴스화 가능
	SubClass subClass = new SubClass();
    subClass.Method_A()
}
```

위 코드처럼 추상 클래스를 바로 인스턴스화하는 것을 불가능하지만 **추상클래스를 상속받은 하위 클래스는 인스턴스화가 가능하다.**

그렇게 되면 제작 의도와 달리 사용자 입장에서는 다르게 해석될 수 있다.
> 🧑‍💻 사용자: " Method_A를 사용할수 있네? 상속 후에 인스턴스화해서 사용하라는 뜻이구나 "

***
## 🎯 인스턴스화를 막는 방법: private 생성자를 만들기

```java
public class UtilityClass{
	//private 생성자
	private UtilityClass();
    ...
}
```
private 생성자로 인해 다음과 같은 효과를 얻을 수 있다.

1. 클래스 바깥에서 접근 불가
2. 상속 불가능


실제로 위에서 말한 유틸리티 클래스에서도 private 생성자를 통해 인스턴스화를 막고 있다. <br>

<img width="700" alt="String" src="https://github.com/kim0527/goorm_Clone_Coding/assets/143387515/60410a87-e5cc-4c18-afb7-147e6670a149"><br>
<img width="700" alt="String" src="https://github.com/kim0527/goorm_Clone_Coding/assets/143387515/aa3229a2-5734-4f0a-abf6-57ab10e64877"><br>
<img width="700" alt="String" src="https://github.com/kim0527/goorm_Clone_Coding/assets/143387515/05be1305-1c94-4a3f-82fe-45d0ff75c58b">

#### 📝 Reference
[Collection vs Collections](https://velog.io/@sw_smj/Java-Collections-클래스)<br>
[static과 Utility Class](https://velog.io/@devrunner21/Static%EA%B3%BC-Utility-Class)<br>
[추상클래스](https://velog.io/@gillog/Java-Abstract-Class추상-클래스)<br>

