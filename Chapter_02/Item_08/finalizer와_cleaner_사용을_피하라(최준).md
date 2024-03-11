# 8️⃣ Item 8: finalizer와 cleaner 사용을 피하라

<br>

## 📌 목차
1. 객체 소멸자
2. 사용 X 이유 1. 즉시 수행된다는 것을 보장하지 않는다. 
3. 사용 X 이유 2. 수행 여부조차 보장하지 않는다. 
4. 사용 X 이유 3. 동작 중 발생한 예외 무시되며 종료된다. 
5. 사용 X 이유 4. 심각한 성능 문제도 동반한다. 
6. 사용 X 이유 5. 공격에 노출되어 심각한 보안 문제 일으킬 수 있다.
7. finalizer & cleaner 대안 
8. finalizer & cleaner 사용처
9. 사용처 1. close 메소드 호출 않는 것의 안전망 역할
10. 사용처 2. 네이티브 피어와 연결된 객체에서의 회수
11. cleaner로 안전망 역할
12. cleaner의 안전망 역할 문제점 확인
13. cleaner의 안전망 역할 문제점 해결

<br>

## ❌ 객체 소멸자

자바는 두 가지 `객체 소멸자`를 제공한다. 

1. finalizer
2. cleaner

### 🔴 finalizer

- finalize() 메서드는 java.lang.Object 클래스에 정의되어 있다.

- 자바에서 객체가 `가비지 컬렉션에 의해 제거될 때 실행`된다.

- 즉, Finalizer는 자바에서 객체가 소멸될 때 마지막으로 수행할 수 있는 작업을 정의하는 데 사용된다. 

- 주로 파일 핸들, 네트워크 연결, 데이터베이스 연결처럼 `시스템 리소스를 정리하는 용도가 이런 작업`이다.

<br>

finalizer를 사용한 코드는 아래와 같다. 

리소스 해제를 명시적으로 호출하지 않고 객체가 `가비지 컬렉션 될 때 finalize() 메소드가 호출`되는 것이다. 

```java
public class Resource {

    private boolean isOpen;

    public Resource() {
        this.isOpen = true; // Resource open.
    }

    // 리소스 사용을 위한 메소드
    public void useResource() {
        if (!isOpen) {
            throw new IllegalStateException("Resource is closed.");
        }
    }

    public void closeResource() {
        isOpen = false; // Resource closed.
    }

    // finalize 메소드 오버라이드
    @Override
    protected void finalize() throws Throwable {

        if (isOpen) { 
            closeResource(); // 리소스가 아직 열려있다면, 리소스를 닫음
        }
        super.finalize();  // finalize 메소드를 super 클래스에게도 호출
    }

    public static void main(String[] args) {

        Resource resource = new Resource(); // 리소스 객체 생성
        resource.useResource(); // 리소스 사용

        // 리소스 해제를 명시적으로 호출하지 않음
        // 객체가 가비지 컬렉션될 때 finalize 메소드가 호출됨
    }
}

```

하지만 finalizer는 예측할 수 없고 상황에 따라 위험할 수 있어 불필요하다. 

오동작, 낮은 성능, 이식성 문제의 원인이 된다. 

`기본적으로 쓰지 말아야 한다.` 

자바 9에서는 `deprecated API`로 지정되었다. 

그렇지만 자바 라이브러리에서 여전히 사용하는 곳이 있다. 

<br>

### 🔴 cleaner

- finalizer가 deprecated API로 지정되고 대안으로 소개 되었다. 

- finalizer보다 `덜 위험`하다. 

<br>

cleaner를 사용한 코드는 아래와 같다. 

cleaner가 작동하기 위해서는 객체에 대한 모든 `강한 참조가 제거`되어야 한다. 

그리고 cleaner는 가비지 콜렉션 수행될 때 cleaner에서 `run 메소드를 호출`한다. 

따라서 cleaner 사용하는 객체 안에 `static 중첩 클래스`를 만들고 `Runnable을 implements` 해야 한다. 

