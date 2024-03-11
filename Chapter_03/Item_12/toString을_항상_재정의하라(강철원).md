# 🚀 item 12. 항상 toString을 재정의하라 Always override toString

왜 toString을 재정의하라고 할까요?   


Object에서 정의한 toString()을 재정의하고 사용하지 않으면 클래스 이름@16진수로 표시한 해시코드 포맷으로 반환합니다.   
사실 16진수를 표시하는 것보다는 실제 데이터를 표시해주는게 훨씬 개발자입장에서는 이해하기 쉽습니다.    
또한 `toString`의 규약에 따르면 ‘간결하면서 사람이 읽기 쉬운 형태의 유익한 정보’를 반환해야 한다.    
`equals`나 `hashCode`처럼 시스템에 오동작을 일으키지는 않지만, item 12 규칙이 주는 개발상 편의는 막대하기 때문에 중요하다. 해당 객체를 사용하기에도, 디버깅하기에도 즐겁습니다.    
이제 예제를 통해 알아봅시다.

```java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix   = rangeCheck(prefix,   999, "프리픽스");
        this.lineNum  = rangeCheck(lineNum, 9999, "가입자 번호");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }
    
    //...
```
```java
public static void main(String[] args) {
        PhoneNumber friend = new PhoneNumber(010, 234, 5561);
        System.out.println("친구 전화번호: " + friend);
}
```
위 코드를 실행하면 아래와 같이 출력합니다.
`친구 전화번호: me.ex.chapter02.item12.PhoneNumber@adbbd`
이런식으로 16진수로 표시한 해시코드를 반환하는 이유는 기본적인 `Object`가 제공하는 `toString`을 사용하면 출력을 하거나 문자열 연결할 때 인스턴스에 문자열을 연산하면 인스턴스에 있는 `toString`이 자동으로 호출이 되는데 그 경유에 이런식으로 출력이 됩니다.

이때 `toString`을 오버라이딩 해서 우리가 원하는 데이터 형태를 구현해주면 가독성이 훨씬 올라갑니다.


>아래처럼 오버라이딩해서 원하는 데이터 형태로 구현해줍니다.
```java
 @Override public String toString() {
        return String.format("%03d-%03d-%04d",
                areaCode, prefix, lineNum);
    }
```
```java
public static void main(String[] args) {
        PhoneNumber friend = new PhoneNumber(010, 234, 5561);
        System.out.println("친구 전화번호: " + friend);
}
```
`친구 전화번호: 010-234-6651`
이렇게 해주면 정상적으로 번호가 출력이 됩니다.

## 😎 구현할 때 고려하면 좋을 점들

### 1️⃣ 객체가 가진 정보 모두를 반혼하는 것이 좋습니다.

❗️ but 객체가 거대하거나 객체의 상태가 문자열로 표현하기에 적합하지 않다면 요약정보를 담는 것도 좋습니다.
->맨해든 거주자 전화번호부(총 1487574개)
-> "Thread[main,5,main]"

### 2️⃣ 포맷을 문서화 할지 정하기

전화번호나 행렬같은 VO는 문서화(주석 달기) 하기를 권합니다.    
포맷을 명시하면 그 객체는 표준적이고 명확하며 사람이 읽을 수 있게 된다. 그대로 입출력에 사용하거나 CSV파일 같은 곳에 저장할 수 도 있습니다.

### 3️⃣ 정적팩터리나 생성자를 함께 제공해주기

포맷을 명시하기로 했다면, 그 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩토리나 생성자를 함께 제공해 주면 좋습니다.

>위 코드를 예시로들면 아래와 같습니다.
```java
public static PhoneNumber of(String phoneNumberString) {
        String[] split = phoneNumberString.split("-");
        PhoneNumber phoneNumber = new PhoneNumber(
                Short.parseShort(split[0]),
                Short.parseShort(split[1]),
                Short.parseShort(split[2]));
        return phoneNumber;
    }
```
```java
 public static void main(String[] args) {
        PhoneNumber phoneNumber = PhoneNumber.of("010-811-5323");
        System.out.println(phoneNumber);
    }
```
이처럼 정적 팩터리 메서드를 만들어 놓으면 훨씬 편리하게 이용할 수 있습니다.

### 4️⃣ 포맷을 명시하든 아니든 의도를 명확히 밝히기

포맷을 명시하려면 아주 정확하게 해야합니다. 

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

>포맷을 명시하지 않기로 했다면 다음처럼 작성할 수 있습니다.
```java
/**
* 이 약물에 관한 대략적인 설명을 반환한다.
* 다음은 이 설명의 일반적인 형태이나,
* 상세 형식은 정해지지 않았으며 향후 변경될 수 있습니다.
*
* "[약물 #9: 유형=사랑, 냄새=테라빈유, 겉모습=먹물]"
*/
@Override public String toString() { ... }
```
이러한 설명을 읽고도 포맷에 맞춰 코딩하거나 특정 값을 빼내어 영구 저장한 프로그래머는 나중에 포맷이 바뀌어 피해를 입어도 자기 자신을 탓할 수밖에 없습니다.


### 5️⃣ toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하기

API를 제공하지 않으면 이 객체를 사용하는 프로그래머가 불필요한 `parsing` 작업을 해야 합니다.    
접근자를 제공하지 않으면, `toString`의 포맷이 사실상 준-표준API처럼 동작하게 됩니다.
예를 들어, `PhoneNumber` 클래스는 지역 코드, 프리픽스, 가입자 번호용 접근자를 제공해야 합니다. 그렇지 않으면 이 정보가 필요한 프로그래머는 `toString` 의 반환값을 파싱할 수 밖에 없습니다.   
접근자를 제공하지 않으면(변경될 수 있다고 문서화했더라도) 그 포맷이 사실상 준 표준 API나 다름없어집니다.

## 📃 정리

모든 구체 클래스에서 `Object`의 `toString`을 재정의합시다. (상위 클래스에서 이미 알맞게 재정의한 경우는 예외)    
`toString`을 재정의한 클래스는 사용하기도 즐겁고 그 클래스를 사용한 시스템을 디버깅하기 쉽게 해줍니다. `toString`은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야합니다.


### 📚 Reference

- [Class Object](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html)
