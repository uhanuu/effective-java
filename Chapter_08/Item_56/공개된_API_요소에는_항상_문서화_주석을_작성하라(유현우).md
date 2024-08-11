# 공개된 API 요소에는 항상 문서화 주석을 작성하라

자바에서 자바독(javadoc)은 소스코드 파일에서 문서화 주석(자바독 주석)이라는 특수한 형태로 기술된 설명을 추려 API 문서로 변환해준다.

### 버전이 올라가며 추가된 중요한 자바독 태그

- 자바 5: @literal, @code
- 자바8: @implSpec
- 자바9: @index

### API를 올바로 문서화하려면 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야한다.

- 직렬화할 수 있는 클래스라면 직렬화 형태에 관해서도 적어야 한다.
- 문서화 주석이 없다면 자바독도 그저 공개 API 요소들의 '선언'만 나열해주는게 전부다.
- **문서가 잘 갖춰지지 않은 API는 쓰기 헷갈려서 오류의 원인이 되기 쉽다.**
- 기본 생성자에는 문서화 주석을 달 방법이 없으니 공개 클래스는 절대 기본 생성자를 사용하면 안된다.
    - 잘 이해가지 않음
- 유지보수까지 고려한다면 대다수의 공개되지 않은 클래스, 인터페이스, 생성자, 메서드, 필드에도 문서화 주석을 달아야 한다.

### 메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다.

- **상속용으로 설계된 클래스의 메서드가 아니라면 어떻게(how) 동작하는지가 아니라 무엇을(what) 하는지를 기술해야 한다.**
- 문서화 주석에는 클라이언트가 **해당 메서드를 호출하기 위한 전제조건을 모두 나열**해야 한다.
    - **일반적으로 전제조건은 @throws 태그로 비검사 예외를 선언하여 암시적으로 기술한다.**
    - 비검사 예외 하나가 전제조건 하나와 연결
- 메서드가 성공적으로 수행된 후에 만족해야 하는 **사후조건도 모두 나열**해야 한다.
- **@param 태그를 이용해 그 조건에 영향받는 매개변수에 기술할 수도 있다.**

### 전제조건과 사후조건뿐만 아니라 부작용도 문서화 해야한다.

부작용이란 사후조건으로 명확히 나타나지는 않지만 시스템의 상태에 어떠한 변화를 가져오는 것을 뜻한다. 

- 백그라운드 스레드를 시작시키는 메서드라면 그 사실을 문서에 밝혀야 한다.

### 메서드 계약을 완벽히 기술하기

- 모든 매개변수에 @param 태그를 달아야 한다.
- 반환 타입이 void가 아니라면 @return 태그를 달아야 한다.
- 발생할 수 있는 모든 예외(체크/언체크 예외)에 @throws 태그를 달아야 한다
- 따르는 코딩 표준에서 허락한다면  @return 태그의 설명이 메서드 설명과 같을 때 @return 태그를 생략해도 좋다.

### 문서화 주석 관례

- @paran 태그와 @return 태그의 설명은 해당 매개변수가 뜻하는 값이나 반환값을 설명하는 명사구를 쓴다.
    - 드물게 명사구 대신 산술 표현식을 쓰기도 한다.
- @throws 태그의 설명은 if로 시작해 해당 예외를 던지는 조건을 설명하는 절이 뒤따른다.
    
    ```java
        /**
         * Returns the element at the specified position in this list.
         *
         * @param index index of the element to return
         * @return the element at the specified position in this list
         * @throws IndexOutOfBoundsException if the index is out of range
         *         ({@code index < 0 || index >= size()})
         */
    ```
    - @param, @return, @throws 태그의 설명에는 마침표를 붙이지 않느다. 한글은 예외다.
    - @throws 절에 {@code} 태그를 달게되면 태그로 감싼 내용을 코드용 폰트로 랜더링하고 태그로 감싼 내용에 포함된 HTML 요소나 다른 자바독 태그를 무시한다.
        - HTML 메타문자인 < 기호 등을 별다른 처리 없이 바로 사용 가능하다.
          
- 자바독 유틸리티는 문서화 주석을 HTML로 변환하므로 문서화 주석 안의 HTML 요소들이 최종 HTML 문서에 반영된다.
    - `*javadoc -d doc *.java*`
- 문서화 주석에 여러 줄로 된 코드 예시를 넣으려면 {@code} 태그를 다시 \<pre> 태그로 감싸면 된다.
    - `<pre>{@code ...}</pre>`로 감싸서 사용하게 되면 HTML 탈출 메타문자를 쓰지 않아도 코드 줄바꿈이 유지된다.
    - @ 기호에는 무조건 탈출문자를 붙여야 하니 문서화 주석 안의 코드에서 애너테이션을 사용하면 주의하자
