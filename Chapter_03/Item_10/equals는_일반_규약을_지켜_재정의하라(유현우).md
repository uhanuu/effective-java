평상시 Intellij를 통해서 재정의하기 쉬웠지만 조심해야 하는 부분을 하나씩 알아보자

- 싱글톤 패턴과 enum은 근본적으로 단 하나만 존재 하기 때문에 equals 메서드가 필요하지 않다.
    - == 비교를 사용하면 된다.

## equals를 재정의 하지 않는게 좋은 경우

### 1. 각 인스턴스가 본질적으로 고유하다.

- 값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스들 (Thread)

### 2. 인스턴스의 ‘논리적 동치성’을 검사할 일이 없다.

- 대표적으로 문자열은 논리적 동치성을 검사한다.

**논리적 동치성이란?**

5천원 짜리 지폐가 2장이 있을 때 질문에 따라서 다르게 말할 수 있다. (논리적 동치성으로 사용함)

- 두 개의 지폐는 값이 같은가요? - 네 (논리적 동치성)
- 두 개의 지폐는 같은 지폐인가요? - 아니요 (고유 번호가 다름)

Pattern 클래스는 equals를 재정의하여 각 인스턴스가 같은 정규표현식을 나타내는지 검사할 수 있지만 굳이 재정의 할 필요가 없다.

```java
void patternTest() {
    final Pattern P1 = Pattern.compile("//.*");
    final Pattern P2 = Pattern.compile("//.*");

    System.out.println(P1.pattern().equals(P1.pattern())); // true
    System.out.println(P1.pattern().equals(P2.pattern())); // true
}
```

### **3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.**

- **List, Set** 의 상위 클래스에 **equals 메서드**가 구현되어있기 때문에 굳이 직접 구현할 필요가 없다.

**AbstractSet**

```java
public boolean equals(Object o) {
        if (o == this)
            return true;

        if (!(o instanceof Set))
            return false;
        Collection<?> c = (Collection<?>) o;
        if (c.size() != size())
            return false;
        try {
            return containsAll(c);
        } catch (ClassCastException | NullPointerException unused) {
            return false;
        }
    }
```

- `AbstractList`,  `AbstractMap`도 있다.

같은 Set인지 구분하기 위해서 사이즈, 내부 값들을 비교하면 되기 때문에 각각 true가 나오게 된다.

```java
public static void main(String[] args) {
        HashMap<Object, Object> hashMap = new HashMap<>();
        hashMap.put("hi", 1);
        TreeMap<Object, Object> treeMap = new TreeMap<>();
        treeMap.put("hi", 1);

        System.out.println(hashMap.equals(treeMap)); //true

				ArrayList<Object> arr = new ArrayList<>();
        arr.add("test");
        LinkedList<Object> link = new LinkedList<>();
        link.add("test");

        System.out.println(arr.equals(link)); //true

				HashSet<Object> hashSet = new HashSet<>();
        hashSet.add("test");
        TreeSet<Object> treeSet = new TreeSet<>();
        treeSet.add("test");

        System.out.println(hashSet.equals(treeSet)); // true
    }
```

### **4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.**

- 위험을 철저히 회피하는 성격이라면 아래처럼 작성 가능

```java
@Override 
public boolean equals(Object o){
    throw new AssertionError(); // 호출 금지
}
```

## equals는 언제 재정의해야 할까?

- 논리적인 동치성을 확인하고 싶은데 상위 클래스에서 재정의가 되어있지 않을 때

## **equals 메서드**를 재정의 할 때 따라야할 규약들

### 1. 반사성

- null이 아닌 모든 참조 값 x에 대해 x.equals(x)를 만족해야 한다.

### 2. 대칭성

- null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true이면, y.equals(x)가 true를 만족해야 한다.

```java
public final class CaseInsensitiveString {

    private final String str;
    
    public CaseInsensitiveString(String str) {
        this.str = Objects.requireNonNull(str);
    }
    
		// 대칭성 위배!
    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) {
            return str.equalsIgnoreCase(((CaseInsensitiveString) o).str);
        }
    
        if (o instanceof String) { // 한 방향으로만 작동한다.
            return str.equalsIgnoreCase((String) o);
        }
        return false;
    }
}

```

- `CaseInsensitiveString`  에서 `String`은 true `String`에서 `CaseInsensitiveString` 는 false가 나오게 된다.

### **3. 추이성**

- null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고, y.equals(z)가 true이면 x.equals(z)도 true이다.

```java
public class Point {

    private final int x;
    private final int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }
        Point p = (Point) o;
        return this.x == p.x && this.y == p.y;
    }
}
```

**상속을 하게 되면 문제가 발생할 수 있다.**

```java
public class ColorPoint extends Point {
   private final Color color;

   public ColorPoint(int x, int y, Color color) {
      super(x, y);
      this.color = color;
   }

	 @Override 
	 public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
   }

	 enum Color { RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET }
}
```

