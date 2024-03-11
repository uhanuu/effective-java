# 🚀 equals는 일반 규약을 지켜 재정의하라

Object는 객체를 만들 수 있는 구체 클래스지만 기봊거으로는 상속해서 사용하도록 설계되엇습니다.     
Object에서 final이 아닌 메서드(equals, hashCode, toString, clone, finalize)는 모두 재정의를 염두에 두고 설계된 것이라 재정의시 지켜야 하는 일반 규약이 명확히 정의되어 있습니다.  그래서 Object를 상속하는 클래스, 즉 모든 클래스는 이 메서드들을 일반 규약에 맞게 재정의 해야 합니다. 메서드를 잘못 구현하면 대상 클래스가 이 규약을 준수한다고 가정하는 클래스(HashMap과 HashSet 등)를 오동작하게 만들 수 있습니다.  

이번 Item10에서는 equals를 재정의할 때 지켜야할 일반 규약에대해 알아보겠습니다.

equals 메서드를 재정의할 때는 반드시 일반 규약을 따라야 합니다.  다음은 Object 명세에 적힌 규약입니다.

<img width="1848" alt="image" src="https://github.com/Growth-Hub/Effective-Java/assets/76567238/3b67d0e0-8f29-4ccd-9b00-c0531de2e117">

>equals 메서드는 동치관계를 구현하며, 다음을 만족합니다.
- 반사성 : null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true
- 대칭성 : null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true ->  y.equals(x)가 true
- 추이성 : null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true && y.equals(z)가 true -> x.equals(z)가 true
- 일관성 : null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하여도 항상 true만 혹은 false만 반환한다.
- null-아님 : null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false

<br>

## 1️⃣ 반사성

**객체는 자기 자신과 같아야 합니다.**
이 요건을 어긴 클래스의 인스턴스를 컬렉션에 넣은 다음 contains 메서드를 호출하면 방금 넣은 인스턴스가 없다고 답합니다.

<br>

## 2️⃣ 대칭성

**두 객체는 서로에 대한 동치 여부에 똑같이 답해야 합니다.**

>대소문자를 구별하지 않는 문자열을 구현한 다음 클래스를 예시로 보겠습니다.
```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

//     대칭성 부합x
    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        if (o instanceof String)  // 한 방향으로만 작동
            return s.equalsIgnoreCase((String) o);
        return false;
    }

    public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        CaseInsensitiveString cis2 = new CaseInsensitiveString("polish");
        String polish = "polish";
        System.out.println(cis.equals(polish));
        System.out.println(cis2.equals(cis));
    }
```
이 예제를 동작시켜보면 아래와 같다.
`cis.equals(s)`  = `true`
`s.equals(cis)` = `false` 를 반환하여, 대칭성을 명백히 위반합니다.

>여기에 더해 list.contains(posish)를 하면 어떻게 출력될까요?
```java
        List<CaseInsensitiveString> list = new ArrayList<>();
        list.add(cis);

        System.out.println(list.contains(polish));
```
정답은 `false`를 반환합니다.   
이처럼 equals 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없습니다.


결국에는 대칭성을 지키면서 올바르게 `equals` 구현하려면 아래처럼 구현해야합니다.
```java
 @Override public boolean equals(Object o) {
        return o instanceof CaseInsensitiveString &&
                ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
    }
```

<br>

## 3️⃣ 추이성

첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 겍체도 같아야 한다는 뜻입니다.

추이성은 상위 클래스에 없는 새로운 필드를 하위 클래스에 추가하며 equals를 재정의할 때 자주 발생하는 문제입니다.

```java
public class Point {
	private final int x;
	private final int y;

	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}

	@Override public boolean equals(Object o) {
		if(!o instanceof Point)
			return false;
		Point p = (Point) o;
		return p.x == x && p.y == y;
	}
}
```

```java
public class ColorPoint extends Point {
	private final Color color;
	
	public ColorPoint(int x, int y, Color color) {
		super(x, y);
		this.color = color;
	}

	...
}
```

### 1. 대칭성 위배
이 코드는 앞전에 배운 대칭성을 위배하는 코드입니다. 
why?
```java
 @Override public boolean equals(Object o) {
    	if(!o instanceof ColorPoint)
    		return false;
    	return super.equals(o) && ((ColorPoint) o).color == color;
    }
```
```java
    public static void main(){
      Point p = new Point(1,2);
      ColorPoint cp = new ColorPoint(1,2, Color.RED);
      p.equals(cp);    // true
      cp.equals(p);    // false
    }
```
`ColorPoint`의 `equals`는 만약 입력 매개변수의 클래스 종류가 다르다며 매번 false만 반환합니다.
따라서 이 코드에서도 `cp.equals(p)`에서 인수로들어간 p는 ColorPoint의 타입이 아니기 때문에 `false`를 반환합니다.

### 2.추이성 위배

