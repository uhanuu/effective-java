앞에서는 다 재정의하라고 하는데 clone에서는 주의해라??  why?
하지만 그전에 clone이 대체 뭘까?


# ⭐️ clone

`clone()` 함수는 객체 지향 프로그래밍에서 객체의 복제를 생성하는 데 사용됩니다. 쉽게 말해, `clone()`을 사용하면 기존 객체의 정확한 복사본을 만들 수 있어요. 이 복사본은 원본 객체와 동일한 값을 가지지만, 메모리 상의 다른 위치에 저장됩니다. 따라서, 복제된 객체를 수정해도 원본 객체에는 영향을 주지 않습니다.


예를 들어,  우리가 어떤 책의 정보를 담고 있는 객체를 가지고 있고, 이 책의 정보를 기반으로 새로운 책의 정보를 만들고 싶지만 일부 정보만 변경하고 싶은 경우, `clone()`을 사용하여 원본 객체의 복사본을 만든 다음, 필요한 정보만 변경할 수 있습니다.

```java
// Java에서의 clone() 사용 예
class Book implements Cloneable {
    private String title;
    private String author;

    // 생성자, 게터, 세터 생략

    public Object clone()throws CloneNotSupportedException{
        return super.clone();
    }
}

public class Main {
    public static void main(String args[]) {
        try {
            Book originalBook = new Book("The Great Gatsby", "F. Scott Fitzgerald");
            Book copiedBook = (Book) originalBook.clone();
            
            // copiedBook은 originalBook의 복사본이지만, 다른 메모리 주소를 가진다.
            // 따라서 copiedBook을 변경해도 originalBook에는 영향을 주지 않는다.
        } catch(CloneNotSupportedException c){}    
    }
}

```

여기서 중요한 것은 `clone()`을 사용하려면 `Cloneable` 인터페이스를 구현해야 하며, 이는 객체가 안전하게 복제될 수 있음을 나타냅니다. `clone()` 메서드는 `CloneNotSupportedException`을 던질 수 있으므로, 이를 호출할 때는 이 예외를 처리해야 합니다.

단순히 모든 필드를 복사하는 "얕은 복사(shallow copy)"와는 달리, 객체 내의 다른 객체를 참조하는 필드가 있을 경우, 이 참조까지 복제하는 "깊은 복사(deep copy)"를 구현하기 위해서는 `clone()` 메서드를 커스터마이징해야 할 수도 있습니다. 이는 복제 과정에서 참조하는 객체들 또한 복제되어야 하기 때문입니다.

## ❓ Clonable

특이하게도 Clonable 자체에는 메서드가 없고, Cloneable을 구현하고 Object의 clone()메서드를 호출하면 값을 복사한 객체를, 그렇지 않으면 CloneNotSupportedException을 반환합니다.

