# 🔟 Item 10: equals는 일반 규약을 지켜 재정의하라

<br>

## 📌 목차
1. equals 메서드란?
2. equals 메서드 문제점
3. equals 메서드 재정의 X 경우
   1. 각 인스턴스가 본질적으로 고유한 경우 
   2. 인스턴스의 논리적 동치성을 검사할 일이 없는 경우
   3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 맞는 경우 
   4. equals 메서드를 호출할 일이 없는 경우
4. equals 메서드 재정의 O 경우
5. equals 메서드 재정의 시 일반 규약
6. 동치 관계
   1. 관계
   2. 동치관계
   3. 동치류
   4. equals에서 동치관계 
7. 1.반사성
   1. 반사성 어긴 예시
8. 2.대칭성
   1. 대칭성 어긴 예시
   2. 대칭성 만족 예시
9. 3.추이성
   1.  추이성 어긴 예시
   2.  추이성 만족….?
   3.  추이성 만족 with 우회
   4.  추이성 만족 with 추상 클래스
10. 4.일관성
    1.  일관성 어긴 예시
11. 5.null 아님
12. equals 메서드 구현 방법
13. 필드 비교 시 타입에 따른 비교 방법
14. equals 메서드 성능
15. equals 메서드 구현 후
16. equals 메서드 구현 주의 사항
    1.  equals를 재정의할 땐 hashCode도 반드시 재정의하자.
    2.  너무 복잡하게 해결하려 들지 말자.
    3.  Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자. 
17. equals 메서드 대신 만들기

<br>

## ❓❓ equals 메서드란?

equals이란 자바의 모든 클래스의 가장 최상위 클래스인 `Object 클래스가 가진 메서드 중 하나`다. 

따라서 모든 클래스들은 equals 메서드를 사용할 수 있다.

equals 메서드는 `두 인스턴스의 주소값을 비교하여 같은 인스턴스인지를 확인`하는 메서드다. 

<br>

하지만 이렇게 물리적으로 주소가 같은 경우가 아니라 `논리적으로 두 인스턴스가 동일하다고 판단될 때` equals 메서드를 사용하려면 equals 메서드를 `재정의` 해야 한다. 

<br>

예를 들어 아래와 같은 경우이다. 

우리는 아래와 같은 코드에서 두 문자열이 같은 “abcd” 값을 가지니 같은 것이라고 하고 싶다. 

```java
String string1 = "abcd";
String string2 = "abcd";

System.out.println(string1.equals(string2));
```

<br>

만약에 equals가 객체의 주소값을 비교하여 같은 객체인지 확인하는 메서드라면 위에서는 false가 나온다. 

하지만 우리가 프로그램상 원하는 것은 `두 문자열이 같은 값을 가지면 equals 메서드를 사용했을 때 true가 나와야 하는 것`이다. 

위처럼 논리적으로 두 객체가 같을 때 equals를 사용해 true가 나오게 하고 싶다면 equals 메서드를 오버라이딩해 프로그램상 같다고 판단할 수 있도록 구현해야 하는 것이다. 

<br>

따라서 String 클래스의 equals를 보면 아래와 같이 오버라이딩 되어 재정의 해서 같은 문자열이면 equals를 사용 시 true가 나온다. 

<br>

**`String 클래스`**

```java
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        return (anObject instanceof String aString)
                && (!COMPACT_STRINGS || this.coder == aString.coder)
                && StringLatin1.equals(value, aString.value);
    }
```

**`StringLatin1 클래스`**

```java
    @IntrinsicCandidate
    public static boolean equals(byte[] value, byte[] other) {
        if (value.length == other.length) {
            for (int i = 0; i < value.length; i++) {
                if (value[i] != other[i]) {
                    return false;
                }
            }
            return true;
        }
        return false;
    }
```

<br>

## ⚠ equals 메서드 문제점

equals 메서드는 재정의 하기 쉬워 보이지만 제대로 만들지 않으면 자칫하면 `끔찍한 결과 초래`한다. 

따라서 문제를 회피하는 가장 쉬운 길은 `아예 재정의 하지 않는 것`이다. 

그냥 두면 그 클래스의 인스턴스는 오직 자기 자신과만 같게 된다. 

<br>

`재정의 하지 않는 경우가 최선인 경우`가 있는데 살펴보자. 

<br>

## ❌ equals 메서드 재정의 X 경우

재정의 하지 않는 경우가 최선인 경우는 `4가지`가 있다. 

1. 각 인스턴스가 본질적으로 고유한 경우 

2. 인스턴스의 논리적 동치성을 검사할 일이 없는 경우 

3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 맞는 경우 

4. equals 메서드 호출할 일 없는 경우 

<br>

각각의 경우에 대해서 자세히 알아보자. 

<br>

### 🟡 1. 각 인스턴스가 본질적으로 고유한 경우

클래스가 값을 표현하는 것이 아니라 `동작하는 개체를 표현`하는 클래스가 이런 경우이다. 

Thread 클래스가 이런 예시로 Object 클래스의 equals 메서드는 이런 클래스에 딱 맞게 구현되었다. 

프로그램 상에서 클래스가 가진 값이 중요한 것이 아니라 그저 어떤 동작을 수행하기 위한 것이라면 재정의해서 클래스가 `가진 값을 비교할 필요가 없다는 것`이다. 

<br>

### 🟡 2. 인스턴스의 논리적 동치성을 검사할 일이 없는 경우

