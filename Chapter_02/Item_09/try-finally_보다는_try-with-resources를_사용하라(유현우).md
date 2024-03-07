# item 9 - try-finally 보다는 try-with-resources를 사용하라
### close 메서드를 호출해 직접 자원을 닫아주는 경우 자바7 버전 부터 **try-finally는 더이상 최선의 방법이 아니다.**

```java
static String firstLineOfFile(String path) throws IOException {
		BufferedReader br = new BufferedReader(new FileReader(path));
		try {
				return br.readLine();
		} finally {
				br.close();
		}
}
```

- InputStrem, OutputStrem, java.sql.Connection등

### ❗️try-finally의 단점

### 1. 자원이 둘 이상이면 try-finally 방식은 가독성이 좋지 않다.

```java
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

- 먼저 OutputStream 을 반납하고 그 뒤에 InputStream 을 반납한다.

### **2. 자원을 반납하지 못하는 경우가 있을 수 있다.**

```java
static void copy(String src, String dst) throws IOException {
      InputStream in = new FileInputStream(src);
      OutputStream out = new FileOutputStream(dst);

      try {
         byte[] buf = new byte[BUFFER_SIZE];
         int n;
         while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);

      } finally {
         in.close(); // 예외 발생
         out.close(); // 자원 회수 못함
      }
   }
```

- **try - catch 블럭을 중첩** 하면 `in.close();` 에서 에러가 나더라도 `out.close();` 를 실행하게 된다.

### 3.  예외가 먹혀서 **에러 스택 트레이스에서 누락된다.**

`readLine()`, `close()`시 예외가 발생한다고 가정한다.

```java
public class BadBufferedReader extends BufferedReader{

    public BadBufferedReader(Reader in, int sz) {
        super(in, sz);
    }

    public BadBufferedReader(Reader in) {
        super(in);
    }

    @Override
    public String readLine() throws IOException {
        throw new CharConversionException();
    }

    @Override
    public void close() throws IOException {
        throw new StreamCorruptedException();
    }
}
```

**😮‍💨 가장 나중에 발생한 예외만 보인다.**

```java
public class Application {