>실제로 Clonable 자체에는 메서드가 없습니다.
![500](https://i.imgur.com/mMlgKVs.png)

<br>

# 📃 clone 메서드 일반 규약

그렇다면 clone 앞전에 배웠던  equals 나 hashCode 처럼 규약이 있을 텐데 무엇일까?


1. `x.clone() != x`  
    이 식은 참이다.
    
2. `x.clone().getClass() == x.getClass()`  
    이 식역시 참이다. 하지만 반드시 만족해야 하는 것은 아니다.
    
3. `x.clone().getClass().equals(x)`  
    이 식은 참이지만 필수는 아니다.
    
4. `x.clone().getClass() == x.getClass()`  
    만약, `super.clone()` 을 호출해 얻은 객체를 `clone()`이 반환한다면 이 식은 참이다. 또한, 관례상 **반환된 객체와 원본객체는 독립적**이어야 한다. 이를 만족하려면 `super.clone()`으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다


### 💎 clone 특징

- `clone`은 메서드가 `super.clone`이 아닌, 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일러는 불평하지 않는 점에서 생성자 연쇄와 비슷합니다.
- `clone`을 재정의한 클래스가 `final`이면 걱정할 하위 클래스가 없습니다. 만약 그렇지 않는다면 클래스의 하위 클래스에서 `super.clone`을 호출하면 잘못된 객체가 만들어져 하위 클래스의 `clone`이 잘 동작하지 않습니다.
- `clone` 메서드는 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장하는 생성자와 같은 효과를 냅니다.

<br>

# 🚀  clone 재정의 방법

그렇다면 clone 재정의는 어떻게 할까?


방법은 상황에 따라 달라집니다.  
1. 모든 필드가 기본 타입이거나 불편 객체를 참조하는 경우
2. 필드가 가변 타입일 경우 1
3. 필드가 가변 타입일 경우 2

<br>

## 1️⃣ 불편 객체를 참조하는 경우

1. `super.clone`을 호출합니다.
2. 모든 필드가 기본 타입이거나 불변 객체를 참조한다면 복사 끝

```java
public class Person implements Cloneable {
    String name;

    public Person(final String name) {
        this.name = name;
    }

    @Override
    public Person clone() throws CloneNotSupportedException {
        try {
            return (Person) super.clone();
        } catch (CloneNotSupportedException cloneNotSupportedException) {
            throw new AssertionError();
        }
    }
}
```


<br>

## 2️⃣ 필드가 가변 타입일 경우 1

1. `super.clone`을 호출한다.
2. 필드에 가변타입이 있다면 내부적으로 `clone`을 재귀적으로 호출하여 복사를 해줘야한다. (ex. 리스트)
    
    1. 필드의 가변타입을 그대로 복사할 경우 문제가 생기는 경우도 존재한다.(링크드 리스트의 node) 그런 경우는 깊은 복사를 지원하도록 한다. 이러한 방법은 재귀를 사용하면 리스트의 원소 수만큼 스택 프레임을 소비하여 스택 오버플로우를 일으킬 위험이 있어 반복자를 써서 순회하는 것이 좋다.

```java
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack(final Object[] elements) {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object obj) {
        ensureCapacity();
        elements[size++] = obj;
    }

    public Object pop() {
        if (size == 0) {
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

    @Override
    protected Stack clone() {
        try {
            return (Stack) super.clone();
        } catch (CloneNotSupportedException cloneNotSupportedException) {
            throw new AssertionError();
        }
    }
}
```

지금 이대로 clone을 하면 안된다.  지금 이대로 clone을 하면 Stack 인스턴스의 size 필드는 올바른 값을 갖겠지만, elements 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조할 것입니다. 이렇게 되면 개발자의 의도대로 코드가 동작하지 않을 수 있습니다.

Stack 클래스의 하나뿐인 생성자를 호출한다면 이러한 상황은 절대 일어나지 않습니다. clone 메서드는 사실상 생성자와 같은 효과를 냅니다. 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 합니다. 그래서 Stack의 clone 메서드는 제대로 동작하려면 스택 내부 정보를 복사해야 하는데, 가장 쉬운 방법은 elements 배열의 clone을 재귀적으로 호출해 주는 것 입니다.

```java
@Override
protected Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException cloneNotSupportedException) {
            throw new AssertionError();
        }
}
```

- `elements.clone()` 을 굳이 `Object[]` 로 형변환 할필요는 없다. `clone`은 런타임 타입과 컴파일타임 타입 모두가 원본 배열과 똑같은 배열을 반환한다. **배열을 복제할때는 `clone` 사용이 권장되는데 배열 복제는 `clone` 기능이 제대로 사용되는 유일한 예라 할 수 있다.**
    
- 하지만, `elements` 필드가 `final` 이었다면 앞서의 방식은 사용할 수 없다. 이는 **Cloneable 아키텍처는 "가변 객체를 참조하는 필드는 final로 선언하라" 는 일반 용법과 충돌**한다. 따라서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 `final`을 제거해야할 수도 있다.

<br>

## 3️⃣ 필드가 가변 타입일 경우 2

필드가 가변 타입일 경우 1번 방법으로 충분하지 않을 때도 있습니다. 즉, clone 메서드를 재귀적으로 호출하는 것만으로 충반하지 않을 때도 있습니다.

> 예시 코드
```java
public class HashTable implements Cloneable{
    private Entry[] buckets = new Entry[50];
    private int size = 0;

    public void put(Entry entry){
        buckets[size++] = entry;
    }

    public void printAll(){
        for (int i=0;i<size;i++){
            System.out.println(buckets[i].toString());
        }
    }

    static class Entry{
        final Object key;
        Object value;
        Entry next;

        public Entry(final Object key, final Object value, final Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        Entry deepCopy(){
            return new Entry(key,value, next==null ? null : next.deepCopy());
        }
    }

    @Override
    protected HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];

            for (int i =0; i < buckets.length; i++){
                if(buckets[i] != null){
                    result.buckets[i] = buckets[i].deepCopy();
                }
            }

            return result;
        }catch (CloneNotSupportedException cloneNotSupportedException){
            throw new AssertionError();
        }
    }
}
```

Stack에서처럼 단순히 버킷 배열의 clone을 재귀적으로 호출해본다면?

```java
    @Override
    protected HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];

            return result;
        }catch (CloneNotSupportedException cloneNotSupportedException){
            throw new AssertionError();
        }
    }
```

- 복제본은 자신만의 버킷 배열을 갖지만, 이 배열은 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 가능성이 생깁니다. ==이를 해결하려면 각 버킷을 구성하는 연결 리스트를 복사해야합니다.==
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
        
        Entry deepCopy() {
        	return new Entry(key,value,
        			next == null ? null : next.deepCopy());   
        }
    }

    @Override
    public HashTable clone() throws CloneNotSupportedException {
	    try{
		    HashTable result = (HashTable) super.clone();
	        result.buckets = new Entry[buckets.length];
	
	        for (int i=0; i<buckets.length; i++) {      
	            if(buckets[i] != null) {
	                result.buckets[i] = buckets[i].deepCopy();
	            }
	        }
	        return result;
		    
	    } catch (CloneNotSupportedException e) {
		    throw new AssertionError();
	    }
        
    }
	... 
}
```


하지만 연결 리스트를 복제하는 방법으로는 그다지 좋지 않습니다. 재귀 호출 때문에 리스트의 원소 수만큼 스택 프레임을 소비하여, 리스트가 길면 스택오버플로를 일으킬 위험이 있기 때문입니다. 이 문제를 피하려면 deepCopy를 재귀 호출 대신 ==반복자를 써서 순회하는 방향으로 수정해야합니다.==
```java
Entry deepCopy() {
            Entry result = new Entry(key, value, next);
            for (Entry p = result; p.next != null; p = p.next) {
                p.next = new Entry(p.next.key, p.next.value, p.next.next);
            }

            return result;
}
```

<br>

## 😡 매번 객체 복사를 위해 이러한 것들을 다 고려하면서 코드를 작성해야 할까?


`Cloneable`을 구현하는 모든 클래스는 `clone`을 재정의해야합니다.
이 때 접근제어자는 `public`으로, 반환 타입은 클래스 자신으로 변경합니다.

또한, `super.clone()`을 호출한 후 필요한 필드를 전부 적절히 수정합니다. 이 말은 그 객체의 내부 깊은 구조에 숨어있는 모든 가변 객체를 복사하고, 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 해야 함을 말합니다.

만약 기본 타입 필드와 불변 객체 참조만 갖는 클래스라면 아무 필드도 수정하지 않아도 됩니다. (단 일련번호나 고유 ID는 기본 타입이나 불변일지라도 수정해야 합니다.)

이미 `Cloneable`을 구현한 클래스를 확장했다면 어쩔 수 없이 `clone`을 잘 작동하도록 구현해야 합니다. 하지만, 그렇지 않은 상황이라면 **복사 생성자와 복사 팩터리로 더 나은 객체 복사 방식을 제공받을 수 있습니다.**


## 복사 생성자 / 복사 팩터리

>복사 생성자
```java
public Yum(Yum yum) {...};
```

>복사 팩터리
```java
	public static Yum newInstance(Yum yum) {...};