논리적 동치성은 위에서 말한 것처럼 프로그램에서 논리적으로 두 인스턴스가 같은지 비교하는 것이다. 

설계자는 클라이언트가 `논리적 동치성을 검사하는 것을 원하지 않거나 애초에 필요하지 않다고 판단`할 수 있다. 

설계자가 후자로 판단했다면 Object의 기본 equals만으로 해결된다. 

<br>

### 🟡 3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 맞는 경우

상위 클래스에서 재정의한 equals가 하위 클래스의 논리적 동치성도 잘 판단해준다면 굳이 재정의할 필요 없이 `상위 클래스의 equals를 사용하면 되는 것`이다. 

<br>

자바에서는 Set, List, Map 구현체는 각각 AbstractSet, AbstractList, AbstractMap이 구현한 equals를 상속 받아서 사용한다. 

<br>

아래는 AbstractSet 클래스의 equals 메서드이다. 

보면 containsAll 메서드를 사용해 Set에 든 원소를 다 가지고 있는지를 확인해 같은지 다른지 판단한다.

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

<br>

### 🟡 4. equals 메서드를 호출할 일이 없는 경우

클래스가 private이거나 package-private이고 `equals 메서드를 호출할 일이 없으면 재정의 할 필요가 없다.` 

위협을 철저히 회피하고 equals가 `실수로라도 호출되는 것을 막고 싶다`면 아래처럼 구현하면 된다. 

```java
@Override public boolean equals(Object o){
	throw new AsserionError(); // 호출 금지!
}
```

<br>

## ⭕ equals 메서드 재정의 O 경우

그렇다면 equals를 `재정의 해야 할 때`는 언제일까?

***객체 식별성 = 두 객체가 물리적으로 같은가가 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때다.***

<br>

주로  Integer, String처럼 값을 표현하는 클래스인 값 클래스가 이런 경우다. 

값 객체를 equals로 비교하는 프로그래머는 두 객체의 주소가 같은지가 아니라 값이 같은지를 알고 싶어한다. 

따라서 equals가 `논리적 동치성을 확인하도록 재정의` 해야 한다. 

이렇게 하면 프로그램에서 프로그래머의 의도대로 할 수 있고 Map의 키와 Set의 원소로 사용할 수 있게 된다.

Map의 키와 Set의 원소는 같은 것이 있으면 안되기 때문에 값을 비교해야 한다. 

그런데 재정의 하지 않으면 주소를 비교하기 때문에 값이 같은 것이 들어갈 수 있다. 

<br>

그런데 값 클래스라도 `아래와 같은 경우에는 equals를 재정의 하지 않아도 된다.` 

1. 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스

2. Enum 클래스

<br>

이런 클래스들은 어차피 `논리적으로 같은 인스턴스가 2개 이상 만들어지지 않는다.`

따라서 논리적 동치성 = 객체 식별성이다. 

따라서 `Object 클래스의 equals 메서드`가 논리적 동치성까지 확인해주는 것이다. 

<br>

## 🔒 equals 메서드 재정의 시 일반 규약

equals 메서드를 재정의 할 때는 반드시 `일반 규약`을 따라야 한다. 

다음은 Object 명세에 적힌 규약이다. 

<br>

---
equals 메서드는 반드시 동치관계(equivalence relation)을 구현하며, 다음을 만족한다.

1. 반사성 (reflexivity) : null이 아닌 모든 참조 값 x에 대해 x.equals(x) 는 true이다.

2. 대칭성 (symmetry) : null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true이면 y.equals(x)도 true이다.

3. 추이성 (transitivity) : null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고 y.equals(z)도 true이면, x.equals(z)도 true이다.

4. 일관성 (consistency) : null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환해야 한다.

5. null 아님 : null이 아닌 모든 참조 값 x에 대하여 x.equals(null)은 false이다.

---
<br>

이 규약은 `굉장히 중요`하다. 

이 규약을 어기면 아래와 같은 `문제점이 발생`한다. 

- 프로그램이 **이상하게 동작하거나 종료**될 것이다.
- 문제의 원인이 되는 코드 **찾기 굉장히 어려울 것**이다.

<br>

왜 이런 문제들이 발생하는가 하면 수 많은 클래스들이 `equals 규약을 지킨다고 가정하고 동작`하기 때문이다. 

<br>

이제 규약 각각에 대해 자세히 알아보자. 

<br>

## 👥 동치 관계

먼저 ‘equals 메서드는 반드시 동치관계(equivalence relation)을 구현하며, 다음을 만족한다.’ 라는 구문에서 `‘동치관계’`가 무엇인지 알아보자. 

<br>

### 🔗 관계

동치관계를 알기 전에 우선 `관계`에 대해서 알아야 한다. 

`집합에서의 관계`는 집합 A와 집합 B가 있다면 두 집합의 데카르트 곱 A x B의 하나의 부분 집합이다. 

이 관계에 해당하는 순서쌍이 (a, b)라면 aRb 이렇게 표현한다. 

그리고 a가 b와 R의 관계가 있다고 한다. 

<br>

수식으로 표현하면 아래와 같다. 

