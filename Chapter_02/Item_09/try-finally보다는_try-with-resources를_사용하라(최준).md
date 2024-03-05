# 9️⃣ Item 9: try-finally보다는 try-with-resources를 사용하라

<br>

## 📌 목차 
1. 자바의 자원 회수
2. try-finally
3. try-finally 문제점
4. try-with-resources
5. try-with-resources 장점
6. 결론

<br>

## 🗑 자바의 자원 회수

자바 라이브러리에는 `close 메소드 호출해 직접 닫아줘야 하는 자원`이 많다. 

예시는 아래와 같다.

- InputStream
- OutputStream
- java.sql.Connection
- ...

<br>

이런 자원들을 `close 하지 않으면 문제점`이 많다.

1. 메모리가 해제 되지 않아 메모리 누수가 생긴다. 
2. 리소스 사용하고 반환하지 않아 리소스 더 이상 사용할 수 없게 된다. 
3. Connection의 경우 연결 유지해 다른 클라이언트가 새로운 연결 생성 못한다. 
4. Connection 반환하지 못해 성능 저하된다. 
5. etc….

<br>

이렇듯 자원 닫기는 중요하다. 

하지만 자원 닫기는 클라이언트가 놓치기 쉬워 예측할 수 없는 성능 문제로 이어지기도 한다. 

이런 자원 중 상당수가 `안전망으로 finalizer를 활용`했다. 

하지만 item 8에서 말했듯이 finalizer를 쓰지 말라고 하고 finalizer는 믿을만하지 못하다. 

그래서 `전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally`가 쓰였다. 

그렇다면 ***`try-finally`*** 가 무엇인가?

<br>

## 🔨 try-finally

예외가 발생했을 때 프로그램이 자동 중단 되는데 이때 프로그램이 자동 중단되지 않고 대처할 수 있도록 처리하는 것이 예외처리다. 

`try-finally는 예외처리할 때 사용하는 구문`이다. 

보통 try-catch-finally라고 많이 한다. 

try,finally 각각에 대해서 보자. 

- `try`: 예외가 발생할지 발생하지 않을지 모르는 코드 블록 정의

- `finally`: try 블록에서 일어난 일에 관계없이 항상 실행이 보장되어야 할 뒷정리용 코드

<br>

finally는 try 블록이 정상 종료가 되던 예외가 발생하던 `실행이 되는 것`이다. 

catch나 finally 블록은 생략할 수 있다. 

try 블록은 catch나 finally 중 적어도 하나 이상의 블록과 함께 사용되어야 한다.

try, catch, finally 블록은 모두 중괄호로 시작하여 중괄호로 끝난다. 

중괄호들은 필수로 요구되는 문법의 일부로서 해당절에 단 하나의 문장만 있더라도 생략할 수 없다.

<br>

`자원이 제대로 닫힘을 보장하는 수단으로 try-finally`를 사용할 때에는 아래와 같이 진행하는 것이다. 

- `try`: try 안에 ***자원을 사용하는 코드를 작성***

- `finally`: finally 안에서 ***자원을 회수하는 close 메소드 호출***

<br>

아래 코드와 같이 사용하는 것이다. 

```java
  // 코드 9-1
	static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
```

<br>

이렇게 해야 예외가 발생하거나 메소드에서 반환되어도 `자원을 제대로 닫을 수 있는 것`이다. 

하지만 try-finally가 `더 이상 자원을 회수하는 최선의 방법이 아니라고 한다.` 

왜 그런지, 어떤 `문제점`이 있는지 보자. 

<br>

## ❎ try-finally 문제점

위에 코드에서 보면 try-finally를 사용해도 나쁘지 않은 것 같다.

<br>

하지만 문제점이 있다. 

자원을 하나 더 사용하는 아래 코드를 보자. 

`자원을 하나 더 사용하니까 try-finally문이 중첩 되어서 코드가 굉장히 지저분`해졌다. 

```java
    // 코드 9-2 자원이 둘 이상이면 try-finally 방식은 너무 지저분하다! (47쪽)
    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }
```

<br>

그리고 또 다른 try-finally의 문제점이 있다. 

`예외는 try 블록, finally 블록 모두에서 실행 될 수 있다.` 

try 블록에서 예외 발생해도 무조건 finally 블록 실행해준다고 했지 finally 블록에서 예외 발생하면 어떻게 해준다는 없다.

<br>

이렇게 try, finally 블록에서 모두 예외가 발생하면 close 메소드도 실패할 것이다. 

이런 상황이면 finally 블록에서 `발생한 두번째 예외가 try 블록에서 발생한 첫번째 예외를 집어 삼킨다.` 

스택 추적 내역에 `첫번째 예외에 관한 정보가 남지 않아 디버깅이 어렵게 된다.`

<br>

물론 두번째 예외 대신 첫번째 예외 기록하도록 코드 수정할 수 있다. 

하지만 그러면 코드가 또 지저분해져서 실제로 그렇게까지 하는 경우는 거의 없다고 한다. 

<br>

## 💎 try-with-resources

그렇다면 도대체 자원 정리에는 어떤 것을 사용해야 하는가?

`현재까지 실패한 목록`

- finalizer
- cleaner
- try-finally

<br>

