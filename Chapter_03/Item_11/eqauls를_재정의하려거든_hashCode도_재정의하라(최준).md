# 1️⃣ Item 11: equals를 재정의하려거든 hashCode도 재정의하라

<br>

## 📌 목차
1. hashCode란
2. hashCode 규약
3. hashCode 재정의 X 문제점
4. 안 좋은 hashCode
5. 좋은 hashCode
6. 좋은 hashCode를 위한 요령
7. 요령 2.2
8. 좋은 hashCode 요령 적용
9. hashCode 구현 후
10. hashCode 주의점 & 유의점
11. Objects.hash
12. Tip
13. 결론

<br>

## ❓ hashCode란

자바의 hashcode() 메서드는 `Object 클래스의 메서드`이다. 

따라서 모든 클래스는 Object 클래스를 상속하기 때문에 사실 상 `모든 객체에서 사용할 수 있다`. 

hashCode() 메서드는 해싱 기법에 사용되는 `해시함수(hash function)을 구현한 것`이다. 

해시함수는 찾고자 하는 값을 입력하면 그 값의 `해시코드(hash code)를 반환`한다. 

해시코드는 일반적으로 각 객체의 주소값을 변환하여 생성한 객체의 고유한 정수값이다. 

<br>

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbchCXO%2FbtrlEJp32FW%2Fl7bhwrCRsUyzx4i3kFrXVK%2Fimg.png)

<br>

## 👮 hashCode 규약

아래는 Object 명세에서 발췌한 `hashCode 규약`이다. 

1. equals 비교에 사용되는 `정보가 변경되지 않았다면` 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 `항상 같은 값을 반환해야 한다`. 

    단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다. 

2. equals(Object)가 `두 객체를 같다고 판단했다면`, 두 객체의 hashCode는 `똑같은 값을 반환`해야 한다. 

3. equals(Object)가 `두 객체를 다르다고 판단했더라도`, 두 객체의 hashCode가 `서로 다른 값을 반환할 필요는 없다`. 

    단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다. 

<br>

## ❎ hashCode 재정의 X 문제점

equals를 재정의한 클래스 모두에서 `hashCode도 재정의`해야 한다. 

그렇지 않으면 위의 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다. 

<br>

hashCode `재정의를 잘못했을 때 크게 문제가 되는 조항은 두번째 조항`이다. 

즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다. 

equals 메서드는 재정의하면 물리적으로 다른 두 객체를 논리적으로 같다고 할 수 있다. 

그런데 `Object의 기본 hashCode 메서드`는 equals에서 같다고 판단한 두 객체가 전혀 다르다고 판단해 규약과 달리 서로 다른 값을 반환한다. 

그래서 equals는 재정의 하고 hashCode를 재정의 하지 않고 `Object의 hashCode 메서드를 사용하면 hashCode 규약 2번을 어기게 되는 것`이다. 

<br>

### ❇ 문제점 예시

Item 10의 PhoneNumber 클래스의 인스턴스를 `HashMap 원소로 사용`한다고 생각해보자. 

