# 3️⃣ Item3: private 생성자나 열거 타입으로 싱글톤임을 보증하라

<br>

## 🧍 싱글톤이란?
싱글톤은 `인스턴스를 오직 하나만 생성할 수 있는 클래스`다.

### 🔎 **싱글톤의 예시**
- 함수와 같은 **무상태 객체**

- **설계상 유일**해야 하는 시스템 컴포넌트

싱글톤 클래스는 테스트하기 어렵다. 

왜냐하면 싱글톤 인스턴스를 mock = 가짜 구현으로 대체할 수 없기 때문이다. 

### 🔨 싱글톤 클래스 만드는 방법 3가지 

보통 방식 1, 2를 많이 사용한다. 

방식 1, 2는 아래와 같은 공통점이 있다. 

1. 생성자는 private

2. 유일하게 인스턴스에 접근할 수 있는 수단인 public static 멤버 하나 있음

이제 각 방식들에 대해서 보자. 

<br>

- **`무상태 객체란?`**
    
      메소드 호출에 따라 상태가 변하지 않고 주어진 입력에 대해 일관된 출력 생성하는 것이다. 
    
      무상태 객체는 멤버 변수를 가지지 않거나 멤버 변수를 가지더라도 그 값이 변경되지 않는 것이다. 
    
      아래와 같은 객체가 무상태 객체다. 
    
    ```java
    public class car{
    	void Car(){
    		System.out.println("hi");
    	}
    }
    ```
    
    ```java
    public class car{
    	static final String CAR_MESSAGE = "hi car";
    	void Car(){
    		System.out.println(CAR_MESSAGE);
    	}
    }
    ```
    
<br>

## 1️⃣ 싱글톤 만드는 방식 1. public static 멤버가 final
이 방식으로 만든 싱글톤은 아래와 같다.

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() { }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }
}
```

`public static final 필드로 객체 필드를 가지고 있는 것`이다. 

private 생성자는 public static final 필드를 초기화 할 때 한번만 호출되는 것이다. 

public, protected 생성자가 없으니 `클래스 초기화 할 때 만들어진 필드로 가지는 객체가 전체 시스템에서 하나뿐임을 보장`한다. 

그런데 예외가 있는데 권한 있는 클라이언트가 리플렉션 API인 AccessibleObject.setAccessible 사용해서 private 생성자 호출할 수 있다. 

이러면 객체가 또 생성될 수 있다. 

이런 공격은 private 생성자를 수정해서 `두번째 객체 생성되려고 할 때 예외 던지면 된다. `

<br>

## 2️⃣ 싱글톤 만드는 방식 2. 정적 팩토리 메소드를 public static 멤버로 제공
이 방식으로 만든 싱글톤은 아래와 같다.

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }
}
```

위의 싱글톤 클래스에서 `정적 팩토리 메소드인 getInstance는 항상 같은 객체의 참조 반환`해서 다른 인스턴스가 만들어지지 않는다. 

방식 1과 다른 점은 `필드가 private static final`이다. 

필드가 private static final이고 생성자도 private이니 `객체를 받을 수 있는 방법은 getInstance 메소드 뿐이다.`

이 방식에도 역시 리플렉션 API 사용해서 private 생성자 호출할 수 있으니 private 생성자에 예외 처리하면 된다. 

<br>

## 3️⃣ 싱글톤 만드는 방식 3. 원소가 하나인 Enum 타입 선언
이 방식으로 만든 싱글톤은 아래와 같다.

```java
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {
        System.out.println("기다려 자기야, 지금 나갈께!");
    }
}
```

방식 1과 비슷하지만 더 간결하다. 

그리고 `추가 노력 없이 직렬화` 할 수 있다. 

그리고 복잡한 직렬화 상황과 리플렉션 공격에도 다른 객체가 생기는 일을 완벽히 막아준다. 

그래서 대부분의 상황에서 이 방식이 싱글톤 만드는 가장 좋은 방법이다. 

다만 만들려는 `싱글톤이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.`

<br>