```java
public class Resource {
    private static final Cleaner Cleaner = Cleaner.create();

    private static class ResourceCleaner implements Runnable {
        @Override
        public void run() {
            // 리소스 정리 로직
        }
    }

    private final Cleaner.Cleanable cleanable;

    public Resource() {
        this.cleanable = Cleaner.register(this, new ResourceCleaner()); // Resource 객체에 대한 정리 작업 등록
    }

    public void useResource() {
        System.out.println("Resource is being used.");
    }

    public static void main(String[] args) {
        Resource resource = new Resource();
        resource.useResource();

        // Resource 객체에 대한 참조를 명시적으로 제거
        // Cleaner가 작동하기 위해서는 객체에 대한 모든 강한 참조가 제거되어야 함
        resource = null;

        // GC 수행시 Cleaner에서 객체 정리 로직 run() 수행
    }
}
```

하지만 `여전히 예측할 수 없고, 느리고, 불필요`하다. 

<br>

### 🔵 C++ destructor

C++의 파괴자와 finalizer, cleaner는 다른 개념이다. 

C++에서 파괴자는 특정 객체와 관련된 자원 회수하는 보편적인 방법이다. 

하지만 자바에서는 이런 회수하는 역할을 가비지 컬렉터가 담당한다.

프로그래머에게는 아무런 작업도 요구 하지 않는다. 

<br>

### ...??

finalizer와 cleaner는 이렇듯 문제가 많고 어디에 쓰이는지 모르겠다. 

`따라서 이번 item에서 사용하지 말라는 것`이다. 

finalizer와 cleaner를 사용하면 안되는지 각 이유 아래와 같다. 

각 이유들을 살펴보자. 

1. 즉시 수행된다는 것을 보장하지 않는다. 
2. 수행 여부조차 보장하지 않는다. 
3. 동작 중 발생한 예외 무시되며 종료된다. 
4. 심각한 성능 문제도 동반한다. 
5. 공격에 노출되어 심각한 보안 문제 일으킬 수 있다. 

<br>

## 1️⃣ 사용 X 이유 1. 즉시 수행된다는 것을 보장하지 않는다.

finalizer와 cleaner는 `즉시 수행된다는 보장이 없다.` 

따라서 여기에서 제때 실행되어야 하는 작업은 절대 할 수 없다. 

finalizer와 cleaner가 얼마나 빨리 수행될지는 전적으로 가비지 컬렉터 알고리즘에 달렸다. 

따라서 finalizer와 cleaner `수행 시점에 의존하는 프로그램은 큰 문제`가 생길 수 있다. 

<br>

왜 이런 문제 발생하는가?

- **`finalizer`**
    - 자바 언어 명세는 어떤 스레드가 finalizer를 수행할지 명시하지 않기 때문이다.
    
    - finalizer 스레드는 다른 어플리케이션 스레드보다 **우선 순위가 낮아서 실행될 기회를 얻지 못하는 것**이다.
    
    - 따라서 이 **문제를 예방할 방법은 없고** finalizer를 사용하지 말아야 한다.

- **`cleaner`**
    - cleaner는 자신을 **수행할 스레드를 제어할 수 있다**는 면이 조금 낫다.
    
    - 하지만 여전히 백그라운드에서 수행되며 가비지 컬렉터의 통제하에 있어 **즉시 수행된다는 것을 보장하지 않는다**.

<br>

## 2️⃣ 사용 X 이유 2. 수행 여부조차 보장하지 않는다.

자바 언어 명세는 finalizer, cleaner의 수행 시점뿐 아니라 `수행 여부조차 보장하지 않는다.` 

finalizer와 cleaner에서 해야 하는 작업을 전혀 수행하지 못한 채 프로그램이 중단될 수 있다는 것이다. 

따라서 프로그램 생애주기와 상관 없는 `상태를 영구적으로 수정하는 작업은 finalizer나 cleaner에서 하면 안된다.` 

<br>

finalizer, cleaner `수행 여부를 보장하는 메소드`가 있는 것 같다.

- System.gc()
    - 강제로 가비지 콜렉션을 실행시키는 메소드.

- System.runFinalization
    - finalizer 실행 대기 중인 모든 객체에서 finalizer 실행하는 메소드.
    - 가비지 콜렉터에 의해 소멸 결졍된 객체를 즉시 소멸 시킨다는 메소드.

