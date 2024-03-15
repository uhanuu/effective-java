# Comparable을 구현할지 고려하라

### Primitive 자료형은 비교 및 정렬이 가능하다.

```java
public static void main(String[] args) {
		int[] age = {11, 31, 52, 25, 27, 8, 76};
		System.out.println(11 < 31); // true
		Arrays.sort(age); // 오름차순 정렬
		System.out.println(Arrays.toString(age)); // [8, 11, 25, 27, 31, 52, 76]
}
```

객체를 정렬하기 위한 인터페이스로 `Comparable` 인터페이스와 `Comparator` 인터페이스 가 존재한다.

- 정렬하기 위한 인터페이스라고 말하기 보다는 **객체를 비교할 수 있는 인터페이스**라고 보는 것이 바람직하다.

`**Comparable`과 `Comparator` 차이점**

- 목적은 비교하는 것이지만 비교 대상이 다르다.
1. Comparable은 `compareTo(T o)`, Comparator의 `compare(T o1, T o2)`로 파라미터의 개수가 다르다.
    - Comparable은 자기 자신과 매개변수 객체를 비교하는 것이고, Comparator는 두 매개변수 객체를 비교하는 것이다.
2. Comparable은 lang 패키지에 있으므로 import를 하지 않아도 괜찮지만 Comparator는 util 패키지에 있으므로 import를 해줘야 한다.

### Reference 자료형은  `Comparable` 인터페이스와 `Comparator` 인터페이스를 이용하자

```java
class Point implements Comparable<Point> {
		int x, y;

		public Point(int x, int y) {
				this.x = x;
				this.y = y;
		}

		@Override
		public int compareTo(Point o) {
				return Integer.compare(this.x, o.x); // x를 기준으로 오름차순 정렬
		}
}
```

- 위에서 말한것 처럼 Comparable은 자기 자신과 파라미터로 들어오는 객체를 비교하고 있다.

`**java.lang.Integer**`

```java
public static int compare(int x, int y) {
        return (x < y) ? -1 : ((x == y) ? 0 : 1);
    }
```

- 표현식의 값이 `음수`, `0`, `양수`일 때 `-1`, `0`, `1`을 반환하도록 정의했다.
- 추가로 비교할 수 없는 타입이 주어지면 `ClassCastException`을 던지면 된다.

## CompareTo 메서드의 일반 규약은 equals와 비슷하다.

- `sgn` 은 수학에서 말하는 부호 함수를 뜻함

### 대칭성

두 객체참조의 순서를 바꿔 비교해도 예상한 결과가 나와야한다.

- `Comparable`을 구현한 클래스는 모든 x, y에 대하여 `sgn(x.compareTo(y)) == -sgn( y.compareTo(x))`여야 한다.
- `x.compareTo(y)`는 `y.compareTo(x)`가 예외를 던질때에 한해 예외를 던져야 한다.

### 추이성

Comparable을 구현한 클래스는 추이성을 보장해야 한다.

- 첫 번째가 두번째보다 크고 두 번째가 세번째보다 크면 첫번째는 세번째보다 커야한다.
- `(x.compareTo(y)>0 && y.compareTo(z) > 0)`이면 `x.compareTo(z) > 0`이다.

### 반사성

크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같다.

- Comparable을 구현한 클래스는 모든 z에 대해 `x.compareTo(y) == 0`이면 `sgn(x.compareTo(z)) == sgn(y.compareTo(z))`이다.

### equals (필수는 아니지만 지키는게 좋다.)

`compareTo` 메서드로 수행한 동치성테스트 결과가 `equals`와 같아야한다.

- `(x.compareTo(y) == 0) == (x.equals(y))`여야 한다. Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다.

**equals와 compareTo 관계**

```java
public static void main(String[] args) {
        BigDecimal bigDecimal1 = new BigDecimal("1.0");
        BigDecimal bigDecimal2 = new BigDecimal("1.00");
        Set<BigDecimal> hashSet = new HashSet<>();
        hashSet.add(bigDecimal1);
        hashSet.add(bigDecimal2);
        System.out.println(hashSet.size()); // 2

        Set<BigDecimal> treeSet = new TreeSet<>();
        treeSet.add(bigDecimal1);
        treeSet.add(bigDecimal2);
        System.out.println(treeSet.size()); // 1

        Set<Double> hashSetDouble = new HashSet<>();
        hashSetDouble.add(1.0);
        hashSetDouble.add(1.00);
        System.out.println(hashSetDouble.size()); // 1
}
```

