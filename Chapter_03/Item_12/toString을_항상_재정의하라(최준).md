# 2️⃣ Item 12: toString을 항상 재정의하라

<br>

## 📌 목차
1. toString이란
2. toString 잘 구현 시 장점
3. 좋은 toString 구현방법
4. 1: 객체가 가진 주요 정보 모두 반환하는 것이 좋다. 
5. 2: 반환값의 포맷을 문서화할지 정해야 한다. 
6. 3: 포맷을 명시하든 아니든 의도를 명확히 밝혀야 한다. 
7. 4: toString 반환 값에 포함된 정보 얻을 수 있는 API 제공해야 한다.
8. toString 재정의 yes or no
   1. toString 재정의 no
   2. toString 재정의 yes
9. tostring 자동 생성
10. 결론

<br>

## ❓ toString이란

toString이란 자바의 모든 클래스의 가장 최상위 클래스인 `Object 클래스가 가진 메서드 중 하나`다. 

따라서 `모든 클래스들은 toString 메서드를 사용`할 수 있다.

toString 메서드는 ***객체가 가지고 있는 정보나 값들을 문자열로 만들어 리턴하는 메서드*** 다. 

<br>

Object의 기본 toString 메서드가 `내가 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없다`.

왜냐하면 이 기본 toString 메서드는 단순히 `‘클래스_이름@16진수로_표시한_해시코드’`를 반환하기 때문이다.

