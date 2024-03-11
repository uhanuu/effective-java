## 🤔 Object의 toString() 이란?
<p align="center">
    <img width="600" alt="image" src="https://github.com/kim0527/Effective-Java/assets/143387515/0d4df472-45c6-48c9-95bf-11fdd144cd46">
    </p>

- 객체가 가진 정보를 문자열로 만들어 출력해준다고 정의되어있다.

### ❗️ 하지만
```java
public class Test {
    private String testName;
    private int startYear;
    public Test(String testName, int startYear) {
        this.testName = testName;
        this.startYear = startYear;
    }

    public static void main(String[] args) {
        Test test = new Test("toString",2024);
        System.out.println(test.toString());
    }
}
```
```
실행결과 : item12.Test@a09ee92
```
- 실제로 위와 같이 객체를 하나 만들고 toString 메서드를 사용해보면 이해하기 힘든 정보가 나온다.
```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```
- 해당 메서드를 살펴보았더니, 객체의 이름과 그것을 16진수로 표시한 해시코드를 반환하고 있었다.
- 하지만 이는 정의되어있는 "객체가 가진 정보"와는 거리가 멀다.

## 🎯 Overide해서 유익한 정보 담기
```java
@Override
public String toString() {
    return "이 실험 이름은 "+testName+"이고, 실험 시작일은 "+startYear+"입니다.";
}
```
```
실행결과 : 이 실험 이름은 toString이고, 실험 시작일은 2024입니다.
```
- toString()를 위와 같이 재정의했고 핵심 내용인 testName과 startYear의 정보가 보이도록 했다.
- 실행결과, 기본 toString 메서드를 사용했을때 보다 이해하기 쉬운 객체 정보를 알 수 있다.
<p align="center">
<img width="600" alt="image" src="https://github.com/kim0527/Effective-Java/assets/143387515/17146cd7-e116-488b-8248-da3e6859c3dc">
</p>

- 실제로 메서드 설명에서도 하위클래스는 해당메서드를 재정의해서 사용하는 것을 추천하고 있다.

## 🧑‍💻 toString Overide의 장점
- toString은 직접 호출하지 않아도 다른 곳에서 많이 사용된다.
- ex) println, printf, 문자열 연결 연산자(+), assert 구문
    ```java
    //println
    public void println(Object x) {
        String s = String.valueOf(x);
        if (getClass() == PrintStream.class) {
            // need to apply String.valueOf again since first invocation
            // might return null
            writeln(String.valueOf(s));
        } else {
            synchronized (this) {
                print(s);
                newLine();
            }
        }
    }
    
    //String.valueOf
    public static String valueOf(Object obj) {
        return (obj == null) ? "null" : obj.toString();  //toString메서드 사용
    }
    ```
- 그래서 toString()은작성한 객체를 참조하는 컴포넌트가 오류 메세지를 로깅할때도 호출될 수 있다.
- 만약 toString을 사람이 이해하기 쉽게 재정의 했다면, 로그 없이 아래 코드만으로 문제를 진단할 수 있다.
  ```java
  System.out.println(Test + "오류 발생 객체");
  ```
## 🙆‍♂️ toString()을 올바르게 재정의하는 방법
#### 1️⃣ 객체가 가진 주요 정보 모두를 반환하는 게 좋다.
- 위 Test의 예시처럼 testName, startYear와 같은 주요 정보들을 모두 보여주도록 재정의 하는 것이 좋다.
- 하지만 만약 객체의 정보가 너무 많다면 아래 예시와 같이 요약정보를 담는 것이 좋다.
    ```
    맨해튼 거주자 전화번호부(총 1487536개)
    ```
#### 2️⃣ 반환값의 포맷을 문서화할지 정하고, 의도 명확히 밝히기.
``` java
/**
* 이 전화번호의 문자열 표현을 반환한다.
* 이 문자열은 "XXX-YYY-ZZZZ"형태의 12자로 구성된다.
* XXX는 지역코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
* 각각의 대문자는 10진수 숫자 하나를 나타낸다.
*
* 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
* 앞에서부터 0으로 채워나간다.예컨대 가입자 번호가 123이라면
* 전화번호의 마지막 네 문자는 "0123"이 된다.
*/
@Override
public String toString() {
  return String.format("%03d-%03d-%04d",
                        areaCode,Prefix,lineNum);
}
```
- 전화번호와 행렬 같은 값 클래스라면 주석을 통해 문서화하는 것을 추천한다.
- 위 와 같이 포맷을 명시하면 객체는 ```표준적이고```, ```명확하고```, ```사람이 읽을 수 있게 된다```는 장점을 가지게 된다.
- 하지만 포맷을 한번 명시하면, 그 ```포멧에 얽매이게 된다```는 단점도 존재한다.

```java
/**
 * Outputs this date-time as a {@code String}, such as
 * {@code 2007-12-03T10:15:30+01:00[Europe/Paris]}.
 * <p>
 * The format consists of the {@code LocalDateTime} followed by the {@code ZoneOffset}.
 * If the {@code ZoneId} is not the same as the offset, then the ID is output.
 * The output is compatible with ISO-8601 if the offset and ID are the same,
 * and the seconds in the offset are zero.
 *
 * @return a string representation of this date-time, not null
 */
@Override  // override for Javadoc
public String toString() {
    String str = dateTime.toString() + offset.toString();
    if (offset != zone) {
        str += '[' + zone.toString() + ']';
    }
    return str;
}
```
- 실제로 ZonedDataTime 클래스는 위와 같이 포맷을 명시하고, 주석으로 의도를 문서화하고 있다.

#### 3️⃣ 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.
- 2️⃣의 phoneNumber 예시로 들면 areaCode, Prefix, lineNum에 접근할 수 있는 ``` getter```를 정의하자.
- 만약 정의해두지 않는다면, 프로그래머는 해당 정보를 필요로 할때 파싱하는 코드를 작성할 수 밖에 없다.
- 이는 성능이 나빠지고, 불필요한 작업이다.

## 🙅‍♂️ toString 재정의가 불필요할때
1. 상위 클래스에서 이미 toString을 올바르게 재정의한 경우
2. 정적 유틸리티 클래스의 경우
    - 정적 유틸리티 클래스는 클래스 정보가 의미가 없다.
    - 단순 정적 메서드를 모아 편하게 사용하기 위함이기 때문에
3. 열거타입(enum)의 경우
    ```java
    public enum Grade {
        BORONZE,
        SILVER,
        GOLD,
        PLATINUM
    }
    public class EnumTest {
        public static void main(String[] args) {
            Grade grade=Grade.PLATINUM;
            System.out.println(grade.toString());  //PLATINUM
        }
    }
    ```
    - 열거타입의 경우 이미 완벽한 toString을 제공하고 있다.
## ➕ @AutoValue
- 프로그래머의 생산성을 높여주는 POJO 헬퍼 라이브러리
- Getter(), Setter(), toString(), equals(), hashCode()을 자동으로 만들어준다.
- 2️⃣의 phoneNumber와 같이 포맷이 있는 경우에는 적합하진 않지만, 기본 Obejct.toString()보다는 훨씬 유용하다.

## ✅ 정리하면
#### 객체의 유용한 정보를 편하게 확인하기 위해서, 웬만하면 객체의 toString()은 재정의해서 사용하자.

### 📝 Reference
[Java 에서 toString 메소드의 올바른 사용 용도에 대하여](https://hudi.blog/java-correct-purpose-of-tostring/)<br>
[AutoValue](https://sdusb.blogspot.com/2017/04/android-antovalue.html)
