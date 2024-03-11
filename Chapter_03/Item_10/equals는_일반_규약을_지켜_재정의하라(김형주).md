## 🤔 Object.equals()란?
```java
public boolean equals(Object obj) {
    return (this == obj);
}
```
- 두 객체의 참조값이 같다면 true를, 다르다면 false를 반환한다.
```java
Test test1 = new Test('123');
Test test2 = new Test('123');
System.out.println(test1.equals(test2)); // false
```
- 위 코드의 경우, 같은 값을 가졌어도 다른 참조값을 가진 객체이기 때문에 equals()는 false를 반환한다.
- 하지만 참조값이 아니라 객체의 값(논리적 동치성)을 비교하고 싶을 때는 어떻게 해야할까?

## ✅ Object.equals() 재정의
- 객체의 논리적 동치성을 확인하고 싶다면, Overide를 통해 재정의 해결할 수 있다.

하지만 재정의할때 equals()의 **일반 규약을 지켜서 재정의 해야한다.** 
- 한 클래스의 인스턴스는 다른 곳으로 빈번히 전달된다.
- 많은 클래스는 전달받은 객체가 **equals 규약을 지킨다고 가정하고 동작한다.**

#### equals()의 일반 규약
1. 반사성 : x.equals(x)는 항상 true이다.
2. 대칭성 : x.equals(y)가 true 이면 y.equals(x) true 이다.
3. 추이성 : x.equals(y)가 true이고 y.equals(z)가 true이면, x.equals(z)도 true이다.
4. 일관성 : x.equals(y)를 매번 실행해도 항상 같은 값을 출력한다.
5. null-아님 : x.equals(null)은 false이다.

규약을 예시와 함께 살펴보자.

## 1️⃣ 반사성 (Reflexivity)
- 객체가 자기 자신과 같아야 한다는 것을 의미한다.

## 2️⃣ 대칭성 (symmetry)
- 두 객체의 서로 동치 여부는 똑같아야 한다는 것을 의미한다.
- x ,y 객체가 있을때 x가 y와 같다면, y도 x와 같아야 한다.

#### 위배 예시
```java
public final class CaseInsensitiveString {
	private final String s;
    
    public CaseInsensitiveString(String s) {
    	this.s = Objects.requireNonNull(s);
    }
    // 대칭성 위배!
    @Override public boolean equals(Object o) {
    	if (o instanceof CaseInsensitiveString)
        	return s.equalsIgnoreCase(
            	((CaseInsensitiveString) o).s);
        if (o instanceof String) // 한 방향으로만 작동한다!
        	return s.equalsIgnoreCase((String) o);
       	return false;
    }
    
    public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        String s = "polish";
        System.out.println(cis.equals(s));  //true
        System.out.println(s.equals(cis));  //false
    }
}
```
- cis.equals(s)의 경우 true가 반환된다.
- 하지만 s.equals(cis)의 경우 False를 반환하여 대칭성을 성립하지 못한다.

```java
public boolean equalsIgnoreCase(String anotherString) {
    return (this == anotherString) ? true
        : (anotherString != null)
        && (anotherString.length() == length())
        && regionMatches(true, 0, anotherString, 0, length()); //ignoreCase가 true
}
```
- 이러한 현상이 발상한 이유를 살펴보면 equalsIgnoreCase에서 regionMatches의 ignoreCase가 true로 되어있다.
- 이는 대,소문자를 구분하지 않겠다는 것을 의미하며, 즉 CaseInsensitiveString.equals()는 대, 소문자를 무시하고 글자만 같다면 true를 반환하고 있다.
- 하지만 String.equals는 대,소문자도 구분하여 동치결과를 반환하기 때문에 대칭성이 깨진다.

```java
public static void main(String[] args) {
    List<CaseInsensitiveString> list = new ArrayList<>();
    list.add(cis);
    System.out.println(list.contains(polish)); //false
}
```
- 위 경우에도 false를 반환하고 있다.
```java
@Override public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString &&
            ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```
