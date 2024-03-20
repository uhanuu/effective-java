# 1️⃣4️⃣ Item 14: Comparable을 구현할지 고려하라

<br>

## 📌 목차
1. Comparable & compareTo란?
   1. Comparable이란?
   2. compareTo란?
2. compareTo 규약
3. 규약 1.
4. 규약 2.
5. 규약 3.
6. 규약 1~3
7. 규약 4.
8. 규약 4 안 지킨 예시
9. compareTo 작성 요령
10. 주의점 1~6
11. Comparator 인터페이스
12. Comparator의 메서드 연쇄 방식
13. Comparator의 메서드 연쇄 방식 예시
14. Comparator의 보조 생성 메서드
15. Comparator의 객체 참조용 비교자 생성 메서드
16. 이상한 compareTo & compare 메서드
17. 결론

<br>

## ❓ Comparable & compareTo란?

### 🆚 Comparable이란?

Comparable은 인터페이스다. 

Comparable 인터페이스는 `객체를 정렬하는데 사용되는 메서드인 compareTo() 메서드를 정의`하고 있다. 

<br>

자바에서 같은 타입의 인스턴스를 `서로 비교해야만 하는 클래스들은 모두 Comparable 인터페이스를 구현`하고 있다. 

따라서 Boolean을 제외한 Wrapper 클래스나 String, Time, Date 같은 클래스의 인스턴스는 모두 정렬 가능하다. 

이때 기본 정렬 순서는 오름차순이다. 

Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있다는 것이다. 

Compareable을 구현하면 `검색, 극단값 계산, 자동 졍렬되는 컬렉션 관리도 쉽게 할 수 있다.` 

<br>

Comparable 인터페이스를 구현하면 이 인터페이스를 활용하는 수많은 제네릭 알고리즘과 컬렉션의 힘을 누릴 수 있다. 

사실상 자바 플랫폼 라이브러리의 모든 값 클래스와 열거 타입이 Comparable을 구현했다. 

`순서가 명확한 값 클래스 작성한다면 Comparable 인터페이스를 구현`하자. 

<br>

Comparable 인터페이스 코드는 아래와 같다. 

```java
public interface Comparable<T>{
	int compareTo(T t);
}
```

<br>

### ✴ compareTo란?

compareTo 메서드는 Object의 메서드가 아니다. 

compareTo 메서드의 `성격은 2가지만 빼면 Object.equals 메서드와 같다`. 

무엇이 다른지 보자. 

1. compareTo는 `단순 동치성 비교 + 순서까지 비교`한다.

2. compareTo는 `제너릭`하다.

<br>

Integer 클래스에서 재정의한 compareTo() 메서드 보면 비교 대상 객체와 현재 객체를 비교하여 다음 세 가지 경우 중 하나의 값을 반환한다.

- 음수: 현재 객체가 비교 대상 객체보다 작음을 나타낸다.
- 양수: 현재 객체가 비교 대상 객체보다 큼을 나타낸다.
- 0: 두 객체가 같음을 나타낸다.

이렇게 큰지 작은지도 반환하여 `순서도 비교`하는 것이다. 

<br>

또한 Comparable 인터페이스를 보면 알 수 있듯이 `제네릭을 사용`하고 있다. 

따라서 compareTo 메서드는 제너릭을 사용하여 `다양한 형태의 객체를 비교할 수 있도록 일반화` 되어 있다.

<br>

## ⚔ compareTo 규약

compareTo 메서드의 일반 규약은 `equals의 규약과 비슷`하다. 

compareTo의 일반 규약은 아래와 같다. 

<br>

>이 객체와 주어진 객체의 순서를 비교한다. 
>
>이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수로 반환한다. 
>
>이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.
>
>다음 설명에서 sgn(표현식) 표기는 수학에서 말하는 부호 함수(signum function)를 뜻하며, 표현식의 값이 음수, 0, 양수일 때 -1, 0, 1을 반환하도록 정의했다.
>
>>- Comparable을 구현한 클래스는 모든 x, y에 대해 
sgn(x.compartTo(y)) == -sgn.(y.compareTo(x))여야 한다. 
>>
>>    (따라서 x.compareTo(y)는 y.compareTo(x)가 예외를 던질때에 한해 예외를 던져야 한다).
>>
>>- Comparable을 구현한 클래스는 추이성을 보장해야 한다. 
>>
>>    즉, (x.compareTo(y) > 0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0이다.
>.
>>- Comparable을 구현한 클래스는 모든 z에 대해 
x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))다.
>>
>>- 이번 권고가 필수는 아니지만 꼭 지키는 게 좋다. 
>>
>>    (x.compareTo(y) == 0) == (x.equals(y))여야 한다. 
>>
>>    Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다. 
>>
>>    다음과 같이 명시하면 적당할 것이다.
>>
>>    "주의 : 이 클래스의 순서는 equals 메서드와 일관되지 않다."

