# 📗 Item 8 finalizer와 cleaner 사용을 피하라

먼저 `finalizer`와 `cleaner`에대해 먼저 알아보겠습니다.
구체적으로 설명하기전에 동화를 통해 전체적인 맥락을 이해해보고 가겠습니다.

옛날 옛적에, 작은 마을에 '자원 회수'라는 중요한 임무를 가진 두 요정, 파이널라이저와 클리너가 있었습니다. 이 두 요정은 마을 사람들이 사용한 후 버린 자원들을 회수하여 자연으로 돌려보내는 역할을 했습니다. 그들의 노력으로 마을은 항상 깨끗하고 정돈되어 있었습니다.

그러나 시간이 흐르며 마을 사람들은 두 요정의 존재를 잊고, 자원을 낭비하기 시작했습니다. 파이널라이저 요정은 자원을 회수하는 데 시간이 많이 걸렸고, 때로는 자원을 제대로 회수하지 못하는 일도 발생했습니다. 클리너 요정도 마찬가지로 느리고 비효율적이었습니다. 더군다나, 이 두 요정은 예측할 수 없게 자원을 회수하러 왔기 때문에, 마을 사람들은 자원이 언제 사라질지 알 수 없었습니다.

이에 지혜로운 마을의 마법사가 나섰습니다. 마법사는 "우리가 직접 책임을 지고 사용한 자원을 정리하는 '자동 자원 회수' 마법을 사용하자"고 제안했습니다. 이 마법은 'try-with-resources'라고 불렸으며, 자원을 사용한 후 즉시 회수하여 자연으로 돌려보내는 방법이었습니다.

마법사의 제안에 따라, 마을 사람들은 파이널라이저와 클리너에 의존하지 않고 직접 자원을 관리하기 시작했습니다. 그 결과, 마을은 더 이상 자원 낭비로 고통받지 않게 되었고, 모든 것이 훨씬 더 효율적이고 예측 가능해졌습니다.

자 그럼 전체적인 맥락을 알았으니  `finalizer`와 `cleaner`에 대해 자세히 알아보겠습니다.
자바는 `finalizer`와 `cleaner`이라는 두 가지 객체 소멸자를 제공함니다. 이들은 GC(가비지 컬렉션)이 더 이상 사용하지 않는 자원에 대한 정리작업을 진행하기 위해 사용됩니다.

<br>

## 📌 finalizer

finalize()메서드는 모든 자바 클래스의 부모인 Object 클래스에 선언된 메소드중 하나입니다. 
객체가 가비지 컬렉터에 의해 수집될 때 실행되도록 설계되었습니다. 가비지 수집 대상이 되었을 때 애플리케이션 개발자가 의도한 기능을 수행하여 별도의 리소스 정리 작업을 할 수 있도록 했습니다.

클래스의 생성(초기화) 시점에 리소스를 획득하고 종료 시점에 리소스를 다시 반납하는 패턴을 RAII(Resource Acqusition Is Initialization) 패턴 이라고 부릅니다. C++ 에서 주로 사용되는 패턴인데, GC 가 따로 없다보니 개발자가 직접 리소스를 할당하고 해제해주어야 했고 까딱 잘못하면 메모리 누수가 나곤합니다. 그래서 위 처럼 패턴을 만들어서 고착화 시킨겁니다. 클래스의 사용 종료는 동시에 리소스의 반납을 보장하는겁니다.

### 특징

- 해당 객체를 가리키는 레퍼런스가 없을 때 가비지 컬렉터에 의해 호출된다.
- 리소스 반납이나 별도의 정리 작업을 수행해야한다.
- Object 클래스의 finalize() 는 그저 비어있는 메소드일 뿐(no-op)이라서 따로 오버라이드 하지 않을 경우 아무 동작도 수행하지 않는다.
- 어떤 스레드가 finalize() 를 호출할지는 모른다.
- finalize() 를 호출하는 스레드는 어떠한 user-visible synchronization lock 도 잡고있지 않는다.
- finalize() 내부에서 발생한 exception 은 무시된다.
- 한 객체에 대해 두 번 이상 호출되지 않는다.

