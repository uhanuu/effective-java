## 🧑‍💻 Finalizer 와 Cleaner
#### finalizer
- 객체를 소멸시키는 역할로, Object 클래스에 포함된 메서드인 finalize()를 호출한다.
- Heap 속 객체가 GC의 대상이 되었을때, 해당 객체의 finalizer를 실행하여 객체를 해제시킨다.
- finalizer는 성능, 보안상의 이유로 자바 9 이후로 deprecated API로 선정되었다.

#### Cleaner
- 자바 9 이후 finalizer의 대안으로 나온 것으로, 동일하게 객체를 해제하여 메모리를 회수하는 역할을 한다.
- Overide해서 사용해야 하는 finalizer와 달리, 구성을 통해 cleaner을 사용한다. ( 주로inner class 구현한다. )
- 하지만 대안으로 나온 Cleaner 또한, 성능의 이유로 불필요하다고 평가되고 있다.

## 🤷‍♂️ 왜 Finalizer 와 Cleaner의 사용을 피하라고 할까?
#### 1️⃣. **실행 시점을 정확히 특정할 수 없고, 실행 여부도 보장해주지 않는다.**
```java
public class FinalizerTest {
    @Override
    protected void finalize() throws Throwable {
        System.out.println("finalizer가 실행되었습니다.");
    }
    public void start(){ 
        System.out.println("start"); 
    }
}

public class SampleRunner {
    public static void main(String[] args) throws InterruptedException {
        SampleRunner runner = new SampleRunner();
        runner.run();
        // 1초 뒤 finalize()는 실행될까?
        Thread.sleep(1000);
    }

    private void run() {
        FinalizerTest finalizerTest = new FinalizerTest();
        finalizerTest.start();
    }
}
```
- main에서 run() 메서드를 통해서 FinalizerTest 객체를 생성했다.
- FinalizerTest객채의 스코프는 run 메서드 범위이기 때문에, run() 실행 후 finilze()가 실행되어야 한다.
```
실행결과: start
```
- 하지만 예상과 달리 1초를 일시정시 시켰음에도 불고하고, finilze()가 실행되지 않아 start만 출력되었다.
- finalizer 스레드는 다른 어플리케이션 스레드보다 우선순위가 낮아서 실행 시점을 정확히 알 수 없다.
- 그러다보니, finalizer 스레드가 실행 될수도, 안될수도 있어 실행 여부도 보장하지 못한다.
- Cleaner도 마찬가지로 실행시점, 실행 여부를 보장하지 않는다.

**그래서, 제때 객체를 해제해줘야 할때는 finalizer와 Cleaner를 사용하면 안된다.**
> 물론, ```System.gc``` , ```System.runFinalization```으로 GC를 작동시켜 finalizer와 Cleaner의 실행 확률을 높일 수 있다.
하지만 해당 메서드는 Full GC를 발생시키고, 이는 Stop-the-world를 수반하기 떄문에 좋은 방법이 아니다.
(추가로 ```System.gc```는 GC 실행을 보장하지 않는다.)

#### 2️⃣. 객체 해제 성능이 좋지 않다.
대표적인 방식은 다음과 같이 2가지이다.
1. finalizer 와 Cleaner
2. AutoCloseable
    <p align="center">
    <img width="456" alt="image" src="https://github.com/kim0527/Effective-Java/assets/143387515/511749d7-c79f-4db7-ae03-ad0a595656a1">
    </p>
    AutoCloseable는 인터페이스로, AutoCloseable.close()는 try-with-resource문으로 관리되는 구현체의 close()를 자동 호출해준다. ( 객체 해제 보장 )

위 두 방식으로 객체 생성에서 파괴까지 시간을 비교 해보았을때, 약 50배 차이가 발생했다.

#### 3️⃣. 예외가 발생하면, 그 즉시 종료된다.
```java
public class FinalizerException {
    public static void main(String[] args) {
        FinalizerObject obj = new FinalizerObject();
        
        obj = null;
        System.gc();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("프로그램 종료");
    }
}

class FinalizerObject {
    @Override
    protected void finalize() throws Throwable {
        // 예외 발생
        throw new Exception("finalizer에서 발생한 예외");
    }
}
```
```
실행결과: 프로그램 종료
```
- finalize 메서드로 예외를 발생시켜보았다.
- 그 결과, 예외로 인해 finalizer가 바로 종료되어 "finalizer에서 발생한 예외" 출력 없이 "프로그램 종료"만 출력되었다.
- 이처럼, 예외가 발생하면 finalizer는 작업이 남아도 그 순간 바로 종료되고, 이는 마무리가 덜 된 객체가 남아있게 만든다.

