# 😎 Item 9 try-finally 보다 try-with-resouces를 사용하라

<br>

>이 글은 Item8을 보고난 후에 보는 것을 추천드립니다.

## 🎾 try-finally

자바에선 `close` 메서드를 호출해 직접 닫아줘야 하는 자원이 많이 존재합니다.
 이러한 자원은 파일 시스템 접근, 네트워크 연결 또는 데이터베이스 연결 등이 있습니다. 

예를 들면 아래와 같습니다.

- 파일 I/O 관련 클래스
    - FileInputStream
    - FileOutputStream
    - RandomAccessFile
    - BufferedReader
    - BufferedWriter
    - FileReader
    - FileWriter
- 네트워크 I/O 관련 클래스
    - Socket
    - ServerSocket
    - DatagramSocket
    - URLConnection
- 데이터베이스 연결 관련 클래스
    - Connection
    - ResultSet
    - Statement
    - PreparedStatement

>이중에서 `BufferedReader`를 사용한 예시를 먼저 보겠습니다.

```java
public class TopLine {
    static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
}
```
>위 코드처럼 구현할 수 있지만 만약 자원이 하나 더 생기면 훨씬 복잡해집니다.
```java
public class Copy {
    private static final int BUFFER_SIZE = 8 * 1024;

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
}
```

정리해보면 try-finally의 단점은 2가지가 존재합니다.

1. 가독성이 좋지 않다.
위 두 예제에서 보았듯이 자원이 하나가 더 추가되자 훨씬 복잡해졌고 여기서 더 추가되면 가독성은 더욱 나빠지게 됩니다.

2. 스택 추적이 어려울 수 있다.

`firstLineOfFile` 메서드를 보면  예외는 try 블록과 finally 블록 모두에서 발생할 수 있습니다.     
예를 들어 기기에 물리적인 문제가 생긴다면 firstLineOfFile메서드 내의 `readLine`메서드가 예외를 던지고, 같은 이유로 `close`메서드도 실패합니다. 이때 예외가 두개가 생기는데 close 예외가 readLine 예외를 얻어버리는 불상사가 일어납니다.
식제로 스택 추적 내역에 첫 번째 예외에 관한 내용은 등장하지 않게 되고, 이는 실제 시스템에서의 디버깅을 몹시 어렵게 합니다.

## 🚀 try-with-resources의 등장 두둥

이전에 보았던 try-finally의 문제들은 try-with-resources에서 해결됩니다.  

try-with-resources는 자바 7에서 도입된 구문으로, 자동 리소스 관리(ARM, Automatic Resource Management)를 지원합니다.     
이 구문은 try 블록을 사용하여 자원을 사용하는 코드를 감싸고, 자원 사용이 끝나면 자동으로 리소스를 해제(즉, close 메서드를 호출)해줍니다. 이 방식은 리소스를 명시적으로 해제하는 코드를 작성할 필요가 없어 코드를 더 간결하고 안전하게 만들어 줍니다.

>기본 구문
```java
try (ResourceType resource = new ResourceType()) {
    // 리소스를 사용하는 코드
}
```
여기서 ResourceType은 java.lang.AutoCloseable 인터페이스 또는 java.io.Closeable 인터페이스를 구현해야 합니다. 이 두 인터페이스는 close() 메서드를 정의하고 있으며, try-with-resources 구문이 종료될 때 자동으로 이 close() 메서드가 호출됩니다.

>파일을 읽는 예시입니다.
```java
try (FileReader fr = new FileReader("file.txt");
     BufferedReader br = new BufferedReader(fr)) {
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
}
```

이 예시에서 FileReader와 BufferedReader는 둘 다 Closeable 인터페이스를 구현하고 있습니다.   
따라서 try-with-resources 구문이 종료되면, BufferedReader와 FileReader의 close() 메서드가 자동으로 호출됩니다. 이는 파일을 명시적으로 닫는 코드를 작성할 필요가 없음을 의미합니다.   


>이전에 보았던 예제에서 처럼 2개 이상의 자원을 사용하는 경우에도 굉장히 깔끔합니다.  
```java
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```

또한, try-finally에 비해 문제를 진단하기 편합니다. readLine과 close 둘 다에서 예외가 발생할 경우, 스택 추적에는 readLine에서 발생한 예외가 먼저 표시되며, close에서 발생한 예외는 'suppressed' 꼬리표와 함께 나중에 출력됩니다. 이는 첫 번째 예외부터 확인할 수 있게 해줍니다.