<br>

모든 객체에 대해 전역 동치 관계를 부여하는 equals 메서드와 달리 `compareTo 메서드는 타입이 다른 객체를 신경 쓰지 않아도 된다.` 

타입이 다른 객체가 주어지면 간단히 ClassCastException 던져도 되고 대부분 그렇게 한다. 

물론 이 규약에서는 다른 타입 사이의 비교도 허용하는데 보통은 비교할 객체들이 구현한 공통 인터페이스를 매개로 이뤄진다. 

<br>

compareTo 규약을 지키지 못하면 `비교를 활용하는 클래스와 어울리지 못한다.`

hashCode 규약 지키지 못하면 HashMap, HashSet 이런 것에서 문제가 생기듯 말이다. 

`비교를 활용하는 클래스`에 예시로는 아래와 같은 것들이 있다. 

- 정렬된 컬렉션 : TreeSet, TreeMap

- 검색, 정렬 알고리즘 활용하는 유틸리티 클래스 : Collections, Arrays

<br>

아래 코드 보면 Compareable 인터페이스를 구현한 Integer 클래스는 Arrays의 sort 메서드를 사용하고 있다. 

하지만 Compareable 인터페이스 구현하지 않은 `NotMakeCompareTo 클래스는 ClassCastException 반생하면 Arrays.sort 메서드 사용하지 못하는 것`을 볼 수 있다. 

<br>

**코드**

```java
public class NotMakeCompareTo {

    private int num;

    public NotMakeCompareTo(int num) {
        this.num = num;
    }
}

----------------------------------------------------------

    public static void main(String[] args) {

        Integer[] temp = new Integer[5];
        for (int i=0;i<5;i++){
            temp[i]=5-i;
        }

        Arrays.sort(temp);
        for (Integer integer : temp) {
            System.out.println("integer = " + integer);
        }

        System.out.println();

        NotMakeCompareTo[] arr = new NotMakeCompareTo[5];
        for (int i=0;i<5;i++){
            arr[i]=new NotMakeCompareTo(i);
        }

        Arrays.sort(arr);

        for (NotMakeCompareTo notMakeCompareTo : arr) {
            System.out.println("notMakeCompareTo = " + notMakeCompareTo);
        }
    }

```

<br>

**결과**

```java
integer = 1
integer = 2
integer = 3
integer = 4
integer = 5

Exception in thread "main" java.lang.ClassCastException: class NotMakeCompareTo cannot be cast to class java.lang.Comparable
	at java.base/java.util.ComparableTimSort.countRunAndMakeAscending(ComparableTimSort.java:320)
	at java.base/java.util.ComparableTimSort.sort(ComparableTimSort.java:188)
	at java.base/java.util.Arrays.sort(Arrays.java:1041)
	at Main.main(Main.java:24)

Process finished with exit code 1
```

<br>

그렇다면 이제 compareTo 규약에 대해서 자세히 알아보자. 

<br>

## 1️⃣ 규약 1.

첫번째 규약의 의미는 `두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다는 것`이다. 

아래의 의미다.

- 객체1 > 객체2 → 객체1.compareTo(객체2) > 객체2.compareTo(객체1)

- 객체1 < 객체2 → 객체1.compareTo(객체2) < 객체2.compareTo(객체1)

- 객체1 == 객체2 → 객체1.compareTo(객체2) == 객체2.compareTo(객체1)

<br>

## 2️⃣ 규약 2.

두번째 규약의 의미는 아래와 같다. 

`객체1>객체2 && 객체2>객체3 → 객체1>객체3`

<br>

## 3️⃣ 규약 3.

세번째 규약은 크기가 `같은 객체들끼리는 어떤 객체와 비교해도 항상 같은 값 나와야 한다는 것`이다. 