#### 4️⃣. 심각한 보안 문제를 일으킨다.
참고 영상: https://www.youtube.com/watch?v=6kNzL1bl1kI
```java
public class Account {
    private String name;

    public Account(String name) throws IllegalAccessException {
        this.name = name;
        if (this.name.equals("Min")){
            throw new IllegalAccessException("Min은 계좌를 개설할 수 없습니다.");
        }
    }
    public void transfer(int amount,String dst){
        System.out.printf("%s(이)가 %s에게 %d원을 송금합니다.",this.name,dst,amount);
    }
}

public class BrokenAccount extends Account{
    public BrokenAccount(String name) throws IllegalAccessException {
        super(name);
    }
    //악의적인 메서드
    @Override
    protected void finalize() throws Throwable {
        this.transfer(100000, "KIM");
    }
}
```
- 계좌 객체를 생성할때, 이름이 "Min"인 경우 예외를 던지는 Account 클래스를 만들었다.
- Account의 하위객체인 BrokenAccount는 finalize에 악의적인 메서드가 담겨있다.
```java
@Test
void FinalizerAttack throws InterruptedException{
    Account account = null;
    try{
        account = new BrokenAccount("Min");
    }catch(Exception exception){
        System.out.println("Min은 안되는데?");
    }
    
    System.gc();
    Thread.sleep(3000L);
}
```
```
실행결과: Min은 안되는데?
        Min가 KIM에게 %100000원을 송금합니다.
```
- 객체를 생성할때 고의로 예외를 발생시키고 finalize를 System.gc로 실행시키면, 다음과 같이 transfer 메소드가 실행된 것을 확인할 수 있다.
- 생성자나 직렬화 과정에서 예외를 발생시킨 후, finalizer를 이용해 악의적인 메서드를 실행시키는 것을 ```finalizer 공격``` 이라고 한다.
- 이처럼 finalizer를 사용한 클래스는 finalizer 공격에 노출되어 보안 문제를 일으킬 수 있다.

> finalizer 공격을 막는 방법은 다음과 같다.
첫번째. ```public class Account``` ➡️  ```public final class Account```
두번쨰. Account 객체에서 finalize를 Overide해서 final 메서드로 만들기

## 🤔  그렇다면 확실한 대안책은 무엇이고, Finalizer 와 Cleaner 언제 사용하는 걸까?
#### 1️⃣. AutoCloseable을 사용하자.
- 위에서 언급했듯이, AutoCloseable는 try-with-resource문으로 관리되는 구현체의 close()를 자동 호출해주기 때문에 객체 해제를 보장해준다.
```java
public class TestObject implements AutoCloseable {
    public void printStart() {
        System.out.println("start");
    }

    @Override
    public void close() throws Exception {
        System.out.println("자동으로 close()가 호출되었습니다.");
    }
}
public static void main(String args[]) {
    try (TestObject testObject = new TestObject()) {
        testObject.printStart();
    } catch (Exception e) {
    }
}
```
```
실행결과: Start
        자동으로 close()가 호출되었습니다.
```
#### 2️⃣. 안전망으로만 Finalizer 와 Cleaner을 사용하자.
- 클라이언트가 close()를 호출하지 않았을때를 대비하여 객체를 해제하기 위한 안전망 역할로만 사용하는 것을 추천하고 있다.
#### 3️⃣. 네이티브 피어와 연결된 객체를 해제할 때, Finalizer 와 Cleaner을 사용하자.
JNI ( Java Native Interface )
- 하드웨어와 운영 체제 플랫폼에 종속된 프로그램(네이티브 프로그램) 혹은 다른 언어로 작선된 라이브러리를 사용할 수 있게 해주는 프로그램을 말한다.

네이티브 피어(Native peer)
- JNI을 통해 생성된 객체를 말한다.

객체를 관리하는 JVM의 GC는 해당 네이티브 피어를 인식하지 못하여 GC 대상으로 지정하지 못한다.
그래서 Finalizer 와 Cleaner가 그러한 객체들을 회수하는 역할로 적당하다.

>하지만 Finalizer 와 Cleaner는 기본적으로 성능이 좋지 않기 때문에, 진짜 해당 방법밖에 없는지 심사숙고해서 사용해야한다.
## ✍️ 정리하면
**웬만하면 Finalizer 와 Cleaner은 사용을 지양하자.**
### 📝 Reference
[finalize 메소드 퇴역 이후](https://www.itworld.co.kr/news/224419)<br>
[try-catch-finally & try-with-resources](https://codechacha.com/ko/java-try-with-resources/)<br>
[AutoCloseable & try-with-resources](https://mangkyu.tistory.com/217)<br>
[Finalizer 공격법](https://hyeonmin.tistory.com/275)<br>


