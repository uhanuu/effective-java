## 🤔 중첩 클래스란?
- 다른 클래스 안에 정의된 클래스를 말한다.
- 자신을 감싼 바깥 클래스에서만 쓰여야 하면, 그 외에서 쓰인다면 톱레벨 클래스로 빼야한다.

## 🌈 중첩 클래스의 종류
- 정적 멤버 클래스
- (비정적) 멤버 클래스
- 익명클래스
- 지역클래스

정적 멤버 클래스를 제외한 나머지는 ```내부 클래스(inner class)```에 해당한다.

## 1️⃣ 정적 멤버 클래스
```java
public class Test {
    private int a = 20;

    private static class innerClass {     // 정적 멤버 클래스
        private int b = 10;
        
        public int add() {
            final Test test = new Test();
            return test.a + b;             // private여도 접근이 가능하다.
        }
    }
}

```
- 다른 클래스 안에 선언되고, 바깥 클래스의 private멤버에도 접근 가능하다. ( 이외에는 일반클래스와 동일하다.)
- 정적 멤버 클래스를 private으로 선언하면 바깥 클래스에서만 접근할 수 있다.

### 정적 멤버 클래스 주 사용처
```java
public class Calculator {
    
    // 중첩된 enum 타입은 암시적으로 static으로 선언되므로 명시적으로 static 키워드를 붙이지 않아도 됨
    // 암시적 static
    public enum Operation {
        PLUS { public int apply(int x, int y) { return x + y; } },
        MINUS { public int apply(int x, int y) { return x - y; } };

        public abstract int apply(int x, int y);
    }
}

class Client{
    public static void main(String[] args) {
        Calculator.Operation operation1 = Calculator.Operation.PLUS;      
        System.out.println("Result: " + operation1.apply(20, 10));     //30

        Calculator.Operation operation2 = Calculator.Operation.MINUS;
        System.out.println("Result: " + operation2.apply(20, 10));     //10
    }
}
```
- 유용한 public 도우미로 주로 쓰인다.
- 위 예시에서는 클라이언트에게 ```Calculator.Operation.PLUS```, ```Calculator.Operation.MINUS```와 같은 형태로 연산 기능을 제공하고 있다.

## 2️⃣ (비정적) 멤버 클래스
정적 멤버 클래스와 구문상 차이는 static이 붙어있고 없고 뿐이지만, 차이는 이외로 크다.
```java
public class Test2 {
    private int a = 20;

    private class innerClass {    //(비정적) 멤버 클래스
        private int b = 10;

        public int add() {
            //final Test test = new Test();   //필요없음.
            return Test2.this.a + b;          // 정규화된 this로 참조 가능
        }
    }
}
```
- 비정적 멤버 클래스의 인스턴스는 **바깥 클래스의 인스턴스와 암묵적으로 연결된다.**
- 정규화된 this(```클래스명.this```)로 참조를 가져올 수 있다.
- 바깥 인스턴스 없이 비정적 멤버 클래스는 생성할 수 없다.

그래서 다음과 같은 기준이 생긴다.

- 중첩 클래스의 인스턴스와 바깥 인스턴스가 독립적으로 존재할 수 있다면?
    -> 정적 멤버 클래스

- 독립적으로 존재할 수 없다면?
    -> 비정적 멤버 클래스

### 비정적 멤버 클래스 주 사용처
주로 어댑터를 정의할때 자주 쓰인다.
```java
public class MySet<E> extends AbstractSet<E> {
    ...
    @Override
    public Iterator<E> iterator() {   // 재정의를 통해 MyIterator 설정
        return new MyIterator();
    }
    
    private class MyIterator implements Iterator<E> {      // Iterator<E>으로 자신만의 반복자 추가
       ...
    }
}
```
- 어댑터란 기존의 클래스를 수정하지 않고도, 특정 인터페이스를 필요로 하는 코드에서 사용할 수 있게 해주는 것을 말한다.
- 위 코드에서는 MySet에 자신만의 Iterator을 추가하고 싶어서 비정적 멤버 클래스를 통해 넣었다.

### 정적 멤버 클래스 🆚  비정적 멤버 클래스

멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자.

- static을 생략하면 바깥 인스턴스와 숨은 참조를 가지게 된다.
- 이를 저장하는 과정에서 시간과 공간이 소비된다.
- 가장 큰 문제점은 이러한 숨은 참조로 인해 GC가 수거해가지 않아 **메모리 누수**가 발생한다는 것이다.

