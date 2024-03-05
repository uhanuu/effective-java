```InputStream```, ```OutputStream```,```java.sql.Connection```
- 위 3가지 객체는 클라이언트가 close 메서드를 호출해 직접 해제해야하는 대표적인 객체이다.
- DB connection이나 File Stream같은 외부 자원을 연결해서 사용하는 객체라는 점이 공통점이다.

## 🤔 이러한 객체는 어떻게 close메서드를 호출할까?
#### 1️⃣. try-catch-finally
try-catch-finally의 구조와 예시는 다음과 같다.
- try : 예외가 발생할 가능성이 있는 구문
- catch : try 구문에서 예외가 발생하면 실행시키는 코드
- finally : 꼭 실행해야하는 코드
- Example
    ```java
    //try-catch-finally 예시
    public static String firstLineOfFile(String path) throw IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            br.close();
        }
    }
    ```
#### 2️⃣. try-catch-finally의 문제점
전통적으로 close()호출을 보장하던 방식이지만, 해당 방식은 몇가지 문제점을 가지고 있다.
1. 자원이 여러개 사용되면 코드의 가독성이 떨어진다.
    ```java
    static void copy(String src, String dst) throws IOException {
    	InputStream in = new FileInputStream(src);
    	try {
    		OutputStream out = new FileOutputStream(dst);
    		// 중첩 발생
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
    - copy 메서드에 2가지 자원 InputStream, OutputStream이 사용되고 있다.
    - 해당 객체들의 close()를 보장하기 위해 각 객체마다 try-catch-finally을 적용하면 중첩이 발생한다.
    - 이렇듯 자원이 사용되는 메서드에서의 try-catch-finally은 코드를 지저분하게 만들고 전체적인 코드의 가독성을 떨어트린다.
    
2. finally에서도 예외가 발생해, 객체가 해제되지 않은 경우도 발생한다.
    ```java
    public class Bomb {
        boolean power = false;
    
        public void activate(){
            power = true;
            System.out.println("폭탄을 활성화 합니다. 삐빅 삐빅");
        }
        public void close(){
            power=false;
            System.out.println("폭탄을 회수합니다. 삐비....픽");
        }
    
        public void process() throws Exception{
            System.out.println("작동중 ...");
            throw new Exception("예외발생! 예외발생!!!! !@#!@#!@#");
        }
    
        public void check() throws Exception{
            System.out.println("폭탄 체크중 ...");
            throw new Exception("폭탄 상태 체크중 예외발생!!!! !@#!@#!@#");
        }
    }
    public class Test {
        public static void main(String[] args) throws Exception {
            Bomb bomb = new Bomb();
            try{
                bomb.activate();
                //예외발생
                bomb.process();
            }finally{
                //예외발생
                bomb.check();
                bomb.close();
            }
        }
    }
    ```
    ```
    실행결과: 작동중 ...
            폭탄 체크중 ...
    ```
    - try와 finally 구문에 process()와 check()로 예외를 발생시켜 보았다.
    - 그 결과, check()의 예외발생으로 종료되면서 bomb.close()가 실행되지 않아 "폭탄을 회수합니다."가 출력되지 않은 모습니다.
    - 이처럼, finally에서 예외가 발생한다면 close()가 호출되지 않아 자원이 회수되지 않은 경우가 발생할 수 있다.

3. 디버깅을 어렵게 만든다.
    ```java
    public class Test {
        public static void main(String[] args) throws Exception {
           try{
               install();
           }catch(Exception e){
               e.printStackTrace();
           }
        }
    
        public static void install() throws Exception{
            Bomb bomb = new Bomb();
            try{
                bomb.activate();
                //예외 발생
                bomb.process();
            }finally{
                //예외 발생
                bomb.check();
                bomb.close();
            }
        }
    }
    ```
    ```
    추적 결과
    java.lang.Exception: 폭탄 상태 체크중 예외발생!!!! !@#!@#!@#
            	at Item_9.ex1.Bomb.check(Bomb.java:22)    //check 예외 지점만 출력
            	at Item_9.ex1.Test.install(Test.java:20)
            	at Item_9.ex1.Test.main(Test.java:6)
    ```
    - Exception.printStackTrace()를 통해서 예와 관련 정보를 출력해 보았다.
    - 그 결과, process() 예외는 finally구문의 check() 예외에 묻혀 보이지 않는 것을 확인할 수 있다.
    - 디버깅을 위해 첫번째 예외 정보가 필요하지만, 관련 정보는 남아있지 않아 디버깅을 어렵게 만든다.

## 🤔  문제점이 생각보다 많은데... 다른 방법은 없을까?
#### 1️⃣. try-with-resources
- 그래서 자바 7이후, 문제점을 보완한  try-with-resources가 등장했다.
- try-with-resources 구조를 사용하려면 자원이 AutoCloseable 인터페이스를 구현해야한다.
    - AutoCloseable란, close() 메서드를 가진 인터페이스이다.
    - AutoCloseable.close()는 try-with-resource문으로 관리되는 구현체의 close()를 자동 호출해준다.
- try(..)에 해제할 객체들을 명시한 객체들은 예외가 발생하면 각 close()를 자동호출 해준다.

#### 2️⃣. try-with-resources 적용해보기
```java
public class Bomb implements AutoCloseable{
    boolean power = false;