객체1.compareTo(객체2) == 0 (객체1==객체2)

→ 객체1.compareTo(아무 객체) == 객체2.compareTo(아무 객체)

## #️⃣ 규약 1~3

규약 1~3은 compareTo 메서드로 수행해야 하는 동치성 검사도 equals 규약과 똑같이 `반사성, 대칭성, 추이성을 충족해야 함`을 뜻한다. 

`규약 1이 대칭성, 규약 2가 추이성, 규약 3이 반사성을 의미`한다. 

<br>

그래서 equals와 주의사항도 똑같다. 

기존 클래스를 확장한 구체 클래스에서 `새로운 값 컴포넌트를 추가했다면 compareTo 규약을 지킬 방법이 없다.` 

객체 지향적 추상화의 이점을 포기하기 전에는 지킬 수 없다.

<br>

따라서 이 문제의 `우회법도 equals와 같다`. 

확장한 구체 클래스에서 값 컴포넌트 추가하고 싶다면 확장 대신 그냥 클래스 만들고 이 클래스에 원래 인스턴스 가리키는 필드 두는 것이다. 

그리고 원래 클래스 인스턴스 반환하는 메서드 제공하면 된다. 

<br>

## 4️⃣ 규약 4.

마지막 규약은 `필수는 아니지만 꼭 지키기를 권한다`고 한다. 

마지막 규약의 의미는 compareTo 메서드로 수행한 동치성 테스트의 결과가 equals 메서드로 수행한 동치성 테스트의 결과와 같아야 한다는 것이다. 

즉 `객체1==객체2 이면 객체1.compareTo(객체2) == 0 && 객체1.equals(객체2) == true`

<br>

이렇게 결과 안 같게 해도 괜찮다. 

그래도 클래스는 여전히 동작한다. 

단, 이 클래스의 `객체를 정렬된 컬렉션에 넣으면` 해당 컬렉션이 구현한 인터페이스 (Collection, Set, Map)에 정의된 동작과 엇박자를 낼 것이다. 

<br>

왜냐하면 이 인터페이스들은 equals 메서드 규약을 따른다고 되어 있다.

하지만 `놀랍게도 정렬된 컬렉션들은 동치성 비교할 때 equals 대신 compareTo를 사용`한다. 

<br>

따라서 논리적으로 같은 두 객체 equals 메서드 재정의 해서 같다고 했는데 compareTo는 재정의 하지 않은 것이다. 

그렇게 되면 컬렉션에 넣으면 이상하게 동작하는 것이다. 

<br>

## ❎ 규약 4 안 지킨 예시

예시를 보자. 

`compareTo와 equals가 일관되지 않는 예시로 BigDecimal`이 있다. 

BigDecimal의 compareTo와 equals 메서드는 다음과 같다. 

<br>

**BigDeciaml.compareTo**

```java
    @Override
    public int compareTo(BigDecimal val) {
        // Quick path for equal scale and non-inflated case.
        if (scale == val.scale) {
            long xs = intCompact;
            long ys = val.intCompact;
            if (xs != INFLATED && ys != INFLATED)
                return xs != ys ? ((xs > ys) ? 1 : -1) : 0;
        }
        int xsign = this.signum();
        int ysign = val.signum();
        if (xsign != ysign)
            return (xsign > ysign) ? 1 : -1;
        if (xsign == 0)
            return 0;
        int cmp = compareMagnitude(val);
        return (xsign > 0) ? cmp : -cmp;
    }
```

<br>

**BigDeciaml.equals**

```java
    @Override
    public boolean equals(Object x) {
        if (!(x instanceof BigDecimal xDec))
            return false;
        if (x == this)
            return true;
        if (scale != xDec.scale)
            return false;
        long s = this.intCompact;
        long xs = xDec.intCompact;
        if (s != INFLATED) {
            if (xs == INFLATED)
                xs = compactValFor(xDec.intVal);
            return xs == s;
        } else if (xs != INFLATED)
            return xs == compactValFor(this.intVal);

        return this.inflated().equals(xDec.inflated());
    }

```
<br>

코드를 다 해석할 필요 없다. 

여기서 중요한 점은 BigDecimal의 equals는 `파라미터로 BigDecimal 타입이 아닌 파라미터가 오면 false를 반환`한다. 