![Untitled](https://file.notion.so/f/f/7d1044b7-24d0-4b32-b24a-1b9061f7bf65/5e4749b0-6907-447e-855f-c4f8cc8dd487/Untitled.png?id=29823b63-36ca-4346-86a0-d6033f99c1d5&table=block&spaceId=7d1044b7-24d0-4b32-b24a-1b9061f7bf65&expirationTimestamp=1710000000000&signature=Aj4n1S7vSIMlrWEf31slGrz5ExM4sVoPd-bygjGLNUw&downloadName=Untitled.png)

<br>

예를 들어보자. 

A = {1, 2, 3, 4}, B = {1, 2, 3, 4} 이고 관계 R은 ‘우항 b보다 좌항 a가 작다’ 이다. 

그렇다면 관계 R은 다음과 같이 표현할 수 있다. 

`R = { (1,2), (1,3), (1,4), (2,3), (2,4), (3,4) }`

<br>

### ⛓ 동치관계

동치관계는 `관계 중에 특별한 조건들을 만족하는 특수한 관계`이다. 

관계 중 `반사율, 대칭률, 추이율을 모두 만족하는 관계`가 동치관계이다. 

<br>

쉽게 이야기 하면 수학적으로 동치관계는 `‘같다’`는 개념의 일반화이다. 

관계에서 동치관계라는 것은 `집합의 원소들이 ‘같다’`라는 것을 의미하는 것이다. 

<br>

### 🧬 동치류

R은 공집합이 아닌 집합 A의 동치 관계이다. 

이때, `A의 원소 a와 관계된 모든 원소의 집합을 a의 동치류`라 한다.

a의 동치류는 [a] 이렇게 표현한다. 

수식으로 보면 `[a]={x∈Aㅣ(a,x)∈R}` 이다.

<br>

무슨 의미인지 알기 어렵다. 

예시를 보고 이해하자. 

<br>

집합 A={1,2,3,4,5}가 있고 동치관계 R={ (1,1), (1,5), (2,2), (2,3), (3,2), (3,3), (4,4), (5,1), (5,5) }이 있다. 

이때 1의 동치류는 [1] = {1,5}, 2의 동치류는 [2] = {2,3}, 3의 동치류는 [3] = {2,3} .. 이렇게 된다. 

<br>

수식에 대입해보면 [1]은 R 중에서 (1,x)인 것을 찾으면 되는데 (1,1), (1,5)가 있기 때문에 1의 동치류는 {1,5}가 되는 것이다. 

<br>

### 👥 equals에서 동치관계

다시 equals로 돌아와 eqauls 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환할 수 있어야 한다. 

즉 `equals가 동치관계를 만족해야 한다는 뜻`이다. 

이제 equals에서 동치관계를 만족시키기 위한 `5가지 요건`을 하나씩 살펴보자. 

<br>

## 🔴 1. 반사성

`반사성 (reflexivity)` : null이 아닌 모든 참조 값 x에 대해 x.equals(x) 는 true이다.

<br>

반사성은 `객체는 자기 자신과 같아야 한다는 뜻`이다. 

이 요건은 일부러 어기는 경우 아니라면 만족시키지 못하기가 더 어려워 보인다. 

<br>

### ❎ 반사성 어긴 예시

만약에 한 클래스에서 equals 메서드에서 반사성을 어기고 구현하면 어떻게 될 것인가?

이 클래스의 인스턴스를 컬렉션에 넣은 다음 contains 메서드를 호출하면 방금 넣은 인스턴스가 없다고 답할 것이다. 

`방금 넣었는데 없다고 답하는 말이 안되는 상황이 발생하는 것`이다. 

따라서 equals 메서드를 재정의 할 때 반사성을 지켜야 한다. 

<br>

## 🔴 2. 대칭성

`대칭성 (symmetry)` : null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true이면 y.equals(x)도 true다.

<br>

대칭성은 `두 객체가 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻`이다. 

<br>

### ❎ 대칭성 어긴 예시

예시로 **대소문자를 구별하지 않는 문자열 클래스**를 구현한 것이다. 

그리고 이 클래스의 equals 메서드에서 대칭성을 위배한 상황이다. 

```java
// 코드 10-1 잘못된 코드 - 대칭성 위배! (54-55쪽)
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
        if (o instanceof String)  // 한 방향으로만 작동한다!
            return s.equalsIgnoreCase((String) o);
        return false;
    }
    
    ...
}
```

<br>

equals 메서드를 보면 순진하게 `일반 문자열 String 타입과도 비교를 시도`한다. 

아래처럼 CaseInsensitiveString과 일반 String 객체가 하나씩 있다고 해보자. 

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";
```

<br>

이렇게 있을 때 cis.equals(s)는 `true`를 반환한다. 

왜냐하면 CaseInsensitiveString의 equals에서 String 타입이 들어와도 CaseInsensitiveString 클래스의 문자열을 대소문자 무시한 후 비교하기 때문이다. 

<br>

문제는 CaseInsensitiveString의 equals는 일반 String 타입을 알고 있다. 

하지만 String의 equals는 CaseInsensitiveString 타입의 존재를 모른다. 

<br>

따라서 s.equals(cis)는 `false`를 반환한다. 

이렇게 되면 `대칭성을 위반`하는 것이다. 

두 객체 CaseInsensitiveString와 String가 동치 여부를 똑같이 답하지 않았다. 

<br>

### ✅ 대칭성 만족 예시

equals 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다. 

이런 문제들을 해결하려면 CaseInsensitiveString의 equals를 `String과 연동하겠다는 생각은 버려야 한다`. 

<br>

그냥 아래처럼 간단히 수정하면 된다. 

equalsIgnoreCase는 두 문자열이 대소문자 상관없이 같은지 확인하는 메서드다. 

```java
    // 수정한 equals 메서드 (56쪽)
    @Override public boolean equals(Object o) {
        return o instanceof CaseInsensitiveString &&
                ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
    }
```

<br>

## 🔴 3. 추이성

`추이성 (transitivity)` : null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고 y.equals(z)도 true이면, x.equals(z)도 true이다.

<br>

추이성은 첫번째 객체와 두번체 객체가 같고, 두번째 객체와 세번째 객체가 같다면, 첫번째 객체와 세번째 객체도 같아야 한다는 뜻이다. 

`a==b, b==c → a==c 의 의미`다. 

<br>

### ❎ 추이성 어긴 예시

`상위 클래스에는 없는 새로운 필드를 하위 클래스에 추가하는 상황`을 생각해보자. 

equals 비교에 영향 주는 정보를 추가한 것이다. 

예시로 **2차원에서의 점을 표현하는 클래스**를 볼 것이다. 

<br>

**`상위 클래스`**

```java
// 단순한 불변 2차원 정수 점(point) 클래스 (56쪽)
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
    
    ...
}
```

**`하위 클래스`**

상위 클래스 Point에서 확장해서 점에 색상 더하기 위해 `color라는 필드를 넣은 것`이다. 

```java
import effectivejava.chapter3.item10.Color;
import effectivejava.chapter3.item10.Point;