    public void activate(){
        power = true;
        System.out.println("폭탄을 활성화 합니다. 삐빅 삐빅");
    }
    public void close() throws Exception{
        power=false;
        System.out.println("폭탄을 회수합니다.");
        throw new Exception("close() 예외발생");
    }

    public void process() throws Exception{
        System.out.println("작동중 ...");
        throw new Exception("process() 예외발생");
    }
}
```
- try-with-resources을 사용해보기 위해 AutoCloseable 인터페이스로 구현한 Bomb 클래스를 만들었다.
```java
public class Test {
    public static void main(String[] args) throws Exception {
       try{
           install();
       }catch(Exception e){
           e.printStackTrace();
       }
    }

    public static void install() throws Exception{
        try ( Bomb bomb = new Bomb();){
            bomb.activate();
            //예외발생
            bomb.process();
        }
    }
}
```
- 위 코드에서는 한개의 자원만 사용하였지만, try(..)에 여러개의 자원을 넣을 수 있다.
- Ex. try( Bomb bomb = new Bomb(); Nuclear nuclear = new Nuclear() )
```
출력 결과: 폭탄을 활성화 합니다. 삐빅 삐빅
         작동중 ...
         폭탄을 회수합니다. //close() 자동 호출 
```
- 실행해본 결과, close()를 작성하지 않았음에도 try-catch-finally와 달리 close()가 자동호출되어 "폭탄을 회수합니다."가 출력된 것을 볼 수 있다.
```
추적 결과
java.lang.Exception: process() 예외발생
        	at Item_9.ex1.Bomb.process(Bomb.java:18)
        	at Item_9.ex1.Test.install(Test.java:16)
        	at Item_9.ex1.Test.main(Test.java:6)
        	Suppressed: java.lang.Exception: close() 예외발생     // 집어삼켜진 예외도 출력
            		at Item_9.ex1.Bomb.close(Bomb.java:13)
            		at Item_9.ex1.Test.install(Test.java:13)
            		... 1 more
```
- 또한, 집어삼켜진 예외들을 Suppressed의 꼬리표로 볼 수 있게 되었다.

#### ✏️ 정리해보면 try-with-resources는 다음과 같은 장점을 가지고 있다.
1. close()를 직접 명시하지 않아도 호출되기 때문에, 코드가 간결하고 가독성이 좋다.
2. 여러개의 자원을 사용할 수 있다.
3. 이전 예외가 집어삽켜지지 않아, 디버깅에 좋다.


## 👨‍⚖️ 결론
#### 객체를 회수해야 할 때는 무조건 try-with-resources를 사용하자.

###Reference
[try-catch와 try-with-resources의 차이점 ](https://codechacha.com/ko/java-try-with-resources/)<br>
[try-with-resources](https://www.baeldung.com/java-try-with-resources)<br>
[getSuppressed](https://lsmman.tistory.com/51)<br>

