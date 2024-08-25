### float과 double이 사용하는 이진 부동소수점의 취약점

```java
public static void main(String[] args) {
    System.out.println(1.03 - 0.42); //0.6100000000000001
    System.out.println(1.00 - 9 * 0.10); //0.09999999999999998
    System.out.println(0.3 - 0.2); //0.09999999999999998
}
```

- `float` 과 `double` 은 이진 부동소수점 연산에 쓰이며 근사치로 계산하도록 설계 되었다.
- `float`와 `double` 타입은 특히 금융 관련 계산과는 맞지 않는다.
- `0.1` 을 2진수로 표현하면 `0.0001100,1100,...` 와 같이 순환소수로  비트 수의 한계까지 표현하다보면 위와 같은 결과가 나온다.

### 해결 방법

- `BigDecimal`, `int` 혹은 `long`을 사용해야 한다.

### `BigDecimal`

```java
public static void main(String[] args) {
    System.out.println(new BigDecimal("1.03").subtract(new BigDecimal("0.42"))); //0.61
    System.out.println(new BigDecimal("1.00").subtract(
        new BigDecimal("9").multiply(new BigDecimal("0.10")))); //0.10
    System.out.println(new BigDecimal("0.3").subtract(new BigDecimal("0.2"))); //0.1
  }
```

- BigDecimal 의 생성자 중 반드시 문자열로 숫자를 받는 생성자를 이용해야 한다. 파라미터로 넣으면서 오버플로우가 발생할 수 있기 때문이다.

### `int` 혹은 `long`

```java
public static void main(String[] args) {
    System.out.println((103 - 42) / 100d); //0.61
    System.out.println((100 - 9 * 10) / 100d); //0.1
    System.out.println((30 - 20) / 100d); //0.1
  }
```

- 정수로 곱한만큼 나눠줘야 한다.
- 자료형의 크기 때문에 다룰 수 있는 값의 크기가 제한되고 소수점을 직접 관리해야 한다.

### 핵심 정리

- 소수점 연산이 필요한 경우 정확한 답이 필요할 때는 `float` 과 `double` 을 사용하지 말고 성능저하가 있더라도 `BigDecimal` 을 사용하는 것이 정확하다.
- `BigDecimal`이 제공하는 여덟 가지 반올림 모드를 이용하여 반올림을 완벽히 제어할 수 있다.
- 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면 `int`나 `long`을 사용하자.
    - 숫자를 9자리 십진수로 표현할 수 있으면 `int` 18자리 십진수로 표현할 수 있다면 `long`을 사용하자.
    - 18자리가 넘어간다면 `BigDecimal` 을 사용해야 한다.