// Point에 값 컴포넌트(color)를 추가 (56쪽)
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
    
    ...
}
```

<br>

equals 메서드를 재정의 하지 않는다면` Point의 equals 메서드 구현이 상속되어 color 필드는 무시한채 비교가 될 것`이다. 

equals 규약을 어긴 것은 아니지만 중요한 정보를 놓치게 되니 받아드릴 수 없는 상황이다. 

<br>

**1️⃣equals 구현 1**

그래서 아래와 같이 equals를 구현한 것이다. 

비교 대상이 또 다른 ColorPoint이고 `위치와 색상이 같을 때만 true를 반환`하는 것이다. 

```java
    // 코드 10-2 잘못된 코드 - 대칭성 위배! (57쪽)
    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
```

<br>

이렇게 구현하면 상위 클래스인 Point를 하위 클래스 ColorPoint에 비교한 결과와 둘을 바꿔 비교한 결과가 다를 것이다. 

```java
Point p = new Point(1,2);
ColorPoint cp = new ColorPoint(1, 2, Color.RED);
```

<br>

왜냐하면 Point의 equals는 색상을 무시하고 비교 진행하여 `true`를 반환한다. 

ColorPoint의 equals는 Point의 클래스 종류가 다르다고 매번 `false`만 반환한다. 

따라서 `대칭성을 어기게 된다`. 

<br>

**2️⃣equals 구현 2**

그렇다면 ColorPoint의 equals 메서드에서 `Point와 비교할 때에는 색상 무시`하도록 하면 해결 될 것 같다. 

아래 코드와 같이 ColorPoint의 equals 메서드를 구현하는 것이다. 

```java
    // 코드 10-3 잘못된 코드 - 추이성 위배! (57쪽)
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

<br>

이렇게 하면 `대칭성은 지키지만 추이성을 깬다.` 

아래와 같이 진행하면 추이성이 깨지는 것을 볼 수 있다. 

```java
        ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
        Point p2 = new Point(1, 2);
        ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
```

<br>

p1.equals(p2), p2.equals(p3)는 `true`를 반환한다. 

그런데 p1.equals(p3)는 `false`를 반환해 추이성을 위반한다. 

<br>

p1과 p2, p2와 p3 비교는 색상을 무시하고 비교해서 true가 나왔다. 

하지만 p1과 p3 비교는 색상 고려하고 비교해서 false가 나온다. 

<br>

그리고 이렇게 equals를 재정의 하면 `무한 재귀에 빠질 위험`도 있다. 

Point의 다른 하위 클래스 SmellPoint를 만들고 equals를 위와 같이 구현한 것이다. 

그 다음에 ColorPoint.equals(SmellPoint) 하면 StackOverflowError를 일으킨다. 

<br>

왜냐하면 ColoPoint에서 다른 클래스이니 그 클래스의 equals를 부른다. 

그러면 SmellPoint의 equals가 호출되는데 여기서도 다른 클래스이니 그 클래스의 equals를 부른다. 

이렇게 무한 재귀에 빠지는 것이다. 

<br>

### ❓ 추이성 만족….?

그렇다면 해법은 무엇일까?

사실 `이 현상은 모든 객체 지향 언어의 동치관계에서 나타나는 근본적인 문제`다. 

구체 클래스를 확장해 ***새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.***

객체 지향적 추상화의 이점을 포기하지 않는한 말이다. 

<br>

**3️⃣ equals 구현 3**

그렇다면 equals 안의 instanceOf 검사를 `getClass 검사로 바꾸면` 규약도 지키고 값도 추가하면서 구체 클래스를 상속할 수 있다는 뜻으로 들린다.

아래처럼 equals를 재정의 하는 것이다. 

```java
    // 잘못된 코드 - 리스코프 치환 원칙 위배! (59쪽)
    @Override public boolean equals(Object o) {
        if (o == null || o.getClass() != getClass())
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
```

<br>

이렇게 getClass를 사용하면 같은 구현 클래스의 객체와 비교할 때만 true를 반환한다. 

`괜찮을 것 같지만 실제 활용할 수 없다.`

<br>

