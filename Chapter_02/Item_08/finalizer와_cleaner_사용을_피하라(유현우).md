# item 8 - finalizer와 cleaner 사용을 피하라
**finalizer와 cleaner를 유용하게 사용할 일은 극히 드물다.**

- 자원의 소유자가 File, DB같은 비메모리 자원에 접근했을 때 close 메서드를 호출하지 않는 것을 대비한 안전망 역활을 하기 위해서 사용할 수도 있다.
- 네이티브 자원은 자바 객체가 아니라서 GC가 처리할 수 없는데 이 때 사용할 수도 있다.

💬 **finalizer와 cleaner**는 GC를 강제로 호출할 수 없기 때문에 문제가 있다. 

- 사용하기 전에 그럴만한 가치가 있는지 성능 저하를 감당할 수 있는지 고민해봐야 한다.

## 단점 1. GC알고리즘에 따라서 실행될 수도 안될 수도 있다.

### 📌 Finalizer 사용하기

main 메서드가 종료되기전에 `GC`가` `FinalizerTest` 객체를 회수하면서 `finalize()`메서드를 호출해야 하는데 GC가 회수하지 않아서 호출되지 않고 있다.

```java
public class FinalizerTest {

    @Override
    protected void finalize() throws Throwable {
        System.out.println("GC야 호출 해줄거야?");
    }

    public static void main(String[] args) throws InterruptedException {
        new FinalizerTest();
        Thread.sleep(1000);
    }
}
```

GC에게 힌트를 줄 수 있는 `System.gc()`,  메서드를 호출하더라도 [JVM 구현 및 시스템 조건에 따라서 회수를 할 수도 하지 않을 수도 있다.](https://www.baeldung.com/java-finalize) `System.*runFinalization*()`도 마찬가지다.

```java
public static void main(String[] args) throws InterruptedException {
        new FinalizerTest();
        System.gc();
        Thread.sleep(1000);
    }
```

- GC가 finalize, cleaner를 실행시키는 우선순위가 다른 객체들을 회수하는 것보다 낮아서 계속 밀릴 수 있기 때문이다.

**즉 접근할 수 없는 일부 객체에 딸린 종료 작업을 전혀 수행하지 못한 채 프로그램이 중단될 수 있다❗️**

- 이런 문제들로 인해서 Deprecated 되었다.

**java.lang.Object**

```java
@Deprecated(since="9")
protected void finalize() throws Throwable { }
```

- java9에서 Deprecate되었으며 주석에도 **`java.lang.ref.Cleaner`와 `java.lang.ref.PantomReference`**를 사용하라고 작성되어 있다.

### 📌 Cleaner를 안전망으로 사용하기

State 인스턴스가 Room 인스턴스를 참조할 경우 순환참조가 발생하고 가비지 컬렉터가 Room을 회수해갈 기회가 오지 않는다. 

- State가 static인 이유도 바깥 객체를 참조하지 않기 위해서이다.

```java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    // Room을 참조하지 말것!!! 순환 참조
    private static class State implements Runnable {
        int numJunkPiles;

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        @Override
        public void run() {  // colse가 호출되거나, GC가 Room을 수거해갈 때 run() 호출
            System.out.println("방 청소");
            numJunkPiles = 0;
        }
    }

    private final State state;
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override
    public void close() {
        cleanable.clean();
    }
}
```

`cleaner`는 내부에서 `PhantomReference` 가 사용되는데 `register` 메서드로 등록해줄 때 종료되면 수행될 `Runnable`를 파라미터로 받는것을 볼 수 있었다.

```java
public Cleanable register(Object obj, Runnable action) {
        Objects.requireNonNull(obj, "obj");
        Objects.requireNonNull(action, "action");
        return new CleanerImpl.PhantomCleanableRef(obj, this, action);
}
```

`cleaner` 역시 GC가 수거할 수도 있고 그렇지 않을 수도 있다.

```java
public static void main(final String[] args) {
        new Room(8);
        System.gc();
        System.out.println("방 쓰레기 생성~~");
}
```

### 단점 2. 성능 문제

System.GC는 실제 회수할지 모르지만 이번에 확인했을 때는 수거하는 것을 볼 수 있었다.

```java
class MyFinalizerWithClose implements AutoCloseable {
    @Override
    public void close() {
        System.out.println("GC야 close 호출?");
    }
}
class MyFinalizer {
    @Override
    protected void finalize() {
        System.out.println("GC야 호출 해줄거야?");
    }
}
```

main 메서드에서 finalizerTest, finalizerWithCloseableTest 각각 따로 수행

```java
public class Main {
    public static void main(String[] args) {
        Main main = new Main();
//        long start1 = System.nanoTime();
//        main.finalizerTest();
//        System.gc();
//        System.out.println(System.nanoTime() - start1); //4083666

        long start2 = System.nanoTime();
        main.finalizerWithCloseableTest();
        System.out.println(System.nanoTime() - start2); //333625
    }
    public void finalizerTest() {
        MyFinalizer test = new MyFinalizer();
    }
    public void finalizerWithCloseableTest() {
        try (MyFinalizerWithClose finalizer = new MyFinalizerWithClose()) {
        }
    }
}
```

System.gc();를 호출하는 것에서 테스트가 올바르지 않지만 try-with-resource를 사용하는게 성능의 장점도 있고 자원을 닫는것도 명확하다는 점을 느낄 수 있었다.

**저자가 작성한 테스트 결과**

- `AutoCloseable` 객체를 생성하고 `try-with-resources`로 자원을 닫아서 가비지컬렉터가 수거하기까지 12ns가 걸렸다면 `finalizer`를 사용한 객체를 생성하고 파괴하니 550ns가 걸렸다. (50배)
- `finalizer`가 가비지 컬렉터의 효율을 떨어지게 한다. 잠시 후 알아볼 안전망 형태로만 사용하면 66ns가 걸린다. 안전망의 대가로 50배에서 5배로 성능차이를 낼 수 있다.

### 단점 3. Finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다.

**간단한 싱글톤**

```java
class MySingleton {
    private String secret;