- 관례상 영문 문서화 주석에서 쓴 "this"는 호출된 메서드가 자리하는 객체를 가리킨다.

### 클래스 상속용 설계 문서화

- 클래스를 상속용으로 설계할 때는 자기사용 패턴(self-use pattern)에 대해서도 문서에 남겨 다른 프로그래머한테 그 메서드를 올바로 재정의하는 방법을 알려줘야 한다.
    - 자기사용 패턴은 자바 8에 추가된 @implSpec 태그로 문서화한다.
    - 하위 클래스들이 그 메서드를 상속하거나 super 키워드를 이용해 호출할 때 그 메서드가 어떻게 동작하는지를 명확히 인지하고 사용하도록 해줘야 한다.
- **일반적은 문서화 주석은 해당 메서드와 클라이언트 사이의 계약을 설명하지만 @implSpec 주석은 해당 메서드와 하위 클래스 사이의 계약을 설명한다.**

@implSpec 태그를 무시하는 경우 `-tag "implSpec:a:Implementation Requirements:"` 를 붙여주자

<img width="1364" alt="image" src="https://github.com/user-attachments/assets/d2090b71-fc5c-437d-8fb1-22bcf8cef37e">

- javadoc -d doc -tag "implSpec:a:Implementation Requirements:" *.java

### API 설명에 HTML 메타문자(<, >, &등) 처리

- {@literal} 태그로 감싸면 HTML 마크업이나 자바독 태그를 무시하게 해준다.
    - {@code} 태그와 비슷하지만 코드 폰트로 렌더링하지 않는다.
- 기호만 감사줘도 되지만 읽기 쉬워야 하는 원칙에 따라서 `*|r| {@literal < } 1*` 보다는 `{@literal |r| < 1}` 를 사용하자

### 문서화의 요약 설명

- 각 문서화 주석의 첫 번째 문장은 해당 요소의 요약 설명(summart description)으로 간주된다.
- 요약 설명은 반드시 대상의 기능을 고유하게 기술해야 한다.
    - 한 클래스 혹은 인터페이스 안에서 요약 설명이 똑같은 멤버가 둘 이상이면 안된다.
    - 다중정의된 메서드들의 설명은 같은 문장으로 시작하는게 자연스럽겠지만 문서화 주석에서는 허용되지 않는다.
- 마침표(.)에 주의해야 한다.
  <img width="1145" alt="image" src="https://github.com/user-attachments/assets/32cbd50f-343a-4ee4-a81c-c37f3a567ee1">
  - 요약설명 중간에 마침표가 아닌 마침표가 들어갈 경우 요약 설명이 끝나는 판단이된다.
  - `Mrs. Peacock.` 부분에서 `Peacock.`가 보이지 않는것을 볼 수 있다.
  - 요약 설명이 끝나는 판단기준은 `{<마침표> <공백> <다음 문장 시작> }` 패턴의 `<마침표>` 까지다.
  - `<공백>` 은 스페이스, 탭, 줄바꿈(첫 번째 블록 태그도)이며 `<다음 문장 시작>` 은 소문자가 아닌 문자다.
    - Mrs. 다음에 공백이 나오고 다음 단어인 P를 소문자로 변경하면 요약설명이 끝났다고 판단하지 않는다.
  - 가장 좋은 해결책은 의도치 않은 마침표를 포함한 텍스트를 {@literal}로 감싸는 것이다.
  - 자바 10 부터는 {@summary}라는 요약 설명 전용 태그가 추가되었다.

### 주석 작성 규약에 따르면 요약 설명은 완전한 문장이 되는 경우가 드물다.
- 메서드와 생성자의 요약 설명은 해당 메서드와 생성자의 동작을 설명하는 동사구여야 한다. (영어 기준)
- ArrayList 생성자와 Collection.size 메서드처럼 2인칭 문장이 아닌 3인칭 문장으로 써야한다. 한글 설명에서는 차이가 없다.
    - ArrayList(int initialCapacity) : `*Constructs an empty list with the specified initial capacity.*`
    - Collection.size(): `*Returns the number of elements in this Collection.*`
- 클래스, 인터페이스, 필드의 요약 설명은 대상을 설명하는 명사절이어야 한다. 클래스와 인터페이스의 대상은 그 인터페이스고, 필드의 대상은 필드 자신이다.
    - Instant: `*An instantaneous point on the time-line.*`
    - Math.PI
        
        ```java
            /**
             * The {@code double} value that is closer than any other to
             * <i>pi</i>, the ratio of the circumference of a circle to its
             * diameter.
             */
        ```
### 자바독이 생성한 HTML 문서에 검색 기능(자바독 9)

