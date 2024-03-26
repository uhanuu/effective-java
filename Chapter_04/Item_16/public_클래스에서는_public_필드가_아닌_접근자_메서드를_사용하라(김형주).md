## 🤔 public클래스에서 필드를 public으로 선언하면 어떻게 될까?
```java
package Test1;  // 다른 패키지에 위치
public class Test1 {
    public double a;      //public 선언
    public double b;
    ...

}

package Test2;
public class Tes2 {
    public static void main(String[] args) {
        Test1 test1 = new Test1(1,2);

        System.out.println(test1);   // a = 1 , b = 2
        test1.a = 8;
        test1.b = 9;
        System.out.println(test1);  // a = 8 , b = 9
    }
}
```
- 패키지가 달라도 쉽게 데이터 필드에 접근이 가능해져 전혀 ```캡슐화와 정보은닉```이 되지 않고 있다.
- 필드는 공개 API가 되버린다.
- API를 수정하지 않고는 내부 표현을 바꿀 수 없다.
- 불변을 보장할 수 없다.

### 즉, 올바른 컴포넌트 설계 방식과 멀어진다.

## 🤷‍♂️ 그렇다면 좋은 방법은 ?
 
```java
public class Test2 {
    private double a;
    private double b;

    public double getA() {        //접근 메서드 제공
        return a;
    }

    public void setA(double a) {      //접근 메서드 제공
        this.a = a;
    }

    public double getB() {        //접근 메서드 제공
        return b;
    }

    public void setB(double b) {    //접근 메서드 제공
        this.b = b;
    }
}
```
- 바로 접근 메서드(Getter,Setter)를 제공하는 것이다.
- 패키지 바깥에서 접근할 수 있는 클래스라면 클래스 내부 표현 방식을 언제든 바꿀 수 있다.

## 🧑‍💻 뭔가 부족해... 확실한 이유를 살펴보자.

#### 1️⃣ Setter가 필요한 이유

```java
public class Person {
    public int age;
}

public static void main(String[] args) {
    Person person = new Person(20);
    
    person.age = 10000;   // 타입의 범위 제한 이외의 제한 X
                         // ex) 120살 미만으로 제한하고 싶은데 제한할 수 없음...
}
```
- 위와 같이 필드를 public으로 하면, 해당 필드의 ```제한을 둘 수가 없게 된다.```
- Ex ) 0보다 작은 나이 값 혹은 120이 넘는 나이값 등등 제한을 둘 수 없다.

```java
class Person{
    private int age;

    public void setAge(int age) {
        if (age > 0 && age<120 ){     // 대입 전 전처리 가능
            this.age = age;
        }else{
            System.out.println("올바르지 않은 값입니다.");
            this.age=0;
        }
    }
}
```

- 필드를 private으로 정의하고 메서드를 제공하면 위와 같이 ```제한없이 접근하는 것을 막을수 있고```, ```값을 대입 전 전처리```가 가능해진다.

#### 2️⃣ Getter가 필요한 이유

```java
class Person{
    public int age;
    public int name;
    
    public String hobby;
    public int grade;
    public String school;
    public int schoolId;
    public String phoneNumber;
    public int gender;
    public int password;
    ...
}
```

-  만약에 위와 같이 클래스에 필드가 무수히 많다고 가정해보자.
-  자주 쓰이고 필요한 정보는 age, name이라고 가정해보자. ```그렇다면 나머지 정보들은 public으로 공개해둘 필요가 있을까?```

```java
class Person{
    private int age;
    private int name;
    
    private String hobby;
    private int grade;
    private String school;
    ...

    public int getAge() {
        return age;
    }

    public int getName() {
        return name;
    }
````

- private로 바꾸고, 자주 쓰이고 필요한 정보들은 get메서드를 제공해주는 방식이 좋다.
- Getter은 필드들의 외부 노출을 제한하고, 노출 범위를 정해준다. (```정보 은닉성```)

## 👀 그러면 필드에 public은 절대 쓰면 안되는건가요?
- 웬만해서는 안쓰는 것이 좋지만, pulic을 사용하면 좋은 특정 경우가 있다.
- ```package-private 클래스```의 필드 , ```private 중첩 클래스```의 필드
```java
public class ColorPoint {
    private String color;

    private static class Point{
        public int x;           //public으로 선언
        public int y;           //public으로 선언
    }

    public Point getPoint() {
        Point point = new Point();  
        point.x = 3;        // 메서드를 제공했을 때보다 코드면에서 깔끔하다.
        point.y = 4;              
        return point;
    }
}
```

- private 중첩 클래스의 경우, 외부클래스 (ColorPoint)에서만 접근이 가능하고, 외부 클래스에서는 접근할 수 없다.
- package-private의 경우에도 같은 패키지 내에서만 조작이 가능하고, 외부 패키지에서는 접근할 수 없다.
- 위 2가지 클래스의 경우 필드를 public해도 ```어차피 내부에서만 사용되어서 상관이 없다.```
- 게다가 메서드를 통해 접근하는 것보다 바로 public으로 접근하는 것이 ```코드면에서 깔끔하다.```

## ❌  필드를 직접 노출시킨 사례 (안좋은 예시)
```java
//java.awt.Point
public class Point extends Point2D implements java.io.Serializable {
    public int x;
    
    public int y;
}
//java.awt.Dimension
public class Dimension extends Dimension2D implements java.io.Serializable {
    public int width;
    
    public int height;
}
```
- ```java.awt.Point```, ```java.awt.Dimension```는 public 클래스의 필드를 직접 노출시킨 사례이다.
- Dimension의 경우 심각한 성능 문제를 가지고 있어 오늘날까지도 해결되지 못했다.

> [item 67] Dimension의 문제점 😥

> getSize 메소드는 Dimension 클래스의 새 인스턴스를 반환한다. Dimension 클래스는 가변으로 설계되어, getSize를 호출할 때마다 새로운 Dimension 인스턴스가 생성하기 때문에 성능 저하를 유발한다.

## 🌈 불변필드의 경우에 public은 어떨까?
```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;
    
    public final int hour;
    public final int minute;

    public Time(int hour, int minute) { ... }
}
```

- public이기 때문에 API를 변경하지 않고서는 표현 방식을 바꿀수 없다는 단점은 여전하다.
- 필드를 읽을때 부수 작업을 수행할 수 없다는 단점도 있다.

### 🥎 정리
package-private 클래스 혹은 private 중첩 클래스의 필드이지 않는 이상,  **필드를  public으로 절대 정의하지 말자.**

### 📝 Reference
[Getter와 Setter 왜 써야 하나요?](https://www.blog.ecsimsw.com/entry/%EC%9E%90%EB%B0%94-Getter%EB%9E%91-Setter%EB%A5%BC-%EC%99%9C-%EC%8D%A8%EC%95%BC%ED%95%B4)

[Dimension의 문제점](https://joyerim.tistory.com/118)