- `HashSet`은 **equals를 기반**으로 비교를하기 때문에 다른 객체로 인식되어 크기가 2가 된다.
- `TreeSet`은 **compareTo를 기반**으로 동치성을 비교하기 때문에 `“1.0” == “1.00”` 같은 값으로 인식되어 compareTo가 0을 반환하기 때문에 크기가 1이 된다.

### 기본 타입 필드가 여럿일 때

```java
public int compareTo(PhoneNumber pn) {
		int result = Short.compare(areaCode, pn.areaCode);
		if (result == 0) {
				result = Short.compare(prefix, pn.prefix);
				if (result == 0) {
						result = Short.compare(lineNum, pn.lineNum);
		}
		return result;
}
```

- 자바 8에서는 Comparator 인터페이스를 활용할 수 있다.

### Comparator (가독성은 좋지만 약간의 성능 저하가 존재한다.)

- `comparingInt`와 `thenComparingInt` 등의 숫자용 기본 타입을 커버하는 보조 생성 메서드들을 갖고 있다. (thenComparingLong 등)

```java
private static final Comparator<Point> COMPARATOR =
			  Comparator.comparingInt((Point point) -> point.y) // 1. y를 기준으로 오름차순
					  .thenComparingInt(point -> point.x); // 2. x를 기준으로 오름차순
```

- comparingInt는 `int` 타입 키에 매핑하는 키 추출 함수를 인수로 받아 그 키를 기준으로 순서를 정하는 비교자를 반환하는 정적 메소드이다.
- `thenComparingInt`는 첫 번째 비교자를 적용한 후 새로운 키로 추가 비교하는 경우 수행한다. 여러 비교할 필드가 존재할 경우 계속해서 체이닝으로 사용가능하다.

### **해시코드 값의 차를 기준으로 Comparator를 사용하는 경우 주의점**

- compare, compareTo 목적은 같으므로 섞어서 사용하겠다.
- Integer.MAX, Integer.MIN 범위 밖을 넘어가는 경우

```java
public int compare(Point o1, Point o2) {
    return o1.hashCode() - o2.hashCode();
}
```

- **정수 오버플로우나 부동소수점 계산 방식에 따른 오류**를 발생할 수 있으므로 해당 방식으로 `-` 는 사용하면 안된다.

**해결 방법**

- 정적메서드를 활용하면 된다.

```java
@Override
public int compareTo(Point o) {
		return Integer.compare(this.hashCode(), o.hashCode());
}
```

- <, >로 비교하고 0,1,-1를 반환하기 때문에 따로 더하거나 빼지 않는다.
- 웬만한 값들은 정수형, 실수형에 맞춰서 정적 메서드를 사용하면 된다. (Double.compare, Float.compare등)

**정수형은 -, + 오버플로우를 조심하고 특히 실수형은 정적 메서드를 이용하자**

```java
// Double.compare
public static int compare(double d1, double d2) {
        if (d1 < d2)
            return -1;           // Neither val is NaN, thisVal is smaller
        if (d1 > d2)
            return 1;            // Neither val is NaN, thisVal is larger

        // Cannot use doubleToRawLongBits because of possibility of NaNs.
        long thisBits    = Double.doubleToLongBits(d1);
        long anotherBits = Double.doubleToLongBits(d2);

        return (thisBits == anotherBits ?  0 : // Values are equal
                (thisBits < anotherBits ? -1 : // (-0.0, 0.0) or (!NaN, NaN)
                 1));                          // (0.0, -0.0) or (NaN, !NaN)
    }
```

- 비트까지 체크해주는거 같다.

## 결론

순서를 고려해야 하는 객체를 다를 경우 `Comparable` 또는 `Comparator` 인터페이스를 구현하자.

- 필드의 값을 비교할 때 부등호가 아닌 래퍼 클래스에서 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.