하지만 이 2개의 메소드는 finalizer와 cleaner가 `실행될 가능성을 높여줄 수는 있으나 보장해주지는 않는다. `

<br>

사실 finalizer와 cleaner `실행을 보장해주겠다는 메소드`가 있었다. 

- System.runFinalizersOnExit

- Runtime.runRinalizersOnExit

하지만 이 두 메소드는 `심각한 결함 때문에 수십 년간 지탄 받고 이제 사용되지 않는다.`

<br>

- ### `System.runFinalizersOnExit`
    
    - 이 방법은 원래 종료 시 실행 중인 finalizer를 활성화 또는 비활성화 하도록 설계되었다.
    
    - 종료 시 finalizer를 실행하는 것은 기본적으로 비활성화 되었다.
    
    - 활성화된 경우, Java 런타임이 종료되기 전에 아직 자동으로 Finalizer가 호출되지 않은 모든 개체의 Finalizer가 실행된다.
    
    - 이것은 본질적으로 안전하지 않다.
    
    - 다른 스레드가 해당 객체를 동시에 조작하는 동안 라이브 객체에서 finalizer가 호출되어 불규칙한 동작이나 deadlock 발생할 수 있다.
    
    - 따라서 이 메소드는 deprecated 되었다. 

<br>    

- ### `Runtime.runRinalizersOnExit`
    
    - 이 방법은 본질적으로 안전하지 않다.
    
    - 다른 스레드가 해당 객체를 동시에 조작하는 동안 라이브 객체에서 finalizer가 호출되어 불규칙한 동작이나 deadlock이 발생할 수 있다.
    
    - 종료 시 finalizer를 활성화 또는 비활성한다.
    
    - 이렇게 하면 Java 런타임이 종료되기 전에 아직 자동으로 호출되지 않은 모든 개체의 Finalizer가 실행되도록 지정된다.
    
    - 기본적으로 종료 시 finalizer는 더이상 사용할 수 없다.
    
    - 보안 관리자가 있는 경우 먼저 checkExit 메서드가 0으로 호출되어 종료가 허용되는지 확인한다.
    
    - 이로 인해 SecurityException이 발생할 수 있다.

<br>    

## 3️⃣ 사용 X 이유 3. 동작 중 발생한 예외 무시되며 종료된다.

또 다른 문제점으로 `finalizer 동작 중 발생한 예외는 무시`되며 처리할 작업이 남았어도 `그 순간 종료`된다. 

이렇게 예외가 무시되면 해당 객체는 마무리가 덜 된 상태로 남을 수 있다. 

그리고 다른 스레드가 `이런 상태의 객체를 사용하려고 한다면 어떻게 동작할지 예측할 수 없다.` 

예외가 발생하면 스레드 중단되고 스택 추적 내역을 출력한다. 

하지만 `finalizer에서는 예외 발생해도 경고조차 출력되지 않는다.` 

그나마 `cleaner는 자신의 스레드를 통제하기 때문에 이런 문제 발생하지 않는다.` 

<br>

## 4️⃣ 사용 X 이유 4. 심각한 성능 문제도 동반한다.

자바에서 권고하는 자원 관리 방법인 `try-with-resources를 사용`해 AutoCloseable 객체를 생성하고 가비지 컬렉터가 수거하기까지 ***`12ns`*** 가 걸렸다. 


- ### `AutoCloseable이란`
    
    - 내가 만든 클래스가 try-with-resources으로 자원이 해제되길 원한다면 AutoCloseable을 implements 해야한다.
    
    - try에 선언된 객체가 AutoCloseable을 구현하면 자바는 try 구문이 종료될 대 객체의 close 메소드를 호출해주는 것이다. 
    
    - try-with-resources 문으로 관리되는 객체일 때 close() 메서드가 자동으로 호출된다. 리소스를 닫고 기본 리소스를 양도한다.
    
    - close() 메소드에서 리소스를 닫도록 하면 된다. 

<br>    

하지만 `finalizer를 사용`한 객체를 생성하고 수거할 때까지는 ***`550ns`*** 가 걸렸다. 

왜 이렇게 성능이 느린가 하면 finalizer가 가비지 컬렉터의 효율을 떨어뜨리기 때문이다. 