아래 코드처럼 사용할 것이다. 

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");
m.get(new PhoneNumber(707, 867, 5309));
```

<br>

이렇게 get에 논리적으로 똑같은 PhoneNumber 객체를 만들어서 넣었다. 

그렇다면 “제니”가 나와야 할 것 같지만 `실제로는 null을 반환`한다. 

<br>

왜 그런가 하면 여기서 `2개의 PhoneNumber 인스턴스가 사용`되었다. 

하나는 put 할 때 new로 생성해서 넣을 때 사용했다.  

하나는 get 할 때 new로 생성해서 HashMap에서 꺼내려고 할 때 사용했다. 

<br>

그런데 hashCode 메서드를 재정의 하지 않아서 `논리적 동치인 두 객체가 서로 다른 해시코드를 반환하는 것`이다. 

equals는 재정의 해서 논리적으로 동치이지만 hashCode는 재정의 하지 않아서 다른 해시코드를 반환해 두번째 규약을 어긴 것이다. 

<br>

따라서 get 메서드는 엉뚱한 해시 버킷에 가서 객체를 찾으려 한 것이다. 

설사 두 인스턴스를 같은 버킷에 담아도 get 메서드는 여전히 null을 반환한다. 

왜냐하면 HashMap은 해시코드가 다른 엔트리끼리는 동치성 비교를 시도조차 하지 않도록 최적화 되어 있기 때문이다. 

<br>

- ### `해시 버킷`
    
    ### 🌐 HashMap & HashTable
    
    `HashMap과 HashTable`은 Java의 API 이름이다. 
    
    HashTable 또한 Map 인터페이스를 구현하고 있기 때문에 HashMap과 HashTable이 제공하는 기능은 같다. 
    
    다만 HashMap은 보조 해시 함수(Additional Hash Function)를 사용하기 때문에 보조 해시 함수를 사용하지 않는 HashTable에 비하여 해시 충돌(hash collision)이 덜 발생할 수 있어 상대으로 성능상 이점이 있다. 
    
    보조 해시 함수가 아니더라도, HashTable 구현에는 거의 변화가 없는 반면, HashMap은 지속적으로 개선되고 있다.
    
    <br>

    HashMap과 HashTable을 정의한다면, `'키에 대한 해시 값을 사용하여 값을 저장하고 조회하며, 키-값 쌍의 개수에 따라 동적으로 크기가 증가하는 associate array'`라고 할 수 있다. 
    
    이 associate array를 지칭하는 다른 용어가 있는데, 대표적으로 Map, Dictionary, Symbol Table 등이다.
    
    <br>

    ### 💥 해시 분포와 해시 충돌
    
    HashMap은 기본적으로 각 객체의 hashCode() 메서드가 반환하는 값을 사용하는 데, 결과 `자료형은 int`다. 
    
    32비트 정수 자료형으로는 완전한 자료 해시 함수를 만들 수 없다. 
    
    논리적으로 생성 가능한 `객체의 수가 2^32보다 많을 수 있기 때문`이다.
    
    또한 모든 HashMap 객체에서 O(1)을 보장하기 위해 랜덤 접근이 가능하게 하려면 원소가 2^32인 배열을 모든 HashMap이 가지고 있어야 하기 때문이다.
    
    따라서 HashMap을 비롯한 많은 해시 함수를 이용하는 associative array 구현체에서는 메모리를 절약하기 위하여 실제 해시 함수의 표현 `정수 범위보다 작은 M개의 원소가 있는 배열만을 사용`한다. 
    
    따라서 다음과 같이 객체에 대한 해시 코드의 `나머지 값을 해시 버킷 인덱스 값으로 사용`한다.
    
    ```java
    int index = X.hashCode() % M; 
    ```
    
    <br>

    이 코드와 같은 방식을 사용하면, 서로 다른 해시 코드를 가지는 `서로 다른 객체가 1/M의 확률로 같은 해시 버킷을 사용`하게 된다. 
    
    이는 해시 함수가 얼마나 해시 충돌을 회피하도록 잘 구현되었느냐에 상관없이 발생할 수 있는 또 다른 종류의 해시 충돌이다. 
    
    이렇게 해시 충돌이 발생하더라도 키-값 쌍 데이터를 잘 저장하고 조회할 수 있게 하는 방식에는 대표적으로 두 가지가 있는데, 하나는 `Open Addressing`이고, 다른 하나는 `Separate Chaining`이다. 
    
    이 둘 외에도 해시 충돌을 해결하기 위한 다양한 자료 구조가 있지만, 거의 모두 이 둘을 응용한 것이라고 할 수 있다.
    
    <br>

    ![image](https://d2.naver.com/content/images/2015/06/helloworld-831311-4.png)
    
    <br>

    `Open Addressing`은 데이터를 삽입하려는 해시 버킷이 이미 사용 중인 경우 다른 해시 버킷에 해당 데이터를 삽입하는 방식이다. 
    
    데이터를 저장/조회할 해시 버킷을 찾을 때에는 Linear Probing, Quadratic Probing 등의 방법을 사용한다.
    
    `Separate Chaining`에서 각 배열의 인자는 인덱스가 같은 해시 버킷을 연결한 링크드 리스트의 첫 부분(head)이다.
    
    `Java HashMap에서 사용하는 방식은 Separate Channing이다.`
    
    <br>

    ### 🥣 해시 버킷
    
    해시 테이블은 각각의 Key값에 해시함수를 적용해 배열의 고유한 index를 생성하고, 이 index를 활용해 값을 저장하거나 검색하게 된다. 
    
    여기서 `실제 값이 저장되는 장소를 버킷 또는 슬롯`이라고 한다.
    
    해시 버킷은 정렬이나 검색 용도로 데이터 항목을 배분하는 데 쓰인다. 
    
    이 작업의 목표는 서로 연결된 목록을 약화하여 특정 항목의 검색 결과에 짧은 기간 내에 액세스할 수 있게 하는 것이다. 
    
    <br>

    ![image](https://velog.velcdn.com/images%2Fsyoung125%2Fpost%2F5c2f57da-642a-4404-bb06-dc1947a5f034%2Fimage.png)
    
    <br>

    자바의 HashMap에서 Separate Channing을 사용하고 나머지 연산을 사용해 `같은 해시 버킷에 저장될 수 있다`. 
    
    해시 코드는 다른데 나머지 연산을 하니 같은 값이 나와 같은 해시 버킷에 들어갈 수 있는 것이다. 
    
    하지만 위에서 말했듯이 `두 인스턴스를 같은 버킷에 담아도 get 메서드는 여전히 null 반환`한다고 했다. 
    
    왜냐하면 HashMap은 해시코드가 다른 엔트리끼리는 동치성 비교를 시도조차 하지 않도록 최적화 되어 있기 때문이다. 
    
<br>

## 👎 안 좋은 hashCode

위의 문제를 해결하기 위해 적절하게 hashCode 메서드를 재정의 해야 한다.

하지만 문제를 해결하려고 `안 좋게 hashCode 메서드를 재정의` 하는 경우가 있다.

<br>

아래가 바로 그런 경우다. 

```java
@Override public int hashCode(){
	return 42;
}
```

<br>

이 코드는 위에서 문제가 되었던 `2번 규약에 대한 문제를 해결`해준다. 

왜냐하면 이 코드는 `동치인 모든 객체에서 똑같은 해시코드인 42를 반환하기 때문`이다. 

<br>

하지만 모든 객체에게 똑같은 값만 반환하니 모든 객체가 해시테이블의 버킷 하나에 담겨 `마치 연결 리스트처럼 동작`한다. 

그 결과 `평균 수행 시간이 O(1)인 해시테이블이 O(n)`으로 느려저 객체가 많아지면 도저히 사용할 수 없다. 

<br>

## 👍 좋은 hashCode

좋은 해시 함수라면 `서로 다른 인스턴스에 다른 해시코드를 반환`한다. 

이것이 hashCode의 세번째 규약이 요구하는 속성이다. 

<br>

이상적인 해시 함수는 주어진 서로 다른 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다. 

이상을 완벽히 실현하기는 어렵지만 비슷하게 만들기는 그다지 어렵지 않다. 

<br>

좋은 hashCode를 작성하는 `요령`을 보자. 

<br>

## 🔢 좋은 hashCode를 위한 요령

1. int 변수인 `result를 선언한 후 값을 c로 초기화`한다.
    
    - 이 때, c는 해당 객체의 `첫번째 핵심 필드`를 단계 2.1 방식으로 계산한 해시코드이다.
    
    - 여기서 핵심 필드는 equals 비교에 사용되는 필드를 말한다.

2. 해당 객체의 `나머지 핵심 필드`인 f 각각에 대해 다음 작업을 수행한다.
    
    1. 해당 필드의 `해시코드 c 를 계산`한다.
        
        - `기본 타입 필드`
            
            - 기본 타입 필드라면, **Type.hashCode(f)** 를 수행한다.
            
            - 여기서 *Type*은 해당 기본타입의 박싱 클래스다.
        
        - `참조 타입 필드`
            
            - 참조 타입 필드면서, 이 클래스의 equals 메소드가 이 필드의 equals를 재귀적으로 호출하여 비교한다면, **이 필드의 hashCode를 재귀적으로 호출**한다.
            
            - 참조 타입 필드면서 hashCode 재귀적으로 호출하는데 계산 더 복잡해질 것 같으면 **이 참조 필드의 표준형 만들어 표준형의 hashCode를 호출**한다.
            
            - 참조 필드의 값이 **null이면 0을 사용한다**. 다른 상수도 괜찮지만 전통적으로 0을 사용한다.
        
        - `배열 타입 필드`
            
            - 필드가 배열이라면, **핵심 원소 각각을 별도 필드처럼 다룬다**.
            
            - 배열 필드에서 위의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시 코드를 계산한 다음 단계 2.2 방식으로 갱신한다.
            
            - 배열 필드에서 **핵심 원소가 하나도 없다면 단순히 상수 = 0을 사용**한다.
            
            - 배열 필드에서 **모든 원소가 핵심 원소라면 Arrays.hashCode를 사용**한다.
    
    2. 단계 2.1에서 계산한 `해시코드 c로 result를 갱신`한다.
        
        - result = 31 * result + c;

3. `result를 반환`한다.

<br>

## 2️⃣.2️⃣ 요령 2.2

요령 2.2에서 곱셈 31*result는 필드를 `곱하는 순서에 따라 result 값이 달라진다`. 

그 결과 클래스에 비슷한 필드가 여러 개일 때 해시 효과를 크게 높여준다. 

<br>

String의 hashCode에서 이렇게 요령 2.2에서 하는 `31 곱하는 것이 없다면` 모든 아나그램의 해시코드가 같아질 것이다. 

<br>

아나그램은 `구성하는 철자가 같고 그 순서만 다른 문자열`이다. 

예를 들어 abcd와 adbc는 아나그램인 것이다. 

a, b, c, d의 해시코드는 각각 같고 31 곱하는 것 없이 result에 각각의 해시코드를 더하기만 하면 똑같은 해시코드가 나오는 것이다. 

<br>

곱할 숫자를 31로 정한 이유는 `31이 홀수이면서 소수`이기 때문이다. 

`홀수를 곱하는 이유`는 2를 곱하는 것은 시프트 연산과 같은 결과를 내기 때문에 숫자가 짝수이고 오버플로가 발생한다면 정보를 읽게 된다. 

`소수를 곱하는 이유`는 명확하지는 않지만 전통적으로 그렇게 해왔다. 

<br>

결과적으로 31을 이용하면 이 `곱셈을 시프트 연산과 뺄셈으로 대체해 최적화 할 수 있다`. 

31*i = (i<<5) - i 로 대체할 수 있다. 

요즘 `VM들은 이런 최적화를 자동`으로 해준다. 

<br>

## ✅ 좋은 hashCode 요령 적용

위의 요령들을 PhoneNumber 클래스에 적용해보자. 

PhoneNumber 클래스의 핵심 필드는 areaCode, prefix, lineNum이 있었다. 

각각 전화번호의 지역 번호, 프리픽스, 가입자 번호를 의미하는 필드들이다. 

<br>

`요령들을 적용해 재정의한 좋은 hashCode`는 아래와 같다. 

```java
    // 코드 11-2 전형적인 hashCode 메서드 (70쪽)
    @Override public int hashCode() {
        int result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        return result;
    }
