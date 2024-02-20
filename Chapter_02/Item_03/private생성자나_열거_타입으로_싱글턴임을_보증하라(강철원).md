# Item 3 private 생성자나 열거 타입으로 싱글턴임을 보증하라

>싱글톤
>오직 하나의 인스턴스만 가지는 클래스

특정 클래스의 인스턴스를 하나만 관리함으로써 중복되는 불필요한 인스턴스 생성을 막을 수 있습니다. 하지만, 이런 싱글톤 클래스들은 테스트를 할 때 어려움이 있습니다. 만약 해당 싱글톤 클래스가 인터페이스를 구현한 구현체라면 해당 인터페이스를 테스트용으로 구현하는 가짜 구현체로 대체가 가능하지만 그게 아니라면 싱글톤 인스턴스를 가짜 구현으로 대체할 수 없기 때문입니다.

## 🎯 싱글톤 생성 방식

<br>

## 🎾 1. private 생성자 + public static final 필드

```java
public class Student {
		public static final Student INSTANCE = new Student();
		private Student(){...}
		
		...
}
```
`private`생성자는 전역 불변 필드인 INSTANCE가 초기화 할 때 단 한 번 호출됩니다. 그 이후에는 접근가느한 생성자가 없기에 외부에서 해당 클래스의 인스턴스를 만들 수 없습니다.
물론, Reflection API를 이용한다면 private 생성자도 호출이 가능한데, 이를 막기 위해서는 생성자를 수정해 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 됩니다.
```java
private Student() {
    if(Objects.nonNull(INSTANCE) {
        throws new RuntimeException();
    }
}
```

이렇게 전역 상수를 통해 싱글톤을 제공하는 방식을 사용한다면 간결하고 따로 메소드 호출을 할 필요가 없으며, final 필드이기 때문에 다른 객체를 참조할수도 없습니다.

### 📌 정리

- 장점
    - 구현이 간결하고 싱글턴임을 API에 들어낼 수 있습니다.
- 단점
    - 싱글턴을 사용하는 클라이언트가 테스트하기 어려워진다.
    - 리플렉션으로 `private`생성자를 호출할 수 있다.
    - 역직렬화 할 때 새로운 인스턴스가 생길 수 있다.  

<br>

## 🎾 2. private 생성자 + 정적 팩터리 메서드

>정적 팩토리 메서드인 `getInstance`를 통해 인스턴스를 반환하며 ReflectionAPI를 제외하면 또 다른 Student 인스턴스를 만들 수 없습니다.
```java
public class Student {
		private static final Student INSTANCE = new Student();
		private Student(){...}
		
		public static Student getInstance(){ return INSTANCE; }
		...
}
```

정적 팩토리 메서드를 사용했을 때의 장점이 필요 없는 상황이라면 첫 번재 방식인 `public static`을 이용한 싱글톤 방식을 사용하면 됩니다.

❗️주의점 
>싱글톤 클래스를 직렬화 할 때는 `Serializable`을 구현한다고 선언(implements)하는 것 만으로는 부족합니다.
구현한다고 선언만 한 상태에서 직렬화 이후 다시 역직렬화(Deserialization)를 하게 되면 새로운 인스턴스가 생성되어 버립니다. 그렇기에 이 문제를 해결하기 위해서는 모든 인스턴스 필드를 일시적(transient)으로 선언한 뒤 readResolve 메소드를 오버라이딩해서 작성해줘야합니다.
```java
private Object readResolve() throws ObjectStreamException{ 
		return INSTANCE; 
}
```

###  정리

- 장점
    - API를 변경하지 않고 싱글턴이 아니게 변경할 수 있다.
    - 정적 팩터리를 제네릭 싱글턴 팩토리로 만들 수 있다.
    - 정적 팩터리의 메서드 참조를 공급자(Supplier)로 사용할 수 있다.
- 단점
    - 싱글턴을 사용하는 클라이언트가 테스트하기 어려워진다.
    - 리플렉션으로 `private`생성자를 호출할 수 있다.
    - 역질렬화 할 때 새로운 인스턴스가 생길 수 있다.  

#### 장점 1) API를 변경하지 않고 싱글턴이 아니게 변경할 수 있습니다.