`cleaner`도 클래스의 모든 인스턴스를 수거하는 형태로 사용하면 각 인스턴스를 수거하는데 ***`500ns`*** 정도 걸린다. 

하지만 cleaner는 잠시 후 볼 안전망 형태로만 사용하면 훨씬 빨라진다. 

`안전방 방식`에서는 객체 하나를 생성, 정리, 파괴하는데 ***`66ns`*** 정도 걸린다고 한다. 

<br>

### 속도 정리

| 방법 | 속도 |
| --- | --- |
| try-with-resources | 12ns |
| finalizer | 550ns |
| cleaner | 500ns |
| cleaner 안전망 | 66ns |

<br>

이렇듯 그냥 `finalizer, cleaner를 사용하면` 자바에서 권고하는 자원 관리 방법보다 `성능적으로 굉장히 느려진다.` 

<br>

## 5️⃣ 사용 X 이유 5. 공격에 노출되어 심각한 보안 문제 일으킬 수 있다.

finalizer를 사용한 클래스는 `finalizer 공격에 노출되어 심각한 보안 문제`를 일으킬 수 있다. 

<br>

- ### `finalizer 공격이란?`
    
    - finalizer 공격 원리는 다음과 같다. 
    
    - 생성자나 직렬과 과정에서 예외 발생하면 이 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 된다. 
    
    - 상위 클래스 생성자에서 조건 충족하지 못하면 생성하지 못하게 한 것이다. 
    
    - 이 finalizer는 하위 클래스의 정적 필드에 자신의 참조 = this를 할당해 다시 객체 접근 할 수 있게 되어 가비지 컬렉터가 수집하지 못하게 막을 수 있다. 
    
    - 가비지 컬렉터가 실행되어 `finalize()를 호출하는데 staic 필드에 this를 넣어 객체에 접근`할 수 있게 되는 것이다. 
    
    - 아래 코드에서 상위 클래스의 생성자와 하위 클래스의 finalize() 메소드를 유심히 보자. 
    
    <br>

    - **상위 클래스 코드**
    
    ```java
    class Vulnerable {
        Integer value = 0;
    
        Vulnerable(int value) {
            if (value <= 0) {
                throw new IllegalArgumentException("Vulnerable value must be positive");
            }
            this.value = value;
        }
    
        @Override
        public String toString() {
            return (value.toString());
        }
    }
    ```
    
    - **하위 클래스 코드**
    
    ```java
    public class AttackVulnerable extends Vulnerable {
        static Vulnerable vulnerable;
    
        public AttackVulnerable(int value) {
            super(value);
        }
    
        public void finalize() {
            vulnerable = this;
        }
    
        public static void main(String[] args) {
            try {
                new AttackVulnerable(-1);
            } catch (Exception e) {
                System.out.println(e);
            }
            System.gc();
            System.runFinalization();
            if (vulnerable != null) {
                System.out.println("Vulnerable object " + vulnerable + " created!");
            }
        }
    }
    ```
    
    - 이런 객체 만들어지면 이 객체의 메소드 호출해 애초에는 허용되지 않았을 작업 수행할 수 있다. 
    
    - `객체가 생성되면 안되는데 생성되어 메소드 호출할 수 있게 되는 것`이다. 
    
    - 객체 생성 막으려면 생성자에서 예외 던지면 되었다. 
    
    - 하지만 `finalizer가 있으면 이것만으로는 막을 수 없는 것이다.` 
    
    - 이해가 어렵다면 아래 references에 `‘finalizer 공격’` 을 보면 좋을 것이다. 

<br>   

이런 공격을 막으려면 `final 클래스를 사용`하면 된다. 

final 클래스는 누구도 하위 클래스 만들 수 없으니 이 공격에서 안전하다. 

하지만 문제는 final 클래스가 아닌 클래스들이다. 

따라서 `final 클래스가 아닌 클래스는 아무일도 하지 않는 finalize 메소드를 만들고 final로 선언하면 된다.` 

<br>

## ✴ finalizer & cleaner 대안

이렇듯 finalizer와 cleaner는 문제점이 많다. 

