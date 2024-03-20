## 📚 Comparable이란?
```java
public interface Comparable<T> {
    /**
     * Compares this object with the specified object for order.  Returns a
     * negative integer, zero, or a positive integer as this object is less
     * than, equal to, or greater than the specified object.
     */
    public int compareTo(T o);
}
```
- ```인터페이스```로 compareTo()메서드를 가지고 있고 해당 인터페이스로 만든 객체는 compareTo를 재정의해줘야한다.
- Generic타입으로 Boolean을 제외한 래퍼 클래스나 String, Time, Date와 같은 클래스의 인스턴스를 받을 수 있다. (유연성)
- compareTo()를 재정의 함으로써 같은 인스턴스끼리 값을 비교할 수 있고, ```순서도 비교하여 정렬도 할 수 있다.```

## 🤔 어떻게 객체를 정렬할까?
<p align = "center">
<img width="600" alt="image" src="https://github.com/kim0527/Effective-Java/assets/143387515/bb54dd27-726e-4657-a5fd-81d37ce81e8c">
</p>

- 원시타입의 경우, 크고 작음이 정의되어 있어 부등호를 통해 쉽게 비교할 수 있고, sort()를 통해 정렬도 간편하게 가능하다.
- 하지만 객체의 경우, 이러한 크고 작음이 정해져 있지 않아 정렬이 불가능하다.
- 객체를 가진 배열을 sort()하는 경우 ```컴파일 에러```가 발생한다.

### Comparable와 compareTo를 정의한 클래스 정렬

```java
class Employee implements Comparable<Employee>{
    private String name;
    private int age;

    public Employee(String name, int age){
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString(){
        return "{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public int getAge() {
        return age;
    }

    @Override
    public int compareTo(Employee o) {      //정렬할 기준
        return this.age - o.getAge();
    }

    public static void main(String[] args){
        List<Employee> employeeList = new ArrayList<>(Arrays.asList(new Employee("KIM", 15),
                                                                    new Employee("LEE", 20),
                                                                    new Employee("MIN", 10)));
                                                           
        Collections.sort(employeeList);      //sort() 수행 (Comparable으로 컴파일 에러 발생 X)
        System.out.println(employeeList);    // 정렬 결과 출력
    }
}
```
```
[실행 결과]
[{name='MIN', age=10}, {name='KIM', age=15}, {name='LEE', age=20}]    // age 기준으로 정렬된 것을 확인할 수 있다.
```
- 위와 같이 Comparable을 인터페이스로 한 Employee 클래스를 생성했다.
- compareTo() 메서드에는 정렬의 기준이 될 age를 비교한 후 결과를 반환한다.
    > 뒤에 나올 귀약에서도 나오겠지만, compareTo()메서드는 아래와 규칙에 따라 int값을 반환한다.
    <br>1️⃣ 이 객체가 주어진 객체보다 작으면 ```음의 정수```를 반환한다.
    <br>2️⃣ 이 객체가 주어진 객체와 같다면 ```0```을 반환한다.
    <br>3️⃣ 이 객체가 주어진 객체보다 크다면 ```양의 정수```를 반환한다.
- 3개의 Employee를 만들고 배열을 만들어 정렬 진행하면 출력결과와 같이 age을 기준으로 객체가 정렬된 것을 확인할 수 있다.

### Collections.sort()에서 무슨일이..?
```java
//Collections.sort() 코드
public static <T extends Comparable<? super T>> void sort(List<T> list) {
    list.sort(null);
}
```
>// list.sort 코드의 주석<br>
If the specified comparator is null, 
then all elements in this list must implement the Comparable interface and ```the elements's natural ordering should be used.```
- Collections.sort()는 List를 인자로 받고 해당의 sort(null)를 수행한다.
- 이때 sort()의 인자가 null(comparator==null)이라면 원소의 기본 정렬을 수행한다고 적혀있다.
### 정리해보면
compareTo()메서드에 정렬 기준을 규약에 맞춰 정의해두면, sort() 실행시  compareTo()를 이용해 원소의 기본 정렬을 수행한다.

## ✅ CompareTo의 일반 규약
- 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다.
- 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ```ClassCastException```을 던진다.
- 아래에 나오는 sgn표기는 부호함수를 뜻하며 표현식의 값이 음수,0,양수일 때 -1,0,1을 반환하도록 정의했다.
1. [대칭성] sgn(x.compareTo(y)) == - sgn(y.compareTo(x))
    <br>( " x > y 이면 y < x 이다 "와 같은 말이다. )
2. [추이성] x.compareTo(y) > 0 이고 y.compareTo(z) > 0 이면,  x.compareTo(z) >0이다
    <br>( " x > y && y > z 이면 x > z이다 "와 같은 말이다. )
3. [반사성] x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))이다.
    <br>( x와 y가 같다면, (x, z) 비교결과와  (y, z)비교결과는 같아야한다. )
4. x.compareTo(y) == 0 이면 x.equals(y)는 True여야 한다.
    <br>( 필수는 아니지만 꼭 지킬 것을 권장한다.)

1번 ~ 3번까지 규약은 equals()의 일반 규약과 매우 유사하다.

