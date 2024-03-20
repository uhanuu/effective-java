# 🚀 Item 14 Comparable을 구현할지 고려하라


우리는 equals 부터 시작해서 hashcode, toString, clone에 이어서 드디어 종착지인 Comparable까지 왔습니다.  이번장에서는  Comparable이 무엇이고  구현할 때 무엇을 고려해야하는지에 대해 알아보겠습니다.

<br>

## ❓ Comparable 

일단 comparable이 뭔지 알아야합니다.

`Comparable` 인터페이스는 Java의 객체 정렬을 위한 인터페이스 중 하나입니다. 이 인터페이스를 구현하면 클래스의 인스턴스들을 서로 비교할 수 있게 되며, 이를 통해 정렬과 같은 작업을 할 때 객체들을 자연스러운 순서로 배열할 수 있습니다. `Comparable` 인터페이스를 구현하는 클래스는 `compareTo(Object o)` 메서드를 오버라이드(재정의)해야 합니다.

`compareTo(Object o)` 메서드는 현재 객체와 매개변수로 전달된 객체를 비교하여, 현재 객체가 매개변수로 전달된 객체보다 작은 경우 음수를, 같은 경우 0을, 큰 경우 양수를 반환합니다. 이 값은 정렬 알고리즘에서 객체들의 순서를 결정하는 데 사용됩니다.

예를 들어, `Person` 클래스가 `age` 필드를 기준으로 정렬되길 원한다면, `Person` 클래스는 `Comparable` 인터페이스를 구현하고, `compareTo()` 메서드를 `age` 값에 따라 다른 `Person` 인스턴스와 비교하도록 정의해야 합니다.

```java
public class Person implements Comparable<Person> {
    private String name;
    private int age;

    // 생성자, 게터, 세터 생략

    @Override
    public int compareTo(Person other) {
        return this.age - other.age;
    }
}


```

이 예시에서, `compareTo()` 메서드는 현재 `Person` 객체의 `age`와 다른 `Person` 객체의 `age`를 비교합니다. 이 메서드의 반환 값은 `Collections.sort()`와 같은 정렬 메소드에서 `Person` 객체들을 나이에 따라 정렬하는 데 사용됩니다.

`Comparable` 인터페이스를 구현함으로써, 개발자는 자신의 클래스에 대한 자연스러운 정렬 순서를 정의할 수 있으며, 이는 해당 클래스의 인스턴스들을 배열, 리스트 등에서 쉽게 정렬할 수 있게 합니다. 이는 데이터를 처리하거나 사용자에게 정보를 표시할 때 매우 유용합니다.


## 🚜 compareble 인터페이스의 compareTo 메서드

`compareTo` 는 해당 객체와 전달된 객체의 순서를 비교합니다.

- `compareTo`는 `Object`의 `euqals`와 두가지 차이점이 있습니다. `compareTo`는 `equals`와 달리 단순 동치성에 더해 순서까지 비교할 수 있으며, 제네릭합니다.
    
- `Comparable`을 구현했다는 것은 그 클래스의 인스턴스에 자연적인 순서가 있음을 뜻합니다. 예를 들어, `Comparable`을 구현한 객체들의 배열은 `Arrays.sort(a)`로 쉽게 정렬이 가능합니다.



## 📌 compareTo 메서드의 일반 규약


1. 반사성 : 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다.

2. 추이성 : 첫 번째보다 두번 째 객체가 크고, 두 번째 객체가 세 번째 객체보다 크면, 세 번째 객체는 첫 번째 객체보다 커야한다.

3. 대칭성 : 크기가 같은 객체들 끼리는 어떤 객체와 비교하더라도 항상 같아야 한다.

이 세가지 규칙은 equals와 같고(item 10), 그렇기에 주의사항과 우회법이 같습니다.

주의사항은 기존 클래스를 확장한 구현체에서 새로운 값을 추가하면 compareTo 규약을 지킬 수 없다는 것이고, 우회법은 상속보다는 조합을 사용하고 조합된 필드에 대한 뷰 메서드(getter)를 활용하는 것입니다.