## 3️⃣ 익명 클래스
```java
class Car {
    public void charge(){
        System.out.println("휘발유를 넣습니다.");
    }
}

//public class electricCar extends Car{               //따로 정의 X
//    @Override
//    public void charge() {
//        System.out.println("전기로 충전합니다.");
//    }
//}

class Main{
    public static void main(String[] args) {
        Car electricCar = new Car(){                //익명 클래스 정의
            @Override
            public void charge() {
                System.out.println("전기로 충전합니다.");
            }
        };
    }
}
```
- 익명 클래스는 바깥 클래스의 멤버가 아니다.
- 멤버와 달리, 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다.
- 정적 문맥에서라도 상수 변수 이외의 정적 멤버는 가질 수 없다.

### 익명 클래스의 단점

1. 선언한 지점에서만 인스턴스를 만들 수 있기 때문에 instanceof 검사를 할 수 없다.
2. 클래스의 이름이 필요한 작업은 수행할 수 없다.
3. 인터페이스를 구현하는 동시에 다른 클래스를 상속할 수도 없다.
4. 코드 중간에 등장하므로 가독성이 떨어진다.
5. 익명클래스가 상위타입에서 상속한 멤버 외에는 호출할 수 없다.

[단점5 참고 코드](https://github.com/kim0527/Effective-Java/blob/main/Chapter_04/Item_24)
```java
public class Car {
    public void charge(){
        System.out.println("휘발유를 넣습니다.");
    }
}

class LpgCar extends Car{
    public void charge(){
        System.out.println("LPG를 넣습니다.");
    }
    public void activate(){
        System.out.println("부릉부릉");
    }
}

class Main{
    public static void main(String[] args) {
        Car electricCar = new LpgCar(){
            @Override
            public void charge() {
                System.out.println("전기로 충전합니다.");
            }
        };

        electricCar.activate();   //컴파일 에러
    }
}
```
<p align = "center">
<img width="700" alt="image" src="https://github.com/kim0527/Effective-Java/assets/143387515/b85dc817-0ea4-489f-aa71-cf1a72fa55aa">
</p>

### 익명 클래스 주 사용처
```java
interface Car {
    void charge();
}
class Main{
    public static void main(String[] args) {
        //Car electricCar = new Car(){
        //    @Override
        //    public void charge() {
        //       System.out.println("전기로 충전합니다.");
        //    }
        //};
        //electricCar.charge();
        
        //람다표현식
        Car electricCar = () -> System.out.println("전기로 충전합니다.");
        electricCar.charge();
    }
}


```

- 자바 람다를 지언하면서 주로 즉석에서 작은 함수 객체나 처리객체를 만드는데 사용한다.
- 이전보다 코드 양이 줄었다.

```java
static List<Integer> intArrayAsList(int[] a) {
      Objects.requireNonNull(a);
    
      return new AbstractList<>() {
            @Override
            public Integer get(int i) {
              return a[i];
            }
    }
}
```
- 정적 팩터리 메서드를 만들때도 사용한다.

## 4️⃣ 지역클래스
```java
public class Outer {
    private int outerNum = 10;

    public void outerMethod() {
        final int localNum = 20;

        class LocalInner {
            final int innerNum=30;
            void innerMethod() {
                System.out.println("outerNum = " + outerNum);
                System.out.println("localNum = " + localNum);
                System.out.println("innerNum = " + innerNum);
            }
        }

        LocalInner localInner = new LocalInner();
        localInner.innerMethod();
    }

    public static void main(String[] args) {
        Outer outer = new Outer();
        outer.outerMethod();          //outerNum = 10
                                      //localNum = 20
                                      //innerNum = 30
    }
}
```
- 지역변수로 선언할 수 있는 모든 곳에서 선언 가능하다.
- 네 가지 중첩 클래스 중 가장 드물게 사용된다.

### 🥎 정리
- 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기 너무 길다면 **멤버 클래스**로 만들자.

정적 or 비정적
- 멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조해야 한다면 **비정적**으로 하자
- 인스턴스가 각각 독립적으로 사용된다면 **정적**으로 하자.

익명 or 지역
- 한 메서드 안에서만 쓰이면서 인스턴스를 생성하는 지점이 단 한곳이고
- 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 존재한다면 **익명 클래스**로 사용하자.
- 위 상황이 아니라면 **지역클래스**를 사용하자.


### 📝 Reference
[익명클래스](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%9D%B5%EB%AA%85-%ED%81%B4%EB%9E%98%EC%8A%A4Anonymous-Class-%EC%82%AC%EC%9A%A9%EB%B2%95-%EB%A7%88%EC%8A%A4%ED%84%B0%ED%95%98%EA%B8%B0)

