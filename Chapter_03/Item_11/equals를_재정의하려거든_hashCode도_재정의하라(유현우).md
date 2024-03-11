# equals를 재정의하려거든 hashCode도 재정의하라

`equals`를 재정의한 클래스는 `hashCode`도 재정의 해야 한다.

## `hashCode` 일반 규약

- `equals`를 재정의한 클래스는 `hashCode`도 재정의 해야 한다.
    - equals는 재정의 했는데 hashCode가 없는 경우 인스턴스를 hashMap이나 HashSet에서 사용할 때 문제가 발생할 수 있다.
    
    ```java
    public static void main(String[] args) {
            Set<Car> cars = new HashSet<>();
            Car car = new Car("오빠차", 9999);
            cars.add(car);
            System.out.println(car.equals(new Car("오빠차", 9999))); // true
            System.out.println(cars.contains(new Car("오빠차", 9999))); // false
    }
    ```
    
    - **`HashMap`은 해시코드가 서로 다른 엔트리끼리는 동치성 비교를 시도조차 않도록 최적화 되어 있기 때문**
- **`equals` 비교에 사용되는 정보가 변경되지 않았다면, `hashCode` 메서드는 몇번을 호출하든, 항상 같은 값을 반환해야 한다. (**애플리케이션을 다시 실행한다면 이 값이 달라져도 상관 없음)
- **`equals`가 두 객체가 같다고 판단했다면, 두 객체의 `hashCode`의 반환값도 같아야 한다.**→ 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.
- **`equals`가 두 객체를 다르다고 판단했더라도, `hashCode`는 꼭 다를 필요는 없다.**하지만, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

### **동치인 모든 객체에서 똑같은 hashCode를 반환하는 코드**

모든 객체에게 똑같은 값만 내어주므로 모든 객체가 해시테이블의 버킷 하나에 담겨 마치 연결리스트처럼 동작하는 문제가 생기게 된다.

```java
@Override 
	public int hashCode() {
		return 42;
	}
```

- 평균 수행 시간이 O(1)인 해시테이블이 O(n)으로 느려져서, 사용할 수 없어진다.

### **좋은 `hashCode`를 작성하는 요령**

1. int 변수인 `result`를 선언한 후 값을 c로 초기화한다.
    - 이 때, c는 해당 객체의 첫번째 핵심 필드(`equals` 비교에 사용되는 필드)를 단계 2.1 방식으로 계산한 해시코드이다.
2. 해당 필드의 해시코드 c 를 계산한다.
    - 기본 타입 필드라면, `Type.hashCode(f)`를 수행한다. 여기서 *Type*은 해당 기본타입의 박싱 클래스다.
    - 참조 타입 필드면서, 이 클래스의 `equals` 메소드가 이 필드의 `equals`를 재귀적으로 호출하여 비교한다면, 이 필드의 `hashCode`를 재귀적으로 호출한다.
        - ex) getLottoMachine().getLottoTicket().getLotto().getLottoNumber()
    - 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다.모든 원소가 핵심 원소라면 `Arrays.hashCode`를 사용한다.
3.  c로 `result`를 갱신하면서 `result`를 반환한다.
    - `result` = 31 * `result` + c;

### 주의할 점

- `equals`비교에 사용되는 필드에 대해서만 해시코드를 계산한다.
- 성능을 높인다고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다.
- 만약 hash로 바꾸려는 필드가 기본 타입이 아니면 해당 필드의 hashCode를 불러 구현한다.
    - 계산이 복잡한 경우는 표준형을 만들어 구현한다.
- 참조 타입 필드가 null일 경우 0을 사용.
- 31을 곱하는 이유는 비슷한 필드가 여러개일 때 해시효과를 크게 높여주기 위함이다.
    - 비슷한 값들이 여러개 있을때 그냥 더하면 같은 값이 나올 확률이 높기 때문에 31을 곱해 큰수로 만들어 해시효과를 증가시킨다.

### 해시코드를 생성하는 비용이 크다면 LazyLoad와 Cache를 적용할 수 있다.

```java
private int hashCode; // primitive 기본값인 0으로 초기화

@Override
public int hashCode() {
  int result = hashCode;

  if(result == 0) {
    result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    hashCode = result;
  }

  return result;
}
```

- 해시코드 생성 비용이 크다면, 한번 필드에 값을 저장해놓고 재활용해도 된다.
    - 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해 둔다.
- LazyLoad도 되고 있어 불리지 않는 한 쓸모 없는 연산은 하지 않을 것이다.
    - 필드를 지연 초기화 하려면 그 클래스가 *thread-safe*가 되도록 초기 생성 시에는 싱크를 맞춰주어야 한다.

## 결론

- `equals`를 재정의할 때는 `hashCode`도 반드시 재정의해야 한다.
    - 프로그램이 제대로 동작하지 않을 수 있다.
- 재정의한 `hashCode`는 `Object`의 API 문서에 기술된 일반 규약을 따라야 하며, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다.
    - hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자 그래야 클라이언트는 이 값에 의지하지 않고, 계산방식을 바꿀 수도 있다.

직접 만들기 보다는 AutoValue 프레임워크 혹은 Intellij IDE 도움을 받아서 `equals`와 `hashCode`를 사용하는 것이 좋다.