Point 클래스는 ColorPoint 타입이 아니기 때문에 위의 if 문에서 false가 반환되는 것을 볼 수 있다.

```java
public static void main(String[] args) {
		Point p = new Point(1, 2);
		ColorPoint cp = new ColorPoint(1, 2, ColorPoint.Color.RED);
		System.out.println(p.equals(cp)); // true
		System.out.println(cp.equals(p)); // false
}
```

**리스코프 치환 원칙**

추이성을 지키기 위해서 Point의 equals를 각 클래스들을 getClass를 통해서 같은 구체 클래스일 경우에만 비교하는 경우 **리스코프 치환 원칙을 지키지 못한다.**

```java
@Override
public boolean equals(Object o) {
    // getClass
    if (o == null || o.getClass() != this.getClass()) {
        return false;
    }

    Point p = (Point) o;
    return this.x == p.x && this.y = p.y;
}
```

- 이는 모든 객체 지향 언어의 동치관계에서 나타나는 근본적인 문제이며, 객체 지향적 추상화의 이점을 포기하지 않는 한 불가능하다.

**상속 대신 컴포지션을 사용해라**

```java
public class ColorPoint {

    private Point point;
    private Color color;

    public ColorPoint(int x, int y, Color color) {
        this.point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint() {
        return this.point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        ColorPoint cp = (ColorPoint) o;
        return this.point.equals(cp) && this.color.equals(cp.color);
    }
}
```

**Point 타입의 뷰를 제공하기 때문에 equals를 이용할 수 있다.**

- ColorPoint는 ColorPoint 끼리 equals 사용해야함

```java
public static void main(String[] args) {
        Point point1 = new Point(1, 2);
        Point point2 = new Point(1, 2);
        ColorPoint colorPoint = new ColorPoint(1, 2, Color.BLUE);
        System.out.println(point1.equals(colorPoint.asPoint())); // true
        System.out.println(colorPoint.asPoint().equals(point2)); // true
        System.out.println(point1.equals(point2)); // true
}
```

### 추상 클레스의 하위 클래스에서라면 equals의 규약을 지키면서도 값을 추가할 수 있다.

- 상위 클래스를 직접 인스턴스로 만드는 것이 불가능 하면 지금까지 이야기한 문제들은 모두 일어나지 않는다.

### **4. 일관성**

- null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.

**java.net.URL**

```java
public static void main(String[] args) throws MalformedURLException {
        URL url1 = new URL("www.xxx.com");
        URL url2 = new URL("www.xxx.com");

        System.out.println(url1.equals(url2)); // 항상 같지 않다.
    }
```

- URL과 매핑된 host의 IP주소를 이용해 비교하기 때문에 같은 도메인 주소라도 나오는 IP정보가 달라질 수 있기 때문에 반복적으로 호출할 경우 결과가 달라질 수 있다.

### **5. null-아님**

- null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false

```java
@Override 
public boolean equals(Object o) {
    // 인텔리제이를 통해서 생성하는 equals
    if (o == null || getClass() != o.getClass()) {
        return false;
    }
    
    // 책에서 추천하는 내용은 null 검사를 할 필요 없이 instanceof를 이용하라는 것이다.
    // instanceof는 두번째 피연산자(Point)와 무관하게 첫번째 피연산자(o)거 null이면 false를 반환하기 때문이다. 
    if (!(o instanceof Point)) {
        return false;
    }
}
```

- `o.equals(null) == true`인 경우는 상상하기 어렵지만, 실수로 `NullPointerException`을 던지는 코드 또한 허용하지 않는다.
- 이러한 `Exception`을 막기 위해서 여러가지 방법이 존재하는데, 그중 하나는 null인지 확인을 하고 `false`를 반환하는 것이다. 하지만 책에서는 다른 방법을 추천하고 있다.

### equals TIP

```java
// null 검사를 할 필요 없이 instanceof를 이용
if (!(o instanceof Point)) {
        return false;
}

// float와 double을 제외한 기본 타입 필드는 ==를 사용한다.
return this.x == p.x && this.y == p.y;

// null 값을 정상 값으로 취급한다면 Objects.equals로 NullPointException을 예방하자.
return Objects.equals(Object, Object);
```

### equals 메서드 재정의 했을 때 주의할 점

- **equals 를 재정의 할 때 hashCode 도 반드시 재정의하자**
    - **필요하다면 핵심필드를 빠짐 없이 비교하며 다섯 가지 규약을 지키자.**
- **어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우하기도 한다. 최상의 성능을 바란다면 다를 가능성이 더 크거나 비교하는 비용이 더 싼 필드를 먼저 비교하자.**
- **Object가 아닌 타입의 매개변수를 받는 equals 메서드는 선언하지 말자**
