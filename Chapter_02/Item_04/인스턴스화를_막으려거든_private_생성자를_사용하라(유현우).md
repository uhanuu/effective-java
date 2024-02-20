### 배경

**단순히 정적 메서드와 정적 필드만 담은 유틸리티성 클래스는 인스턴스를 만들지 않는 걸 권장하는 경우가 있다.**

- 객체 지향적인 사고로 봤을 때 좋은 방법이 아니다.

## **1. 기본 타입 값이나 배열 관련 메소드를 모은 클래스**

- `java.lang.Math` 와 `java.util.Arrays`

```java
public class Arrays {
		// Suppresses default constructor, ensuring non-instantiability.
    private Arrays() {}
		public static boolean isArray(Object o) { ...}
    public static Object[] asObjectArray(Object array) { ...}
    public static List<Object> asList(Object array) { ...}
    public static <T> boolean isNullOrEmpty(T[] array) { ...}
	...
}

public final class Math {

    /**
     * Don't let anyone instantiate this class.
     */
		private Math() {}
		...
}
```

- 내부 메소드들이 전부 static으로 선언되어 있고 생성자가 private로 선언된 것을 확인할 수 있다.

### ❗️정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한게 아니다.

- 생성자를 명시하지 않으면 컴파일러는 매개변수를 받지 않는 public 생성자를 만들기 때문에 private으로 막아준 것을 볼 수 있다.
- 생성자가 분명 존재하는데 호출할 수 없다는 것은 직관적이지 않기 때문에 주석을 달아준 것도 확인할 수 있었다.

## 2. 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메소드(or 팩토리)를 모은 클래스

`java.util.Collections`

```java
public class Collections {
    // Suppresses default constructor, ensuring non-instantiability.
    private Collections() {
    }
		...
}

```

- static으로만 구성되어 있으며 `private 생성자 + 주석`이 존재하는 것을 확인할 수 있다.

## 3. final 클래스와 관련된 메소드들을 모아놓을 때

**final 클래스는 보안 상의 이유로 상속을 금지 시킨다.**

- 하위 클래스를 만드는 것이 시스템의 문제가 발생할 수 있는 경우에 사용할 수 있다.
- final 클래스를 상속해서 하위 클래스에 메서드를 넣는 건 불가능 하기 때문에 모아서도 사용한다.

대표적인 final class `java.lang.String`

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence,
               Constable, ConstantDesc {
		...
}
```

### ❗️추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.

`org.springframework.util.StringUtils` 를 보면 유틸리티성 클래스임에도 불구하고 생성자가 public으로 열려있다.
![image](https://github.com/uhanuu/Effective-Java/assets/110734817/ab4cfb07-930f-477c-a035-4541d2f5399a)
`추상클래스 + public 생성자` 인 경우

- 하위 클래스를 만들어서 인스턴스화가 가능하다.
- abstract 클래스는 보통 클래스들의 공통 필드와 메소드를 정의하는 목적으로 만들기 때문에 상속해서 사용하라는 의미로 오해할 수 있다.
- 확장해서 사용하는 용도가 맞다면 주석을 넣어주거나 private 생성자 + 주석을 사용하자.

### 인스턴스를 만들 수 없는 유틸리티 클래스

`java.util.Objects` 를 보면 `private + error` 를 이용한 것을 볼 수 있다.

- 예외 메시지를 보면 인스턴스를 생성하지 말라고 주석에 역활까지 하는것을 볼 수 있다.

```java
public final class Objects {
    private Objects() {
        throw new AssertionError("No java.util.Objects instances for you!");
    }
		...
}
```

- Error를 던질 필요는 없지만 클래스 안에서 실수라도 생성자를 호출하지 않도록 도와준다.

**인스턴스화를 막는 것을 잊지말고 private 생성자를 사용해서 문서화를 하도록 하자!**