### 4번 규약 위배 예시
```java
public static void main(String[] args) {
    BigDecimal A = new BigDecimal("1.0");
    BigDecimal B = new BigDecimal("1.00");

    Set<BigDecimal> set = new HashSet<>();
    set.add(A);
    set.add(B);

    System.out.println("A.equals(B) 결과: "+ A.equals(B));      // false     (다르다고 인식)
    System.out.println("set 사이즈: "+ set.size());             // 2         (다르다고 인식해 중복처리 X)
}
```
- HashSet에서는 두 객체가 다르다고 판단하여 size()를 2로 출력한다.
```java
public static void main(String[] args) {
    BigDecimal A = new BigDecimal("1.0");
    BigDecimal B = new BigDecimal("1.00");

    Set<BigDecimal> tree = new TreeSet<>();
    tree.add(A);
    tree.add(B);

    System.out.println("A.compareTo(B) 결과: "+ A.compareTo(B));       // 0  (같다고 인식)
    System.out.println("set 사이즈: "+ tree.size());                   // 1  (같다고 인식해 중복처리)
}
```
- 하지만 TreeSet에서는 두 객체가 같다고 판단하여 size()를 1로 출력한다.

> TreeSet 주석 중에...
<br>This is so because the ```{@code Set} interface``` is defined in terms of the ```{@code equals} operation```
, but a ```{@code TreeSet} instance``` performs all element comparisons ```using its {@code compareTo}``` (or {@code compare}) method
>
- 객체 동치를 Set에서는 equals()를 통해 판단하고, TreeSet에서는 compareTo()를 통해 판단한다.
- 이처럼 equals와 compareTo의 결과가 다를 경우 위와 같은 문제가 발생한다.

## 🙆‍♂️ compareTo 작성 요령
전체적으로 equals() 작성요령과 유사하며, 차이점은 아래와 같다.
1. 제너릭 인터페이스 이므로, 타입 확인 혹은 형변환이 필요없다.
    <br>( 어차피 타입이 잘못됬을 경우, ClassCastException으로 인해 컴파일이 되지 않는다.)
2. null을 인수로 넣어 호출하면 NullPointerException을 던져야 한다.

이제 객체 상태에 따른 재정의 방법에 대해 알아보자.

### 1️⃣ 객체 참조 필드가 하나뿐인 경우
#### 추천하지 않는 방법
```java
public class Book implements Comparable<Book>{
    private Integer price;

    @Override
    public int compareTo(Book book) {
        if (this.price == book.price) {
            return 0;
        } else if (this.price > book.price) {
            return 1;
        } else {
            return -1;
        }
    }

    public static void main(String[] args) {
        Book book1 = new Book();   //Null
        Book book2 = new Book();   //Null

        System.out.println(book1.compareTo(book2));   //NullPointException을 반환하지 않고 0을 반환함.

    }
}
```
- Book의 price이 Null이지만 compareTo은 NullPointException을 반환하지 않고 0을 반환한다.
- 이처럼 compareTo 메서드에서 관계 연산자 <와>를 사용하는 이전 방식은 거추장스럽고 오류를 유발하니, 추천하지 않는다.

#### 추천 방법
```java
public class Book implements Comparable<Book>{
    private Integer price;

    @Override
    public int compareTo(Book book) {
        return Integer.compare(price, book.price);  //비교자 사용
    }

    public static void main(String[] args) {
        Book book1 = new Book();   //Null
        Book book2 = new Book();   //Null

        System.out.println(book1.compareTo(book2));   //NullPointException

    }
}
```
### 2️⃣ 객체 필드가 여러개 일때
```java
public int compareTo(PhoneNumber pn) {
  int result = Short.compare(areaCode, pn.areaCode);    //가장 중요한 필드
  if (result == 0) {
    result = Short.compare(prefix, pn.prefix);       // 두번째로 중요한 필드
    if (result == 0) {
      result = Short.compare(lineNum, pn.lineNum); //세번째로 중요한 필드
  }
  return result;
}
```
- 필드가 여러개 일때는 중요한 핵심필드 부터 먼저 비교해주자.

## ➕ Comparator (비교자)
- Comparator 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자 생성할 수 있게 되었다.
- 하지만 약간의 성능 저하가 존재한다.
```java
private static final Comparator<PhoneNumber> COMPARATOR =
            comparingInt((PhoneNumber pn) -> pn.areaCode)  // 1순위 areaCode로 정렬
                    .thenComparingInt(pn -> pn.prefix)    // 2순위 prefix 정렬
                    .thenComparingInt(pn -> pn.lineNum);  // 3순위 lineNum 정렬

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```
- Comparator는 Comparable과 달리 기본 정렬 기준과 다르게 정렬하고 싶을 때 주로 사용한다.
- ```comparingInt```, ```thenComparingInt```와 같이 자바의 숫자용 기본타입을 모두 커버하는 변형 메서드가 존재한다.

## ⚠️ 주의사항
```java
static Comparator<Object> hashCodeOrder = new Comparator<>(){
	public int compare(Object 01, Object 02){
		return o1.hashCode() - o2.hashCode();
	}
};
```
- 위 코드 처럼 값을 차를 기준으로 하는 메서드는 ```정수 오버플로우``` 혹은 ```부동소수점 계산 방식```에 의해 오류가 발생할 수 도 있다.

#### 대안책 1 : 정적 compare 메서드를 활용한 비교자
```java
static Comparator<Object> hashCodeOrder = new Comparator<>(){
	public int compare(Object 01, Object 02){
		return Integer.compare(o1.hashCode() , o2.hashCode());
	}
};
```

#### 대안책 2 : 비교자 생성 메서드를 활용한 비교자
```java
static Comparator<Object> hashCodeOrder = 
        Comparator.comparingInt(o -> o.hashCode());
```

## 🥎 정리
- 순서를 고려한 값 클래스가 있다면 Compareable 인터페이스 혹은 Comparator 인터페이스를 구현하자.

### 📝 Reference
[[JAVA] 리스트(List) 정렬](https://engineerinsight.tistory.com/32)<br>
[[Java] Comparable와 Comparator의 차이와 사용법](https://gmlwjd9405.github.io/2018/09/06/java-comparable-and-comparator.html)




