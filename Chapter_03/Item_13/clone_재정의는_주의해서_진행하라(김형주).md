## 🎯 clone()과 Cloneable
### clone()
- 번역 그대로 복제를 위한 메소드로, 해당 인스턴스를 복제하여 새로운 인스턴스를 생성해 그 참조값을 반환한다.

### Cloneable
- 인터페이스로, clone()메서드를 사용할 객체는 Cloneable 인터페이스로 구현해줘야 한다.
- 즉, Cloneable 인터페이스로 객체가 구현되어 있어야 clone()를 통해서 복제할 수 있다.
- 구현되어 있지 않은 객체에 clone()를 사용하면 ```CloneNotSupportedException```을 발생시킨다.

>#### 🤔 그렇다면 Cloneable 인터페이스 안에 clone()이 구현되어 있는건가요? ➡️ 🙅‍♂️❌
>
>![image](https://github.com/kim0527/Effective-Java/assets/143387515/3486c552-970d-47db-8365-4830f518508f)
>![image](https://github.com/kim0527/Effective-Java/assets/143387515/db09c1e9-8e85-43c4-b795-942a4f5dc4f8)
>
- Cloneable 인터페이스는 메서드가 없는 인터페이스 이다.
- Cloneable 인터페이스는 ```단순히 복제해도 되는 클래스임을 명시하는 용도```이다.
- clone()은 Object의 native의 메서드로 되어 있다. (아래 링크 참고)
    - [native 코드로 정의된 clone()](https://github.com/openjdk/jdk/blob/3f41fdecdb6d131a5afe6e0a39d7414c222fe4fb/src/hotspot/share/prims/jvm.cpp#L636)
- 게다가 protected되어 있어 Cloneable을 구현하는 것만으로는 사용이 불가능기 때문에 ```@Overide```가 재정의가 필요하다.

## ✅ Clone()의 일반 규약
아래 규약은 '복사'라는 행위의 일반적 의도이며, **항상 이 요구를 반드시 만족해야 하는 것은 아니다.**
1. ```x.clone() != x``` 는 True이다. 
    - 복사한 인스턴스는 원본 객체와 서로 다른 참조 값을 가져야 한다.
2. ```x.clone().getClass() == x.getClass()``` 는 True이다.
    - 반환한 인스턴스는 같은 클래스를 가지고 있어야 한다.
3. ```x.clone().equals(x)```는 True이다.
    - 원본 객체와 복사한 객체는 논리적으로 동치해야 한다.

clone 메서드의 일반 규약은 위와 같이 강제성이 없기 때문에 허술하다.

## 📚 Clone() 재정의 방법
### 1️⃣ 잘못된 재정의 방식
```java
public class CloneUseNewMethod implements Cloneable{
    private int number;

    public int getNumber() {
        return number;
    }

    public CloneUseNewMethod(int number) {
        this.number = number;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        CloneUseNewMethod that = (CloneUseNewMethod) o;
        return number == that.number;
    }
    
    @Override
    protected Object clone() throws CloneNotSupportedException {
        CloneUseNewMethod result = new CloneUseNewMethod(this.getNumber());  //new 연산자를 이용한 복사
        return result;
    }
}

public static void main(String[] args) throws CloneNotSupportedException {
    //TEST 1
    CloneUseNewMethod A = new CloneUseNewMethod(10);

    System.out.println("A.clone() != x");       //일반 규약 1번
    System.out.println( A.clone() != A);        //True 
    System.out.println("A.clone().getClass() == A.getClass()");        //일반 규약 2번
    System.out.println( A.clone().getClass() == A.getClass());         //True
    System.out.println("A.clone().equals(A)");          //일반 규약 3번
    System.out.println(A.clone().equals(A));            //True
}
```
- 규약에 따라서 재정의해보면 ```new연산자```로 새로운 인스턴스를 만들어 복제하는 것도 가능하다.
- 하지만 이 방식은 상속했을때 문제가 발생한다.
- 하위 객체에서 상위 객체가 정의한 clone() 사용시, 상위 객체의 인스턴스를 만들어 반환하기 때문에 복제의 역할을 수행하지 않는다.

그래서 일반 규약에서도 관례를 제공하고 있다.
> 관례상, 이 메서드가 반환하는 객체는```super.clone```을 호출해 얻어야 한다.
> <br>관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 ```super.clone```으로 얻은 객체의 필드 중 하나 이상을 반환전에 수정해야 할 수 도 있다.

#### 이제 위 일반 규약과 관례를 가지고 복하려는 클래스 종류에 따른 올바른 재정의 방식을 알아보자.

### 2️⃣ 올바른 재정의 방식

#### Ⅰ. 모든 필드가 기본 타입이거나 불변 객체의 경우
```java
public class PhoneNumber implements Cloneable{
    private int first, middle, end;

    public PhoneNumber(int first, int middle, int end) {
        (생략...)
    }

    @Override
    public boolean equals(Object o) {
       (생략...)
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        try{
            return (PhoneNumber) super.clone();   //공변 반환 타이핑
        }catch(CloneNotSupportedException e){
            throw new AssertionError();
        }
    }
}
```
- 공변 반환 타이핑을 통해 Object의 하위 객체인 PhoneNumber를 반환한다.
> 공변 반환 타이핑(covariant return typing)이란?<br>
부모 클래스의 메소드를 오버라이딩하는 경우, 부모 클래스의 반환 타입은 자식 클래스의 타입으로 변경해주는 것

#### Ⅱ. 필드에서 가변 상태를 참조하는 경우

#### 🤔 이 경우 방법 Ⅰ과 동일하게 clone()을 재정의하면 어떻게 될까?
```java
public class PhoneNumberNote implements Cloneable{
    private ArrayList<String> note = new ArrayList<String>();

    public void addNote(String number) {
        this.note.add(number);
    }

    public ArrayList<String> getNote() {
        return note;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        try{
            return (PhoneNumberNote) super.clone();    //방법 Ⅰ과 동일하게 재정의
        }catch(CloneNotSupportedException e){
            throw new AssertionError();
        }
    }
}

public static void main(String[] args) throws CloneNotSupportedException {
    PhoneNumberNote note1 = new PhoneNumberNote();
    note1.addNote("02-123-456");

    PhoneNumberNote note2 = note1.clone();
    System.out.println("Note 1: "+note1.getNote());
    System.out.println("Note 2(clone): "+note2.getNote());

    System.out.println("");
    note1.addNote("010-1234-5678");   //변동사항 발생
    System.out.println("Note1에 전화번호 추가 발생 후");

    System.out.println("Note 1: "+note1.getNote());
    System.out.println("Note 2(clone): "+note2.getNote());
}
```
```
[실행 결과]
    Note 1: [02-123-456]
    Note 2(clone): [02-123-456]
    
    Note1에 전화번호 추가 발생 후
    Note 1: [02-123-456, 010-1234-5678]       
    Note 2(clone): [02-123-456, 010-1234-5678]        // Note2에 넣지 않았음에도 같이 넣어짐.
```
- 동일하게 재정의한 결과, Note1에만 넣었던 전화번호가 Note2에도 들어가 있는 것을 확인할 수 있다.

#### 🤷‍♂️ 왜 이런 문제가 발생했을까?
```java
//객체의 참조값
System.out.println(System.identityHashCode(note1));   //821270929
System.out.println(System.identityHashCode(note2));   //1160460865
//필드의 참조값
System.out.println(System.identityHashCode((note1.getNote())));  // 821270929
System.out.println(System.identityHashCode((note2.getNote())));  // 821270929
```
- 원시타입과 달리 참조타입의 경우 객체의 대한 복제(super.clone)만 수행시, **필드의 참조는 원본 필드와 동일한 참조를 가지게 된다.**

#### 🎁 해결방법
```java
public class PhoneNumberNote implements Cloneable{
    private ArrayList<String> note = new ArrayList<String>();

    @Override
    protected Object clone() throws CloneNotSupportedException {
        try{
            PhoneNumberNote result = (PhoneNumberNote) super.clone(); // 객체 clone()
            result.note = (ArrayList<String>) note.clone();     //필드에도 clone()
            return result;
        }catch(CloneNotSupportedException e){
            throw new AssertionError();
        }
    }
}
```
```
[실행결과]
    Note 1: [02-123-456]
    Note 2(clone): [02-123-456]
    
    Note1에 전화번호 추가 발생 후
    Note 1: [02-123-456, 010-1234-5678]
    Note 2(clone): [02-123-456]                 //Note2에는 추가가 안된 모습!!
```
- clone()를 재정의할때 필드도 같이 clone() 수행해주면, 필드도 서로 다른 참조값을 가지게 된다.

#### Ⅲ. 복잡한 가변 상태를 객체의 경우
#### 이번에는 객체가 list 혹은 Array와 같은 collection에 들어가있는 경우는 어떻게 될까?
- 간단하다. collection에 들어갈 객체의 필드까지 복제해주면 된다.
- 책 속 예제에서은 깊은 복사(deepcopy)를 통해 객체의 필드까지 복사했다.

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
        			next == null ? null : next.deepCopy());    //깊은 복사 메서드 (재귀)
        }
    }

    @Override
    public HashTable clone() throws CloneNotSupportedException {
        HashTable result = (HashTable) super.clone();
        result.buckets = new Entry[buckets.length];

        for (int i=0; i<buckets.length; i++) {       //반복문을 통해 배열 속 모든 객체들 깊은 복사 수행
            if(buckets[i] != null) {
                result.buckets[i] = buckets[i].deepCopy();
            }
        }
        return result;
    }
}
```
- 배열 안에 들아갈 객체 Entry에 재귀를 이용한 깊은 복사(deepcopy) 메서드를 정의했다.
- 그리고 clone()에서 원본 배열 bucket의 길이만큼 새로운 배열을 만들고, 반복문을 통해 원본 배열 속 모든 Entry객체를 deepcopy 했다.

#### ➕  다른 깊은 복사 구현 방법
```java
//깊은 복사 메서드 (재귀)
Entry deepCopy() {
	return new Entry(key,value,
			next == null ? null : next.deepCopy());   
}
//깊은 복사 메서드 (반복)
Entry deepCopy() {
    Entry result = new Entry(key,value,next);
    for (Entry p = result; p.next ! = null; p = p.next){
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    }
    return result
	return new Entry(key,value,
			next == null ? null : next.deepCopy());    //깊은 복사 메서드 (재귀)
}       
```
- 재귀를 이용한 방법도 괜찮지만, 배열이 길어졌을 경우 ```재귀호출로 인해 스택 오버플로우```가 발생할 위험이 있다.
- 그래서 위와 같이 반복문을 통해서도 동일한 동작을 구현할 수 있다.
> #### 🔄 recursive call stack overflow error <br>
> 아래 그림처럼 메서드가 실행될때마다 JVM의 호출스택이 쌓이게 된다. 이때 재귀로 인해 메서드가 처리되지 않고 계속 호출된다면, 계속 메서드가 쌓이게 될 것이고 스택이 가득차 stack overflow가 발생한다. 이때 발생하는 에러를 recursive error라고 한다.<br>
>![image](https://github.com/kim0527/Effective-Java/assets/143387515/607980c5-7e21-4fa2-b591-edb7d1aaf134)

## ⚠️ Clone() 재정의시 주의사항
### 1. 재정의한 public clone 메서드에서는 throws 절을 없애자.
- Object.clone 메서드에서 ```검사예외 "CloneNotSupportedException"```을 던진다.
- [item 71] 적절한 검사예외 사용은 프로그램의 안정성을 높여주지만, 예외 남용은 사용자가 쓰기 어렵게 만든다.

>#### ➕ 검사 예외  vs  비검사 예외<br>
>그림과 같이 JAVA의 예외 처리는 모두 Throwable을 상속하고 있고 크게 Exception과 Error로 나뉜다.
>![image](https://github.com/kim0527/Effective-Java/assets/143387515/d1dc28a0-f6ba-418e-a2ca-4c036898af67) 
><br>**검사 예외 (Exception)** : 개발자가 명시함으로써 어플리케이션 수행 중 일어날 법한 예외를 검사하고 대비하라는 목적으로 사용한다.
><br>**비검사 예외(Error)** : Error는 시스템적인 예외를 의미한다. 개발자가 예외를 try-catch 로 잡지 않았을 때 발생한다.

### 2. 상속용 클래스는 Cloneable를 구현해서는 안된다.
### 3. Clone 메서드를 적절히 동기화해줘야 한다.
- Object.clone()은 동기화를 신경쓰지 않는다.
- 만약 어떤 메서드 안에서 Object.clone()를 수행하면, Object.clone()는 동기화를 신경쓰지 않기 때문에 메서드 완료되기 전에 복제를 해버린다.
- 동시성에 문제가 있어, 적절한 동기화가 필요하다.
- 동기화 문제 예시: https://javabom.tistory.com/15

## 🥎 중간정리
1. Cloneable을 구현하는 모든 클래스는 Clone을 재정의해야한다.
2. 접근제한자는 public으로, 반환 타입은 클래스 자신으로 해야한다. (공변 반환 타이핑)
3. 객체 뿐만 아니라 필드까지 clone() 해줘야 한다.
4. 만약 필드 또한 참조타입이라면, 깊은 복사를 통해 내부 깊은 구조까지 복사하자.
5. 깊은 복사는 재귀 혹은 반복을 통해 구현할 수 있다.

## 😥 위 작업이 너무 많다면?
### 복사생성자 & 복사 팩토리
- Cloneable을 구현한 클래스가 clone을 재정의해야하지만, 그렇지 않다면 복사 생성자와 복사 팩터리를 사용해보자.

**복사 생성자**
```java 
public class Stack {
    private Object[] elements;
    private int size = 0;
    
    public Stack(Stack stack) {
        this.elements = s.elements.clone();
        this.size = s.size;
    };
}
```
- 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다.
    
**복사 팩토리**
```java 
public class Stack {
    private Object[] elements;
    private int size = 0;
    
    public static Stack newInstance(Stack stack) {
        return new Stack(stack.elements, stack.size);
    }
}
```
- 복사 생성자를 모방한 정적 팩토리이다.

**복사 생성자와 복사 팩토리의 장점**
1. 생성자를 사용하지 않아 안정적이다.
2. 인터페이스 타입의 인스턴스를 인수로 받을 수 있다.
3. 복제본의 타입을 직접 선택할 수 있다.

### 📝 Reference 
[Cloneable에 대한 고찰](https://velog.io/@suky/Java-Cloneable%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B3%A0%EC%B0%B0)<br>
[공변,무공변,반공변](https://sungjk.github.io/2021/02/20/variance.html)<br>
[검사예외, 비검사예외](https://jaehoney.tistory.com/383)