하지만 BigDecimal의 compareTo 메서드는 파라미터로 String이 오면 BigDecimal의 `String을 받는 생성자가 실행돼 BigDecimal을 만들어 넣어줘서 실행`한다. 

<br>

따라서 아래 코드를 보면 결과가 `0, false`가 나온다. 

```java
        BigDecimal t1= new BigDecimal("10.0");
        BigDecimal t2= new BigDecimal("10.000");

        System.out.println(t1.compareTo(t2));
        System.out.println(t1.equals(t2));
        
        //결과
        -------------------
        0
        false
```

<br>

이렇게 되면 `동치 비교에 compareTo 사용하는 컬렉션과 equals 사용하는 컬렉션에 동작이 이상해진다`. 

위처럼 BigDecimal을 생성하고 HashSet에 넣으면 HashSet은 원소 2개를 갖는다. 

HashSet은 equals 메서드를 사용해서 동치를 판단하기 때문이다. 

<br>

하지만 TreeSet에 두 인스턴스를 넣으면 원소 1개만 갖는다. 

TreeSet은 compareTo 메서드로 비교하기 때문이다. 

<br>

따라서 compareTo 메서드로 수행한 동치성 테스트의 결과가 equals 메서드로 수행한 동치성 테스트의 결과와 `같게 하는 것이 좋다는 것`이다. 

<br>

## 🧾 compareTo 작성 요령

compareTo 메서드 작성 요령은 equals와 비슷하다. 

`몇 가지 차이점만 주의`하면 된다. 

<br>

주의점은 아래와 같다. 

1. compareTo 메서드 파라미터 타입 확인하거나 형변환할 필요 없다. 

2. null 파라미터로 넣어 호출하면 NullPointerException 던져야 한다. 
3. 객체 참조 필드 비교하려면 compareTo 메서드 재귀적으로 호출해야 한다. 
4. 객체 참조 필드 중 Comparable 구현하지 않은 필드나 표준 아닌 순서로 비교해야 한다면 비교자 대신 사용한다. 
5. 기본 타입 필드 비교하려면 박싱된 기본 타입 클래스에 새로 추가된 정적 메서드인 compare 사용해야 한다. 
6. 클래스에서 핵심 필드 여러 개라면 어느것 먼저 비교하느냐가 중요하다. 

<br>

이제 각 주의점들에 대해서 보자. 

<br>

## 🔴 주의점 1

첫번째 주의점은 compareTo 메서드 파라미터 타입 확인하거나 형변환할 필요 없다는 것이다. 

<br>

왜 그런가 하면 Comparable은 타입을 파라미터로 받는 `제네릭 인터페이스`다. 

따라서 compareTo 메서드의 파라미터 타입은 컴파일 타임에 정해진다. 

파라미터 타입이 잘못됐다면 컴파일 자체가 되지 않는다. 

따라서 `파라미터 타입 확인하거나 형변환할 필요 없는 것`이다. 

<br>

## 🟠 주의점 2

두번째 주의점은 null 파라미터로 넣어 호출하면 NullPointerException 던져야 한다는 것이다.

<br>

`구현할 때 null 들어왔는지 확인해야 한다는 것`이다. 

물론 compareTo 메서드에서 파라미터의 멤버에 접근하려고 하는 순간 파라미터가 null이니 예외는 던져질 것이다. 

<br>

## 🟡 주의점 3

세번째 주의점은 `객체 참조 필드 비교하려면 compareTo 메서드 재귀적으로 호출해야 한다는 것`이다. 

<br>

compareTo 메서드는 각 필드가 동치인지 비교하는 것이 아니라 순서를 비교한다. 

따라서 참조 필드의 경우 주소 값의 순서 비교하게 될 것이다.

제대로 된 compareTo 메서드 구현하려면 item 13에서 본 clone 메서드처럼 재귀적으로 compareTo 메서드를 호출해 비교해야 한다는 것이다. 

<br>

## 🟢 주의점 4

네번째 주의점은 `객체 참조 필드 중 Comparable 구현하지 않은 필드나 표준 아닌 순서로 비교해야 한다면 비교자 대신 사용한다는 것`이다. 

<br>

비교자 = Comparator는 아래에서 나오니 확인하자. 