동치성 비교에서 equals와 동일한 결과를 반환하는 것도 중요합니다. TreeSet처럼 순서가 중요한 컬렉션에선 동치성을 비교할 때 equlas가 아닌 compareTo를 사용합니다.


## ❗️ 구현시 주의 사항

Comparable은 제네릭 인터페이스이므로 입력 인수의 타입을 확인하거나 형변환 하지 않아도 됩니다.

내부 필드 중 객체 참조 필드는 compareTo를 재귀적으로 호출하고, Comparable을 구현하지 않은 필드는 Comparator을 대신 활용합니다. Comparator는 직접 구현해도 되고 자바 라이브러리에서 재공하는 것을 사용해도 됩니다.

원시 타입인 경우 랩핑 클래스의 정적 메서드 compareTo를 이용하면 됩니다.


## 📌 CompareTo  고려할 점


- `Comparable`은 타입을 인수로 받는 제네릭 인터페이스이므로 `compareTo`의 인수타입은 컴파일 시에 정해지기 때문에 입력 인수 확인이나 형변환을 할 필요가 없습니다.
- `null`을 인수로 넣으면 `NullPointerException`을 던져야합니다.
- `compareTo`는 동치가 아닌 순서를 비교합니다.
- 객체 참조 필드를 비교하려면 `compareTo` 메서드를 재귀적으로 호출합니다.
- `Comparable`을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 `Comparator`을 대신 사용합니다.

#### ❗️ 주의

compareTo 메서드에서 관계연산자 (<, >)를 사용하지 않는것을 추천한다.

why?
박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드 `compare`를 대신 이용하면됩니다.
관계연산자(`<`,`>`)는 오류를 유발할 가능성이 있기때문에 추천하지 않는다.


## 📌 Comparator 인터페이스

자바 8에서 `Comparator` 인터페이스가 일련의 비교자 생성 메서드와 메서드 연쇄방식으로 비교자를 생성할 수 있게 되었습니다.


- 비교자들은 `compareTo` 메서드를 구현하는데 활용될 수 있습니다.
	- 이 방식은 간결하지만 약간의 성능 저하가 발생합니다.
	- 자바의 정적 임포트 기능을 활용하면 정적 비교자 생성 메서드들을 그 이름만으로 사용할 수 있어 코드가 훨씬 깔끔해집니다.
- `Comparator`는 `comparingInt`와 `thenComparingInt`등의 숫자용 기본 타입을 커버하는 보조 생성 메서드들을 가지고 있습니다.
- `comparing`와 `thenComparing`이란 객체 참조용 비교자 생성 메서드 또한 가지고 있습니다.

>비교 생성 메서드를 활용한 비교자
```java
private static final Comparator<PhoneNumber> COMPARATOR =
            Comparator.comparingInt((PhoneNumber phoneNumber) -> phoneNumber.areaCode)
                    .thenComparingInt(phoneNumber -> phoneNumber.prefix)
                    .thenComparingInt(phoneNumber -> phoneNumber.lineNum);

public int compareTo(PhoneNumber phoneNumber) {
	    return COMPARATOR.compare(this, phoneNumber);
}
```

- 위 코드는 비교자 생성 메서드 2개를 이용해 비교자를 생성합니다.
	- 첫 번째는 `comparingInt` 두 번째는 `thenCompaingInt`입니다.
- `comparingInt`는 객체 참조를 `int` 타입 키에 매핑하는 키 추출 함수를 인수로 받아 그 키를 기준으로 순서를 정하는 비교자를 반환하는 정적 메소드입니다.
- `thenComparingInt`는 `Comparator`의 인스턴스 메서드로, `int` 키 추출 함수를 입력받아 다시 비교자를 반환합니다. 
	- `thenComparingInt`는 연달아 호출이 가능합니다.
	- 자바의 타입 추론 능력으로 인해 호출할 때 타입을 명시하지 않습니다. 