>이제 Point도 지원하도록 추가
```java
@Override public boolean equals(Obejct o){
      if(!(o instanceof Point))
        return false;
      if(!(o instanceof ColorPoint))
        return o.equals(this);
      return super.equals(o) && ((ColorPoint) o).color == color;
    }
```
```java
    public static void main(){
      ColorPoint p1 = new ColorPoint(1,2, Color.RED);
      Point p2 = new Point(1,2);
      ColorPoint p3 = new ColorPoint(1,2, Color.BLUE);
      p1.equals(p2);    // true 
      p2.equals(p3);    // true 
      p1.equals(p3);    // false
    }
```

`p1.equals(p2)`와 `p2.equals(p3)`는 `true`를 반환하는데, `p1.equals(p3)`;는 `false`를 반환하기 때문에 바로 앞전에 살펴보았던 개념인 추이성에 위배됩니다. 또한 콜스택이 쌓이면서 스택 오버플로우가 발생할 수 있습니다.

그렇다고 해법이 존재하는 것도 아닙니다.  구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않습니다. 


### 3. 리스코프 치환 원칙 위배

> 그럼 기존에 `instanceof` 대신 `getClass`를 사용하면 어떨까요? 
```java
@Override public boolean equals(Object o){
  if(o == null || o.getClass() != getClass())
    return false;
  Point p = (Point) o;
  return p.x == x && p.y == y;
}
```
 
위의 코드는 같은 구현 클래스의 객체와 비교할 때만 true를 반환합니다. 
이렇게 한다면 리스코프 치환 원칙을 위배합니다.
`Point`의 하위 클래스는 여전히 `Point`이므로 어디서든 `Point`로써 활용될 수 있어야 합니다.

**리스코프 치환 원칙**
<img width="750" alt="image" src="https://github.com/Growth-Hub/Effective-Java/assets/76567238/943b5829-d7f0-45a3-839d-552207e1904f">

>리스코프 치환 원칙은 1988년 바바라 리스코프(Barbara Liskov)가 올바른 상속 관계의 특징을 정의하기 위해 발표한 것으로, 서브 타입은 언제나 기반 타입으로 교체할 수 있어야 한다는 것을 뜻합니다.
교체할 수 있다는 말은, 자식 클래스는 최소한 자신의 부모 클래스에서 가능한 행위는 수행이 보장되어야 한다는 의미입니다.
즉, 부모 클래스의 인스턴스를 사용하는 위치에 자식 클래스의 인스턴스를 대신 사용했을 때 코드가 원래 의도대로 작동해야 한다는 의미입니다.
이것을 부모 클래스와 자식 클래스 사이의 행위가 일관성이 있다고 말합니다.


### 해결법 1 - 컴포지션 활용하기(Not 상속)

```java
public class ColorPoint{
  private final Point point;
  private final Color color;

	public ColorPoint(int x, int y, Color color) {
		point = new Point(x, y);
		this.color = Objects.requireNonNull(color);
	}

	/* 이 ColorPoint의 Point 뷰를 반환한다. */
  public Point asPoint(){ // view 메서드 패턴
    return point;
  }

  @Override public boolean equals(Object o){
    if(!(o instanceof ColorPoint)){
      return false;
    }
    ColorPoint cp = (ColorPoint) o;
    return cp.point.equals(point) && cp.color.equals(color);
  }
}
```

`Point`를 상속하는 대신 `Point`를 `ColorPoint`의 `private` 필드로 두고, `ColorPoint`와 같은 위치의 일반 `Point`를 반환하는 뷰(view 메서드)를 `public`으로 추가하는 식으로 구현이 가능합니다.

<br>

## 4️⃣ 일관성

두 객체가 같다면(어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 합니다.

```java
while(x.equals(y) == x.equals(y))
```

<br>

## 5️⃣ null-아님

>잘못된 명시적 null 검사
```java
@Override
public boolean equals(Object o) {
  if(o == null) { 
      return false;
  }
}
```
>올바른 묵시적 null 검사
```java
@Override
public boolean equals(Object o) {
  if(!(o instanceof MyType)) { 
      return false;
  }
  MyType myType = (MyType) o;
}
```

동시성을 검사하려면 `equals`는 건네받은 객체를 적절히 형변환한 후 필수 필드들의 값을 알아내야 합니다.
따라서, 형변환에 앞서 `instanceof` 연산자로 입력 매개변수가 올바른 타입인지 검사해야 합니다.
입력이 `null`이면 타입 확인 단계에서 `false`를 반환하므로 `null` 검사를 명시적으로 하지 않아도 됩니다.

<br>

---

### 📚 Reference

- [Class Object](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html)
- [리스코프 치환 원칙](https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EC%95%84%EC%A3%BC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-LSP-%EB%A6%AC%EC%8A%A4%EC%BD%94%ED%94%84-%EC%B9%98%ED%99%98-%EC%9B%90%EC%B9%99)