    public void setSecret(String secret) {
        this.secret = secret;
    }

    public String getSecret() {
        return secret;
    }
}

class MySingletonFactory {
    private static MySingleton INSTANCE = new MySingleton();

    public static MySingleton getINSTANCE() {
        return INSTANCE;
    }
}
```

상속을 통해서 부모 생성자에 접근하는데 접근하지 못하고 실패하게 된다.

- GC가 자식 객체를 수거하면서 finalize()메서드를 호출하는데 부모 메서드를 호출하는 문제가 발생한다.

```java
@Getter
//@Component
public class MySecret {

    //@Value로 환경설정에서 셋팅했다고 가정
    private String accessKey = "wuqhdqwdbnjklqwd";
    //@Value로 환경설정에서 셋팅했다고 가정
    private String secret = "PASSWORD";

    public MySecret(String accessKey) {
        if (!this.accessKey.equals(accessKey)) {
            throw new IllegalArgumentException("접근할 수 없습니다.");
        }
        this.accessKey = accessKey;
    }
}

class BrokenSecret extends MySecret {
    public BrokenSecret(String accessKey) {
        super(accessKey);
    }

    @Override
    protected void finalize() throws Throwable {
        String secret = super.getSecret();
        MySingletonFactory.getINSTANCE().setSecret(secret);
    }

    public static void main(String[] args) {
        MySingleton instance = MySingletonFactory.getINSTANCE();
        while (instance.getSecret() == null) {
            try {
                new BrokenSecret("아 몰랑~~");
            } catch (Exception e) {
                //예외 삼키기
            }
        }
        System.out.println(instance.getSecret()); //패스워드 출력
    }
}
```

### 😛 상속으로 인해서 생긴 문제라서 간단하게 final 키워드로 방어할 수 있다.

**상속 막기**

```java
public final class MySecret { ... } 
```

- private 생성자를 만드는 방법도 있다.

**finalize 메서드 부모 클래스에서 override 막아버리기**

```java
public final class MySecret { 
		@Override
    protected final void finalize() throws Throwable {
    }
}
```

### ❗️결론

finalizer(Deprecated)는 사용하지 말고 Cleaner를 사용한다면 안전망 역활이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자.

- 불확실성과 성능 저하에 주의해야 한다.
- 자원의 소유자가 try-with-resource를 사용하는게 좋다.

**📚 Reference**

- https://luckydavekim.github.io/development/back-end/java/phantom-reference-in-java/
- https://pro-dev.tistory.com/111
- https://www.baeldung.com/java-finalize
- [https://inpa.tistory.com/entry/JAVA-☕-가비지-컬렉션-GC-튜닝-맛보기](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98-GC-%ED%8A%9C%EB%8B%9D-%EB%A7%9B%EB%B3%B4%EA%B8%B0)
