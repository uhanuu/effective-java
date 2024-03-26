# 1️⃣6️⃣ Item 16 : public 클래스에서는 public 필드가 아닌 접근자 메서드 사용하라

<br>

## 📌 목차
1. public 클래스의 public 필드
2. 인스턴스 필드 모아놓는 클래스의 public 필드
3. public 클래스의 불변 public 필드
4. package-private 클래스, private 중첩 클래스의 public 필드
5. public 필드 노출 X 규칙 어긴 사례
6. 결론

<br>

## 🔓 public 클래스의 public 필드

public 클래스의 public 필드는 좋지 않은 선택이다. 

***`public 클래스의 필드를 직접 노출하지 말아야 한다.`***

<br>

지금부터 다양한 상황에서 public 필드를 가질 때 어떤 클래스에서 어떤 문제가 있고 어떻게 해야 하는지 보자. 

<br>

## 📚 인스턴스 필드 모아놓는 클래스의 public 필드

인스턴스 필드들을 모아 놓는 일 외에는 아무 목적도 없는 퇴보한 클래스가 있다. 

아래와 같이 2차원 점을 표현하기 위해 Point라는 클래스를 만들고 안에 x 좌표, y 좌표를 표현하기 위한 double 필드들이 있는 것이다. 

```java
public class Point{
		public double x;
		public double y;
}
```

<br>

### ❗ 문제점

이런 public 클래스이고 인스턴스 필드만 모아 놓은 클래스에서 `필드가 public이라면 문제점`이 있다. 

아래와 같은 문제점이 있다. 

- 데이터 필드에 직접 접근하니 `캡슐화의 이점을 제공하지 못한다`.

- API 수정하지 않고 `내부 표현 바꿀 수 없다`.

- `불변식을 보장할 수 없다`.

- 외부에서 필드에 접근할 때 `부수 작업 수행할 수 없다`.

<br>

철저한 객체 지향 프로그래머는 이런 클래스를 상당히 싫어한다. 

<br>

### ⭕ 해결책

그렇다면 이런 클래스는 `어떻게 바꿔야 하는가?`

철저한 객체 지향 프로그래머는 이런 클래스에서 `필드는 모두 private`으로 바꾸고 `public 접근자 = getter를 추가`한다. 

<br>

아래 코드처럼 하는 것이다.

```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

<br>

public 클래스에서라면 이렇게 해야 한다. 

이렇게 하면 위의 문제점들을 어떻게 해결하는지 보자. 

- `private를 사용`하여 캡슐화의 이점을 제공한다.

- 접근자를 제공해 클래스 내부 표현 방식을 언제든 바꿀 수 있는 `유연성 제공`한다.

- 접근자를 제공해 `접근자 안에서 부수 작업을 수행`할 수 있다.

<br>

위의 코드는 `불변식을 보장하지는 않는다`. 

왜냐하면 필드가 그저 private이고 이 필드를 수정할 수 있는 `setter가 있기 때문`이다. 

따라서 불변식까지 보장하려면 다음과 같이 코드를 작성해야 한다. 

```java
public final class Point {
    private final double x;
    private final double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }
}
```

<br>

## 🚫 public 클래스의 불변 public 필드

public 클래스의 필드가 불변인 경우가 있다. 

이 경우도 여전히 좋은 생각은 아니다. 

<br>

### 👍 장점

위의 public 클래스에서 가변 public 필드를 가지는 것보다는 장점은 있다. 

- 필드가 불변이기 때문에 직접 노출할 때의 단점은 조금 줄어든다.
- 필드의 불변식은 보장할 수 있다.

<br>

### 👎 단점

하지만 여전히 다음과 같은 단점은 계속 가지고 있다. 

- API를 변경하지 않고는 표현 방식 바꿀 수 없다.
- 필드 읽을 때 부수 작업을 수행할 수 없다.

<br>

### 👏 예시

이런 형태의 예시로 아래 코드를 보자. 

시간을 나타내는 Time 클래스이다. 

```java
// 코드 16-3 불변 필드를 노출한 public 클래스 - 과연 좋은가? (103-104쪽)
public final class Time {
    private static final int HOURS_PER_DAY    = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("Hour: " + hour);
            
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("Min: " + minute);
            
        this.hour = hour;
        this.minute = minute;
    }

    // 나머지 코드 생략
}
```

<br>

이렇게 하면 각 인스턴스가 유효한 시간을 표현하는 것은 보장한다. 

하지만 이렇게 하는 것이 과연 좋은지는 의문이 남는다. 

<br>

## 📂 package-private 클래스, private 중첩 클래스의 public 필드

public 필드가 모두 문제될 것 같지만 `그렇지 않은 경우`가 있다. 

package-private 클래스와 private 중첩 클래스의 경우 필드를 public으로 해서 노출해도 문제 없다. 

이때는 그 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 된다. 

<br>

`package-private 클래스와 private 중첩 클래스의 경우` 클래스 선언 면에서나 사용하는 클라이언트 코드 면에서나 접근자 방식보다 public 필드를 제공하는 것이 풜씬 깔끔하다. 

<br>

왜 그런지 이유는 아래와 같다. 

- 클라이언트 코드가 이 클래스 내부 표현에 묶이기는 하나, 클라이언트도 어차피 이 `클래스를 포함하는 패키지 안에서만 동작하는 코드`이다.

- 따라서 `패키지 바깥 코드`는 전혀 손대지 않고도 데이터 표현 방식 바꿀 수 있다.

- private 중첩 클래스는 이 클래스 포함하는 `외부 클래스 바깥 코드`는 전혀 손대지 않고도 데이터 표현 방식 바꿀 수 있다.

<br>

이렇듯 package-private 클래스는 public 필드를 제공해도 같은 **패키지내의 나를 사용하는 코드만 수정하면 된다**. 

private 중첩 클래스는 public 필드를 제공해도 더 좁은 범위인 **중첩 클래스 포함하는 외부 클래스 코트만 수정하면 된다**. 

<br>

## ❌ public 필드 노출 X 규칙 어긴 사례

자바 라이브러리에도 public 클래스의 필드를 직접 노출하지 말라는 `규칙을 어기는 사례가 종종 있다`.

대표적인 예가 java.awt.package 패키지의` Point와 Dimension 클래스`다. 

내부를 노출한 Dimension 클래스의 심각한 `성능 문제`는 오늘날까지도 해결되지 못했다. 

<br>

Dimension 클래스 코드를 보면 아래와 같다. 

```java
public class Dimension extends Dimension2D implements java.io.Serializable {

    /**
     * The width dimension; negative values can be used.
     *
     * @serial
     * @see #getSize
     * @see #setSize
     * @since 1.0
     */
    public int width;

    /**
     * The height dimension; negative values can be used.
     *
     * @serial
     * @see #getSize
     * @see #setSize
     * @since 1.0
     */
    public int height;

		...
}

```

<br>

보면 Dimension 클래스가 public 클래스인데 width와 height 필드도 public인 문제 있는 클래스다. 

<br>

## ‼ 결론

`public 클래스는 절대 가변 필드를 직접 노출해서는 안된다`. 

불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. 

하지만 package-private 클래스나 private 중첩 클래스에서는 종종 불변 또는 가변 필드를 노출하는 편이 나을 때도 있다.