```

<br>

이 메서드는 PhoneNumber 인스턴스의 핵심 필드 3개만 사용해 간단한 계산만 수행한다. 

이 과정에서 비결정적 요소는 전혀 없어서 동치인 PhoneNumber 인스턴스들은 같은 해시코드 가지는 것이 확실하다. 

단순하고 충분히 빠르고 서로 다른 전화번호들은 다른 해시 버킷들로 제법 훌륭히 분배해준다. 

<br>

## 🆗 hashCode 구현 후

hashCode를 `다 구현했다면 할 일`들은 아래와 같다. 

- hashCode 메서드가 ***동치인 인스턴스에 대해 똑같은 해시코드를 반환하는지 자문***

- 똑같은 해시코드 반환할 것이라는 직관을 검증할 ***단위 테스트 작성***

- 동치인 인스턴스가 서로 다른 해시코드 반환한다면 ***원인 찾아 해결***

<br>

단위 테스트의 경우 equals와 hashCode 메서드를 자동 생성 프레임워크인 AutoValue로 생성했다면 건너 뛰어도 좋다고 한다. 

<br>

## ⁉ hashCode 주의점 & 유의점

hashCode를 구현할 때 `주의할 점, 알아야 할 점`들이 있다. 

- 파생 필드는 해시코드 계산에서 제외해도 된다.

- equals 비교에 사용되지 않은 필드는 반드시 제외해야 한다.

- 클래스가 불변이고 해시코드를 계산하는 비용이 크다면 매번 새로 계산하기보다는 캐싱 방식 고려해야 한다.

- 성능 높인다고 해시코드 계산할 때 핵심 필드를 생략해서는 안된다.

- hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말아야 한다.

<br>

각 항목들에 대해서 자세히 알아보자. 

<br>

### 1️⃣ 1. 파생 필드는 해시코드 계산에서 제외해도 된다.

말 그대로 다른 필드로 부터 계산해 낼 수 있는 필드 즉 `파생 필드는 모두 무시해도 된다는 것`이다. 

요령의 2단계에서 `파생 필드들에 대한 해시코드를 계산할 필요가 없다는 것`이다. 

<br>

### 2️⃣ 2. equals 비교에 사용되지 않는 필드는 반드시 제외해야 한다.

중요한 항목으로 `반드시` 제외해야 한다. 

그렇지 않으면 `hashCode 2번째 규약` ‘equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다. ‘를 어기게 될 위험이 있다. 

<br>

equals에서는 a, b 필드만 사용해서 판단하는 것이다. 

그런데 hashCode에서는 a, b, c 필드 사용해서 판단하는 것이다. 

따라서 객체가 a, b 필드는 같아서 equals에서는 같다고 판단한다. 

하지만 c 필드가 다를 수 있다.

그렇다면 hashCode에서는 다른 값을 반환해 `규약 2를 어기는 것`이다. 

<br>

### 3️⃣ 3. 클래스가 불변이고 해시코드를 계산하는 비용이 크다면 매번 새로 계산하기보다는 캐싱 방식 고려해야 한다.

이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 `인스턴스가 만들어질 때 해시코드를 계산해 둬야 한다는 것`이다. 

<br>

객체가 해시의 키로 사용되지 않는 경우라면 hashCode 메서드가 처음 불릴 때 계산하는 `지연 초기화 전략 사용`을 고려해 볼 수 도 있다. 

필드를 지연 초기화 하려면 그 클래스를 `Thread-safe`하게 만들도록 신경 써야 한다. 

hashCode 필드의 초깃값은 흔히 생성되는 객체의 해시 코드와는 달라야 한다. 

<br>

아래는 이 방식을 사용한 PhoneNumber 클래스의 hashCode의 예시 코드다. 

```java
    // 해시코드를 지연 초기화하는 hashCode 메서드 - 스레드 안정성까지 고려해야 한다. (71쪽)
    private int hashCode; // 자동으로 0으로 초기화된다.

    @Override public int hashCode() {
        int result = hashCode;
        if (result == 0) {
            result = Short.hashCode(areaCode);
            result = 31 * result + Short.hashCode(prefix);
            result = 31 * result + Short.hashCode(lineNum);
            hashCode = result;
        }
        return result;
    }