### 문제점

- 객체의 수명 연장
    - 바로 위에서 설명한 것처럼 finalize() 메소드를 오버라이드한 객체들은 일반 객체들 보다 한 사이클정도 수명이 깁니다. 보통은 이 하나의 사이클이 별거 아닌 것 처럼 느껴지겠지만 tenured 세대에 있는 객체들은 이 사이클이 상당히 긴 시간이 될 수 있습니다. 이는 곧 리소스가 낭비되는 시간이 더 길어진다는 의미입니다.
- 예외 처리
    - finalize() 메소드 실행 도중 발생한 예외는 아무도 처리할 수 없습니다. finalize() 메소드를 실행하는 스레드는 유저 애플리케이션 컨텍스트가 없기 때문이죠. 그리고 finalize() 를 실행하는 스레드는 또 별도로 생성되기 때문에 이 오버헤드 또한 감수해야합니다.
- 언제 실행될지 알 수 없다.
    - 개발자가 동적 메모리를 수동으로 처리하는 C++ 와 달리 Java 는 할당할 가용 메모리가 부족하면 가비지 콜렉터가 그때그때 반사적으로 가비지 수집을 수행합니다. 가비지 수집이 언제 일어날지 아무도 모르기 때문에 finalize() 메소드도 언제 실행될 지 알수가 없습니다. 가비지 수집 자체가 언제 실행될지 알 수 없는 상황에서 리소스 반납을 finalize() 에 맡긴다는 것 자체가 말이 되지 않습니다.
- 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다.
    - 공격 원리는 간단합니다.  생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되다 만 객체에서 악의적인 하위 클래스의 `finalizer`가 수행될 수 있게 됩니다.

>`finalizer` 공격예시
>간단한 계좌시스템을 예로 들어봅시다.
```java
import java.math.BigDecimal;

public class Account {

    private String accountId;

    public Account(String accountId) {
        this.accountId = accountId;

        if (accountId.equals("블랙ID")) {
            throw new IllegalArgumentException("블랙ID는 거래가 불가능합니다.");
        }
    }

    public void transfer(BigDecimal amount, String to) {
        System.out.printf("transfer %f from %s to %s\n", amount, accountId, to);
    }
}
```

블랙ID이면 계좌생성을 아예 막습니다.
테스테 해보면 아래와 같습니다.

```java
class AccountTest {

    @Test
    void 일반_계정() {
        Account account = new Account("cheolwon");
        account.transfer(BigDecimal.valueOf(33),"father");
    }

    @Test
    void 블랙_계정() {
        Account account = new Account("블랙ID");
        account.transfer(BigDecimal.valueOf(33),"father");
    }
}
```

<img width="1274" alt="image" src="https://github.com/Growth-Hub/Effective-Java/assets/76567238/a0db5f35-edae-4238-a824-b224f3ec9ace">

정상적으로 블랙 ID를 막고 있습니다.

이제 공격을 해봅시다.
>Account 클래스를 상속받아서  `finalize`메서드를 오라이드 한 뒤에 원하는 코드를 작성해줍니다.
```java
public class BrokenAccount extends Account {

    public BrokenAccount(String accountId) {
        super(accountId);
    }

    @Override
    protected  void finalize() throws Throwable {
        this.transfer(BigDecimal.valueOf(55000), "father");
    }
}
```

>TEST
```java
class AccountTest {

    @Test
    void 일반_계정() {
        Account account = new Account("cheolwon");
        account.transfer(BigDecimal.valueOf(33),"father");
    }

    @Test
    void 블랙_계정() {
        Account account = new Account("블랙ID");
        account.transfer(BigDecimal.valueOf(33),"father");
    }

    @Test
    void 블랙_계정_공격() throws InterruptedException {
        Account account = null;
        try {
            account = new BrokenAccount("블랙ID");
        } catch (Exception exception) {
            System.out.println("공격");
        }

        System.gc();
        Thread.sleep(2000L);
    }

}
```