    public static void main(String[] args) {
        try {
            Application.read();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void read() throws IOException {
        BadBufferedReader badBufferedReader = null;
        try {
            badBufferedReader = new BadBufferedReader(new InputStreamReader(System.in));
            badBufferedReader.readLine(); // 첫 번쨰 예외 발생
        } finally {
            badBufferedReader.close(); // 두 번째 예외 발생
        }
    }
}
```

**⚠️ stack trace**

- readLine() 메서드에서 발생한 예외는 확인할 수 없다.

```java
java.io.StreamCorruptedException
	at com.example.test.item9.BadBufferedReader.close(BadBufferedReader.java:21)
	at com.example.test.item9.Application.read1(Application.java:25)
	at com.example.test.item9.Application.main(Application.java:11)
```

**read 메서드에서 catch로 stack trace를 찍는 방법도 있다.**

```java
public static void read() throws IOException {
        BadBufferedReader badBufferedReader = null;
        try {
            badBufferedReader = new BadBufferedReader(new InputStreamReader(System.in));
            badBufferedReader.readLine(); // 20번
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            badBufferedReader.close(); // 43번
        }
    }
```

**⚠️ stack trace**

```java
/Users/ryu/Library/Java/JavaVirtualMachines/corretto-17.0.4.1/Contents/Home/bin/java -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=52184:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/ryu/개발/test/out/production/classes:/Users/ryu/개발/test/out/production/resources:/Users/ryu/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-starter/3.2.2/dc04714f9295297f92fa8099eb51edc54dbe67db/spring-boot-starter-3.2.2.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-autoconfigure/3.2.2/5c407409f8d260a4bc6e173d16fc3b36e6adec21/spring-boot-autoconfigure-3.2.2.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot/3.2.2/9f274d1bd822c4c57bb5b37ecae2380b980f567/spring-boot-3.2.2.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-starter-logging/3.2.2/3347c3b1cec6cf2d5fa186d1e49d2f378a6b7cae/spring-boot-starter-logging-3.2.2.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/jakarta.annotation/jakarta.annotation-api/2.1.1/48b9bda22b091b1f48b13af03fe36db3be6e1ae3/jakarta.annotation-api-2.1.1.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/org.springframework/spring-core/6.1.3/a002e96e780954cc3ac4cd70fd3bb16accdc47ed/spring-core-6.1.3.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/org.yaml/snakeyaml/2.2/3af797a25458550a16bf89acc8e4ab2b7f2bfce0/snakeyaml-2.2.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/org.springframework/spring-context/6.1.3/c63f038933701058fd7578460c66dbe2d424915/spring-context-6.1.3.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/ch.qos.logback/logback-classic/1.4.14/d98bc162275134cdf1518774da4a2a17ef6fb94d/logback-classic-1.4.14.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/org.apache.logging.log4j/log4j-to-slf4j/2.21.1/d77b2ba81711ed596cd797cc2b5b5bd7409d841c/log4j-to-slf4j-2.21.1.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/org.slf4j/jul-to-slf4j/2.0.11/279356f8e873b1a26badd8bbb3284b5c3b22c770/jul-to-slf4j-2.0.11.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/org.springframework/spring-jcl/6.1.3/a715e091ee86243ee94534a03f3c26b4e48de31e/spring-jcl-6.1.3.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/org.springframework/spring-aop/6.1.3/4d9bd4bd9b8bedf9ef151b45c79766b336117b9a/spring-aop-6.1.3.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/org.springframework/spring-beans/6.1.3/c2df4210e796d3a27efc1f22621aa4e2c6cd985f/spring-beans-6.1.3.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/org.springframework/spring-expression/6.1.3/7c35fc3d7525a024fdde8a5d7597a6a8a4e59d7/spring-expression-6.1.3.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/io.micrometer/micrometer-observation/1.12.2/e082b05a2527fc24ea6fbe4c4b7ae34653aace81/micrometer-observation-1.12.2.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/ch.qos.logback/logback-core/1.4.14/4d3c2248219ac0effeb380ed4c5280a80bf395e8/logback-core-1.4.14.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/org.slf4j/slf4j-api/2.0.11/ad96c3f8cf895e696dd35c2bc8e8ebe710be9e6d/slf4j-api-2.0.11.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/org.apache.logging.log4j/log4j-api/2.21.1/74c65e87b9ce1694a01524e192d7be989ba70486/log4j-api-2.21.1.jar:/Users/ryu/.gradle/caches/modules-2/files-2.1/io.micrometer/micrometer-commons/1.12.2/b44127d8ec7b3ef11a01912d1e6474e1167f3929/micrometer-commons-1.12.2.jar com.example.test.item9.Application
java.io.CharConversionException
	at com.example.test.item9.BadBufferedReader.readLine(BadBufferedReader.java:16)
	at com.example.test.item9.Application.read1(Application.java:22)
	at com.example.test.item9.Application.main(Application.java:11)
java.io.StreamCorruptedException
	at com.example.test.item9.BadBufferedReader.close(BadBufferedReader.java:21)
	at com.example.test.item9.Application.read1(Application.java:27)
	at com.example.test.item9.Application.main(Application.java:11)
```

## **try-with-resouces 를 사용하자 (**자바 7)

try-finally 방식의 단점을 보완하기 위해서 try-with-resources를 사용하려면 AutoCloseable 인터페이스를 구현해야 한다.

- 자원 반납을 따로 하지 않아도 된다.

Java에서 자원에 접근할 때 사용하는 모든 구현체들은 Closeable을 구현하고 있다.

```java
public abstract class Reader implements Readable, Closeable { ... }
public abstract class Writer implements Appendable, Closeable, Flushable { ... }
public abstract class InputStream implements Closeable { ... }
public abstract class OutputStream implements Closeable, Flushable { ... }
```

`Closeable` 인터페이스는 `AutoCloseable` 인터페이스를 상속하고 있다. 

```java
// functional interface 아니다.
public interface Closeable extends AutoCloseable {
    public void close() throws IOException;
}
```

- 이전 try-finally 코드들은 전부 try-with-resources 사용이 가능하다.

### 1️⃣ 가독성이 좋다.

```java
static String firstLineOfFile(String path) throws IOException {
		try(BufferedReader br = new BadBufferedReader(new FileReader(path))) {
				return br.readLine();
		}
}
```

**자원이 둘 이상일 때도 try - catch 블럭을 중첩해서 사용하지 않아도 되기 때문에 깔끔하다.**

```java
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

### 2️⃣ 예외를 잡아먹지 않는다.

```java
public class Application {

    public static void main(String[] args) {
        try {
            Application.read();

        } catch (Exception e) { //11번
            e.printStackTrace();
        }
    }

    public static void read() throws IOException {
        try (BadBufferedReader badBufferedReader = new BadBufferedReader(new InputStreamReader(System.in))) {
            badBufferedReader.readLine();
        }
    }
}

```

**⚠️ stack trace**

```java
java.io.CharConversionException
	at com.example.test.item9.BadBufferedReader.readLine(BadBufferedReader.java:16)
	at com.example.test.item9.Application.copy(Application.java:31)
	at com.example.test.item9.Application.main(Application.java:11)
	Suppressed: java.io.StreamCorruptedException
		at com.example.test.item9.BadBufferedReader.close(BadBufferedReader.java:22)
		at com.example.test.item9.Application.copy(Application.java:30)
		... 1 more
```

**결론**

close() 메서드를 통해 자원을 회수해야 하는 경우 try-with-resources를 사용하자. 

- 가독성이 좋다.
- 쉽게 자원을 회수할 수 있다.
- 예외가 먹히지 않는다.