그렇다면 종료해야 할 자원을 담고 있는 객체의 클래스에서 `finalizer와 cleaner를 대신해줄 묘안`이 무엇인지 알아보자. 

대안은 바로 `AutoCloseable을 구현`하고 클라이언트에서 인스턴스를 다 쓰고 나면 `close 메소드를 호출`하면 된다. 

그리고 예외가 발생해도 제대로 종료되도록 `try-with-resources를 사용`해야 한다. 

그리고 각 인스턴스는 자신이 닫혔는지 추적하는 것이 좋다. 

즉 close 메소드에서 이 객체는 `더 이상 유효하지 않다고 필드에 기록`하는 것이다. 

그리고 `필드를 검사해서 객체가 닫힌 후에 객체 호출하면 IllegalStateException을 던지는 메소드도 만들면 된다`. 

코드로 만들어보면 아래와 같을 것이다. 

```java
import java.io.IOException;

public class MyResource implements AutoCloseable {
    private boolean closed = false;

    // 메서드를 호출할 때마다 유효성을 검사하는 보조 메서드
    private void checkValidity() {
        if (closed) {
            throw new IllegalStateException("Resource is closed");
        }
    }

    public void doSomething() {
        checkValidity();
        System.out.println("Doing something with the resource");
    }

    @Override
    public void close() throws IOException {
        if (!closed) {
            // 자원을 닫는 추가 작업이 있다면 여기서 수행
            closed = true;
            System.out.println("Resource closed");
        }
    }

    // 클라이언트가 직접 호출하여 자원을 닫을 수 있도록 하는 메서드
    public void closeResource() throws IOException {
        close();
    }

    public static void main(String[] args) {
        try (MyResource resource = new MyResource()) {
            // 리소스를 사용하는 코드
            resource.doSomething();
            // 예외가 발생하는 상황 시뮬레이션
            throw new IOException("Simulated exception");
        } catch (IOException e) {
            System.out.println("Caught exception: " + e.getMessage());
        }
    }
}
```

<br>

## ♻ finalizer & cleaner 사용처

그러면 finalizer와 cleaner는 절대 쓰지 말아야 하는가? 

그렇다면 왜 만든 것인가?

finalizer와 cleaner는 `적절한 쓰임새가 2가지`가 있다. 

1. close 메소드 호출 않는 것의 안전망 역할
2. 네이티브 피어와 연결된 객체에서의 회수

각각의 쓰임에 대해서 보자. 

<br>

## 1️⃣ 사용처 1. close 메소드 호출 않는 것의 안전망 역할

`자원의 소유자가 close 메소드를 호출하지 않을 수 있다.` 

이것에 대한 `안전망의 역할`을 finalizer와 cleaner가 하는 것이다. 

위에서 봤듯이 finalizer와 cleaner는 즉시 또는 끝까지 호출되리라는 보장은 없다. 

그래도 `자원 소유자가 close를 호출하지 않으면 절대 자원 회수가 일어나지 않는다.` 

차라리 finalizer, cleaner를 사용해 `자원 회수를 늦게라도 해주는 것이 아예 안 하는 것보다 낫다.` 

그래도 이런 안전망 역할의 finalizer를 작성할 때에는 그럴만한 값어치가 있는지 심사숙고 해야 한다. 

<br>

## 2️⃣ 사용처 2. 네이티브 피어와 연결된 객체에서의 회수

`네이티브 피어와 연결된 객체에서 finalizer와 cleaner가 사용이 된다. `

네이티브 피어는 일반 자바 객체가 `네이티브 메서드 기능`을 통해 기능을 위임한 네이티브 객체다. 

<br>

- ### `네이티브 메서드`
    
    - 네이티브 메소드는 `다른 언어로 작성된 코드를 자바에서 호출하도록` 만들어진 규약이다.
    
    - 자바 네이티브 인터페이스 = JNI는 자바 언어와 네이티브 언어 (C/C++) 언어 간의 상호 작용을 위한 인터페이스다. 
    
    - JNI를 사용하면 자바 어플리케이션에서 C/C++ 함수를 호출하거나 C/C++ 어플리케이션에서 자바 클래스 및 메소드를 호출할 수 있다. 
    
    - 이를 통해 자바의 플랫폼 독립성을 유지하면서도 하드웨어의 특성을 활용하는 네이티브 코드를 사용할 수 있다.