이런 문제들을 해결해 줄 것이 바로 자바 7에서 나온 **`try-with-resources`** 이다. 

try-with-resources를 사용하려면 `AutoCloseable 인터페이스를 구현`해야 한다. 

item 8에서도 설명했지만 AutoCloseable 인터페이스는 close 메소드 하나만 정의한 인터페이스다. 

자바 라이브러리와 서드파티 라이브러리들의 수많은 클래스와 인터페이스가 이미 AutoCloseable을 구현하거나 확장해뒀다. 

<br>

아래 코드는 InpuStream 코드로 AutoCloseable의 자식 인터페이스인 Closeable을 구현한 모습이다. 

```java
public abstract class InputStream implements Closeable {
	...
}

public interface Closeable extends AutoCloseable {
    public void close() throws IOException;
}
```

<br>

이렇듯 회수해야 하는 자원에 AutoCloseable 인터페이스를 구현해 놓으면 아래과 같이  try-with-resources를 쓸 수 있는 것이다. 

`try에 자원 객체를 전달하면 try 블록이 끝나면 자동으로 close 메소드를 호출해줘서 자원을 회수해주는 것`이다. 

따라서 catch나 finally 블록에 자원 회수하는 코드 넣지 않아도 된다. 

try에는 복수의 자원 객체를 전달할 수 있다. 

<br>

위에서 try-finally를 사용한 코드들을 try-with-resources를 사용한 코드들로 바꿔보자. 

9-1 코드를 9-3 코드로, 9-2 코드를 9-4 코드로 바꾼 것이다. 

```java
    // 코드 9-3 try-with-resources - 자원을 회수하는 최선책! (48쪽)
    static String firstLineOfFile(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(
                new FileReader(path))) {
            return br.readLine();
        }
    }
```

```java
    // 코드 9-4 복수의 자원을 처리하는 try-with-resources - 짧고 매혹적이다! (49쪽)
    static void copy(String src, String dst) throws IOException {
        try (InputStream   in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        }
    }
```

<br>

## 👍 try-with-resources 장점

### 1️⃣ 장점 1
바뀐 코드를 보니 확실히 try-with-resources 버전이 `짧고 읽기 수월`하다. 

<br>

### 2️⃣ 장점 2
또한 try-finally의 문제점 중 하나였던 `예외가 스택 추적 내역에서 누락되는 문제를 해결`해준다. 

즉 문제를 진단하기도 훨씬 좋다는 것이다. 

어떻게 문제 진단하기 좋은지 보자. 

<br>

자원을 사용하는 곳과 close 메소드가 자동으로 호출되는 곳 2군데에서 모두 예외가 발생하면 `close에서 발생한 예외는 숨겨지고` 자원 사용한 곳에서 발생한 예외가 기록된다고 한다. 

여러 예외들이 숨겨질 수 있는데 이런 `숨겨진 예외들도 그냥 버려지지는 않는다.` 

스택 추적 내역에 ‘suppressed’ 라는 꼬리표를 달고 출력된다. 

자바 7에서 Throwable에 추가된 getSuppressed 메소드를 이용하면 프로그램 코드에서 가져올 수도 있다. 

<br>

### 3️⃣ 장점 3
또한 보통의 try-finally처럼 try-with-resources에서도 `catch절을 쓸 수 있다.` 

catch절 덕분에 `try문을 더 중첩하지 않고도 다수의 예외를 처리할 수 있다.` 

아래 코드처럼 사용하면 된다. 

아래 코드는 9-3 코드에서 예외 터지면 예외 던지는 대신 기본값 반환하게 처리한 것이다. 

```java
    // 코드 9-5 try-with-resources를 catch 절과 함께 쓰는 모습 (49쪽)
    static String firstLineOfFile(String path, String defaultVal) {
        try (BufferedReader br = new BufferedReader(
                new FileReader(path))) {
            return br.readLine();
        } catch (IOException e) {
            return defaultVal;
        }
    }
```

<br>

따라서 ***`try-with-resouces의 장점`*** 을 정리하면 아래와 같다. 

1. 코드가 짧고 읽기 수월하다. 

2. 예외가 스택 추적 내역에서 누락되는 문제를 해결해 문제 진단하기 좋다. 

3. 보통 try-finally처럼 catch절 쓸 수 있어 try문 중첩하지 않고 다수의 예외 처리할 수 있다. 

<br>

## 📣 결론

꼭 회수해야 하는 자원을 다룰 때에는 try-finally말고 `반드시 try-with-resources를 사용`하자. 

`코드가 더 짧고 분명해지고 만들어지는 예외 정보도 훨씬 유용`하다. 

try-finally로 작성하면 코드 지저분해지는 경우라도 try-with-resources로는 정확하고 쉽게 자원 회수할 수 있다. 

<br>

## 📍 references

- [try-finally]([https://velog.io/@idkwhattodo/Java-예외처리-throw-try-catch-finally](https://velog.io/@idkwhattodo/Java-%EC%98%88%EC%99%B8%EC%B2%98%EB%A6%AC-throw-try-catch-finally))
- [try-with-resources 1](https://ryan-han.com/post/java/try_with_resources/)
- [try-with-resources 2](https://mangkyu.tistory.com/217)