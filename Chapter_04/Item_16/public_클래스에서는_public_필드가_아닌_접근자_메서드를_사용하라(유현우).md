# public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라
**개발을 하다보면 인스턴스 필드들만 모아놓는 목적만 가진 아래와 같은 클래스를 구성하는 경우가 있다.**

## **캡슐화의 이점을 제공하지 못하는 클래스**

```java
class Point {
	public int x;
	public int y;

	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}
}
```

### 필드가 public 이라서 생기는 문제들

캡슐화의 이점을 제공하지 못한다.

1. API를 수정하지 않고는 내부 표현을 바꿀 수 없다.
    - public 필드이기 때문에 내부 표현을 변경하기 위해서는 필드 자체를 변경해야 하고 이 필드를 사용하는 클라이언트 코드들도 전부 수정해야 한다.
2. 불변식을 보장할 수 없다.
    
    클라이언트에서 직접적으로 필드에 접근하고 있으므로 클라이언트에 의해서 변경될 수 있다.
    
    - 자기 자신이 상태를 관리하는게 아니다.
    
    ```java
    public static void main(String[] args) {
            Point point = new Point(1, 3);
            point.x = 3;
            point.y = 5;
            System.out.println(point); //Point{x=3, y=5}
    }
    ```
    
3. 외부에서 필드에 접근할 때 부수적인 로직을 추가할 수 없다.
    - 만약 x를 변경할 때 30이상을 넘어가면 안된다면 조건을 추가할 수 있는 방법이 없다.

## **철저한 객체 지향 프로그래머가 캡슐화한 클래스**

위에 코드에서 필드는 모두 private으로 변경하고 public 접근자(getter), 변경자(setter) 추가했다.

```java
class Point {
  private double x;
  private double y;

  public Point (double x, double y) {
    this.x = x;
    this.y = y;
  }

  public double getX() { return x; }
  public double getY() { return y; }

  public void setX(double x) { this.x = x; }
  public void setY(double y) { this.y = y; }
}
```

### 장점

클래스의 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 제공하게 된다.

- getter/setter 혹은 또 다른 메소드를 통해 부수적인 로직을 언제든 추가할 수 있다.

### **package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다고 해도 문제될 것이 없다.**

**같은 패키지 안에서 어떤 특정 이유 때문에 사용하던가 탑 레벨 클래스에서만 접근하기 때문에 위에서 언급한 단점들이 나타날 이유가 없어 문제될 것이 없다.**

- Package-private : 같은 패키지 및 같은 클래스에서만 접근 가능
- private 중첩 클래스 : 내부 클래스가 private 형태로 된 클래스

```java
public class TopPoint {
	private static class Point {
		public int x;
		public int y;
	}

	public Point getPoint() {
		return new Point(1,5);
	}
}
```

### private class인 경우

TopPoint 클래스에서는 얼마든지 Point 클래스의 필드를 조작할 수 있지만 외부 클래스에서는 Point 클래스의 필드에 직접 접근할 수 없다.

### **package-private class인 경우**

해당 클래스가 포함되는 패키지 내에서만 조작할 수 있지만 패키지 외부에서는 접근할 수 없다.

### **불변 필드를 노출한 public 클래스**

```java
public class Time {
	private static final int HOURS_PER_DAY = 24;
	private static final int MINUTES_PER_HOUR = 60;

	public final int hour;
	public final int minute;

	public Time(int hour, int minute) {
		if (hour < 0 || hour > HOURS_PER_DAY) {
			throw new IllegalArgumentException("시간: " + hour);
		}
		if (minute < 0 || minute > MINUTES_PER_HOUR) {
			throw new IllegalArgumentException("분: " + minute);
		}
		this.hour = hour;
		this.minute = minute;
	}

	...
}
```

- 불변식은 사용이 가능하지만 나머지 문제는 해결하지 못하고 있다.

### 개인적으로 생각하는 방식

- 필드는 private final으로 불변식을 사용하고 getter(접근자)로 유연성을 얻는 방식
    - setter(수정자)는 추가지하지 말고 데이터를 수정하고 싶으면 객체를 새로 만들어주면 된다.

```java
class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;

    private final int hour;
    private final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour > HOURS_PER_DAY) {
            throw new IllegalArgumentException("시간: " + hour);
        }
        if (minute < 0 || minute > MINUTES_PER_HOUR) {
            throw new IllegalArgumentException("분: " + minute);
        }
        this.hour = hour;
        this.minute = minute;
    }

    public int getHour() {
        return hour;
    }

    public int getMinute() {
        return minute;
    }
}
```

java 14 이상이라면 record를 사용해서 더 깔끔한 코드와 불변을 지킬 수 있다.

```java
record Time (int hour, int minute) {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;

    Time {
        if (hour < 0 || hour > HOURS_PER_DAY) {
            throw new IllegalArgumentException("시간: " + hour);
        }
        if (minute < 0 || minute > MINUTES_PER_HOUR) {
            throw new IllegalArgumentException("분: " + minute);
        }
    }
}
```

### **정리**

public 클래스의 경우 가변 필드의 접근 제한자를 public으로 두면 안 되며, 불변 필드라고 해도 덜 위험하지만 안심할 수 없다. 

- 만약 가변 필드를 노출하고 싶을 땐, package-private 클래스나 private 중첩 클래스를 활용하자
