# 🚀 Item 16 public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

public 클래스에서 public 필드를 사용하면 어떨까요?

먼저 코드를 같이보시죠

```java
class point {
	public double x;
	public double y;
}
```

이렇게 데이터 필드를 public으로 해놓으면  아래와 같은 단점이 존재합니다.

1. 캡슐화의 이점을 제공하지 못합니다. (Item 15)
2. API를 수정하지 않고는 내부 표현을 바꿀 수 없습니다.
3. 불편식을 보장할 수 없습니다.
4. 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없습니다.

>그래서 보통 아래처럼  필드를 모두 private으로 바꾸고 public 접근자(getter)를 추가합니다.

```java
class Point {
	private double x;
	private double y;

	public Point(double x, double y) {
		this.x = x;
		this.y = y;
	}

	public double getX() {return x;}
	public double getY() {return y;}

	public void setX(double x) { this.x = x;}
	public void setY(double y) { this.y = y;}
}
```

이는 아래의 장점과 같습니다.

- 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있습니다.

```
❗️ package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를  노출한다 해도 하등의 문제가 없습니다
```
.

## 🤔 자바 플랫폼 라이브러리에도 public 클래스의 필드를 직접 노출하지 말라는 규칙을 어기는 사례가 종종있습니다.

대표적인 예가 `java.awt.package` 패키지의 `Point`와 `Dimension` 클래스입니다.  

### Dimension

![](https://i.imgur.com/LaOvvfm.png)


`java.awt.Dimension` 클래스는 public 필드인 `width` 와 `height` 를 직접 노출하는 방식을 사용합니다. 
이는 객체의 캡슐화 원칙에 어긋나는 설계로 간주될 수 있으며, 필드가 public 으로 선언되어 있기 때문에 클라이언트 코드에서 이 필드들을 직접 수정할 수 있게 됩니다. 이로 인해 객체의 무결성이 손상될 수 있으며, 향후 클래스를 변경하거나 확장하는 데 있어 제약이 될 수 있습니다. 따라서 ==공개 클래스를 설계할 때는 데이터 필드에 직접 접근하는 대신 접근자(getter) 및 설정자(setter)메서드를 사용하는 것이 좋습니다.==  이를 통해 데이터 은닉을 유지하고, 클래스의 인터페이스를 통한 상호작용만을 허용함으로써 클래스의 내부 구현을 캡슐화할 수 있습니다.


## ❓❗️public 클래스의 필드가 불변이라면? (public final)

public 클래스의 필드가 불변이라면 직접 노출할 때의 단점이 조금은 줄어들지만, 여전히 결코 좋은 생각이 아닙니다.  

아래와 같은 단점이 존재합니다.

- API를 변경하지 않고는 표현 방식을 바꿀 수 없습니다.
- 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점이 여전히 존재합니다.

단, 기존 public 필드와는 다르게 불변식은 보장할 수 있습니다. 


## 🎯 정리

- public 클래스는 절대 가별 필드를 직접 노출해서는 안됩니다.
- 불변 필드라면 노출해도 덜 위험하지만 여전히 단점은 존재합니다.
- pacakge-private 클래스나 private 중첩 클래스에서는 종종 필드를 노출하는 편이 나을 때도 있습니다.



### 📚 Reference

- [Dimension](https://github.com/fanhongtao/JDK/blob/master/src/java/awt/Dimension.java)
