중첨 클래스(nested class)는 자신을 감싼 바깥 클래스에서만 쓰여야 한다.

- 그 외 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.

### 중첩 클래스 종류

- 정적 멤버 클래스
- (비정적) 멤버 클래스
- 익명 클래스
- 지역 클래스

**정적 멤버 클래스를 제외한 나머지는 내부 클래스(inner class)다.**

## 정적 멤버 클래스 (static → O)

- 다른 클래스 안에 선언되며 바깥 클래스의 private 멤버에도 접근할 수 있다는 점만 제외하면 일반 클래스와 동일하다.
- 다른 정적 멤버와 동일한 접근 규칙을 가지기 때문에 private으로 선언시 바깥 클래스에서만 접근이 가능하다.

```java
class Calculator {
    public static long plus(long left, long right) {
        return Operation.PLUS.operator.applyAsLong(left, right);
    }

    public static long minus(long left, long right) {
        return Operation.MINUS.operator.applyAsLong(left, right);
    }

		// 열거 타입도 암시적 static
    private enum Operation {
        PLUS((l, r) -> l + r),
        MINUS((l, r) -> l - r);
        private final LongBinaryOperator operator;

        Operation(LongBinaryOperator operator) {
            this.operator = operator;
        }
    }
```

- 정적 멤버 클래스와 비정적 멤버 클래스의 차이는 구문상 static이 있고 없고 차이다.
- 바깥 클래스가 표현하는 객체의 한 부분(구성요소)일 때 사용하면 된다.

## (비정적) 멤버 클래스 (static → X)

- 숨은 외부참조로 바깥 클래스의 인스턴스와 암묵적으로 연결된다.
    - `클래스명.this` 형태로 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다.
- 바깥 인스턴스 없이는 생성할 수 없다.

```java
class Outer {
    private String name = "현우";

    class Inner {
        public void printName() {
            System.out.println(Outer.this.name);
        }
        public void updateAndPrintName(String updateName) {
            Outer.this.updateName(updateName);
            System.out.println(Outer.this.name);
        }
    }

    private void updateName(String name) {
        this.name = name;
    }

    public static void main(String[] args) {
        Inner inner = new Outer().new Inner();
        inner.printName(); // 현우
        inner.updateAndPrintName("메롱"); // 메롱
    }
}
```

### 주의점!

**멤버 클래스에서 바깥 인스턴스에 접근할 일이 없으면 무조건 static을 붙여주자**

- 비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이의 관계는 멤버 클래스가 인스턴스화 될 때 확립되며, 더 이상 변경할 수 없다.
- 즉, static을 생략하면 바깥 인스턴스로의 숨은 참조가 생기면서 메모리 공간을 차지하며 생성 시간도 더 걸리게 된다.
    - 추가로 가비지 컬렉션이 바깥 클래스의 인스턴스를 수거하지 못할 수 있다.

### 언제 사용하면 좋을까?

- 비정적 멤버 클래스는 어댑터를 정의할 때 자주 쓰인다. 즉, 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용하는 것이다.
- Map 인터페이스의 구현체들은 보통 (`keySet()`, `entrySet()`, `values()` 메서드가 반환하는) 자신의 컬렉션 뷰 를 구현할 때 사용한다.
    
    ```java
      public class HashMap<K, V> extends AbstractMap<K, V>
            implements Map<K, V>, Cloneable, Serializable {
    
        final class EntrySet extends AbstractSet<Map.Entry<K, V>> {
    													...
        }
    
        final class KeySet extends AbstractSet<K> {
    													...
        }
    
        final class Values extends AbstractCollection<V> {
    													...
        }
    }
    ```
    

## 익명 클래스

- 익명 클래스는 바깥 클래스의 멤버가 아니다.
- 멤버와 달리 선언하는 시점에 인스턴스가 만들어진다.
- 코드의 어디서든 만들 수 있다.
- 비정적인 문맥에서 사용될때만 바깥 클래스의 인스턴스를 참조할 수 있다.
    
    ```java
    class OuterClass {
        private final String NAME = "현우";
    
        static void anonymousClass() {
            Anonymous anonymous = new Anonymous() {
                @Override
                public void hello() {
                    System.out.println(OuterClass.this.NAME); // 컴파일 에러
                }
            };
            anonymous.hello();
        }
    
        interface Anonymous{
            void hello();
        }
    }
    ```
    

### 단점

- 여러 인터페이스를 구현할 수 없고, 인터페이스를 구현하는 동시에 다른 클래스를 상속할 수도 없다.
- instanceof 검사나 클래스의 이름이 필요한 작업은 수행할 수 없다.
- 편리하지만 재사용이 안된다.
- 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없다.
    
    ```java
    interface Animal {
        void sound();
    }
    
    class Dog implements Animal {
        @Override
        public void sound() {
            System.out.println("멍멍");
        }
    
        public void bark() {
            System.out.println("개가 짖습니다.");
        }
    }
    
    class Main {
        public static void main(String[] args) {
            Animal animal = new Dog() {
                @Override
                public void sound() {
                    System.out.println("월월");
                }
            };
    
            animal.sound(); // 월월
            // 익명 클래스에서 Dog 클래스의 멤버인 bark() 메서드에 접근할 수 없음
            // animal.bark() 컴파일 에러
        }
    }
    ```
    
- 초기화된 `final` 기본 타입과 문자열 필드만 가질 수 있다.

### 언제 사용하면 좋을까?

- 즉석에서 작은 함수 객체나 처리 객체를 만드는 데 주로 사용되었지만 람다 등장 이후로 람다가 이 역할을 대체한다. (메서드 참조면 더 깔끔해짐)
- 익명 클래스는 정적 팩터리 메서드를 구현할때 자주 쓰이곤 한다.

```java
class OuterClass {
    private final String NAME = "현우";

    void anonymousClass(final String s) {
        Anonymous anonymous = () -> System.out.println(s); // 람다로 대체
        anonymous.hello();
    }

    interface Anonymous{
        void hello();
    }
}
```

- 인터페이스에 메서드가 여러개면 람다를 사용하지 못한다.

## 지역 클래스

- 네가지 중첩 클래스중 가장 드물게 사용된다.
- 지역 변수를 선언할 수 있는 곳이면 어디서든 선언이 가능하며 유효 범위도 지역변수와 같다.
- 지역 클래스가 작성된 메서드 내부에서 생성해서 사용이 가능하다.

```java
class OuterClass {
    public void print() {
        class LocalClass {
            private void print() {
                System.out.println("지역 클래스");
            }
        }
        new LocalClass().print();
    }
    
    public static void main(String[] args) {
        final OuterClass outerClass = new OuterClass();
        outerClass.print(); // 지역 클래스
    }
}
```

### 다른 중첩 클래스와 공통점

- 멤버 클래스처럼 이름이 있고 반복해서 사용이 가능하다.
- 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있다.
- 정적 멤버는 가질 수 없으며 가독성을 위해 짧게 작성해야 한다.

### 정리

- 중첩 클래스는 4가지 종류가 있다.
- 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기에 너무 길다면 `멤버 클래스`로 만들자.
- 멤버 클래스의 인스턴스가 바깥 인스턴스를 참조한다면 `비정적`으로 만들고 그렇지 않다면 `정적`으로 만들자.
- 중첩 클래스가 `한 메서드` 안에서 쓰이면서 `그 인스턴스를 생성하는 지점`이 `한 곳` 일 경우 (익명 vs 지역)
    - 해당 타입으로 쓰기에 적합한 `클래스나 인터페이스`가 이미 있다면 `익명 클래스`
    - 그렇지 않다면 `지역 클래스`로 만들자.
