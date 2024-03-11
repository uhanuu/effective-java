# 🚀 Item 11 equals를 재정의하려거든 hashCode도 재정의하라


why❓ equals를 재정의한 곳에서 왜 hashCode도 재정의 해야할까요?

equals를 재정의한 곳에서 hashCode를 재정의하지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킵니다.  

📌 Object 명세서
<img width="1859" alt="image" src="https://github.com/Growth-Hub/Effective-Java/assets/76567238/acc89625-e199-434f-bfda-35b664306400">
- equals 비교에 사용되는 정보가 변경되지 않았다면, hashCode 도 변하면 안 된다.
    - (애플리케이션을 다시 실행한다면 이 값이 달라져도 상관 없음)
- equals가 두 객체가 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환한다.
    - 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.
- equals가 두 객체를 다르다고 판단했더라도, hashCode는 꼭 다를 필요는 없다.
    - 하지만, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

<br>

## ♟️ 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.

위 명세에서 가장 큰 문제가 되는 조항이 바로 두 번째 조항인 논리적으로 같은 객체는 같은 해시코드를 반환 입니다.

```java
public class HashMapTest {

    public static void main(String[] args) {
        Map<PhoneNumber, String> map = new HashMap<>();

        PhoneNumber number1 = new PhoneNumber(123, 456, 7890);
        PhoneNumber number2 = new PhoneNumber(123, 456, 7890);

//         TODO 같은 인스턴스인데 다른 hashCode
        System.out.println(number1.equals(number2)); // true
        System.out.println(number1.hashCode()); // 566034357
        System.out.println(number2.hashCode()); // 940553268

        map.put(number1, "제니");
        map.put(number2, "안유진");
        
        String s = map.get(number2);
        System.out.println(s); // 안유진
    }
}
```
위 예제에서 PhoneNumber은 이전 Item 10에서 처럼 equals는 재정의했지만 hashcode는 재정의하지 않은 클래스입니다.  
이런식으로  두 객체가 같지만  서로 다른 인스턴스이기 때문에 해시코드도 달라지게 됩니다.  

이때 중요한 것이 있습니다.  값 클래스는 또 다른 같은 값 클래스를 넣어줘도 동일하게 동작해야합니다. 
그렇다면  `number2`대신에  `new PhoneNumber(123, 456, 7890)`를 넣어줘도 동일한 결과가 나와야하는데 과연 그럴까요?

```java
String s = map.get(new PhoneNumber(123, 456, 7890));
System.out.println("map.get(new PhoneNumber(123, 456, 7890) = " + s);
```
<img width="525" alt="image" src="https://github.com/Growth-Hub/Effective-Java/assets/76567238/d60ad044-6e7d-4735-a95d-29da3dc50185">
제대로 동작되지 않습니다. 

 
why❓
`new PhoneNumber(123, 456, 7890)`가 key로 `map.get()`에 넘겨줄 때  이 키에대한 해시코드 값을 먼저 가져와서 그 해시에 해당하는 버킷에 들어있는 오브젝트를 꺼냅니다. 하지만 `new PhoneNumber(123, 456, 7890)`에 대한 해시코드는 다르기 때문에 HashMap에 이러한 key에 대한 값이 안들어있다고 판단내리고 null을 출력하게 됩니다.   

이러한 이유때문에 반드시 같은 해시값을 가질 수 잇도록 hashCode도 재정의해야합니다.


<br>

### 📌 알아두면 좋은 - HashMap의 동작 과정

HashMap은 키-값 쌍으로 데이터를 저장하는 구조입니다.

- 해싱
    - HashMap에 데이터(키-값 쌍)를 추가하려고 할 때, 먼저 키의 hashCode() 메서드를 호출하여 해시코드를 계산합니다. 
        - 이 해시코드는 키가 저장될 배열 인덱스를 결정하는 데 사용됩니다.
- 인덱스 계산
    - 해시코드를 바탕으로 배열의 인덱스를 계산합니다. 이때, 해시 충돌을 줄이기 위해 특정 함수를 사용하여 계산된 해시코드를 배열의 크기로 나누어서 인덱스를 결정합니다. 이렇게 계산된 인덱스는 데이터가 저장될 위치를 가리킵니다.
- 값 저장
    - 계산된 인덱스에 해당하는 배열 위치에 키-값 쌍을 저장합니다. 만약 그위치에 이미 다른 키-값 쌍(엔트리)이 있다면, 해시 충돌이 발생한 것입니다. HashMap은 이러한 충돌을 처리하기 위해 연결 리스트 또는 레드-블랙트리를 사용하여 같은 인덱스에 여러 엔트리를 저장할 수 있습니다.
- 해시 충돌 처리 
    - 동일한 인덱스에 여러 개의 키-값 쌍이 저장될 경우, HashMap은 연결 리스트나 레드-블랙 트리를 통해 여러 엔트리를 관리합니다. 새로운 엔트리가 추가될 때, 리스트나 트리를 순회하면서 동일한 키가 이미 존재하는지 검사합니다. 이미 존재하는 키의 경우, 그 키에 연결된 값을 새로운 값으로 대체합니다. 새로운 키인 경우, 연결 리스트의 끝에 추가하거나 레드-블랙 트리에 삽입합니다.
- 데이터 접근 
    - 키를 사용하여 HashMap에서 데이터를 검색할 때도 키의 hashCode() 메서드로 해시코드를 계산한 다음, 인덱스를 찾아 해당 위치에서 연결 리스트나 레드-블랙 트리를 통해 키-값 쌍을 검색합니다. 해당 키가 발견되면 연결된 값을 반환합니다.

<br>

## 그렇다면 hashCode를 어떻게 구현해야할까?

>클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다 캐싱을 고려해야 합니다.
해시의 키로 사용되지 않는 경우라면 hashCode가 처음 불릴 때 계산한느 지연 초기화하면 좋다.
```java
private int hashCode;

@Override
public int hashCode() {
      	int result = hashCode; // 초기값 0을 가진다.
        if(result == 0) {
        int result = Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(areaCode);
        hashCode = result;
        }
        return result;
}
```
<br>

## 정리

equals를 재정의할 때는 hashCode도 반드시 재정의해야 합니다. 그렇지 않으면 프로그램이 제대로 동작하지 않습니다.    
재정의한 hashCode는 Object의 API문서에서 기술된 일반 규약을 따라야 하며, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 합니다. 물론 `AutoValue` 프레임워크를 사용하면 멋진 equals와 hashCode를 자동으로 만들어줍니다. 


<br>
<br>

---

### 📚 Reference

- [Object](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html)
- [HashMap은 어떻게 동작할까?](https://woodcock.tistory.com/24)