- **`직렬화란?`**
    
      자바 직렬화란 자바 시스템 내부에서 사용되는 객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 바이트(byte) 형태로 데이터 변환하는 기술과 바이트로 변환된 데이터를 다시 객체로 변환하는 기술(역직렬화)을 아울러서 말한다.
    
      시스템 적으로는 JVM(Java Virtual Machine)의 메모리에 상주(heap 또는 stack) 되어 있는 객체 데이터를 바이트 형태로 변환하는 기술과 직렬화된 바이트 형태의 데이터를 객체로 변환해서 JVM으로 상주시키는 형태를 말한다.
      
      자바 직렬화를 하기 위해서 type이 primitive type(기본형 타입)이거나 java.io.Serializable 인터페이스를 상속 받아야 한다.

<br>

## 👍1️⃣ 방식 1의 장점
1. 해당 클래스가 싱글톤임이 `API에 명백히 드러난다.`

2. `간결하다.`

<br>

## 👍2️⃣ 방식 2의 장점
1. `API를 바꾸지 않고` 싱글톤이 아니게 `변경할 수 있다.` 

2. 원한다면 정적 팩토리 메소드를 `제네릭 싱글톤 팩토리로 만들 수 있다.`

3. 정적 팩토리 메소드의 참조를 `공급자(supplier)로 사용할 수 있다.` 

이런 장점들이 필요하지 않다면 방식 1번이 좋다.

<br>

- **`공급자(supplier)란?`**
    
    Supplier<T>의 내부 코드는 아래와 같다. 
    
    ```java
    public interface Supplier<T>{
    	T get();
    }
    ```
    
    Supplier는 매개변수를 받지 않고 단순히 반환하는 추상 메소드가 있는 것이다. 
    
    그래서 아래와 같이 사용하면 객체를 return 할 수 있다. 
    
    ```java
    Supplier<Elvis> elvis = () -> new Elvis();
    
    elvis.get();
    ```

<br>

## 📀 싱글톤 클래스 직렬화
방식 1, 2로 만든 싱글톤 클래스를 직렬화 하려면 `Serializable을 구현한다고 선언만 하면 안된다.`

모든 `인스턴스 필드에 일시적(transient)라 선언`하고 `readResolve 메소드를 제공`해야 한다. 

이렇게 하지 않으면 `역직렬화할 때마다 새로운 인스턴스가 만들어져`서 싱글톤이 아니게 된다. 

싱글톤임을 보장하려면 readResolve 메소드를 제공해야 한다. 

아래 코드가 readResolve 메소드이고 여기서 INSTANCE는 위의 싱글톤 클래스에서의 인스턴스 필드다. 

```java
public Object readResolve(){
	return INSTANCE;
}
```

### ❓ Why?
왜 이렇게 해야 하는가 하면 `역직렬화 할 때 readObject()라는 메소드가 자동으로 호출`이 된다. 

readObject() 메소드는 `항상 새로 생성된 인스턴스를 반환`해서 싱글톤이 아니게 된다. 

readResolve 메소드를 통해서 최초에 생성된 인스턴스를 전달하면 readObject 메소드에 의해 `생성된 인스턴스는 최초에 생성된 인스턴스로 대체`된다. 

그리고 싱글톤 클래스에서는 readObject를 통해 새로 생성되는 인스턴스가 필요 없으니 `인스턴스 필드도 직렬화할 필요가 없는 것`이다. 

따라서 모든 인스턴스 필드를 transient로 선언해야 하는 것이다. 

<br>

- **`일시적(transient)란?`**
    
      직렬화하는 대상에서 제외하고 싶은 항목에 사용하는 것이다. 
    
      직렬화 후 역직렬화 하게 되면 제외된 항목에는 null값이 들어간다. 
    
      아래와 같이 사용하는 것이다. 
    
    ```java
    public class Information implements Serializable{
    	private String id;
    	private transient String password;
    	...
    }
    ```
<br>

## 📍 references

- [무상태 객체](https://kyeoneee.tistory.com/54)

- [Supplier](https://hwan33.tistory.com/17)

- [직렬화 1](https://www.happykoo.net/@happykoo/posts/257)

- [직렬화 2](https://madplay.github.io/post/what-is-readresolve-method-and-writereplace-method)

- [직렬화 3](https://madplay.github.io/post/java-serialization)

- [직렬화 4](https://brunch.co.kr/@oemilk/183)