비교자는 직접 만들거나 자바가 제공하는 것 중에 골라서 사용하면 된다. 

<br>

## 🔵 주의점 5

다섯번째 주의점은 `기본 타입 필드 비교하려면 박싱된 기본 타입 클래스에 새로 추가된 정적 메서드인 compare 사용해야 한다는 것`이다. 

<br>

이전에는 compareTo 메서드에서 정수 기본 타입 필드 비교할 때에는 관계 연산자인 <와 >를 사용하라고 했다. 

그리고 실수 기본 타입 필드 비교할 때는 정적 메서드인 Double.compare와 Float.compare 사용하라고 했다. 

<br>

그런데 `자바 7부터 상황이 바뀌었다.` 

박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드인 compare 사용하면 된다. 

`이전의 방식은 거추장스럽고 오류를 유발해 이제는 추천하지 않는다.` 

<br>

## 🟣 주의점 6

여섯번째 주의점은 클래스에서 핵심 필드 여러 개라면 `어느 것 먼저 비교하느냐가 중요하다는 것`이다. 

<br>

compareTo를 구현할 때에는 `가장 핵심적인 필드부터 비교`해나가자.

compareTo의 결과가 0이 아니라면 즉 순서가 결정되면 더이상 다른 필드 비교하지 않아도 된다. 

따라서 `순서 결정되면 바로 결과 반환`할 수 있다. 

핵심적인 필드 비교해서 얼른 비교를 끝내는 것이다. 

<br>

핵심 필드가 똑같다면 `똑같지 않은 필드를 찾을 때까지` 그 다음으로 중요한 필드를 비교해 나가야 한다. 

<br>

예시로 아래와 같이 compareTo 메서드 작성하면 된다. 

아래에 3개의 필드 areaCode, prefix, lineNum이 있다. 

중요도는 areaCode>prefix>lineNum인 것이다. 

```java
    // 코드 14-2 기본 타입 필드가 여럿일 때의 비교자 (91쪽)
    public int compareTo(PhoneNumber pn) {
        int result = Short.compare(areaCode, pn.areaCode);
        if (result == 0)  {
            result = Short.compare(prefix, pn.prefix);
            if (result == 0)
                result = Short.compare(lineNum, pn.lineNum);
        }
        return result;
    }
```

<br>

## ⚖ Comparator 인터페이스

Comparator는 Java.util의 Comparator.class에서 Comparator는 하나의 public abstract 메서드를 구현해야 하는 함수형 인터페이스다. 

두 개의 객체를 넘겨받아 그것의 `크기 비교를 수행하는 추상 메서드인 compare를 구현`하도록 강제하고 있다.

compare 메서드는 두 객체 중 만약 앞의 객체가 더 작다면 음수, 같다면 0, 크다면 양수를 반환해야 한다.

<br>

코드는 아래와 같다. 

```java
@FunctionalInterface
public interface Comparator<T> {

    /**
    * Returns:
    * a negative integer, zero, or a positive integer as the first
    * argument is less than, equal to, or greater than the second.
    */
    
    int compare(T o1, T o2);

    . . .
}
```

<br>

Comparator를 만들고는 아래와 같이 사용하는 것이다. 

```java
//Comparator 인터페이스 구현한 구현체가 MyComparator 클래스인 것이다. 
MyComparator comparator = new MyComparator();
comparator.compare(객체1, 객체2);
```

<br>

## ⛓ Comparator의 메서드 연쇄 방식

자바 8에서는 Comparator 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 `메서드 연쇄 방식으로 비교자 생성`할 수 있게 되었다. 

그리고 이 Comparator들을 Comparable 인터페이스가 원하는 compareTo 메서드 구현하는데 멋지게 활용할 수 있다. 

<br>

>메서드 연쇄 방식은 `여러 메서드 호출을 연달아 이어서 호출하는 방식`을 의미한다. 
>
>각 메서드는 `객체를 반환`하고, 이를 통해 연쇄적으로 다른 메서드를 호출할 수 있는 것이다. 
>
>이는 코드를 보다 간결하고 읽기 쉽게 만들어준다. 

<br>

Comparator 인터페이스에 추가된 메서드 중에서 **`Comparator.comparing()`** 메서드는 주어진 키(필드, 메서드 등)를 기준으로 객체를 비교하는 Comparator를 생성한다. 

