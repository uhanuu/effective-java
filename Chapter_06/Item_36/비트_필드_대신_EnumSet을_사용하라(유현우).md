열거한 값들이 주로 집합으로 사용될 경우 과거에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해왔다.

- 비트 연산을 하는 가장 큰 이유는 연산속도다.  bit는 최소정보단위로 다른 추상화된 정보들과 다르게 즉시 해석되어 전달되므로 연산 속도가 매우 빠르다는 장점이 있다. (유지보수가 힘들다)

## **비트 필드 열거 상수**

- 합집합 교집합 같은 집합 연산을 효율적으로 수행할 수 있다.

```java
public static final int STYLE_BOLD = 1 << 0; // 1
public static final int STYLE_ITALIC = 1 << 1; // 2
public static final int STYLE_UNDERLINE = 1 << 2; // 4
public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

public String applyStyles(int styles) {
    StringJoiner sj = new StringJoiner(", ");

    if((styles & STYLE_BOLD) == 1) {
        sj.add("BOLD");
    }

    if((styles & STYLE_ITALIC) == 2) {
        sj.add("ITALIC");
    }

    if((styles & STYLE_UNDERLINE) == 4) {
        sj.add("UNDERLINE");
    }

    if((styles & STYLE_STRIKETHROUGH) == 8) {
        sj.add("STRIKETHROUGH");
    }

    return sj.toString();
}

public static void main(String[] args) {
    String text = applyStyles(STYLE_BOLD | STYLE_ITALIC);
    System.out.println(text); // BOLD, ITALIC
}
```

### 단점

- 정수 열거 상수의 단점을 그대로 가져간다.
- 단순한 정수 열거 상수를 출력할 때보다 해석하기 어렵다. (유지보수 어려움)
- 모든 원소를 순회하기 까다롭다.
- 최대 몇비트가 필요한지 API 작성 시 미리 예측하여 적절한 타입을 선택해야 한다.
    - API를 수정하지 않고는 비트 수(32bit, 64bit)을 늘릴 수 없기 때문

## EnumSet으로 비트 필드를 대체하자

### EnumSet

원소가 총 64개 이하(대부분)라면 RegularEnumSet을 사용하여 비트 필드에 비견되는 성능을 보여주며 그 이상은 JumboEnumSet을 사용하도록 내부적으로 판단해서 크기에도 유연하다.
![image](https://github.com/uhanuu/effective-java/assets/110734817/36d87015-56cf-4b9f-a727-af8d703e7958)
- 비트를 직접 다룰 때 난해한 작업을 EnumSet이 다 처리해주기 때문에 흔히 겪는 오류들에서 해방된다.

### 코드 대체하기

```java
enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

public static String applyStyles(Set<Style> styles) {
  return styles.stream()
      .map(Style::name)
      .collect(Collectors.joining(", "));
}

public static void main(String[] args) {
  System.out.println(applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC)));
}
```

- 이전 비트 코드랑 비교했을 때 코드가 짧고 깔끔하고 안전하다
- 원소를 순회하기 쉽다.

### 정리

열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다. 

- EnumSet이 비트 필드 수준의 명료함과 성능을 제공하며 열거 타입의 장점도 얻을 수 있기 때문이다.

불변을 원한다면 성능이 조금 희생되지만 Collections.unmodifiableSet으로 EnumSet을 감싸 사용할 수 있다.