왜냐하면 Point의 하위 클래스는 정의상 여전히 Point이므로 어디서든 Point로써 활용될 수 있어야 한다. 

그런데 이 방식에서는 그렇지 못하다. 

왜 그렇지 못한지 예시로 보자. 

<br>

### ⁉ 추이성 만족...? 예시

주어진 점이 반지름이 1인 단위 원 안에 있는지 판별하는 메서드가 필요하다. 

아래는 그 메서드의 코드이다. 

```java
    // 단위 원 안의 모든 점을 포함하도록 unitCircle을 초기화한다. (58쪽)
    private static final Set<Point> unitCircle = Set.of(
            new Point( 1,  0), new Point( 0,  1),
            new Point(-1,  0), new Point( 0, -1));

    public static boolean onUnitCircle(Point p) {
        return unitCircle.contains(p);
    }
```

<br>

이제 `값을 추가하지 않는 방식으로 Point를 확장한 CounterPoint 클래스`를 만들 것이다.  

만들어진 인스턴스의 개수를 생성자에서 세도록 구현하였다. 

<br>

**`하위 클래스`** 

```java
import java.util.concurrent.atomic.*;

// Point의 평범한 하위 클래스 - 값 컴포넌트를 추가하지 않았다. (59쪽)
public class CounterPoint extends Point {
    private static final AtomicInteger counter =
            new AtomicInteger();

    public CounterPoint(int x, int y) {
        super(x, y);
        counter.incrementAndGet();
    }
    public static int numberCreated() { return counter.get(); }
}
```

<br>

- **`리스코프 치환 원칙이란?`**
    
    리스코프 치환 원칙은 1988년 바바라 리스코프(Barbara Liskov)가 올바른 상속 관계의 특징을 정의하기 위해 발표한 것이다. 
    
    **서브 타입은 언제나 기반 타입으로 교체할 수 있어야 한다는 것**을 뜻한다.
    
    교체할 수 있다는 말은, 자식 클래스는 최소한 자신의 부모 클래스에서 가능한 행위는 수행이 보장되어야 한다는 의미이다.
    
    즉, 부모 클래스의 인스턴스를 사용하는 위치에 자식 클래스의 인스턴스를 **대신 사용했을 때 코드가 원래 의도대로 작동**해야 한다는 의미이다.
    
    이것을 부모 클래스와 자식 클래스 사이의 행위가 일관성이 있다고 말한다.
    
    쉽게 말하면 **다형성 원리**를 이야기 하는 것이다. 
    
<br>

리스코프 치환 원칙에 따르면 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다. 

따라서 `그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 동작`해야 한다. 

즉 앞에서 말한 Point의 하위 클래스는 정의상 여전히 Point이므로 어디서든 Point로써 활용될 수 있어야 한다의 의미다. 

앞에서 `getClass를 사용해 equals를 재정의 하면 위의 문장 즉 리스코프 치환 원칙을 만족하지 못한다는 것`이다. 

<br>

왜 리스코프 치환 원칙을 만족하지 못하는지 보자. 

위에서 정의한 하위 클래스 CounterPoint의 인스턴스를 onUnitCircle 메서드에 넘기면 어떻게 되는지 보자. 

Point 클래스의 equals를 getClass를 사용해 작성했다면 CounterPoint의 x, y와 무관하게 `항상 false를 반환`할 것이다. 

<br>

왜 그런가 하면 원인은 컬렉션 구현체에서 주어진 원소를 담고 있는지 확인하는 방법에 있다. 

onUnitCircle 메서드에서 Set을 사용하는데 Set을 포함한 대부분의 `컬렉션은 contains 메서드에서 equals 메서드를 사용`한다. 

그런데 Set에 Point 객체 넣어 놨으니 `Point의 equals를 사용`한다. 

그러면 Point의 equals에서 getClass를 사용하니 CounterPoint의 인스턴스는 어떤 Point 인스턴스와 같을 수 없으니 contains에서 false가 나온다. 

<br>

### ➰ 추이성 만족 with 우회

구체 클래스의 하위 클래스에서 값을 추가하고 equals 규약 만족 시킬 방법은 없다. 

하지만 `괜찮은 우회 방법`이 있다. 

<br>

- **`컴포지션이란?`**
    
    컴포지션은 parts 즉 클래스를 구성하는 부분의 합으로 정의한다
    
    **클래스 필드 내에 private or public 필드로 클래스의 인스턴스를 참조**하게 하고 해당 클래스를 구성하는 부분의 합으로 정의된다.
    
    클래스의 구성요소로 쓰인다는 뜻에서 composition이라고 한다.
    
<br>

`“상속 대신 컴포지션을 사용하라”`는 것이다. 

Point를 상속하는 대신 `Point를 ColorPoint의 private 필드로` 둔다. 

그리고 ColorPoint와 같은 위치의 일반 `Point를 반환하는 view 메서드를 public으로 추가`하는 것이다. 

<br>

아래 코드처럼 구현하는 것이다. 

Point를 가지고 Point의 equals와 색상을 비교하는 equals 같이 사용하는 것이다. 

