# 3️⃣5️⃣ Item 35 : ordinal 메서드 대신 인스턴스 필드를 사용하라

<br>

## 📌 목차
1. ordinal 메서드
2. ordinal 메서드 문제점
3. 해결책

<br>

## ❓ ordinal 메서드

대부분의 열거 타입 상수는 자연스럽게 `하나의 정숫값에 대응`된다. 

열거 타입은 각 상수에 정수 값이 자동으로 할당된다. 

그리고 그 `정수 값은 0부터 시작`한다. 

하지만 명시적으로 값을 정의하지 않았을 때에만 적용된다. 

<br>

그리고 모든 열거 타입은 해당 상수가 그 `열거 타입에서 몇 번째 위치인지를 반환하는 ordinal이라는 메서드`를 제공한다. 

ordinal 메서드는 열거 타입 상수의 순서를 반환하는데 열거 타입 상수는 선언된 순서대로 0부터 시작되는 index가 할당된다. 

위에서 말한 `정수 값이 index`인 것이다. 

`ordinal 메서드`는 이 열거 타입 상수의 `index를 반환`하는 것이다. 

<br>

열거 타입 상수와 연결된 정수 값이 필요하면 ordinal 메서드를 이용하고 싶은 유혹에 빠진다. 

하지만 item 제목에서부터 알 수 있듯이 `ordinal 메서드를 사용하면 안된다.` 

왜 사용하면 안되는지 알아보자. 

<br>

## ❌ ordinal 메서드 문제점

예시를 통해 ordinal 메서드의 문제점을 알아보자. 

아래 코드는 합주단의 종류를 연주자가 1명인 solo부터 연주자가 10명인 detect까지 정의한 열거 타입이다. 

```java
public enum Ensemble{
		SOLO, DUET, TRIO, QUARTET, QUINTET,
		SEXTET, SEPTET, OCTET, NONET, DECTET;
		
		public int numberOfMusicians(){ return ordinal()+1; } 
}
```

<br>

ordinal 메서드의 문제점은 아래와 같다. 

1. `상수 선언 순서를 바꾸는 순간` ordinal 메서드 사용한 곳이 오동작해 유지보수하기 어렵다. 

2. 이미 사용 중인 정수와 `값이 같은 상수는 추가할 방법이 없다`. 

3. `값을 중간에 비워둘 수 없다`. 

<br>

각 문제점에 대해 예시로 알아보자. 

<br>

### 1️⃣ 문제점 1

첫번째 문제점은 상수 선언 순서가 바뀌는 순간 `ordinal이 반환하는 값이 달라진다는 것`이다. 

DUET과 TRIO의 순서를 바꾸면 DUET은 1을 반환하다가 2를 반환하게 된다. 

그러면 이 ordinal 메서드를 사용하는 numberOfMusicians 메서드가 생각한 것과 다르게 오동작 하는 것이다. 

<br>

### 2️⃣ 문제점 2

두번째 문제점은 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없다는 것이다. 

`연주자가 8명인 합주단의 종류는 2가지`가 있다.

- 8중주

- 복4중주

<br>

이 복4중주를 의미하는 DOUBLE_QUARTET도 열거 타입에 넣고 싶어서 아래와 같이 추가했다. 

```java
public enum Ensemble{
		SOLO, DUET, TRIO, QUARTET, QUINTET,
		SEXTET, SEPTET, OCTET, DOUBLE_QUARTET, NONET, DECTET;
		
		public int numberOfMusicians(){ return ordinal()+1; } 
}
```

<br>

그런데 하나의 정수 값에 대응한다고 했다. 

그래서 ordinal 메서드의 값이 `OCTET이 7이면 DOUBLE_QUARTET은 8`이 된다. 

그래서 numberOfMusicains 메서드를 호출하면 DOUBLE_QUARTET은 값이 9가 나온다. 

<br>

그렇기 때문에 ordinal 메서드를 사용하면` 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없다는 것`이다. 

<br>

### 3️⃣ 문제점 3

세번째 문제점은 `값을 중간에 비워둘 수 없다는 것`이다. 

`연주자가 12명`인 3중 4중주 = TRIPLE_QUARTET를 `열거 타입에 추가하고 싶다`. 

이 열거 타입 상수가 ordinal 메서드 호출했을 때 11 반환하려면 10을 반환하는 열거 타입 상수가 있어야 한다. 

<br>

그래서 `중간에 연주자가 11명인 열거 타입 상수도 채워야 한다`. 

`그런데 연주자가 11명인 합주단의 이름이 없다`. 

그러면 쓰이지 않는 `dummy 상수를 추가`해야 TRIPLE_QUARTET 상수를 추가할 수 있다. 

```java
public enum Ensemble{
		SOLO, DUET, TRIO, QUARTET, QUINTET,
		SEXTET, SEPTET, OCTET, NONET, DECTET,
		DUMMY, TRIPLE_QUARTET;
		
		public int numberOfMusicians(){ return ordinal()+1; } 
}
```

<br>

이렇게 되면 다음과 같은 문제가 생긴다. 

- 코드가 깔끔하지 못하다.

- DUMMY 값이 많아질수록 실용성이 떨어진다.

따라서 ordinal 메서드를 사용하면 값을 중간에 비워둘 수 없는 것이다. 

<br>

## ⭕ 해결책

그렇다면 이런 ordinal 메서드의 문제를 어떻게 해결할 것인가?

열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고 `인스턴스 필드에 저장`하면 된다. 

아래 코드처럼 작성하면 된다. 

```java
// 인스턴스 필드에 정수 데이터를 저장하는 열거 타입 (222쪽)
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

<br>

그렇다면 위의 문제점을 아래와 같이 해결해준다. 

- 열거 타입 상수의 순서가 바뀌어도 `필드에 저장된 값이 달라지지 않아` 문제가 생기지 않는다.
- 값이 같은 상수여도 `필드에 각각 같은 값을 저장`하면 된다.
- `중간에 빈 값이 있어도 문제가 없다`.

<br>

Enum의 API 문서를 보면 ordinal에 대해 다음과 같이 쓰여 있다. 

“대부분 프로그래머는 이 메서드를 쓸 일이 없다. 이 메서드는 EnumSet과 EnumMap 같이 `열거 타입 기반의 범용 자료구조에 쓸 목적`으로 설계 되었다.”

따라서 `이런 용도가 아니라면 ordinal 메서드는 절대 사용하지 말자는 것`이다.