<img width="1269" alt="image" src="https://github.com/Growth-Hub/Effective-Java/assets/76567238/8ae9e8d7-4ca6-400c-8e3c-6cbb93cbb47b">

이런식으로 공격이 가능합니다.  
이게 가능한 이유는 다음과 같습니다.
`finalizer`는 정적 필드에 자신의 참조를 할당하여 가비지 컬렉터가 수집하지 못하게 막을 수 있습니다. 이렇게 일그러진 객체가 만들어지고 나면, 이 객체의 메서드를 호출해 에초에는 허용되지 않았을 작업을 수행하는 건 일도 아닙니다.     
객체 생성을 막으려면 생성자에서 예외를 던지는 것만으로 충분하지만, `finalizer`가 있다면 그렇지 않습니다.    
    
#### 해결책

- final 클래스로 클래스를 만들기 
    - final 클래스들은 그 누구도 하위 클래스를 만들 수 없으니 이 공격에서 안전합니다.
- 추가적으로 finalize 메서드를 만들고 final로 선언하기(final이 아닌 클래스일 때)


>❗️참고
>현재는 JDK 9 버전에서 Deprecated 되었습니다.
>https://openjdk.org/jeps/421

<br>
<br>

## 📌 cleaner

Cleaner는 객체 참조와 해당 객체가 더 이상 필요 없을 때 실행할 정리 동작을 관리하는 클래스입니다. 이 클래스는 객체가 "팬텀 도달 가능(phantom reachable)" 상태가 되었음을 알린 후에 정리 동작을 실행하도록 등록합니다. 이를 위해 PhantomReference와 ReferenceQueue를 사용하여 도달 가능성이 변경되었을 때 알림을 받습니다.

각 Cleaner 인스턴스는 독립적으로 작동하며, 대기 중인 정리 동작을 관리하고, 더 이상 사용되지 않을 때 스레딩 및 종료를 처리합니다. 객체 참조와 해당 정리 동작을 등록하면 Cleanable이 반환됩니다. 가장 효율적인 사용 방법은 객체를 닫거나 더 이상 필요 없을 때 명시적으로 clean 메소드를 호출하는 것입니다. 정리 동작은 객체가 팬텀 도달 가능 상태가 되었을 때 한 번만 호출될 Runnable입니다. 단, 이미 명시적으로 정리된 경우에는 호출되지 않습니다. 중요한 점은 정리 동작이 등록된 객체를 참조해서는 안 된다는 것입니다. 그렇지 않으면 객체가 팬텀 도달 가능 상태가 되지 않고 자동으로 정리 동작이 호출되지 않습니다.

정리 동작의 실행은 클리너와 연관된 스레드에 의해 수행되며, 정리 동작에서 발생하는 모든 예외는 무시됩니다. 클리너와 다른 정리 동작은 정리 동작에서 발생하는 예외의 영향을 받지 않습니다. 이 스레드는 등록된 모든 정리 동작이 완료되고 클리너 자체가 가비지 컬렉터에 의해 회수될 때까지 실행됩니다.

System.exit 동안 클리너의 동작은 구현에 따라 달라집니다. 정리 동작이 호출되는지 여부와 관련하여 어떠한 보장도 이루어지지 않습니다.

클래스나 메소드에 null 인수를 전달하면 NullPointerException이 발생한다는 점 외에 특별히 명시된 경우를 제외하고는, API 주석에서 설명하듯이, 관련 객체가 팬텀 도달 가능 상태가 된 후에만 정리 동작이 호출됩니다. 따라서 정리 동작을 구현하는 객체는 정리 대상 객체에 대한 참조를 보유하지 않아야 합니다. 예제에서는 정적 내부 클래스가 정리 상태와 동작을 캡슐화합니다. 익명 클래스나 "내부" 클래스는 외부 인스턴스에 대한 암시적 참조를 포함하기 때문에 사용해서는 안 됩니다. 새 클리너를 사용할지 기존 클리너를 공유할지는 사용 사례에 따라 결정됩니다.