클래스, 메서드, 필드 같은 API 요소의 색인은 자동으로 만들어지며, 원한다면 {@index} 태그를 사용해 API에서 중요한 용어를 추가로 색인화할 수 있다.

- `* This method complies with the {@index IEEE 754} standard.`

### 제네릭, 열거 타입, 애너테이션은 특별히 주의해야 한다.

제네릭 타입이나 제네릭 메서드를 문서화할 때는 모든 타입 매개변수에 주석을 달아야 한다.

```java
/**
 * An object that maps keys to values.  A map cannot contain duplicate keys;
 * each key can map to at most one value.
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 */
public interface Map<K, V> { ... }
```

열거 타입을 문서화할 때는 상수들에도 주석을 달아야한다.

```java
/**
	* Defines the automatic redirection policy.
  */
public enum Redirect {

  /** Never redirect. */
  NEVER,

	/** Always redirect. */
	ALWAYS,

	/** Always redirect, except from HTTPS URLs to HTTP URLs. */
	NORMAL
}
```

- 열거 타입 자체와 그 열거타입의 public 메서드도 마찬가지다.

애너테이션 타입을 문서화할 때는 멤버들에도 모두 주석을 달아야 한다.

```java
/**
 * 이 애너테이션이 달린 메서드는 명시한 예외를 던져야만 성공하는 테스트 메서드임을 나타낸다.
 */
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface ExceptionTest {
	/**
	 * 이 애너테이션을 단 테스트 메서드가 성공하려면 던저야 하는 예외.
	 * (이 클래스의 하위 타입 예외는 모두 허용된다.)
	 */
  Class<? extends Throwable> value();
}
```

- 애너테이션 타입의 요약 설명은 프로그램 요소에 이 애너테이션을 단다는 것이 어떤 의미인지를 설명하는 동사구로 한다.
- 한글로 쓴다면 동사로 끝나는 평범한 문장이면 된다.

패키지를 설명하는 문서화 주석은 package-info.java 파일에 작성하고 자바9 부터 지원하는 모듈 시스템 관련 설명은 module-info.java 파일에 작성하면 된다.

### API 문서화에서 자주 누락되는 스레드 안전성과 직렬화 가능성

- 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든, 스레드 안전 수준을 반드시 API 설명에 포함해야 한다.
- 직렬화할 수 있는 클래스라면 직렬화 형태도 API 설명에 기술해야 한다.

### 자바독 메서드 주석 상속

- 문서화 주석이 없는 API 요소를 발견하면 자바독이 가장 가까운 문서화 주석을 찾아준다.
    - 이때 상위 '클래스'보다 그 클래스가 구현한 인터페이스를 먼저 찾는다.
- {@ingeritDoc} 태그를 사용해 상위 타입의 문서화 주석 일부를 상속할 수 있다.
    - 클래스는 자신이 구현한 인터페이스의 문서화 주석을 복붙하지 않고 재사용할 수 있지만, 사용하기 까다롭고 제약도 조금 있다.

### 문서화 주석의 주의사항

여러 클래스가 상호작용하는 복잡한 API라면 문서화 주석 외에도 전체 아키텍쳐를 설명하는 별도의 설명이 필요할 때가 왕왕 있다.

- 이런 설명 문서가 있다면 관련 클래스나 패키지 문서화 주석에서 그 문서의 링크를 제공해주면 좋다.

### 자바독 문서 검사

- 자바독은 프로그래머가 자바독 문서를 올바르게 작성했는지 확인하는 기능을 제공한다.
- 자바 7에서는 명령줄에서 -Xdoclint 스위치를 켜주면 이 기능이 활성화되고 자바 8은 기본으로 작동한다.
- 체크스타일 같은 IDE 플러그인을 사용하면 더 완벽하게 검사된다.
- 자바독이 생성한 HTML 파일을 HTML 유효성 검사기로 돌리면 문서화 주석의 오류를 한층 줄일 수 있다.
    - 잘못 사용한 HTML 태그 확인 용도
- 자바 9와 10의 자바독은 기본적으로 HTML4.01 문서를 생성하지만, 명령줄에서 -html5 스위치를 켜면 HTML5 버전으로 만들어준다.
- 정말 잘 쓰인 문서인지 확인하는 유일한 방법은 자바독 유틸리티가 생성한 웹페이지를 읽어보는 길뿐이다.

### 핵심 정리

- 문서화 주석은 API 문서를 문서화하는 가장 훌륭하고 효과적인 방법이다.
- 공개 API라면 빠짐없이 설명을 달아야 한다.
- 표준 규약을 일관되게 지키자.
- 문서화 주석에 임의의 HTML 태그를 사용할 수 있음을 기억하자.
    - 단 HTML 메타문자는 특별하게 취급해야 한다. (특히 애노테이션 코드 작성할 때 @)