<br>    

네이티브 피어는 `자바 객체가 아니라 가비지 컬렉터가 존재를 알지 못한다.` 

그래서 이때 finalizer와 cleaner를 사용해 네이티브 피어의 자원을 회수하는 것이다. 

하지만 `성능 저하를 감당할 수 있고` 네이티브 피어가 `심각한 자원을 가지고 있지 않을 때`에만 finalizer와 cleaner를 사용해야 한다. 

성능 저하 감당할 수 없거나 네이티브 피어가 사용하는 자원 즉시 회수해야 하는 심각한 자원이면 `close 메소드를 사용`해야 한다. 

<br>

## 🔗 cleaner로 안전망 역할

cleaner의 사용처 1인 안전망 역할을 하는 것을 보자. 

`Room이라는 클래스를 예시`로 안전망 역할을 어떻게 하는지 볼 것이다. 

```java
package effectivejava.chapter2.item8;

import java.lang.ref.Cleaner;

// 코드 8-1 cleaner를 안전망으로 활용하는 AutoCloseable 클래스 (44쪽)
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다!
    private static class State implements Runnable {
        int numJunkPiles; // Number of junk piles in this room

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        // close 메서드나 cleaner가 호출한다.
        @Override public void run() {
            System.out.println("Cleaning room");
            numJunkPiles = 0;
        }
    }

    // 방의 상태. cleanable과 공유한다.
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override public void close() {
        cleanable.clean();
    }
}
```

Room 클래스는 AutoCloseable을 구현했다. 

따라서 try-with-resources와 사용하면 자동으로 close 메소드가 호출해 자원 회수 될 것이다. 

하지만 여기서는 `cleaner를 안전망으로 사용해 자원 회수하려는 것`이다.

---

<br>

static으로 사용된 중첩 클래스인 State는 cleaner가 방을 청소할 때 `수거할 자원인 numJunkPiles 필드`를 가지고 있다. 

이 State 클래스는 Runnable을 구현하고 오버라이딩 해야 하는 run 메소드는 cleanable에 의해 딱 한번만 호출될 것이다. 

이 `run 메소드에 리소스 정리 로직`이 들어가 있는 것이다. 

이 State 인스턴스는 **`절대 Room 인스턴스를 참조`** 하면 안된다. 

왜냐하면 `순환 참조`가 생겨 가비지 컬렉터가 Room 인스턴스를 회수할 기회가 오지 않는다. 

State가 정적 중첩 클래스인 이유가 이것 때문이다. 

static 아니면 중첩 클래스가 외부 클래스 참조 가지기 때문에 순환 참조 생긴다. 

그리고 `람다` 역시 바깥 객체의 참조 갖기 쉬우니 사용하지 않는 것이 좋다. 

------

<br>

cleanable 객체는 Room 생성자에서 `cleaner에 Room과 State를 등록`하면서 얻는다. 

리소스를 정리하는 로직인 `run 메소드가 호출되는 상황`은 **2가지**가 있다. 

1. Room의 close 메소드 호출
2. close 메소드 호출 안되었다면 cleaner가 호출

<br>

`close 메소드에서 cleanable.clean() 호출`하면 여기서 run 메소드를 호출해 리소스를 정리해준다. 

하지만 실수로 `close 메소드를 호출하지 않으면` 가비지 컬렉터가 Room 회수할 때 `cleaner가 run 메소드 호출해주는 것`이다. 

그런데 이 ***`cleaner가 run 메소드 호출해주는 것이 보장되지 않는 것이 문제다.`***

<br>

## ❎ cleaner의 안전망 역할 문제점 확인

cleaner가 run 메소드 호출 보장하지 않는 것이 문제라고 했다. 

그래서 한번 cleaner가 `잘 run 메소드 호출해서 리소스를 정리해주는지 보자.` 

아래 코드는 Room을 사용하고 “아무렴”을 출력하는 것이다. 

cleaner가 run 메소드를 잘 호출하면 “아무렴”이 출력이 되고 “방 청소”가 출력될 것이다. 