CleaningExample이 try-finally 블록에서 사용되면, close 메소드는 정리 동작을 호출합니다. close 메소드가 호출되지 않으면, CleaningExample 인스턴스가 팬텀 도달 가능 상태가 되었을 때 Cleaner에 의해 정리 동작이 호출됩니다.

Cleaner도 Finalizer와 마찬가지로, 리소스를 정리하기 위해 사용되기 위한 목적으로 설계되었다. 하지만, Cleaner는 Finalizer보다는 덜 위험하지만, 여전히 예측 불가능하고, 느리고 일반적으로 불필요합니다.

<br>

## Cleaner가 해결한 Finalizer의 문제

### 1. 제어 가능한 스레드
`Cleaner`는 자체적으로 관리하는 스레드를 사용하여 정리 작업을 수행하기 때문에, `finalize()`가 실행되는 스레드가 불명확하고 제어하기 어려운 것과 대비됩니다.
`Cleaner`의 스레드는 특정 정리 작업에 할당되어 더 효율적이고 관리하기 쉽습니다.

### 2. 안정성과 예측 가능성:
`Finalizer`는 예외 발생 시 객체 정리가 중단될 수 있으며, 이로 인해 리소스 누수가 발생할 수 있습니다.
반면, `Cleaner`는 예외 처리를 좀 더 안정적으로 관리할 수 있으며 예측 가능한 방식으로 리소스를 정리합니다.

### 3. 리소스 누수와 안전성
`Finalizer`는 잘못 사용될 경우 리소스 누수를 일으킬 수 있다. `Cleaner`는 이러한 리소스 누수의 위험을 줄이고, 보다 안전한 리소스 관리를 제공합니다.

<br>
 

## Cleaner가 여전히 가지고 있는 문제

### 1. 비동기적 실행

Cleaner의 정리 작업은 객체가 가비지 컬렉터에 의해 회수된 후 비동기적으로 수행됩니다.
이는 Finalizer와 유사하게, 정리 작업의 실행 시점을 정확히 예측할 수 없게 만듭니다.

### 2. 리소스 해제 지연

Cleaner도 Finalizer와 마찬가지로, 리소스가 즉시 해제되지 않을 수 있습니다.
가비지 컬렉터가 객체를 회수하고, Cleaner가 해당 작업을 수행하는 데까지 시간이 걸릴 수 있으며, 이는 특히 리소스가 제한적인 환경에서 문제가 될 수 있다.


<br>

## ❓ 그렇다면 finalizer와 cleaner 말고 어떤 것을 사용해야할까?

### 1. AutoCloseable 인터페이스 구현

단순히 `AutoCloseable` 인터페이스를 구현하여 `close()` 메서드를 호출하는 것으로 명시적으로 리소스를 해제할 수 있습니다.
```java
public class CustomResource implements AutoCloseable {
    public CustomResource() {
        //Resource opened.
    }

    @Override
    public void close() {
        //Resource closed.
    }

    public static void main(String[] args) {
        CustomResource resource = null;
        try {
            resource = new CustomResource();
            //resource do something...
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (resource != null) {
                try {
                    resource.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

### 2. try-with-resources 구문 사용

`try-with-resources` 구문은 `Finalizer`와 `Cleaner`의 대안으로 가장 권장되는 방법입니다.
`try-with-resources` 구문은 `AutoCloseable` 인터페이스를 구현하는 객체를 자동으로 관리하기 때문에, 이 구문 안에서 선언된 리소스는 구문이 종료될 때 자동으로 `close()` 메서드를 호출하여 리소스를 안전하게 닫는다.

```java
public class ResourceManagementExample {

    public static void main(String[] args) {
        // 파일 읽기
        try (BufferedReader reader = new BufferedReader(new FileReader("input.txt"))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 파일 쓰기
        try (BufferedWriter writer = new BufferedWriter(new FileWriter("output.txt"))) {
            writer.write("Hello, world!");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

<br>
<br>

---

### 📚Referece
- [java.lang.ref.Cleaner - an alternative to finalization](https://bugs.openjdk.org/browse/JDK-8138696)