```java
import effectivejava.chapter3.item10.Color;
import effectivejava.chapter3.item10.Point;

import java.util.Objects;

// 코드 10-5 equals 규약을 지키면서 값 추가하기 (60쪽)
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
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

<br>

### ➿ 추이성 만족 with 추상 클래스

`추상 클래스의 하위 클래스에서라면 equals 규약 지키면서도 값을 추가할 수 있다.` 

아무런 값을 갖지 않는 추상 클래스 Shape을 만든다. 

그리고 Shape을 확장해 radius 필드 추가한 Circle 클래스 만든다. 

그리고 Shape을 확장해 length, width 필드 추가한 Rectangle 클래스 만든다. 

`상위 클래스를 직접 인스턴스로 만드는 것이 불가능`하면 지금까지 이야기한 문제들은 일어나지 않는다. 

<br>

## 🔴 4. 일관성

`일관성 (consistency)` : null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환해야 한다.

<br>

일관성은 `두 객체가 같아면 어느 하나 혹은 두 객체 모두 수정되지 않는 한 앞으로도 영원히 같아야 한다는 뜻`이다. 

가변 객체는 비교 시점에 따라 서로 다를 수도 혹은 같을 수도 있다. 

하지만 불변 객체는 한번 다르면 끝까지 달라야 한다. 

불변 객체는 equals가 한번 같다고 한 객체는 영원히 같다고 답해야 한다. 

그리고 한번 다르다고 한 객체는 영원히 다르다고 답해야 한다. 

<br>

### ❎ 일관성 어긴 예시

java.net.URL의 equals는 주어진 URL과 매핑된 호스트의 IP 주소 이용해 비교한다. 

그런데 호스트의 이름을 IP 주소로 바꾸려면 `네트워크 통해야 하는데 그 결과가 항상 같다고 보장할 수 없다.` 

왜냐하면 equals 메서드는 실제로 해당 호스트에 대한 DNS 쿼리를 시도한다. 

그런데 네트워크를 통해서 가기 때문에 가는 와중에 호스트 이름과 매핑된 IP 주소가 달라질 수 있기 때문이다. 

이는 URL의 equals가 일반 규약을 어기게 하고 실무에서도 문제를 일으킨다. 

<br>

URL의 equals를 이렇게 구현한 것은 `커다란 실수`였다고 한다. 

이런 문제 피하려면 equals는 `항시 메모리에 존재하는 객체만을 사용`한 결정적 계산만 수행해야 한다. 

<br>

## 🔴 5. null 아님

`null 아님` : null이 아닌 모든 참조 값 x에 대하여 x.equals(null)은 false이다.

<br>

null 아님은 `모든 객체가 null과 같지 않아야 한다는 뜻`이다. 

객체.equals(null)해서 true를 반환하는 상황은 상상하기 어렵다. 

왜냐하면 객체가 null이라도 .equals 하면 NullPointException이 난다. 

그리고 null이 들어와도 null을 걸러내는 코드가 없다면 null에 접근하거나 사용하려는 코드가 있다면 또 NullPointException이 날 수 있다. 

<br>

따라서 수많은 클래스들이 아래 코드처럼 `입력이 null인지 확인해 자신을 보호`한다. 

```java
//명시적 null 검사 - 필요 없다!
@Overrid public boolean equals(Object o){

	if(o==null)
		return false;
		...
}
```

<br>

하지만 이렇게까지 `명시적인 null 검사는 필요 없다.` 

왜냐하면 동치성을 검사하려면 equals는 건네받은 객체를 적절히 형변환한 후 필수 필드들의 값을 알아내야 한다. 

그러려면 형변환에 앞서 instaceof 연산자로 입력 매개변수가 올바른 타입인지 검사해야 한다. 

<br>

아래 코드처럼 하는 것이다. 

`instanceof에서 null이면 걸러질 것`이다. 

```java
//묵시적 null 검사 - 이쪽이 낫다. 
@Overrid public boolean equals(Object o){

	if(!(o instanceof MyType))
		return false;

	MyType mt = (MyType) o;
		...
}
```

<br>

equals가 타입을 확인하지 않으면 잘못된 타입이 인수로 주어졌을 때 ClassCastException을 던져서 일반 규약을 위배하게 된다. 

그런데 `instanceof는 첫번째 피연산자가 null이면 false를 반환`한다. 

따라서 `null 검사를 명시적으로 하지 않아도 되는 것`이다. 

<br>

## ⚙ equals 메서드 구현 방법

지금까지 본 equals 메서드를 재정의 할 때는 반드시 따라야 하는 일반 규약에 따라 `양질의 equals 메서드를 구현하는 방법`을 단계별로 보자. 

<br>

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다. 자기 자신이면 true를 반환한다. 

2. instanceof 연산자로 입력이 올바른 타입인지 확인한다. 그렇지 않다면 false를 반환한다.

3. 입력을 올바른 타입으로 형변환 한다. 

4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.

<br>

각 단계별로 특징을 알아보자. 

<br>

### 1️⃣ 1번 단계

== 연산자를 사용해 입력이 자기 자신의 참조인지 확인하는 것은 단순한 `성능 최적화용`이다. 

비교 작업이 복잡한 상황의 경우 값어치를 할 것이다. 

<br>

### 2️⃣ 2번 단계

instanceof 연산자로 입력이 올바른 타입인지 확인한다는 이전 null 아님에서 봤듯이 `null을 걸러주는 역할`도 한다. 

<br>

여기서 올바른 타입은 보통 equals가 정의된 클래스이다. 

그런데 가끔은 `그 클래스가 구현한 특정 인터페이스`가 될 수도 있다. 

왜냐하면 어떤 인터페이스는 `자신을 구현한 서로 다른 클래스끼리도 비교할 수 있도록 equals 규약을 수정`하기도 한다. 

이런 인터페이스를 구현한 클래스라면 equals에서 `instanceof 할 때 인터페이스 타입 사용`해야 한다. 

<br>

위의 경우 아래 코드처럼 해야 한다는 것이다. 

```java
@Overrid public boolean equals(Object o){

	if(!(o instanceof 인터페이스 타입))
		return false;
		
		...
}
```

<br>

### 3️⃣ 3번 단계

입력을 올바른 타입으로 형변환 하는 것은 이전에도 봤듯이 `동치성 검사하려면 equals는 건네받은 객체를 적절히 형변환한 후 필수 필드들의 값을 알아내야 하기 때문`이다. 

<br>

형변환 하는 것은 2번 단계에서 instanceof 검사를 했기 때문에 `100% 성공한다.`

<br>

### 4️⃣ 4번 단계

입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다는 `논리적 동치성을 위해서 해야 하는 것`이다. 

`모든 필드가 일치하면 true를, 하나라도 다르면 false를 반환`하는 것이다. 

<br>

2단계에서 instanceof를 인터페이스 타입으로 했다면 입력의 필드 값 가져올 때도 그 `인터페이스의 메서드를 사용`해야 한다. 

<br>

## 🔠 필드 비교 시 타입에 따른 비교 방법

equals에서 핵심 필드들을 비교할 때 `여러 타입에 따라 비교 방법이 다르기도 하다`. 

<br>

아래의 타입들에 대해 어떻게 비교하는지 보자. 

1. 참조 타입 필드
2. float, double 제외 기본 타입 필드
3. float, double 타입 필드
4. 배열 필드
5. null도 정상 값 취급하는 참조 필드
6. 비교하기 아주 복잡한 필드

<br>

### 🔵 1. 참조 타입 필드

참조 타입 필드는 지금까지 봤듯이 각각의 `equals 메서드로 비교`하면 된다. 

<br>

### 🔵 2. float, double 제외 기본 타입 필드

float, double 제외 기본 타입 필드는 `== 연산자로 비교`하면 된다. 

<br>

### 🔵 3. float, double 타입 필드

float, double 필드는 `각각 정적 메서드를 사용`해야 한다. 

- float 타입: Float.compare(float, float)
- double 타입: Double.compare(double, double)

<br>

왜 float, double 타입을 특별 취급하는가 하면 `Float.NaN, -0.0f, 특수 부동 소수 값 등을 다뤄야 하기 때문`이다. 

Float.equals, Double.equals 메서드 대신 사용할 수 있지만 이 메서드들은 오토박싱을 수반할 수 있어 `성능상 좋지 않다`. 

<br>

### 🔵 4. 배열 필드

배열 필드는 배열의 `원소 각각을 위의 1~3번 방법으로 비교`한다. 

배열의 `모든 원소가 핵심 필드라면` Array.equals 메서드들 중 하나를 사용하자. 

<br>

### 🔵 5. null도 정상 값 취급하는 참조 타입 필드

이런 필드는 정적 메서드인 `Objects.equals(Object, Object)`로 비교해 NullPointException 발생을 예방하자. 

모든 클래스의 상위 클래스인 `Object가 아니라 Objects.equals이다`. 

<br>

Objects.equals(Object, Object)는 아래와 같은 특징이 있다. 

- 두 개체가 모두 null 을 가리키면 equals()는 true를 반환한다.
- 개체 중 하나가 null 을 가리키면 equals()는 false를 반환한다.
- 두 개체가 모두 같으면 equals()는 true를 반환한다.
- 그렇지 않으면, 등식 방법을 사용하여 등식을 결정한다.

<br>

코드는 아래와 같다. 

![Untitled](https://file.notion.so/f/f/7d1044b7-24d0-4b32-b24a-1b9061f7bf65/03313095-4049-41e4-a04b-836e0a7a2569/Untitled.png?id=73745d8d-0bae-4612-9288-8f6cb1646d87&table=block&spaceId=7d1044b7-24d0-4b32-b24a-1b9061f7bf65&expirationTimestamp=1710000000000&signature=yW2MDKaqc3qv5eE3ogoAfCi8zPq6pKpViqeWPWZrzvU&downloadName=Untitled.png)

<br>

### 🔵 6. 비교하기 복잡한 필드

비교하기 복잡한 필드는 그 필드의 표준형(canonical form)을 저장해둔 후 `표준형끼리 비교`하면 훨씬 경제적이다. 

필드를 좀 더 쉬운 것으로 만든 후 비교하는 것이다. 

이 기법은 `불변 클래스에서 제격`이다. 

`가변 객체라면 값이 바뀔 때마다 표준형을 최신 상태로 갱신`해줘야 한다. 

<br>

## 🚘 equals 메서드 성능

어떤 `필드를 먼저 비교하느냐`가 equals의 성능을 좌우하기도 한다. 

최상의 성능을 바란다면 아래의 방법을 참고하자. 

- 다를 가능성이 더 크거나 비교하는 비용이 싼 필드 먼저 비교
- 동기화용 락 필드 같이 객체의 논리적 상태와 관련없는 필드 비교 X
- 파생 필드가 객체 전체 상태 대표 시 파생 필드 비교

<br>

마지막 `파생 필드`가 무엇인지 보자. 

파생 필드는 `핵심 필드로부터 계산해낼 수 있는 것`이다. 

파생 필드는 굳이 비교할 필요는 없지만 파생 필드를 비교하는 쪽이 더 빠를 때도 있다고 한다. 

`파생 필드가 객체 전체의 상태를 대표하는 상황`에 그렇다.

<br>

## 🔎 equals 메서드 구현 후

equals를 다 구현했다면 `할 일`들이 있다. 

1. 대칭적인가 자문
2. 추이성이 있는가 자문 
3. 일관적인가 자문
4. 단위 테스트 작성해 확인

<br>

1~3번 중 하나라도 실패한다면 원인을 찾아서 고치자.

나머지 요건인 반사성과 null 아님도 만족해야 하지만 `이 둘이 문제되는 경우는 별로 없다`. 

<br>

## ⚠ equals 메서드 구현 주의 사항

equals 메서드 구현할 때 `주의 사항`들을 보자. 

1. equals를 재정의할 땐 hashCode도 반드시 재정의하자. 

2. 너무 복잡하게 해결하려 들지 말자.

3. Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자. 

<br>

각 주의 사항들에 대해서 보자. 

<br>

### 🟢 1. equals를 재정의할 땐 hashCode도 반드시 재정의하자.

이것은 `다음 item의 주제`로 다음 item에서 확인하자.

<br>

### 🟢 2. 너무 복잡하게 해결하려 들지 말자.

필드들의 동치성만 검사해도 equals 규약을 어렵지 않게 지킬 수 있다. 

오히려 `너무 공격적으로 파고들다가 문제` 일으키기도 한다. 

<br>

예를 들어 File 클래스라면 심볼릭 링크를 비교해 같은 파일을 가리키는지 확인해 비교하는 `어렵고 쓸데없는 짓을 하면 안된다는 것`이다. 

<br>

### 🟢 3. Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.

equals 메서드의 `매개변수는 반드시 Object 타입`이여야 한다. 

타입이 Object가 아니면 Object.equals를 재정의 하는 것이 아니라 `다중정의 하는 것`이다. 

<br>

이처럼 `타입을 구체적으로 명시한 equals는 오히려 해가 된다`. 

이 메서드는 하위 클래스에서의 @Override annotation이 긍정 오류를 내게 한다. 

잘못된 equals를 오버라이딩 할 수 있다는 의미 같다. 

그리고 보안 측면에서도 잘못된 정보를 준다. 

<br>

@Override annotation을 일관되게 사용하면 이런 실수를 예방할 수 있다. 

처음에 `상위 클래스에서 equals를 작성할 때 아래처럼 @Override 하면` 컴파일 오류가 발생할 것이다. 

```java
@Overrid public boolean equals(MyClass o){
	...
}
```

<br>

## 🦾 equals 메서드 대신 만들기

equals를 작성하고 테스트하는 일은 지루하다. 

그리고 이것을 테스트하는 코드도 항상 뻔하다. 

<br>

다행히 이 `작업을 대신해줄 오픈소스`가 있다. 

바로 `AutoValue 프레임워크`다. 

<br>

클래스에 `@AutoValue` 붙이면 AutoValue가 equals를 알아서 작성해준다. 

그리고 우리가 직접 작성하는 것과 근본적으로 똑같은 코드를 만들어줄 것이다. 

<br>

`대다수의 IDE도 같은 기능 제공하지만` 생성된 코드가 AutoValue만큼 깔끔하거나 읽기 좋지는 않다. 

그리고 IDE는 나중에 `클래스 수정된 것을 자동으로 알아채지 못해 테스트 코드 작성해 놔야 한다.` 

그래도 사람이 직접 작성하는 것보다 `IDE에 맡기는 편이 낫다.` 

IDE는 사람처럼 부주의한 실수 저지르지 않기 때문이다. 

<br>

## ❗ 결론

꼭 필요한 경우가 아니면 equals를 재정의하지 말자. 

많은 경우 Object의 equals가 우리가 원하는 비교를 정확히 수행한다. 

재정의 해야 할 때는 클래스의 핵심 필드 모두를 빠짐없이 다섯가지 규약 지켜가며 비교해야 한다. 

<br>

## 📍 references

- [equals란?](https://velog.io/@kekim20/JAVA-equals%EB%A9%94%EC%84%9C%EB%93%9C-Object-String-Integer%ED%81%B4%EB%9E%98%EC%8A%A4%EC%99%80-%EC%98%A4%EB%B2%84%EB%9D%BC%EC%9D%B4%EB%94%A9)
- [관계](http://www.ktword.co.kr/test/view/view.php?no=444)
- [동치관계, 동치류1](http://www.ktword.co.kr/test/view/view.php?no=4958)
- [동치관계, 동치류2](https://gosamy.tistory.com/373)
- [동치관계, 동치류3](https://bite-sized-learning.tistory.com/395)
- [동치류](https://m.blog.naver.com/kyj0833/221018308669)
- [URL의 equals](https://blog.naver.com/PostView.nhn?blogId=horajjan&logNo=220606665532)
- [Objects.equals](https://velog.io/@jipark09/Java-Object.equals)
- [리스코프 치환 원칙](https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EC%95%84%EC%A3%BC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-LSP-%EB%A6%AC%EC%8A%A4%EC%BD%94%ED%94%84-%EC%B9%98%ED%99%98-%EC%9B%90%EC%B9%99)
- [컴포지션](https://velog.io/@vino661/%EC%83%81%EC%86%8D%EA%B3%BC-%EC%BB%B4%ED%8F%AC%EC%A7%80%EC%85%98%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C)