```

<br>

- ### `지연 초기화란?`
    ### 🔜 지연 초기화
    지연 초기화는 객체나 변수를 `처음으로 사용할 때까지 초기화를 미루는 방법`을 말한다.
    
    이를 통해 프로그램의 `성능을 향상`시킬 수 있다.

    <br>
    
    가장 간단한 지연 초기화 방법 중 하나는 `객체를 필요로 할 때까지 초기화를 미루는 방식`이다. 
    
    예를 들어, 인스턴스 변수를 선언할 때 null로 초기화하고, 이 변수를 처음 사용할 때 실제로 객체를 생성하여 할당하는 방식이다.
    
    ```java
    public class MyClass {
        private MyObject myObject;
    
        public MyObject getMyObject() {
            if (myObject == null) {
                myObject = new MyObject();
            }
            return myObject;
        }
    }
    ```
    
    <br>

    지연 초기화은 Java 8에 포함된 `Supplier<T>`로 구현해볼 수 도 있다.
    
    Supplier<T> 함수 안에서 클래스의 생성자를 사용하면, `get() 메서드가 불려야 해당 클래스의 객체가 생성`된다.
    
    객체가 필요할 때 Supplier의 get() 메서드 호출해 객체를 받는 것이다. 

    <br>
    
    아래 코드처럼 사용하는 것이다. 
    
    ```java
        public static void main(String[] args) {
            Supplier<MyObject> lazyObjectSupplier = () -> new MyObject("lazyObject");
            lazyObjectSupplier.get();
            lazyObjectSupplier.get();
            lazyObjectSupplier.get();
        }
    ```
    
    <br>

    ### ⚠ 지연 초기화 문제점
    하지만 지연 초기화에는 `문제점`이 있다. 
    
    `multi Thread 환경`이라고 생각해보자. 
    
    2명의 사용자가 동시에 객체를 생성하는 메서드를 호출했다. 
    
    그렇다면 정확하게 인스턴스는 한번만 만들어지는가? 아니다.
    
    위의 늦은 초기화 방법에서 `객체가 null인지 판단하는 조건문`에 대해서 여러 thread가 `동시에 들어올 가능성`이 존재하기 때문에 위험하다. 

    <br>
    
    ### ⭕ 지연 초기화 문제점 해결책
    따라서 지연 초기화를 Thread-safe하게 하려면 `volatile 키워드`와 `synchronized 블록`을 사용해야 한다. 
    
    아래 코드와 같이 사용하는 것이다. 
    
    ```java
    public class MyClass {
        private volatile MyObject myObject;
    
        public MyObject getMyObject() {
            if (myObject == null) {
                synchronized (this) {
                    if (myObject == null) {
                        myObject = new MyObject();
                    }
                }
            }
            return myObject;
        }
    }
    ```
    <br>


    synchronized 블록을 사용하면 특정 코드 블록을 동기화할 수 있다. 
    
    이때 특정 객체의 락을 사용하거나, 클래스의 락을 사용할 수 있다.
    
    하지만 `이것만 사용하면 deadlock`에 걸릴 수 있다. 
    
    <br>

    따라서 `volatile 키워드 사용해야 한다`. 
    
    volatile 키워드를 사용하면 변수의 변경 사항이 즉시 다른 스레드에게 알려지도록 보장한다. 
    
    따라서 변수에 대한 모든 변경 사항이 메인 메모리에 즉시 반영되어 다른 스레드에서 변수를 읽을 때 항상 최신 값을 얻을 수 있다. 
    
    이를 통해 위의 문제를 방지할 수 있다.
    
<br>

### 4️⃣ 4. 성능 높인다고 해시코드 계산할 때 핵심 필드를 생략해서는 안된다.

`핵심 필드를 생략하면 속도는 빨라지겠지만 해시 품질이 나빠져` 해시테이블의 성능을 심각하게 떨어뜨릴 수도 있다. 

특히 어떤 필드는 특정 영역에 몰린 인스턴스들의 해시코드를 `넓은 범위로 고르게 퍼트려주는 효과`가 있을지도 모른다. 

하필 이런 필드를 생략하면 해당 영역의 수많은 인스턴스가 단 몇 개의 해시코드로 집중되어 해시테이블의 속도가 선형으로 느려질 것이다. 

<br>

### 5️⃣ 5. hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말아야 한다.

공표하지 말아야 하는 이유는 그래야 클라이언트가 hashCode가 반환하는 값에 `의지하지 않게 되고 추후에 계산 방식을 바꿀 수 있기 때문`이다. 

생성 규칙 자세히 공표하면 프로그래머가 이 규칙에 맞게 로직을 구현하거나 해서 나중에 생성 규칙이 달라지면 프로그래머가 작성한 로직도 다시 고쳐야 하는 의지하는 문제가 생기는 것이다. 

<br>

String, Integer를 포함해 자바 라이브러리의 많은 클래스에서 hashCode 메서드가 반환하는 정확한 값을 알려준다. 

즉 hashCode의 해시 기능이 정해졌다는 것이다. 

바람직하지 않은 실수지만 바로잡기에는 이미 늦었다. 

향후 릴리스에서 해시 기능을 개선할 여지도 없애버렸다. 

<br>

자세히 규칙을 공표하지 않는다면, `해시 기능에서 결함을 발견했거나 더 나은 해시 방식을 알아낸 경우 다음 릴리스에서 수정할 수 있는 것`이다. 

<br>

## 🏷 Objects.hash

모든 클래스의 상위 클래스인 Object 클래스 말고 `Objects 클래스`는 임의의 개수만큼 객체 받아 해시 코드를 계산해주는 `정적 메서드 hash를 제공`한다. 

이 메서드를 활용하면 앞서 요령대로 구현한 코드와 비슷한 수준의 hashCode 함수를 단 한 줄로 작업할 수 있다. 

<br>

하지만 `아쉽게도 속도는 더 느리다`. 

왜냐하면 입력 인수를 담기 위한 `배열이 만들어진다`. 

그리고 입력 중에 기본 타입이 있다면 `박싱과 언박싱`도 거쳐야 한다. 

그래서 더 느리다. 

그래서 hash 메서드는 성능에 민감하지 않은 상황에서만 사용하자. 

<br>

Objects의 hash 코드는 아래와 같다. 

```java
    public static int hash(Object... values) {
        return Arrays.hashCode(values);
    }
    
    
    //Arrays.hashCode() 코드
    public static int hashCode(Object a[]) {
        if (a == null)
            return 0;

        int result = 1;

        for (Object element : a)
            result = 31 * result + (element == null ? 0 : element.hashCode());

        return result;
    }