```

복사 생성자와 복사 팩터리는 `Cloneable/clone` 방식처럼 정상적인 `final` 필드 용법과 충돌하지 않으며, 불필요한 검사예외(`Exception`) 처리를 하지 않아도 되고 형변환도 필요하지 않으며 객체 생성 메커니즘(생성자를 쓰지 않는 방식)을 사용하지도 않습니다.

해당 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있어 이들을 이용하면 복제본 타입을 선택하는데 있어 유연성이 향상될 수 있습니다.

<br>

## 🚜 정리
- 인터페이스를 만들때 절대 `Cloneable`을 확장해서는 안됩니다. `Cloneable`은 믹스인 용도로 만들어졌습니다.
- `final` 클래스라면 성능 최적화 관점에서 검토후 문제가 없을때만 `Cloneable`을 구현합니다.
- 객체의 복제 기능은 `Cloneable/clone` 방식보다 **복사 팩터리와 복사 생성자**를 이용하는것이 가장 좋습니다. 단, 배열같은 경우는 `clone`방식을 가장 적합하므로 예외로 

### 🤔 여담

<img width="668" alt="image" src="https://github.com/Growth-Hub/Effective-Java/assets/76567238/941c18e7-5c11-4947-9ea7-f4606eadc916">


<br>
<br>

---

### 📚 Reference

- [Clonse()](https://www.geeksforgeeks.org/clone-method-in-java-2/)
- [# Java Object clone() Method - Cloning in Java](https://www.digitalocean.com/community/tutorials/java-clone-object-cloning-java)
