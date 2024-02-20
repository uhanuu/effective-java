### 배경

**애플리케이션을 만들다보면 어떤 인스턴스가 애플리케이션에서 하나만 있어야 하는 경우가 있다.**

- 객체의 인스턴스가 오직 1개만 생성되는 디자인 패턴인 싱글턴 패턴을 이용할 수 있다.
  [spring의 싱글턴](https://mangkyu.tistory.com/151)를 참고하셔도 좋을거 같아요

## 싱글톤을 만드는 방법

## **👻 private 생성자 + public static final 필드**

```java
public class MyInstance {
    public static MyInstance INSTANCE = new MyInstance();
    private MyInstance() {
    }
    ...
}
```

**MyInstance** 클래스를 사용하는 클라이언트 코드는 **간결하게 인스턴스를 사용**할 수 있다.

```java
public static void main(String[] args) {
	MyInstance instance = MyInstance.INSTANCE;
}
```

private 생성자는 **public static MyInstance INSTANCE = new MyInstance();** 필드를 초기화할 때 단 한번만 호출된다.

- public 이나 protected 생성자가 없기 때문에 내 애플리케이션에서 하나의 인스턴스만 존재하는것이 보장된다.

#### 장점

- 해당 클래스가 싱글턴임이 API에 명백히 드러난다.
- 간결하다.

### 단점 1.**😮‍💨 싱글톤을 사용하는 클라이언트 코드를 테스트하기 어려워진다.**

**MyService를 사용하는 Client는 MyService를 직접 사용하고 있기 때문에 테스트하기 어려워진다.**

```java
class MyService {
    public static final MyService INSTANCE = new MyService();
    private MyService() {
    }
    public String action() {
        return "External service operations";
    }
}

class Client {
    private final MyService myService;

    public Client(MyService myService) {
        this.myService = myService;
    }

    public String action() {
        return myService.action();
    }
}
```

> 테스트

```java
@Test
public void clientTest() {
	Client client = new Client(MyService.INSTANCE);
	assertEquals(client.action(), "External service operations");
}
```

- 코드는 간단해 보이지만 **MyService** 클래스가 **외부 API를 호출**하는 경우나 **연산이 오래걸리는 작업**을 테스트 코드를 통해 호출하는건 좋지 않다.

### **🙌🏻 인터페이스를 사용하자!**

**💬 Mock 객체를 사용해도 좋지만 익명 클래스를 활용해서 테스트시 Spring Boot를 올리지 않고 사용할 수 있을거 같다.**

```java
interface Service {
    String action();
}

class MyService implements Service{
    public static final MyService INSTANCE = new MyService();
    private MyService() {
    }
    @Override
    public String action() {
        return "External service operations";
    }
}
class Client {
    private final Service myService;

    public Client(Service myService) {
        this.myService = myService;
    }

    public String action() {
        return myService.action();
    }
}
```

> 테스트

```java
@Test
public void clientTest() {
	Service testService = new Service() {
		@Override
		public String action() {
			return "test code";
		}
	};

	Client client = new Client(testService);
	assertEquals(client.action(), "test code");
}
```

**람다로 익명 함수 대체하기**

```java
@Test
public void clientTest() {
    Service testService = () -> "test code";

    Client client = new Client(testService);
    assertEquals(client.action(), "test code");
}
```

인터페이스에 메서드가 한개인 경우는 람다를 활용할 수 있지만 웬만하면 힘들거 같고 공통적으로 구성할 수 있는 환경은 통합을 해서 Mock 객체를 사용하는 것도 좋아보인다. 아니면 Test 전용 Class를 만들어서 Service를 상속받아서 구현할 수 있을거 같다.

### 단점 2.클라이언트에서 사용하지 않더라도 인스턴스가 항상 생성되기 때문에 메모리가 낭비된다.

### **❗️ 주의점**

### **1️⃣ 클라이언트에서 리플렉션을 이용하면 private 생성자 호출이 가능하다.**

**리플렉션 API인 AccessibleObject.setAccessible 을 이용해서 private 생성자 호출이 가능하다.**

- 필드에서 최초의 인스턴스 생성 이후 생성자에 접근하면 예외가 발생하도록 방어할 수 있다.

```java
public class MyInstance {
    private static MyInstance INSTANCE = new MyInstance();
    *// 기본값 false*private static boolean isCreated;

    public static MyInstance getInstance() {
        return INSTANCE;
    }

    private MyInstance() {
    *// 방어 코드*if (isCreated) {
            throw new IllegalStateException();
        }
        isCreated = true;
    }
}

class Client {
    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        *// private static ConstructTest INSTANCE = new ConstructTest(); 필드를 초기화하면서 private 기본 생성자 호출*
        MyInstance.getInstance();

        Constructor<MyInstance> constructor = MyInstance.class.getDeclaredConstructor();
        *// private 접근*
        constructor.setAccessible(true);
        *// 리플렉션을 통해서 private 생성자 호출하면서 예외 발생*
        MyInstance myConstruct = constructor.newInstance();
    }
}
```

- 코드가 간결해지는 장점이 사라진다.

### 2️⃣ 역직렬화 할 때 새로운 인스턴스가 생길 수 있다.

[여기서](https://devlog-wjdrbs96.tistory.com/268) 직렬화, 역직렬화에 대해서 잘 설명해준다.

역직렬화시 **새로운 인스턴스**가 나오기 때문에 **readResolve 메서드를 선언**해서 **기존에 사용하던 인스턴스를 리턴**하도록 하면 된다.

```java
class MyService implements Service, Serializable {
    public static final MyService INSTANCE = new MyService();
    private MyService() {
    }
    @Override
    public String action() {
        return "External service operations";
    }

    *// 문법적으로 오버라이딩은 아니지만 역직렬화시 해당 메서드가 사용이 된다.*private Object readResolve() {
        return INSTANCE;
    }
}
```

- 주의해야 하는 점들을 보완하면 싱글턴의 간결한 코드라는 장점이 사라지게 된다.

### **💬 @Override가 아닌 문법을 왜 제공하는지 생각해봤다.**

**@Override를 사용해야 한다면 readResolve가 public으로 열려서 외부로 공개된다.**

- 클라이언트 쪽에서 readResolve를 호출해서 객체를 반환할 수 있으며 캡슐화가 깨지게 된다.
- 코드에 일관성이 깨지는 문제를 막아준게 아닌가 싶다.

## **👻 정적 팩터리 방식의 싱글턴**

**private 생성자 + public static final 필드 방식의 주의할 점들이 동일하게 적용된다.**

```java
public class MyInstance {
    private static MyInstance INSTANCE = new MyInstance();
    public static MyInstance getInstance() {
        return INSTANCE;
    }

    private MyInstance() { ... }
}
```

### **장점 1. API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.**

```java
public class MyInstance {
    public static MyInstance getInstance() {
        return new MyInstance();
    }
    private MyInstance() {}
}

class Client {
    public static void main(String[] args) {
        MyInstance instance = MyInstance.getInstance();
    }
}
```

### **장점 2. 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.**

**제네릭한 타입으로 동일한 싱글턴 인스턴스를 사용하고 싶을 때**

- 제네릭 타입이 다르기 때문에 equals로 비교해야 한다.

```java
public class MyInstance<T> {
    private static MyInstance<?> INSTANCE = new MyInstance<>();
    public static <T>MyInstance<T> getInstance() {
        return (MyInstance<T>) INSTANCE;
    }
    public void printItem(T item) {
        System.out.println(item);
    }
    private MyInstance() {}
}

class Client {
    public static void main(String[] args) {
        MyInstance<String> instance1 = MyInstance.getInstance();
        MyInstance<Integer> instance2 = MyInstance.getInstance();

// System.out.println(instance1 == instance2); 타입 미스 매치
        System.out.println(instance1.equals(instance2));
    }
}
```

- 인스턴스는 동일하지만 원하는 타입으로 형변환을 해줄 수 있다는 장점이 있다.

### **장점 3. 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다.**

- **java가 제공하는 함수형 인터페이스를 사용하면 된다.**

**인터페이스와 정적 팩터리 메서드로 만들어주자**

```java
public interface Singer {
    void sing();
}

class HyoshinPark implements Singer {
    private static final HyoshinPark INSTANCE = new HyoshinPark();

    public static HyoshinPark getINSTANCE() {
        return INSTANCE;
    }

    private HyoshinPark() {
    }

    @Override
    public void sing() {
        System.out.println("좋은 사람 사랑했었다면~~~ 헤어져도 슬픈 게 아니야아우어워~");
    }
}
```

`Supplier<Singer>` 를 통해서 유연하게 사용할 수 있다.

```java
class Concert {
    void start(Supplier<Singer> singerSupplier) {
        Singer singer = singerSupplier.get();
        singer.sing();
    }
    public static void main(String[] args) {
        Concert concert = new Concert();
        concert.start(HyoshinPark::getINSTANCE);
    }
}
```

**💬 이런 장점들이 필요없으면 간단하게 private 생성자 + public static final 필드 방식을 사용해도 좋을거 같다.**

## **👻 열거 타입**

**Enum 역시 class이기** 때문에 **상속은 불가능** 하지만 **인터페이스를 구현할 수 있습니다.**

```java
public interface Member {
    String speak();
}

enum Hyunwoo implements Member {
    INSTANCE;
    @Override
    public String speak() {
        return "메롱";
    }
}
```

**클라이언트에서 간단하게 사용할 수 있습니다.**

```java
class Client {
    public static void main(String[] args) {
        Hyunwoo hw = Hyunwoo.INSTANCE;
        System.out.println(hw.speak());
    }
}
```

**Enum 은 리플렉션을 내부코드로 막아 놓았기 때문에 생성자를 불러오려고 하면 에러가 발생한다.**

> Enum은 인터페이스를 구현할 수 있기 때문에 테스트 코드도 작성할 수 있고 생성자가 없기 때문에 리플렉션에 안전하고 역직렬화에도 안전하다.!