```

<br>

PhoneNumber 클래스의 hashCode 메서드를 Objects.hash를 이용해 구현하면 아래와 같다. 

```java
    // 코드 11-3 한 줄짜리 hashCode 메서드 - 성능이 살짝 아쉽다. (71쪽)
    @Override public int hashCode() {
        return Objects.hash(lineNum, prefix, areaCode);
    }
```

<br>

## ➕ Tip

이번 아이템에서 소개한 해시 함수 제작 요령은 최첨단은 아니지만 `충분히 훌륭`하다. 

품질 면에서나 해싱 기능 면에서나 자바 플랫폼 라이브러리가 사용한 방식과 견줄만하다. 

그리고 대부분의 쓰임에도 문제가 없다. 

<br>

단, `해시 충돌이 더욱 적은 방법`을 꼭 써야 한다면 `구아바의 com.google.common.hash.Hashing을 참고`하자. 

<br>

- ### `구아바의 com.google.common.hash.Hashing 란?`
    
    Guava(구아바)는 구글이 개발한 자바 프로그래밍 언어를 위한 오픈 소스 라이브러리이다.
    
    com.google.common.hash.Hashing는 Guava 라이브러리의 일부로 해시 함수를 다루는 유틸리티 클래스다. 
    
    `해시 관련 static 유틸리티 메서드`들이 있는 곳이다. 
    
    여기에는 다양한 해시 함수들이 있다. 
    
<br>

## ❗ 결론

equals를 재정의할 때는 hashCode도 반드시 재정의 해야 한다. 

그렇지 않으면 프로그램이 제대로 동작하지 않을 것이다. 

재정의한 hashCode는 Object의 API 문서에 기술된 일반 규약을 따라야 한다. 

그리고 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다. 

이렇게 구현하는 것이 어렵지는 않지만 조금 따분하기는 하다. 

하지만 AutoValue 프레임워크를 사용하면 멋진 equals와 hashCode를 자동으로 만들어준다. 

IDE들도 이런 기능을 일부 제공한다. 

<br>

## 📍 references

- [hashCode란 1](https://brunch.co.kr/@mystoryg/133)
- [hashCode란 2](https://studyandwrite.tistory.com/475)
- [해시 버켓 1](https://d2.naver.com/helloworld/831311)
- [해시 버켓 2](https://www.databricks.com/kr/glossary/hash-buckets)
- [해시 버켓 3](https://velog.io/@syoung125/%ED%95%B4%EC%8B%9C%ED%85%8C%EC%9D%B4%EB%B8%94%EC%9D%B4%EB%9E%80)
- [지연 초기화 1](https://velog.io/@9geunu/Java-Lazy-Initialization)
- [지연 초기화 2](https://sabarada.tistory.com/128)
- [구아바](https://guava.dev/releases/21.0/api/docs/com/google/common/hash/Hashing.html)