또한 **`thenComparing()`** 메서드는 두 번째 기준으로 객체를 비교하는 Comparator를 생성하고, 앞의 비교가 같다고 나온 경우에만 이를 적용한다.

<br>

이러한 메서드를 연쇄적으로 사용하여 Comparator를 생성하면 다음과 같이 보다 간결한 코드를 작성할 수 있다

```java
// Comparator 인터페이스의 정적 메서드를 이용하여 Comparator 생성
Comparator<Person> comparator = Comparator.comparing(Person::getLastName)
                                          .thenComparing(Person::getFirstName);

```

<br>

위의 예제에서는 **Person** 객체를 비교하는 Comparator를 생성하는데, 먼저 성(LastName)을 기준으로 정렬하고, 동일한 성일 경우에는 이름(FirstName)을 추가 기준으로 정렬 하는 것이다. 

<br>

많은 프로그래머가 이 방식의 간결함에 매혹되지만 `약간의 성능 저하가 뒤따른다.` 

<br>

## 🧬 Comparator의 메서드 연쇄 방식 예시

참고로 `자바의 static import 기능 이용`하면 static Comparator 생성 메서드들은 그 이름만으로 사용할 수 있어 `코드가 더 깔끔`해진다. 

<br>

아래 코드와 같이 사용하는 것이다. 

```java
    // 코드 14-3 비교자 생성 메서드를 활용한 비교자 (92쪽)
    private static final Comparator<PhoneNumber> COMPARATOR =
            comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.prefix)
                    .thenComparingInt(pn -> pn.lineNum);

    public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
    }
```

<br>

이 코드는 comparintInt, thenComparingInt 2개의 비교자 생성 메서드 이용해 Comparator를 생성한다. 

<br>

### ◼ comparingInt

`comparingInt`는 객체 참조를 int 타입 키에 매핑하는 키 추출 함수를 인수로 받는다. 

그리고 그 `키를 기준으로 순서를 정하는 비교자를 반환하는 메서드`다. 

위의 코드에서는 람다를 인수로 받고 있다. 

이 람다는 PhoneNumber 클래스에서 추출한 areaCode 필드를 기준으로 PhoneNumber의 순서 정하는 Comparator<PhoneNumber>를 반환한다. 

<br>

>여기 파라미터인 `람다에서 파라미터 타입을 명시`했다. 
>
>(PhoneNumber pn) 이렇게 명시한 것을 볼 수 있다. 
>
>왜냐하면 `자바의 타입 추론 능력이 처음부터 타입 알아낼만큼 똑똑하지 못하다`. 
>
>따라서 프로그램이 컴파일 되도록 우리가 도와줘야 하는 것이다. 

<br>

### ◼ thenComparingInt

두 PhoneNumber 객체의 areaCode 필드가 같을 수 있다. 

따라서 `다음 비교를 진행`해야 한다. 

이 일은 두번째 Comparator 생성 메서드인 `thenComparingInt`가 한다. 

<br>

thenComparingInt 메서드는 Comparator의 인스턴스 메서드다. 

역시 thenComparingInt 메서드도 int 키 추출자 함수를 파라미터로 입력 받아 다시 Comparator를 반환한다. 

이 Comparator는 앞의 `첫번째 Comparator를 적용한 다음에 새로 추출한 키로 추가 비교 수행하는 것`이다.

thenComparingInt 메서드는 `원하는 만큼 연달아 호출`할 수 있다. 

 <br>

>이번에 thenComparingInt의 파라미터인 람다를 보면 `파라미터에 타입을 명시하지 않았다.`
>
>자바의 타입 추론 능력이 `앞의 것을 보고 이제는 추론할 수 있기 때문`에 명시하지 않아도 되는 것이다. 

<br>

## 🆘 Comparator의 보조 생성 메서드

Comparator는 수많은 `보조 생성 메서드`들로 중무장하고 있다. 

- `long, double용` : comparingInt, thenComparingInt 변형 메서드 사용 
(comparingLong, comparingDouble)

- `short용` : int보다 작은 정수 타입은 comparingInt, thenComparingInt 사용

- `float용` : float는 double용 사용

<br>

이런 식으로 자바의 숫자용 기본 타입을 모두 커버하는 메서드들이 있다. 

<br>

## 🆙 Comparator의 객체 참조용 비교자 생성 메서드

