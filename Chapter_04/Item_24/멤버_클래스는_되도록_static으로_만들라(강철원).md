# 🚀 Item 24 멤버 클래스는 되도록 static으로 만들라

이 책에서는 제일 처음 중첩 클래스 부터 언급하며 시작합니다.  

## ❓ 중첩클래스 

**중첩클래스**는 다른 클래스 내부에 선언된 클래스를 말합니다.  
중첩 클래스는 그들을 둘러싸고 있는 외부 클래스에 대한 깊은 연결을 가지며, 자바에서는 이러한 중첩 클래스를 사용하여 더욱 구조화되고 캡슐화된 코드를 작성할 수 있습니다. 
중첩 클래스는 크게 네 가지 유형으로 분류됩니다

- 정적 멤버 클래스
- 멤버  클래스
- 익명  클래스
- 지역  클래스


정적 멤버 클래스를 제외한 나머지는 전부 내부 클래스(inner class)입니다.

그럼 지금부터 여기에 있는 각각의 클래스에 대해서 자세 알아보겠습니다.

## 1️⃣ 정적 멤버 클래스(static member class)

다른 클래스 내부에 선언되지만, 해당 외부 클래스의 인스턴스에 독립적으로 존재할 수 있는 클래스입니다. 정적 멤버 클래스는 외부 클래스의 인스턴스 없이도 생성할 수 있으며, 주로 외부 클래스와 함께 사용될 때 의미가 있지만, 외부 클래스의 인스턴스 상태에 의존하지 않는 경우에 사용됩니다.



>ex) 자동차와 엔진
```java
public class Car {
    private static int carCount = 0;
    private String model;
    private int year;

    // Car 클래스의 생성자
    public Car(String model, int year) {
        this.model = model;
        this.year = year;
        carCount++;  // 자동차 인스턴스 생성 시, 총 자동차 수를 증가
    }

    // Car 인스턴스의 정보를 반환하는 메서드
    public void displayInfo() {
        System.out.println(year + "년형 " + model + " 자동차");
    }

    // 정적 멤버 클래스 Engine 정의
    public static class Engine {
        private String engineType;

        // Engine 클래스의 생성자
        public Engine(String engineType) {
            this.engineType = engineType;
        }

        // 엔진 정보를 출력하는 메서드
        public void displayType() {
            System.out.println("엔진 유형: " + engineType);
        }
    }

    // 자동차 개수를 반환하는 정적 메서드
    public static int getCarCount() {
        return carCount;
    }
}

public class Main {
    public static void main(String[] args) {
        // Car 인스턴스 생성
        Car myCar = new Car("소나타", 2020);
        myCar.displayInfo();

        // Car.Engine 인스턴스 생성
        Car.Engine myEngine = new Car.Engine("V6");
        myEngine.displayType();

        // 자동차 총 개수 출력
        System.out.println("총 자동차 수: " + Car.getCarCount());
    }
}

```

위 예제에서 `Car` 클래스는 자동차를 나타내며, `Engine` 클래스는 엔진을 나타냅니다. `Engine`은 `Car`의 정적 멤버 클래스로서, 엔진은 자동차와 밀접한 연관이 있지만 각 엔진 인스턴스는 특정 `Car` 인스턴스에 종속되지 않습니다. 따라서 `Car.Engine`의 형태로 인스턴스를 생성할 수 있으며, 이를 통해 엔진의 유형을 관리합니다.

이렇게 정적 멤버 클래스를 사용함으로써, 관련된 클래스들을 논리적으로 그룹화하여 코드의 구조를 명확하게 할 수 있고, 각 클래스의 역할을 분명히 할 수 있습니다.

#### ❓ 언제 사용하는 게 좋을까?

멤버 클래스에서 바깥 인스턴스에 접근할 일이 없으면 무조건 static을 붙여서 정적 멤버 클래스로 만들자
- why?
	- static을 생략하면 바깥 인스턴스로의 숨은 참조가 생기고, 참조를 저장하기 위해 시간과 공간이 낭비됩니다.
	- 가비지 컬렉션이 바깥 클래스의 인스턴스를 수거하지 못합니다.

## 2️⃣ 비정적 멤버 클래스(non-static member class)

비정적 멤버 클래스는 외부 클래스의 인스턴스에 속한 인스턴스를 가지며, **외부 클래스의 인스턴스 필드와 메서드에 직접 접근할 수 있는 특성을 가집니다**. 이러한 특성 때문에, 비정적 멤버 클래스는 외부 클래스의 인스턴스와 깊은 연결을 가지며, 특정 인스턴스의 상태를 관리하거나 조작하는 데 사용될 수 있습니다.