>클라이언트 코드를 변경시키지 않고 생성 방식을 변형시킬 수 있다.
```java
public class MethodSingleton {
    private MethodSingleton() {}

    public static MethodSingleton getInstance() {
        return new MethodSingleton();
    }
}
```
 `MethodSingleton` 클래스는 싱글턴 패턴을 사용하여 인스턴스를 제공하지만, 생성자를 `private`으로 설정함으로써 인스턴스 생성을 제한합니다. 그럼에도 불구하고, `getInstance` 메서드를 통해 새로운 인스턴스를 반환하도록 변경할 수 있습니다. 이는 클라이언트 코드가 `MethodSingleton`의 인스턴스 생성 방식에 의존하지 않도록 하여, 나중에 싱글턴 패턴이 아닌 다른 패턴으로 변경해야 할 때 클라이언트 코드를 수정할 필요가 없게 만듭니다. 따라서, 이는 소프트웨어의 유지보수성과 확장성을 향상시킵니다.
 
 
 #### 장점 2) 정적 팩터리를 제너릭 싱글턴 팩토리로 만들 수 있습니다.
 
 Generic 을 활용하여 같은 인스턴스를 반환하지만 타입을 변환하여 사용할 수 있다.
 ```java
 public class GenericSingleton<T> {
    public static final GenericSingleton<?> INSTANCE = new GenericSingleton<>();

    private GenericSingleton() {
    }

    public static <T> GenericSingleton<T> getInstance() {
        return (GenericSingleton<T>) INSTANCE;
    }

    public boolean send( T message ) {
        // blah~ blah~
        return true;
    }
}
 ```
 
 `GenericSingleton` 클래스를 사용하는 이점은 제네릭을 통한 타입 안전성과 유연성입니다. 이 클래스는 모든 타입의 객체에 대해 단일 인스턴스를 제공할 수 있으며, `getInstance` 메서드를 통해 요청된 타입으로의 명시적 캐스팅을 허용합니다. 이는 다양한 타입의 객체를 처리해야 하는 경우에 유용하며, 타입 변환 오류를 컴파일 시간에 잡아낼 수 있어 런타임 오류의 가능성을 줄여줍니다. 또한, 이 방식은 타입에 따라 다른 인스턴스를 반환할 필요가 없이, 단일 인스턴스를 유지하면서 타입의 유연성을 제공합니다.

#### 장점 3) 정적 팩터리의 메서드 참조를 공급자(Supplier)로 사용할 수 있습니다.

>`Supplier` 를 활용하여, `lambda expression` or `method reference` 를 사용할 수 있다.
```java
public class SingletonSupplier {

    public void start( Supplier<ISingleton> supplier ) {
        ISingleton instance = supplier.get();
        instance.send( "test" );
    }

    public static void main( String[] args ) {
        SingletonSupplier singletonSupplier = new SingletonSupplier();
        singletonSupplier.start( MockSingleton::getInstance );
    }
}
```

SingletonSupplier 클래스의 접근 방식은 높은 수준의 추상화와 유연성을 제공합니다. Supplier<ISingleton> 인터페이스를 사용함으로써, 클라이언트 코드는 싱글턴 인스턴스의 생성 방식을 몰라도 됩니다. 이는 메서드 참조(MockSingleton::getInstance)를 통해 싱글턴 인스턴스를 지연 생성하거나 필요에 따라 다른 인스턴스를 제공할 수 있도록 해줍니다. 따라서, 이 방식은 인스턴스의 생성과 사용 사이의 결합도를 낮추며, 테스트 용이성을 향상시키고, 다양한 실행 컨텍스트에서의 유연한 인스턴스 관리를 가능하게 합니다.

<br>

## 🎾 3. 열거 타입 싱글톤

지금까지 살펴본 싱글톤을 사용하려면 고려해야할 부분들이 몇 가지가 있는데, 이런 부분들은 열거 타입 방식의 싱글톤으로 만들면 해결이 가능합니다.  이 세번째 방식을 사용한다면 직렬화 문제, 리픅렉션 공격들도 막을 수 있습니다.
```java
package me.catsbi.effectivejavastudy.item3;

public enum SingletonV3 implements SingletonBase{
    INSTANCE;

    @Override
    public void print() {
        System.out.println("싱글톤 V3버전 열거 타입");
    }
}
```

- 열거타입 싱클톤의 최고의 강점은 사용이 간단합니다.  
- 대부분의 상황에서는 원소가 하나뿐인 결거 타입이 싱글턴을 만드는 가장 좋은 방법입니다.