- String과 연동하겠다는 꿈은 버리면 다음과 같은 간단한 코드로 할 수 있다.

## 3️⃣ 추이성 (transitivity)
- x=y, y=z이면 x=z이다.
- 간단하지만 자칫하면 어기기 쉽고, 상위클래스를 상속 후 새로운 값을 넣는 과정에서 주로 발생한다.
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
        if (!(o instanceof Point)) 
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y; // x,y값 동치 확인
    }
}
```
```java
public enum Color{
    RED,
    GREEN,
    BLUE
}
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
    
    public static void main(String[] args) {
        Point p = new Point(1, 2);
		ColorPoint cp = new ColorPoint(1, 2, ColorPoint.Color.RED);
		System.out.println(p.equals(cp)); // true
		System.out.println(cp.equals(p)); // false
    }
}
```
- 기존에 2차원 좌표를 나타내는 Point가 있었고 Point를 상속해 색상값을 추가한 ColorPoint가 있다고 가정해보자.
- p.equals()의 경우, ColorPoint는 color의 인스턴스이기 때문에 if문을 통과한 후 x,y값이 같은 것을 확인 true를 반환한다.
- 하지만 cp.equals()의 경우 color는 ColorPoint의 인스턴스가 아닌 상위클래스이기 때문에 if문으로 인해 항상 false를 반환한다.

이로써 위 코드는 ``대칭성(symmetry)``을 위배한다.

```java
@Override public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;
    
    // o가 일반 Point면 색상을 무시하고 비교한다.
    if (!(o instanceof ColorPoint))
        return o.equals(this);
    
    // o가 ColorPoint면 색상까지 비교한다.
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```
```java
public static void main(String[] args) {
    ColorPoint p1 = new ColorPoint(1, 2, Color.RED); 
    Point p2 = new Point(1, 2);
    ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
    
    //대칭성을 성립
    System.out.println(p2.equals(p1)); // true
    System.out.println(p1.equals(p2)); // true
    //하지만 추이성은 위배
    System.out.println(p1.equals(p2)); // true
	System.out.println(p2.equals(p3)); // true
	System.out.println(p1.equals(p3)); // false
}
```
- 이번에는 일반 Point이면 색상을 무시하여 비교하는 코드를 추가해보았다.
- 그랬더니 p1.equals(p2)와 p2.equals(p1)가 둘다 true를 반환하면서 대칭성은 성립되었다.
- 하지만 추이성을 확인하는 과정에서 p1.equals(p3)가 색상이 달라 false가 반환되면서 추이성은 위배한다는 것을 확인할 수 있다.

이로써 위 코드는 ``추이성 (transitivity)``을 위배한다.

### 그렇다면 해결방법은 무엇인가?
- **없다.** 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.
```java
@Override public boolean equals(Object o){
  if(o == null || o.getClass() != getClass())
    return false;
  Point p = (Point) o;
  return p.x == x && p.y == y;
}
```
- 이번에는 추이성을 위해 instanceof 대신에 getClass를 사용하여 같은 객체끼리 비교할 때만 true를 반환하도록 했다.
- 추이성은 지켰지만 해당 코드는 ``리스코프 치환 원칙``을 위배했다.

### 리스코프 치환 원칙 ( Liscov Substitution Principle )
```java
ColorPoint p1 = new ColorPoint(1, 2, Color.RED); 
Point p2 = new Point(1, 2);
// getClass()로 인해 ColorPoint는 Point로 활용될 수 없음 
System.out.println(p1.equals(p2)); //false
```
- 부모 객체와 자식 객체가 있을 때, 부모 객체를 호출하는 동작에서 **자식 객체가 부모 객체를 완전히 대체할 수 있다는 원칙**
- ColorPoint는 Point의 인스턴스이기 때문에 Point로써 활용될 수 있어야 한다.
- 하지만 getClass로 같은 객체끼리 비교로만 제한해버리면 ColorPoint는 Point로써 활용될 수 없게 된다. 

### 우회 방법
- 구체 클래스의 하위 클래스에서 값을 추가할 방법은 없지만, 괜찮은 우회 방법은 존재한다.
- 상속 대신 컴포지션을 사용하는 것이다.

### 컴포지션 ( composition )
- 구성이라는 뜻으로, public 혹은 private필드로 **클래스의 인스턴스를 참조하여 해당 클래스를 구성하는 것**을 말한다. 
```java
public class ColorPoint {
    // Point와 Color 클래스 인스턴스로 ColorPoint 클래스 구성
    private final Point point; 
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = color;
    }

    /**
     * 이 ColorPoint의 Point 뷰를 반환한다.
     */
    public Point asPoint() {
        return point;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
}
```
## 4️⃣ 일관성 ( consistency )
- 두 객체가 같다면 영원히 같아야 하고, 다르다면 영원히 달라야 한다.

자원에 따라 일관성이 깨지는 것처럼 보일 수 있다.
- ```java.net.URL```의 equals()는 주어진 URL과 매핑된 호스트의 IP주소를 이용해 비교한다.
- 하지만 호스트 이름은 같아도 IP 주소는 언제나 바뀔 수 있기 때문에 반복해서 equals()를 호출시 다른 결과가 나올 수 있다.

#### 즉, equals()의 판단에 신뢰할 수 없는 자원은 끼어들게 하지말자.

## 5️⃣ Null-아님
- 모든 객체가 null이 아니여야 한다는 것을 의미한다.
```java
@Overrid public boolean equals(Object o){
	if(o==null)
		return false;
	(이하 생략)
}
```
- 객체가 null인지 아닌지 if문으로 확인하는 코드이다.
- 하지만 논리적 동치성을 검사하는 순서는 다음과 같다.
>1. 전달받은 객체을 올바르게 형변환한다.
>2. 이후 필드 값을 비교한다.

```java
@Override
public boolean equals(Object o){
	if(!(o instanceof MyType))
		return false;
	(이하 생략)
}