Comparator에는 `객체 참조용 Comparator 생성 메서드`도 준비되어 있다. 

<br>

우선 `comparing`이라는 정적 메서드 2개가 다중정의 되어 있다. 

1. 키 추출자 받아 그 키의 자연적 순서 사용하는 메서드

2. 키 추출자 하나와 추출된 키를 비교한 Comparator 이렇게 2개 받아 사용하는 메서드
   
<br>

코드를 보면 아래와 같다. 

```java
// 1번 메서드
    public static <T, U extends Comparable<? super U>> Comparator<T> comparing(
            Function<? super T, ? extends U> keyExtractor)
    {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2));
    }

// 2번 메서드
    public static <T, U> Comparator<T> comparing(
            Function<? super T, ? extends U> keyExtractor,
            Comparator<? super U> keyComparator)
    {
        Objects.requireNonNull(keyExtractor);
        Objects.requireNonNull(keyComparator);
        return (Comparator<T> & Serializable)
            (c1, c2) -> keyComparator.compare(keyExtractor.apply(c1),
                                              keyExtractor.apply(c2));
    }
```

<br>

그리고 `thenComparing`이란 인스턴스 메서드가 3개 다중정의 되어 있다. 

1. Comparator 하나 파라미터로 받아 이 Comparator로 순서 정하는 메서드

2. 키 추출자 파라미터로 받아 그 키의 자연적 순서로 순서 정하는 메서드

3. 키 추출자 하나와 추출된 키를 비교할 Comparator 2개 파라미터로 받아 사용하는 메서드

<br>

코드를 보면 아래와 같다. 

```java
// 1번 메서드 
    default Comparator<T> thenComparing(Comparator<? super T> other) {
        Objects.requireNonNull(other);
        return (Comparator<T> & Serializable) (c1, c2) -> {
            int res = compare(c1, c2);
            return (res != 0) ? res : other.compare(c1, c2);
        };
    }
    
//  2번 메서드
    default <U extends Comparable<? super U>> Comparator<T> thenComparing(
            Function<? super T, ? extends U> keyExtractor)
    {
        return thenComparing(comparing(keyExtractor));
    }

// 3번 메서드
    default <U> Comparator<T> thenComparing(
            Function<? super T, ? extends U> keyExtractor,
            Comparator<? super U> keyComparator)
    {
        return thenComparing(comparing(keyExtractor, keyComparator));
    }
```

<br>

## ❌ 이상한 compareTo & compare 메서드

가끔 `값의 차를 기준`으로 첫번째 값이 두번째 값보다 작으면 음수를, 두 값이 같으면 양수를 첫번째 값이 두번째 값보다 크면 양수를 반환하는 compareTo, compare 메서드가 있다. 

<br>

예시 코드는 아래와 같다. 

```java
static Comparator<Object> hashCodeOrder = new Comparator<>(){
	public int compare(Object 01, Object 02){
		return o1.hashCode() - o2.hashCode();
	}
};
```

<br>

이렇게 `사용하면 안된다.` 

이 방법은 `정수 오버플로 일으키거나 IEEE 754 부동소수점 계산 방식에 따른 오류를 낼 수 있다.` 

그렇다고 위에서 설명한 방식대로 구현한 코드보다 월등히 빠르지도 않을 것이다. 

<br>

이 방법 말고 아래의 `두 방식 중 하나를 사용`하자. 

1. 정적 compare 메서드를 활용한 Comparator

```java
static Comparator<Object> hashCodeOrder = new Comparator<>(){
	public int compare(Object 01, Object 02){
		return Integer.compare(o1.hashCode() , o2.hashCode());
	}
};
```

<br>

2.  comparator 생성 메서드 활용한 Comparator

```java
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```

<br>

## ‼ 결론

`순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현`해야 한다. 

구현해서 그 인스턴스들을 쉽게 정렬, 검색하고 비교 기능 제공하는 컬렉션과 함께 쓰일 수 있게 해야 한다. 

<br>

## 📍 references

- [Comparable](https://tcpschool.com/java/java_collectionFramework_comparable)
- [compareTo 1](https://lifeonguide.tistory.com/123)
- [compareTo 2](https://mine-it-record.tistory.com/133)
- [comparator](https://jaehee329.tistory.com/11)