>예시 : 도서관과 도서
```java
public class Library {
    private String libraryName;

    // Library 클래스의 생성자
    public Library(String libraryName) {
        this.libraryName = libraryName;
    }

    // 도서 정보를 출력하는 메서드
    public void displayLibraryInfo() {
        System.out.println("도서관 이름: " + libraryName);
    }

    // 비정적 멤버 클래스 Book 정의
    public class Book {
        private String bookTitle;
        private String author;

        // Book 클래스의 생성자
        public Book(String bookTitle, String author) {
            this.bookTitle = bookTitle;
            this.author = author;
        }

        // 도서 정보를 출력하는 메서드
        public void displayBookInfo() {
            System.out.println("제목: " + bookTitle + ", 저자: " + author + ", 소속 도서관: " + libraryName);
        }
    }

    public static void main(String[] args) {
        // Library 인스턴스 생성
        Library myLibrary = new Library("중앙 도서관");

        // Library 인스턴스를 통해 Book 인스턴스 생성
        Book myBook = myLibrary.new Book("이펙티브 자바", "조슈아 블로크");
        myBook.displayBookInfo();  // 도서 정보 출력
    }
}


```

이 예제에서 `Library` 클래스는 도서관을 나타내며, `Book` 비정적 멤버 클래스는 도서관 내의 도서를 나타냅니다. `Book` 클래스는 `Library`의 인스턴스에 속하므로, 각 `Book` 인스턴스는 특정 `Library` 인스턴스와 연결됩니다. 이 연결을 통해 `Book` 클래스는 외부 클래스인 `Library`의 `libraryName` 필드에 직접 접근할 수 있습니다.

비정적 멤버 클래스를 사용함으로써, 특정 외부 클래스의 인스턴스와 깊은 관계를 가지는 내부 클래스를 정의할 수 있습니다. 이를 통해 외부 클래스의 인스턴스별로 상태를 가지는 내부 클래스의 인스턴스를 관리하고, 외부 클래스와 내부 클래스 간의 캡슐화를 유지할 수 있습니다.


#### ❓ 언제 사용하는 게 좋을까?

어댑터를 정의할 때 자주 사용됩니다. 
즉, 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용할 수 있습니다.

```
❓어댑터 패턴
어댑터 패턴은 한 클래스의 인터페이스를 클라이언트에서 기대하는 다른 인터페이스로 변환해주는 디자인 패턴입니다. 이를 통해 호환성이 없는 인터페이스들이 함께 작동할 수 있게 해줍니다. 비정적 멤버 클래스를 사용하는 이유 중 하나는, 이들이 외부 클래스의 인스턴스와 연결될 수 있으므로, 외부 클래스의 상태에 접근하고 조작할 수 있다는 점입니다. 이러한 특성은 어댑터 패턴을 구현할 때 매우 유용하게 활용될 수 있습니다.
```


## 3️⃣ 익명 클래스

익명 클래스(Anonymous Class)는 이름이 없는 내부 클래스로, 주로 한 번만 사용되는 클래스를 신속하게 생성하고 인스턴스화하기 위해 사용됩니다. 익명 클래스는 인터페이스나 추상 클래스의 구현체를 즉석에서 제공하거나, 기존 클래스의 확장을 바로 정의할 때 유용합니다. 이러한 클래스는 이벤트 리스너나 작은 콜백 객체를 정의하는 데 종종 사용됩니다.

익명 클래스는 선언과 동시에 인스턴스화되며, `new` 연산자 다음에 인터페이스 이름이나 클래스 이름을 명시하고 바로 중괄호 `{}`를 사용해 클래스 본문을 정의합니다.

>예시) 버튼 클릭 이벤트 리스너
```java
import javax.swing.JButton;
import javax.swing.JFrame;
import java.awt.event.ActionListener;
import java.awt.event.ActionEvent;

public class ButtonClickListener {
    public static void main(String[] args) {
        JFrame frame = new JFrame("익명 클래스 예제");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(300, 200);

        JButton button = new JButton("클릭");
        // 버튼 클릭 이벤트를 처리하기 위한 ActionListener 익명 클래스 정의
        button.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("버튼이 클릭되었습니다!");
            }
        });

        frame.add(button);
        frame.setVisible(true);
    }
}

```

이 코드에서는 `JButton`에 `ActionListener`를 추가하고 있습니다. `ActionListener`의 `actionPerformed` 메서드를 오버라이드하는 익명 클래스를 정의하여, 버튼 클릭 이벤트가 발생했을 때 수행할 작업을 구현합니다. 여기서 익명 클래스는 `new ActionListener()` 구문 바로 뒤에 중괄호 `{}`를 사용하여 정의됩니다.

익명 클래스를 사용함으로써, 별도의 `ActionListener` 구현 클래스를 작성하지 않고도 이벤트 처리 로직을 간단히 추가할 수 있습니다. 이러한 방식은 코드를 더 간결하게 만들고, 단일 사용 목적의 구현체를 빠르게 생성할 수 있게 해줍니다. 익명 클래스는 주로 간단한 인터페이스의 구현이나 작은 콜백 메서드를 정의할 때 유용하게 사용됩니다.