![Untitled](https://file.notion.so/f/f/7d1044b7-24d0-4b32-b24a-1b9061f7bf65/9fb82bc6-e4e8-4cee-8aa2-64de4017137e/Untitled.png?id=4556ffff-91c7-46ce-83a6-f55fc5477947&table=block&spaceId=7d1044b7-24d0-4b32-b24a-1b9061f7bf65&expirationTimestamp=1710007200000&signature=T9QVVudNWNwBP19n16W5F_tUljANfs_c_dSZ-QfIP_o&downloadName=Untitled.png)

<br>

toString의 일반 규약에 따르면 `‘간결하면서 사람이 읽기 쉬운 형태의 유익한 정보’`를 반환해야 한다. 

그리고 toString 규약은 `‘모든 하위 클래스에서 이 메서드를 재정의하라’`고 한다. 

<br>

## 👍 toString 잘 구현 시 장점

toString을 잘 구현한 클래스는 `장점`이 있다. 

1. 사용하기 훨씬 즐겁다.
2. 클래스 사용할 때 디버깅이 쉽다. 

<br>

아래의 사용 예시를 보며 toString의 장점을 보자. 

<br>

toString 메서드가 `사용되는 경우`는 아래와 같다. 

1. 객체를 println, printf, 문자열 연결 연산자(+), assert 구문에 넘길 때 
2. 디버거가 객체를 출력할 때 

<br>

즉 toString은 내가 `직접 호출하지 않아도 다른 어딘가에서 사용`이 된다. 

예를 들어 내가 작성한 객체를 참조하는 컴포넌트가 오류 메시지를 로깅할 때 자동으로 호출 될 수 있다. 

이때 toString을 `제대로 재정의 하지 않는다면 쓸모없는 메시지만 로그에 남아 디버깅하기 어려울 것`이다. 

toString을 재정의 하면 아래와 같은 코드만으로 문제를 진단하기에 충분한 메시지 남길 수 있다. 

```java
System.out.println(객체 + "오류 메시지");
```

<br>

toString을 재정의했든 아니든 프로그래머 대부분은 진단 메시지를 위처럼 출력할 것이다. 

그런데 toString을 `재정의 하지 않았으면 쓸모 없는 메시지가 출력`되는 것이다. 

<br>

## 📚 좋은 toString 구현 방법

좋은 toString은 이 인스턴스를 포함하는 객체에서 유용하게 쓰인다. 

<br>

좋은 toString을 구현하는 방법에는 `4가지`가 있다. 

1. 객체가 가진 주요 정보 모두를 반환하는 것이 좋다. 
2. 반환값의 포맷을 문서화 할지 정해야 한다. 
3. 포맷을 명시하든 아니든 의도를 명확하게 밝혀야 한다. 
4. toString 반환 값에 포함된 정보를 얻어올 수 있는 API 제공해야 한다. 

<br>

각 방법에 대해서 살펴보자. 

<br>

## 1️⃣ 1. 객체가 가진 주요 정보 모두 반환하는 것이 좋다.

실전에서 toString을 구현할 때는 `그 객체가 가진 주요 정보를 모두 반환하는 것이 좋다`.

{Jenny=PhoneNumber@ad08bb38d} 보다 {Jenny=707-867-5309} 이렇게 하는 것이 좋다는 것이다. 

<br>

하지만 객체가 거대하거나 객체의 상태가 문자열로 표현하기에 적합하지 않으면 `주요 정보를 모두 반환하기 어려울 수 있다`. 

이럴 때에는 “맨해튼 거주자 전화번호부 (총 1487536개) 이렇게 `요약 정보를 담으면 된다`. 

이상적으로는 객체 스스로를 완벽히 설명하는 문자열이여야 한다. 

하지만 `이상적으로 하기 어렵다면 요약 정보를 담으라는 것`이다. 

<br>

## 2️⃣ 2. 반환값의 포맷을 문서화할지 정해야 한다.

toString 구현할 때는 `반환값의 포맷을 문서화 할지 정해야 한다`. 

전화번호나 행렬 같은 `값 클래스라면 문서화 하기를 권한다`.

<br>

포맷을 명시하면 그 객체는 다음과 같은 `이점`을 얻는다. 

1. 표준적
2. 명확해짐
3. 사람이 읽을 수 있게 됨

<br>

따라서 객체를 출력한 값 그대로 입출력에 사용하거나 CSV 파일처럼 사람이 읽을 수 있는 데이터 객체로 저장할 수 있다. 

<br>

포맷을 명시하기로 했다면 `명시한 포맷에 맞는 문자열과 상호 전환할 수 있는 정적 팩터리나 생성자를 함께 제공`해주면 좋다. 

자바의 많은 값 클래스가 이 방식을 따르고 BigInteger, BigDeciaml 등이 예시다. 

아래의 BigInteger 생성자를 보면 String을 받고 주석의 설명을 보면 String을 BigInteger로 바꿔준다고 한다. 

```java
    /**
     * Translates the decimal String representation of a BigInteger
     * into a BigInteger.  The String representation consists of an
     * optional minus or plus sign followed by a sequence of one or
     * more decimal digits.  The character-to-digit mapping is
     * provided by {@link Character#digit(char, int)
     * Character.digit}.  The String may not contain any extraneous
     * characters (whitespace, for example).
     *
     * @param val decimal String representation of BigInteger.
     * @throws NumberFormatException {@code val} is not a valid representation
     *         of a BigInteger.
     */
    public BigInteger(String val) {
        this(val, 10);
    }
```

<br>

하지만 포맷을 명시하면 그 객체는 다음과 같은 `단점`을 얻는다. 

1. 포맷에 얽매이게 된다. 

<br>

포맷 명시하고 그 클래스가 많이 쓰인다면 `평생 그 포맷에 얽매이게 된다`.

왜냐하면 프로그래머들이 `명시된 포맷에 맞춰 코드를 작성`할 것이다. 

포맷을 바꾸면 이 포맷을 사용하던 코드와 데이터들은 엉망이 될 것이다. 

<br>

따라서 포맷을 명시하지 않으면 그 객체는 다음과 같은 `장점`을 얻는다. 

1. 향후에 정보 더 넣거나 포맷을 개선할 수 있는 `유연성` 얻게 된다. 

<br>

## 3️⃣ 3. 포맷을 명시하든 아니든 의도를 명확히 밝혀야 한다.

포맷을 명시하든 아니든 `의도를 명확히 밝혀야 한다`. 

`포맷을 명시하려면` 아래처럼 아주 정확하게 해야 한다. 

```java
    /**
     * 이 전화번호의 문자열 표현을 반환한다.
     * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
     * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
     * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
     *
     * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
     * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
     * 전화번호의 마지막 네 문자는 "0123"이 된다.
     */
      @Override public String toString() {
          return String.format("%03d-%03d-%04d",
                  areaCode, prefix, lineNum);
      }
```

<br>

`포맷을 명시하지 않기로 했다면` 아래처럼 설명을 작성할 수 있을 것이다. 

```java
/**
 * 이 약물에 관한 대략적인 설명을 반환한다. 
 * 다음은 이 설명의 일반적인 형태이나,
 * 상세 형식은 정해지지 않았으며 향후 변경될 수 있다. 
 *
 * "[약물 #9: 유형-사랑, 냄새=테레빈유, 겉모습=먹물]"
 */
 @Overrid public String toString(){
	 ...
	}
```

<br>

이렇게 toString 위에 작성한 설명을 읽고도 자기 스스로 포맷에 맞춰 코딩하거나 특정 값을 빼내어 영구 저장한 프로그래머는 나중에 포맷이 바뀌어도 자기 자신 탓할 수 밖에 없다. 

<br>

## 4️⃣ 4. toString 반환 값에 포함된 정보 얻을 수 있는 API 제공해야 한다.

포맷 명시 여부와 상관 없이 `toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공`하자. 

<br>

예를 들어 전화번호를 의미하는 `PhoneNumber 클래스`는 지역 코드, 프리픽스, 가입자 번호를 필드로 가질 것이다. 

<br>

전화번호: 010-1234-5678

지역코드: 010

프리픽스: 1234

가입자 번호: 5678

<br>

그리고 이 `정보들을 조합해 전화번호로 만든 후 toString으로 반환`할 것이다. 

그렇다면 PhoneNumber 클래스는 지역 코드, 프리픽스, 가입자 번호용 `접근자 (getter)를 제공`해야 한다는 것이다. 

<br>

그렇지 않다면 다음과 같은 `문제가` 생긴다. 

1. toString의 반환 값을 파싱해야 한다. 
2. 포맷 바뀌면 시스템 망가진다. 

<br>

이런 정보가 필요한 프로그래머는 파싱 작업을 수행해야 한다. 

이렇게 되면 `성능이 나빠지고 접근자 제공하면 필요하지 않은 작업`이다. 

<br>

그리고 향후 `포맷이 바뀌면 파싱하는 로직도 바뀌어야 한다`. 

따라서 로직이 바뀌지 않았다면 시스템이 망가지게 된다. 

<br>

## ⭕❌ toString 재정의 yes or no

toString을 `재정의 꼭 해야 하는 것`이 있고 `재정의 하지 않아도 되는 것`이 있다. 

이것들에 대해서 보자.

<br>

### ❌ toString 재정의 no

`정적 유틸리티 클래스`는 toString을 제공할 이유가 없으니 재정의 하지 않아도 된다. 

또한 대부분의 `열거 타입 클래스`도 자바가 이미 완벽한 toString을 제공하니 따로 재정의 하지 않아도 된다. 

<br>

### ⭕ toString 재정의 yes

`하위 클래스들이 공유해야 할 문자열 표현이 있는 추상 클래스`라면 toString을 재정의 해야 한다. 

자바에서 대다수의 컬렉션 구현체는 추상 컬렉션 클래스들의 toString 메서드를 상속해 사용한다. 

<br>

아래 코드는 ArrayList가 상속하고 있는 AbstractCollection의 toString이다. 

```java
    public String toString() {
        Iterator<E> it = iterator();
        if (! it.hasNext())
            return "[]";

        StringBuilder sb = new StringBuilder();
        sb.append('[');
        for (;;) {
            E e = it.next();
            sb.append(e == this ? "(this Collection)" : e);
            if (! it.hasNext())
                return sb.append(']').toString();
            sb.append(',').append(' ');
        }
    }
```

<br>

따라서 ArrayList를 출력해보면 아래와 같이 나오는 것이다. 

```java
        ArrayList<Integer> arrayList = new ArrayList();
        arrayList.add(1);
        arrayList.add(2);
        arrayList.add(3);

        System.out.println(arrayList);

```

![Untitled](https://file.notion.so/f/f/7d1044b7-24d0-4b32-b24a-1b9061f7bf65/87434a11-67be-4c91-9a26-45e657ed372d/Untitled.png?id=ca0525bd-72f5-4423-9050-1ae08741fdc0&table=block&spaceId=7d1044b7-24d0-4b32-b24a-1b9061f7bf65&expirationTimestamp=1710007200000&signature=Xu8OODG0yjLLLUSnJZRd_jNE4elUvADDvej5H9hGne4&downloadName=Untitled.png)

<br>

## 🦾 tostring 자동 생성

구글의 `AutoValue 프레임워크는 toString도 생성`해준다. 

또한 `대부분의 IDE도 toString 생성`해준다.

<br>

아래 코드는 String name과 int phoneNumber 필드로 가지는 클래스 인텔리제이에서 자동 생성한 toString이다. 
```java
    @Override
    public String toString() {
        return "MyClass{" +
                "name='" + name + '\'' +
                ", phoneNumber=" + phoneNumber +
                '}';
    }
```

<br>

AutoValue는 각 필드의 내용을 멋지게 나타내 준다. 

하지만 `클래스의 의미까지 파악하지는 못한다`. 

<br>

예를 들어 전화번호를 뜻하는 PhoneNumber 클래스는 `자동 생성하면 우리가 원하는 것이 나오지 않을 것`이다. 

우리는 ‘010-1234-5678’ 이런 문자열을 원하는데 ‘areaCode=010, prefix=1234, lineNum=5678’ 이런 문자열이 돌아올 것이다. 

전화번호는 `표준 체계를 따라야 하기 때문에 자동 생성 toString이 적합하지 않다`. 

<br>

비록 자동 생성 toString에 적합하지 않더라도 `객체의 값에 관해 아무것도 알려주지 않는 Object의 toString보다는 자동 생성된 toString이 훨씬 유용`하다. 

따라서 자동 생성된 것이라도 재정의 하자는 것이다. 

<br>

## ❗ 결론
모든 구체 클래스에서 Object의 toString을 재정의하자. 

상위 클래스에서 이미 알맞게 재정의한 경우는 예외다. 

toString을 재정의한 클래스는 사용하기도 즐겁다. 

그리고 그 클래스를 사용한 시스템을 디버깅하기 쉽게 해준다. 

toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다. 

<br>

## 📍 references

- [toString이란](https://sungmin1.tistory.com/124)