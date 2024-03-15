# clone 재정의는 주의해서 진행하라
## `Cloneable` 의 역활

- 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스
- 메서드가 하나도 없지만 Object 클래스의 clone() 메서드 동작 방식을 결정한다.

```java
public class Entry implements Cloneable{
    String key;
    String value;

    public Entry(String key, String value) {
        this.key = key;
        this.value = value;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

- `Cloneable`을 구현하지 않은 인스턴스에서 `clone()`을 호출하면 `CloneNotSupportedException` 가 발생하기 때문에 구현해줘야 한다.

## `Cloneable`의 문제점

- `Cloneable` 인터페이스의 `clone()` 을 구현하는게 아니라 `Object` 클래스에 `protected` 접근자로 된 `clone()` 메서드를 오버라이드 해야 한다.
- 자바의 기본 의도와 다르게 생성자를 호출하지 않고 객체를 생성할 수 있게 되어버린다.

## `clone()` 메서드의 일반 규약

`x.clone() != x` (true)

- 복사된 객체가 원본이랑 같은 주소를 가지면 안된다.

 `x.clone().getClass() == x.clone().getClass()` (true)

- 복사된 객체가 같은 클래스여야 한다는 뜻이다.

`x.clone().equals()` (true가 필수 X)

- 복사된 객체가 논리적 동치는 일치해야 한다는 뜻이다. (필수 X)

## `clone()` 메서드의 문제점

- super.clone()을 사용하지 않으면, 상속한 하위 클래스에서 super.clone()을 호출했을 때 엉뚱한 결과가 나올 수 있다.
- `final` 클래스라면 사용하지 않아도 괜찮다. (자기 밑으로 하위 클래스가 없기 때문)
    - `super.clone()`을 호출하지 않을거면 Cloneable을 구현할 필요가 없다.

## 가변 상태를 참조하지 않는 클래스용 clone() 메서드

- `가변객체`란, `instance` 생성 이후에도 내부 상태 변경이 가능한 객체를 말한다.
- super.clone을 호출한다.
    - 클래스에 정의된 모든 필드는 원본 필드와 똑같은 값을 가진다.
    - 모든 필드가 `primitive` 타입이거나 `final` 이라면 super.clone 메서드만 호출하면 된다.

```java
@Override
public PhoneNumber clone() {
  try {
    return (PhoneNumber) super.clone();
  } catch (CloneNotSupportedException e) {
    throw new AssertionError(); // 일어날 수 없는 일이다.
  }
}
```

- 자바가 공변 반환 타이핑을 지원하기 때문에 `PhoneNumber` 타입으로 형변환해서 반환하자.
    - 클라이언트가 형변환하지 않아서 사용하기 쉽다. (절대 실패하지 않음)
- try-catch 블록으로 감싸자
    - Object.clone() 메서드가 `CloneNotSupportedException`을 던지기 때문이다.
    - `CloneNotSupportedException`이 `unchecked` 예외를 던지면 사용하지 않아도 됬었다.

## 가변 상태를 참조하는 클래스용 clone() 메서드

```java
static class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if(size == 0) {
            throw new EmptyStackException();
        }

        Object result = elements[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

- `super.clone()`으로 복제하면, `primitive` 타입의 값은 올바르게 복제된다.

**clone된 Stack에서 pop 메서드를 호출하고 main에서 pop을 호출하면 null이 발생하게 된다.**

```java
public static void main(String[] args) throws CloneNotSupportedException {
        Stack stack = new Stack();
        stack.push("kk");
        Stack clone = stack.clone();
        clone.pop();
        System.out.println(stack.pop()); //null 발생
}
```

복제된 `Stack`의 `elements`는 원본 Stack 인스턴스와 똑같은 배열을 참조한기 때문

```java
class Stack {
		...  
    public Object pop() {
        if(size == 0) {
            throw new EmptyStackException();
        }

        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
}
```

- `Stack` 클래스의 유일한 생성자를 이용하면 이런 문제는 발생하지 않지만 값이 복사되지 않는 문제가 있다.

### 해결 방법

`elements` 배열에 `clone()` 메서드를 이용해 따로 복사해주면 된다.

```java
@Override
public Stack clone() {
  try {
  	Stack clonedStack = (Stack) super.clone();
    clonedStack.elements = this.elements.clone();
    return clonedStack;
  } catch (CloneNotSupportedException e) {
    throw new AssertionError(); // 일어날 수 없는 일이다.
  }
}
```

- 값도 모두 복사되고 서로 같은 `elements` 배열을 참조하지 않는다.
- `elements`가 `final`로 선언되어 있었다면, 앞의 방식은 작동하지 않는다.
    - `clonedStack.elements = this.elements.clone();` 이 구문을 실행할 수 없다.
    - **가변 객체를 참조하는 필드는 `final`로 선언하라**는 일반 용법과 충돌한다.

## 복잡한 가변 상태를 갖는 경우

`clone()` 메서드를 사용하면, 복제된 `HashTable`은 자신만의 `buckets`는 가지고 있지만 `buckets` 내부에 있는 객체들은 원본 객체들을 가리키고 있다.

```java
static class HashTable implements Cloneable {
    private Entry[] buckets;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }

    @Override
    public HashTable clone() {
	    try {
		    HashTable result = (HashTable) super.clone();
        result.buckets = buckets.clone();
        return result;
	    } catch (CloneNotSupportedException e) {
			  throw new AssertionError(); // 일어날 수 없는 일이다.
			}
    }
}
```

### 해결 방법

```java
static class HashTable implements Cloneable {
    private Entry[] buckets;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
				// 링크드리스트처럼 next로 연결된다
        Entry deepCopy() {
            Entry result = new Entry(key, value, next);
            for(Entry p = result; p.next != null; p = p.next) {
                p.next = new Entry(p.next.key, p.next.value, p.next.next);
            }
            return result;
        }
    }

    @Override
    public HashTable clone() throws CloneNotSupportedException {
        HashTable result = (HashTable) super.clone();
        result.buckets = new Entry[buckets.length];

        for (int i=0; i<buckets.length; i++) {
            if(buckets[i] != null) {
                result.buckets[i] = buckets[i].deepCopy();
            }
        }

        return result;
    }
}
```

`buckets`의 들어 있는 Entry 객체들은 계속 객체로 연결되어 있다.

- deepCopy 메서드를 통해서 객체를 연쇄적으로 계속 같은 값을 지닌 새로운 객체로 복사해주어야 한다.

### 재귀를 이용할 수도 있다.

```java
Entry deepCopy() {
	return new Entry(key,value,
			next == null ? null : next.deepCopy());
}
```

- 재귀 호출로 리스트의 원소 수만큼 스택 프레임을 소비하여, 리스트가 길면 스택 오버플로를 일으킬 위험이 있다.

**key-vale 쌍 각각에 대해 복제본 테이블의 put(key, value) 메서드를 호출해서 Entry를 넣어줄 수 있다.**

- 저수준의 방법을 활용할 때보다는 동작속도가 조금 느릴 것이다.

**생성자에서 `HashTable`을 받고 `put()` 메서드로 복사한다면, `put`은 `final`이거나 `private`이어야 한다.**

- 생성자에는 재정의될 수 있는 메서드를 호출하면 안되기 때문이다.

## `clone()` 메서드 주의사항

- 기본 타입이나 불변 객체 참조만 가지면 아무것도 수정할 필요 없지만 `일련번호` 혹은 `고유 ID`와 같은 값을 가지고 있다면, 비록 불변일지라도 새롭게 수정해주어야 한다.
- `Object.clone()`은 동기화를 신경쓰지 않은 메서드이다.
    - 동시성 문제가 발생할 수 있다.
- `clone()`을 막고 싶다면 `clone()` 메서드를 재정의하여, `CloneNotSupportedException()`을 던지도록 하자.
    
    ```java
    	@Override
    protected final Object clone() threows CloneNotSupportedException {
    	throw new CloneNotSupportedException();
    }
    ```
    

## 복사 생성자와 복사 팩터리

- 복사 생성자란 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다.
- 더 정확한 이름은 변환 생성자(conversion constructor)와 변환 팩터리(conversion factory)이다.

**복사 생성자**

```java
public Yum(Yum yum) { ... };
```

**복사 팩터리**

```java
public static Yum newInstance(Yum yum) { ... };
```

### clone() 보다 좋은 점

- 생성자를 쓰지 않는 생성방식을 쓰지 않는다.
- 정상적 `final` 필드 용법과 충돌하지 않는다.
- 불필요한 검사 예외를 던지지 않는다.
- 형변환도 필요 없다.
- `인터페이스` 타입의 `인스턴스`도 인수로 받을 수 있다.

## 결론

- 인터페이스를 만들 때는 절대 `Cloneable`을 확장해선 안된다.
    - `Cloneable`은 클래스의 믹스인(사용) 의도로 만들어진 것이다.
- `final` 클래스라면 `Cloneable`을 구현해도 하위 클래스에서 위험은 크지 않지만, 성능 최적화 관점에서 검토 후에 드물게 허용해야 한다.
- 복사 생성자와 복사 팩터리를 통해서 복사 기능을 사용하자