- 단점 
	- 선언한 지점에서만 인스터를 만들 수 있다.
	- instanceof 검사나 클래스의 이름이 필요한 작업은 수행할 수 없다.
	- 인터페이스를 구현하는 동시에 다른 클래스를 상속할 수 없다.
	- 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없다.
	- 익명 클래스는 표현식 중간에 등작하므로 (10줄 이하로) 짧지 않으면 가독성이 떨어진다.

#### ❓ 언제 사용하는 게 좋을까?

주로 즉석에서 작은 함수 객체나 처리 객체를 만드는 데 사용하면 좋습니다.  
>but 람다 등장 이후로 람다가 이 역할을 대체하였습니다.
```java
List<Integer> list = Arrays.asList(10, 5, 6, 7, 1, 3, 4);

// 익명 클래스 사용
Collections.sort(list, new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return Integer.compare(o1, o2);
    }
});

// 람다 도입 후
Collections.sort(list, Comparator.comparingInt(o -> o));
```


## 4️⃣ 지역 클래스(Local Class)

지역 클래스(Local Class)는 메서드 내부에 정의되는 클래스로, 선언된 메서드 내에서만 사용할 수 있습니다. 지역 클래스는 특정 메서드의 실행 도중에만 필요한 동작을 구현할 때 유용하게 사용될 수 있습니다. 이러한 클래스는 해당 메서드의 지역 변수처럼 취급되며, 메서드의 실행이 끝나면 이 클래스의 유효 범위도 종료됩니다.

지역 클래스는 일반적으로 해당 메서드에서만 의미가 있는 클래스를 정의할 때 사용됩니다. 이는 메서드 내부의 로직을 보다 명확하게 표현하고, 클래스를 외부로 노출시키지 않으면서도 특정 기능을 수행하는 데 필요한 클래스를 정의할 수 있게 합니다.

>예시) 유효성 검사를 수행하는 지역 클래스
```java
public class InputValidator {
    
    // 사용자 입력 유효성 검사를 수행하는 메서드
    public static void validateInput(String input) {
        // 지역 클래스 Validator 정의
        class Validator {
            private String data;

            Validator(String data) {
                this.data = data;
            }

            // 입력 데이터의 유효성 검사를 수행하는 메서드
            boolean isValid() {
	            // 예제 조건: 최소 5글자 이상이어야 함
                return data != null && data.length() >= 5; 
            }
        }

        // 지역 클래스의 인스턴스 생성 및 사용
        Validator validator = new Validator(input);
        if (validator.isValid()) {
            System.out.println("입력이 유효합니다: " + input);
        } else {
            System.out.println("입력이 유효하지 않습니다: " + input);
        }
    }

    public static void main(String[] args) {
        validateInput("Hello");  // 유효한 입력
        validateInput("Hi");     // 유효하지 않은 입력
    }
}

```

이 예제에서 `Validator` 클래스는 `validateInput` 메서드 내에서만 정의되고 사용됩니다. 이 지역 클래스는 입력된 문자열의 유효성을 검사하는 `isValid` 메서드를 포함하고 있습니다. `validateInput` 메서드는 사용자로부터 입력받은 문자열을 `Validator` 인스턴스에 전달하고, 유효성 검사 결과에 따라 적절한 메시지를 출력합니다.

지역 클래스를 사용함으로써, `Validator`의 구현이 `validateInput` 메서드와 긴밀하게 연결되어 있음을 명확하게 할 수 있으며, `Validator`의 사용 범위를 `validateInput` 메서드로 한정짓습니다. 이러한 접근은 클래스의 사용 범위를 제한하고, 코드의 가독성을 높이는 데 도움이 됩니다.


### ❗️ 주의

비정적 멤버 클래스와 지역 클래스를 헷갈리면 안 된다.
비정적 멤버 클래스와 지역 클래스 모두 특정 범위 내에서만 유효한 클래스를 정의할 수 있는 방법을 제공합니다. 그러나 `비정적 멤버 클래스`는 **외부 클래스와의 밀접한 관계를 통해 인스턴스 멤버에 대한 접근을 필요로 할 때** 주로 사용되며, `지역 클래스`는 **특정 메서드 내에서만 필요한 복잡한 동작을 구현하는 데** 사용됩니다.



## 정리

```java

if(메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기에 너무 길다면){

	if(멤버 클래스의 인스턴스가 바깥 인스턴스를 참조하면) {
		비정적 멤버 클래스
	} else if(멤버 클래스의 인스턴스가 바깥 인스턴스를 참조안하면) {
		정적 멤버 클래스
	}
}

if(중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한곳이고 해당 타입으로 쓰기에 적잡한 클래스나 인터페이스가 있다면) {
	익명 클래스
}else{
	지역 클래스
}
```