```
- 이처럼 형변환이 먼저 이루어져야 하기 때문에 instanceof로 형변환이 가능한 인스턴스인지 확인해보는 것이 우선이다.
- 그리고 instanceof는 null을 자동으로 false를 반환하기 때문에 자동으로 null도 방지할 수 있다.

## 🙆‍♂️ 양질의 equals 메서드 만드는 순서

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 **핵심**필드들이 모두 일치하는지 하나씩 검사한다.

## 🙅‍♂️ equals() 재정의를 하지 않는 경우
-  equals()를 무분별하게 재정의하는 것은 좋지 않다.
-  아래와 같은 상황에는 재정의하지 않는 것이 최선이니 명심하자.

1. 객체가 값을 표현하는 것이 아니라 동작하는 개체를 표현하는 경우
2. 인스턴의 "논리적 동치성"을 검사할 일이 없는 경우
3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는 경우
4. 클래스가 private 혹은 package-private이거나, equals()를 호출할 일이 없는 경우

### 📝 Reference
[RegionMatches](https://learn.microsoft.com/ko-kr/dotnet/api/java.lang.string.regionmatches?view=xamarin-android-sdk-13#java-lang-string-regionmatches(system-boolean-system-int32-system-string-system-int32-system-int32))<br>
[리스코프 치환원칙](https://velog.io/@harinnnnn/OOP-%EA%B0%9D%EC%B2%B4%EC%A7%80%ED%96%A5-5%EB%8C%80-%EC%9B%90%EC%B9%99SOLID-%EB%A6%AC%EC%8A%A4%EC%BD%94%ED%94%84-%EC%B9%98%ED%99%98-%EC%9B%90%EC%B9%99-LSP)<br>
[상속과 컴포지션](https://velog.io/@vino661/%EC%83%81%EC%86%8D%EA%B3%BC-%EC%BB%B4%ED%8F%AC%EC%A7%80%EC%85%98%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C)


