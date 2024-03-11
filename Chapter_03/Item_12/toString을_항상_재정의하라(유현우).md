toString의 규약은 “모든 하위 클래스에서 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보로 재정의하라”이다.

## t**oString을 왜 재정의 해야할까?**

**Object.toString()**

- `클래스_이름@16진수로_표시한_해시코드`를 반환한다

```java
public String toString() {
		return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

**Object.toString()** 을 재정의하지 않았을 때 

```java
public class Book {
    private String title;
    public Book(String title) {
        this.title = title;
    }
    public static void main(String[] args) {
        Book book = new Book("함께 자라기");
        System.out.println(book); // com.example.test.item12.Book@816f27d
    }
}
```

- `com.example.test.item12.Book@816f27d` 로 출력되는 것 보다 재정의를 해줘서 `Book{title='함께 자라기'}` 과 같이 객체의 정보를 직접 알려주는 형태가 훨씬 유익하다.

### toString은 자동으로 불린다?

- `println`, `printf`, `문자열 연산자(+)`, `assert` 구문에 넘길 때 혹은 `디버거가 객체를 출력`할 때 `자동`으로 불린다.

```java
System.out.println(book)
```

- 우리가 방금 호출했을 때도 toString()이 호출된 것을 볼 수 있었다.

**`java.io.PrintStream` println() 메서드만 확인해서 보면**

```java
public void println(Object x) {
        String s = String.valueOf(x);
        if (getClass() == PrintStream.class) {
            // need to apply String.valueOf again since first invocation
            // might return null
            writeln(String.valueOf(s));
        } else {
            synchronized (this) {
                print(s);
                newLine();
            }
        }
}
```

String의 `valueOf`메서드를 통해서 String 형태로 변환해주는 것을 볼 수 있다.

```java
public static String valueOf(Object obj) {
		return (obj == null) ? "null" : obj.toString();
}
```

- 여기서 toString을 호출하고 있었다.

### 컬렉션을 이용하는 경우

```java
public static void main(String[] args) {
		Map<String, Book> books = new HashMap<>();
		books.put("김창준님", new Book("함께 자라기"));
		System.out.println(books); // {김창준님=함께 자라기}
}
```

- map 객체의 경우 `{김창준님=com.example.test.item12.Book@816f27d}` 보다 `{김창준님=함께 자라기}` 라는 메시지가 훨씬 가독성이 높다.

## **어떻게 재정의해야 할까?**

### **객체가 가진 주요 정보 모두를 반환하는게 좋다.**

- x,y 좌표를 나타내는 Point에서 x,y 중에서 하나만 반환하면 좌표를 확인할 수 없다.

```java
class Point {
    private int x;
    private int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public String toString() {
        return "Point{" +
                "x=" + x +
                ", y=" + y +
                '}';
    }
}
```

- 만약 객체가 거대하거나 객체의 상태가 문자열로 표현하기에 적합하지 않다면 요약정보를 담아야 한다.
    - `ex) 맨해튼 거주자 전화번호부(총 1487536개)`

### **toString을 구현할 때면 반환값의 포맷을 문서화할지 정해야 한다.**

- 전화번호부나 행렬 같은 값 클래스라면 문서화를 권장한다. 포맷을 명시하면 객체는 표준적이고, 명확하고, 사람이 읽을 수 있게 된다.

```java
public String toString() {
    return String.format("%s-%s-%s", areaCode, prefix, lineNumber);
}
```

```java
// 포맷 적용 전,
PhoneNumber{areaCode='031', prefix='789', lineNumber='1234'}

// 포맷 적용 후,
031-789-1234
```

- 포맷을 하면 읽기 편하고 좋지만, 포맷을 한번 명시하면 (그 클래스가 많이 쓰인다면) 평생 그 포맷에 얽매이게 된다. (수정이 어렵다)
- 반대로 포맷을 명시하지 않는다면 향후 릴리스에서 정보를 더 넣거나 포맷을 개선할 수 있는 유연성을 얻게 된다.

### 포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자

위의 예제 클래스에서 areaCode, prefix, lineNumber용 **접근자를 제공하지 않으면** 이 정보가 필요한 프로그래머는 toString의 반환값을 파싱해야 한다.

- 성능이 떨어지고 불필요한 작업이다.
- 향후 포맷을 바꾸면 시스템이 망가지는 결과를 초래할 수 있다.
- 변경될 수 있다고 문서화했더라도 포맷이 사실상 준 표준 API나 다름이 없어진다.

### **toString을 재정의 하지 않아도 되는 경우**

- 정적 유틸리티 클래스
- 이미 완변한 toString이 제공되는 Enum 타입

### **결론**

포맷까지는 하지 않더라도 객체의 값에 관해 아무것도 알려주지 않는 Object의 toString보다는 Intellij를 통해서 toString 생성을 하거나 Lombok의 @toString을 이용을 생활화 해보자