```java
package effectivejava.chapter2.item8;

import java.util.concurrent.TimeUnit;

// cleaner 안전망을 갖춘 자원을 제대로 활용하지 못하는 클라이언트 (45쪽)
public class Teenager {
    public static void main(String[] args) {
        new Room(99);
        System.out.println("Peace out");

        // 다음 줄의 주석을 해제한 후 동작을 다시 확인해보자.
        // 단, 가비지 컬렉러를 강제로 호출하는 이런 방식에 의존해서는 절대 안 된다!
//      System.gc();
    }
}
```

<br>

하지만 출력해보니 “아무렴”만 나온다. 

즉 `run 메소드 호출하지 않아서 리소스 정리가 되지 않은 것`이다. 

cleaner의 문제점인 호출이 보장되지 않는 것이 나타난 것이다. 

![image](https://cj-1998.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F7d1044b7-24d0-4b32-b24a-1b9061f7bf65%2F55d6d37a-92c2-48f5-952c-7e1c297c9c31%2FUntitled.png?table=block&id=75b2628e-c240-4f4e-a0bc-5361b308cf46&spaceId=7d1044b7-24d0-4b32-b24a-1b9061f7bf65&width=660&userId=&cache=v2)

<br>

cleaner의 명세에는 System.exit 호출할 때의 cleaner의 동작은 구현하기 나름이고 청소가 이뤄질지 보장하지 않는다고 나와 있다. 

cleaner가 안전망이기는 한데 `아주 부실한 안전망`인 것이다. 

<br>

## ✅ cleaner의 안전망 역할 문제점 해결

그렇다면 이런 문제를 어떻게 해결해야 하는가?

앞에서 봤듯이 Room 생성을 `try-with-resources로 감싸면` Room 클래스가 AutoCloseable을 구현했기 때문에 `무조건 close 메소드 불려서 리소스가 정리된다.` 

```java
package effectivejava.chapter2.item8;

// cleaner 안전망을 갖춘 자원을 제대로 활용하는 클라이언트 (45쪽)
public class Adult {
    public static void main(String[] args) {
        try (Room myRoom = new Room(7)) {
            System.out.println("안녕~");
        }
    }
}
```

<br>

실행해보면 “방 청소”가 호출이 되었다. 

`리소스 정리가 잘 된 것`이다. 

![image](https://cj-1998.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F7d1044b7-24d0-4b32-b24a-1b9061f7bf65%2F2b19193f-ff05-4f70-b8a2-f0540e4ca282%2FUntitled.png?table=block&id=e42d34f6-477c-44be-9f69-50139f522986&spaceId=7d1044b7-24d0-4b32-b24a-1b9061f7bf65&width=570&userId=&cache=v2)

<br>

따라서 아주 부실한 안전망인 cleaner보다 `튼튼하고 무조건 막아주는 try-with-resource를 사용해야 하는 것`이다. 

<br>

## 📍 references

- [finalizer & cleaner](https://olrlobt.tistory.com/76)
- [System.gc](https://velog.io/@jsj3282/17.-%EB%8F%84%EB%8C%80%EC%B2%B4-GC%EB%8A%94-%EC%96%B8%EC%A0%9C-%EB%B0%9C%EC%83%9D%ED%95%A0%EA%B9%8C)
- [System.runFinalization 1](https://codedragon.tistory.com/4608)
- [System.runFinalization 2](https://docs.oracle.com/javase/8/docs/api/java/lang/System.html)
- [System.runFinalizersOnExit](https://docs.oracle.com/javase/8/docs/api/java/lang/System.html)
- [Runtime.runRinalizersOnExit](https://docs.oracle.com/javase/7/docs/api/java/lang/Runtime.html#runFinalizersOnExit(boolean))
- [AutoCloseable 1](https://velog.io/@sa1341/AutoCloseable-%ED%81%B4%EB%9E%98%EC%8A%A4)
- [AutoCloseable 2](https://hyoj.github.io/blog/java/basic/java7-autocloseable.html#method-detail)
- [finalizer 공격](https://yangbongsoo.tistory.com/8)
- [네이티브 메소드 1](https://roughexistence.tistory.com/81)
- [네이티브 메소드 2](https://gdngy.tistory.com/26)