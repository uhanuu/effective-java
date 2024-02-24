# 🚀 인스턴스화를 막을려거든 private 생성자를 사용하라

## ❓ 왜 인스턴스화를 막을려고 하는 걸까

OOP의 핵심은 코드의 재사용입니다.

이 목적을 위해서 우리는,
일단 어떤 틀의 형태인 클래스를 만들고,
그를 통해 인스턴스를 만들어 사용하는 방식을 사용하기로 했습니다..


그런데 간혹가다 단순히 정적 메서드와 정적 필드만을 담은 클래스를 만들고 싶을 때가 있습니다. 
또는 싱글톤 패턴을 사용하고 싶을 때가 있습니다.  

>**❗️ 싱글톤 패턴**    
>싱글톤 패턴은 객체지향 디자인 패턴의 한 종류로, 어떤 클래스의 인스턴스가 단 하나만 생성되어 시스템 어디에서든지 그 인스턴스에 접근할 수 있도록 보장하는 패턴입니다. 이 패턴은 주로 공유 자원에 대한 접근 제어, 로깅, 데이터베이스 연결, 프린터 스풀러 등과 같이 애플리케이션 전체에서 단 하나의 인스턴스만 필요한 경우에 사용됩니다.

>- 싱글톤 패턴을 사용하려는 주된 이유는 다음과 같습니다
>-  - 제어된 접근: 싱글톤 패턴은 전역 인스턴스를 제공하므로, 이를 통해 어디서든지 인스턴스에 접근할 수 있으며, 이는 특정 자원에 대한 접근을 중앙집중화하고 제어할 수 있게 합니다.
>-  - 자원의 낭비 방지: 여러 인스턴스를 생성할 필요 없이 단일 인스턴스만 유지함으로써 메모리 낭비를 방지할 수 있습니다.
>-  - 일관성 있는 자원 접근: 모든 컴포넌트가 동일한 인스턴스와 자원을 공유하기 때문에 자원의 상태 관리가 용이해집니다

여기서의 핵심은 인스턴스를 사용하도록 객체가 설계되었지만 이런 특정한 경우에는 아예 인스턴스를 사용하지 못하게 막아줘야합니다.

> **❓ 생성자를 만들지 않으면?**   
>자바에서 객체는  생성자를 만들지 않으면 자동으로 public으로 된 default 생성자를 만듭니다.

>**❓ 추상 클래스로 만든다면?**   
아래는 스프링에 있는 추상클래스중 하나입니다.
<img width="901" alt="image" src="https://github.com/Growth-Hub/Effective-Java/assets/76567238/20537046-4764-4aad-b505-17b9a35bcb6f">

> 추상클래스도 아래처럼 코드를 구현하면 인스턴스를 가지고 호출할 수 있습니다.
```java
import org.springframework.context.annotation.AnnotationConfigUtils;

public class DefaultUtilityClass extends AnnotationConfigUtils {

    public static void main(String[] args) {
        DefaultUtilityClass utilityClass = new DefaultUtilityClass();
        utilityClass.processCommonDefinitionAnnotations(null);
    }
}
```

## 🎯 private 생성자를 사용해서 인스턴스화 막기

인스턴스화를 막기위한 해결법은 정말 간단합니다.    
`private` 생성자를 사용하면 클래스의 인스턴스화를 막을 수 있습니다.

```java

public class UtilityClass {

  private UtilityClass() {
    throw new AssertionError();
  }
  ... 나머지 코드 생략
}
```

명시적 생성자가 `private`이니 클래스 바깥에서는 접근할 수 없습니다.    
꼭 `AssertionError`를 던질 필요는 없지만, 클래스 안에서 실수로라도 생성자를 호출하지 않도록 도와줍니다.    
    
이 방식은 `private`으로 선언 했으니 하위 클래스가 상위 클래스의 생성자에 접근할 길이 막혀서  상속을 불가능하게 하는 효과도 있습니다.

❓ `AssertionError`
>AssertionError는 자바 개발 언어에서 발생하는 오류입니다. 이 오류는 자바 프로그램이 예상하는 결과가 나타나지 않을 때 발생합니다. 이 오류는 런타임 오류로 인식되며, 코드에서 의도한 것과 다른 결과가 발생하는 문제를 식별하는 데 도움이 됩니다.
간단히 말해서, AssertionError는 프로그램이 "이렇게 동작해야 해"라고 우리가 가정한 규칙을 따르지 않을 때, "잠깐, 이건 맞을 수 없어!"라고 알려주는 오류라고 생각하시면 됩니다.  이 오류는 예외를 처리하기위한 에러가 아닙니다.  try-catch에서 사용하고자 하는 에러가 아니고 정말 만나면 안되는 상황의 경우에 배치하고 혹시라도 발생하면 이건 무조건 잘못된 거다라는 곳에서 사용합니다. 

<br>
<br>

### 📚 Reference

- [Java에서 assert와 exception](https://dkswnkk.tistory.com/710)
- [AssertionError (Java Platform SE 8 )](https://docs.oracle.com/javase/8/docs/api/java/lang/AssertionError.html)
- [[이펙티브 자바] 객체의 생성과 파괴 Item4 - 인스턴스화를 막으려거든 private 생성자를 사용하라](https://velog.io/@holidenty/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-%EA%B0%9D%EC%B2%B4%EC%9D%98-%EC%83%9D%EC%84%B1%EA%B3%BC-%ED%8C%8C%EA%B4%B4-Item4-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4%ED%99%94%EB%A5%BC-%EB%A7%89%EC%9C%BC%EB%A0%A4%EA%B1%B0%EB%93%A0-private-%EC%83%9D%EC%84%B1%EC%9E%90%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC)
