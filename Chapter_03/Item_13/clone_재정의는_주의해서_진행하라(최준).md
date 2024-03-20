# 1️⃣3️⃣ Item 13: clone 재정의는 주의해서 진행하라

<br>

## 📌 목차
1. Cloneable & Clone 메서드 개요

2. clone 메서드
   1. 아무 코드도 없는데 어떻게 동작?
   2. clone 메서드는 얕은 복사
3. Cloneable 인터페이스
   1. marker interface란?
   2. Cloneable 인터페이스 용도
4. clone 메서드 규약
5. 잘못된 clone 메서드 재정의
6. 올바른 clone 메서드 재정의 - 기본 타입, 불변 객체
7. 올바른 clone 메서드 재정의 - 가변 객체 참조 (배열)
8. 올바른 clone 메서드 재정의 - final 가변 객체 참조 
9. 올바른 clone 메서드 재정의 - 가변 객체 참조 안에 가변 객체 참조
10. 올바른 clone 메서드 재정의 - 복잡한 가변 객체 복제
11. 주의점
12. 주의점 1. clone 메서드에서는 재정의 될 수 있는 메서드 호출 X.
13. 주의점 2. 재정의한 clone 메서드는 throws 절을 없애야 함.
14. 주의점 3. 상속용 클래스는 Cloneable 구현 X.
15. 주의점 4. Cloneable을 구현한 Thread-safe 클래스 작성시 clone 메서드 역시 동기화.
16. 요약
17. 복사 생성자 & 복사 팩토리
18. 결론

<br>

## 📊 Cloneable & Clone 메서드 개요

>Java 언어에서는 `인스턴스의 복사를 실행하는 도구로 clone 메소드`가 준비되어 있다.
>
>clone 메서드는 모든 클래스의 상위 클래스인 java.lang.Object 클래스에 정의되어 있다. 
>
>clone 메소드를 실행할 경우에는 `복사 대상이 되는 클래스`는 java.lang.Cloneable `인터페이스를 구현할 필요`가 있다.  
>
>복사 대상이 되는 클래스가 직접 java.lang.Cloneable 인터페이스를 구현해도 상관 없고, 하위 클래스의 어딘가에서 Clonable 인터페이스를 구현해도 상관없다.

<br>

Cloneable 인터페이스를 구현한 클래스의 인스턴스는 clone 메소드를 호출하면 복사된다. 

그리고 clone 메소드의 반환값은 복사해서 만들어진 인스턴스가 된다.

그 내부에서 하는 일은 원래의 인스턴스와 같은 크기의 메모리를 확보한 뒤, 그 인스턴스의 필드 내용을 복사하는 것이다.

즉 `원본 객체를 필드값과 동일한 값을 가지는 새로운 객체 생성하는 것`이다. 

<br>

만약 Cloneable 인터페이스를 구현하지 않는 클래스가 clone 메소드를 호출하면 `예외 CloneNotSupportedException이 발생`한다.

그리고 clone 메서드는 `재정의해서 사용`해야 한다. 

왜냐하면 Object 클래스의 clone 메서드는 protected로 선언되어 있어 public으로 재정의 해야 한다.

<br>

clone 메서드, Cloneable 인터페이스에 대해서 자세히 알아보자. 

<br>

## 👯 clone 메서드

Object의 clone 메서드는 아래와 같다. 

```java
    @IntrinsicCandidate
    protected native Object clone() throws CloneNotSupportedException;
```

<br>

### ⁉ **아무 코드도 없는데 어떻게 동작??**

clone 메서드 앞에 `native라는 키워드`가 있는데 이 키워드는 자바 코드 내에서 직접적인 JVM의 네이티브 코드로 구현된 메서드임을 나타낸다. 

JVM의 네이티브 코드는 JVM에서 실행되는 자바 프로그램의 성능 향상 시키기 위해 C, C++ 등의 네이티브 언어로 작성된 코드를 말한다. 

따라서 Object 클래스의 clone 메서드에 아무런 코드가 없는 것이다. 

<br>

>openjdk 소스코드 내 jvm.cpp에 clone 메서드의 네이티브 코드가 있다.
>
>[openjdk 소스코드 내 jvm.cpp](https://github.com/openjdk/jdk/blob/3f41fdecdb6d131a5afe6e0a39d7414c222fe4fb/src/hotspot/share/prims/jvm.cpp#L636)
>
>이 코드에 보면 `Cloneable 인터페이스를 체크하는 로직`이 있다. 
>
>이 설명에 대해 잘 설명해준 글이 references에 ‘Cloneable 3’이다. 
>
>이 글을 읽어보면 좋을 것 같다. 

<br>

### 🖨 **clone 메서드는 얕은 복사**

clone 메서드는 `필드의 내용을 그대로 복사`한다. 

만약에 필드가 값이 아니라 참조 값을 가지고 있다면 참조 값을 복사하는 것이다. 

예를 들어 필드가 배열이면 배열의 값들을 하나하나 복사하는 것이 아니라 배열의 참조 값만 복사한다. 

이것 때문에 추후에 `문제점`이 생긴다. 

<br>

이런 `참조 값을 복사하는 것을 얕은 복사`라고 한다. 

이의 반대되는 `깊은 복사는 실제 값을 메모리 상에 복사하는 것`이다. 

그림은 아래와 같다. 

<br>

**`얕은 복사`:**

![image](https://velog.velcdn.com/images/dooboocookie/post/a12fb88e-b537-49e2-9620-a3b7322d257b/image.png)

<br>

**`깊은 복사`:**

![image](https://velog.velcdn.com/images/dooboocookie/post/1fe54429-32a8-4f15-84c8-d2d887e878aa/image.png)

<br>

이렇게 얕은 복사 수행하기 때문에 clone 메서드를 재정의하는 것도 있다. 

<br>

## 🧑‍🤝‍🧑 Cloneable 인터페이스

Cloneable 인터페이스 내부에 clone 메서드 선언되어 있는 것 같은데 그것이 아니다. 

`Cloneable 인터페이스에는 메서드가 하나도 없다.` 

코드를 보면 아래와 같다. 

```java
public interface Cloneable {
}
```

<br>

이 인터페이스는 단지 ‘clone 메서드에 의해 복사할 수 있다’는 표시로 사용되고 있다. 

이런 인터페이스를 `marker interface`라고 한다. 

<br>

### 🔖 **marker interface란?**

marker interface는 Cloneable 인터페이스처럼 인터페이스 내부에 상수도 메서드도 없는 인터페이스를 말한다. 

marker interface는 `객체의 타입과 관련된 정보 제공`하는 것이다. 

이 인터페이스 구현하는 클래스가 특정 속성 가진다고 표시하는 인터페이스다. 

따라서 객체에 대한 런타임 유형 정보 제공해서 컴파일러와 JVM은 이 `marker interface를 통해 객체에 대한 추가 정보 얻는 것`이다. 

<br>

### 📂 Cloneable 인터페이스 용도

Cloneable 인터페이스는 위에서 봤듯이 `clone 메서드에 의해 복사할 수 있다는 표시로 사용`된다. 

또 다른 하는 일은 Object의 protected 메서드인 clone 메서드의 동작 방식 결정하는 것이다. 

위에서 봤듯이 Cloneable 구현한 객체가 clone 메서드 호출하면 복사한 객체 반환한다고 했다. 

Cloneable 구현하지 않은 객체가 clone 메서드 호출하면 예외 던친다고 했다. 

이렇게 `Cloneable 구현했는가 아닌가에 따라 clone 메서드 호출했을 때 동작 방식 정하는 것`이다. 

<br>

>Cloneable 인터페이스는 표시로 사용되는 인터페이스지만 아쉽게도 의도한 목적 제대로 이루지 못했다.
>
>왜냐하면 `clone 메서드가 선언된 곳이 Cloneable이 아니라 Object이고 그마저도 protected이기 때문`이다. 
>
>그래서 클래스에서 그냥 Cloneable만 구현했다고 clone 메서드 호출할 수 있는 것이 아니다. 
>
>따라서 사용하려면 Object 클래스의 `clone 메서드를 오버라이딩해서 재정의해서 사용`해야 한다. 

<br>

원래 인터페이스라면 Cloneable 인터페이스 안에 clone 메서드 있어서 clone 메서드 오버라이딩 하라고 나오면서 알려줘야 한다. 

하지만 Cloneable 인터페이스에서는 clone 메서드가 Object 클래스에 있어서 알려주지도 않고 protected라 clone 메서드 쓰려고 해도 쓸 수 가 없다. 

<br>

명세에서는 이야기하지 않지만 `실무에서는 Cloneable 구현한 클래스는 clone 메서드 재정의 해서 public으로 제공하고 clone 메서드가 복제 제대로 이뤄진다고 기대`한다. 

이렇게 하려면 `Cloneable 구현한 클래스와 모든 상위 클래스는 규약을 지켜야 한다`. 

그런데 규약을 지키면 위험하고 모순적인 메커니즘이 탄생한다. 

바로 생성자를 호출하지 않고도 객체를 생성할 수 있다는 것이다. 

<br>

이렇듯 `Cloneable은 문제점이 많은데 많이 쓰이고 있다`. 

<br>

## 🧾 clone 메서드 규약

`clone 메서드의 일반 규약은 허술`하다. 

clone 메서드의 규약을 보자. 

    이 객체의 복사본을 생성해 반환한다. ‘복사’의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다.

    일반적인 의도는 다음과 같다.

    어떤 객체 x에 대해 다음 식은 참이다.

        x.clone() != x

    또한 다음 식도 참이다.

        x.clone().getClass() = x.getClass()

    다음 식도 일반적으로 참이지만, 역시 필수는 아니다.

        x.clone() .equals(x)

    관례상, 이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야 한다.

    이 클래스와 (Object를 제외한) 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.

        x.clone().getClass() = x.getClass()

    관례상, 반환된 객체와 원본 객체는 독립적이어야 한다.

    이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

<br>

규약을 보면 중요한 점이 clone 메서드가 반환하는 객체는 `super.clone 호출`해서 얻어야 한다고 되어 있다. 

이렇게 되면 `생성자 연쇄와 비슷한 메커니즘`이다. 

생성자 연쇄는 부모 클래스와 자식 클래스 있을 때 자식 클래스의 생성자가 호출될 때 자동으로 부모 클래스의 생성자가 호출되는 것이다. 

여기서도 자식 클래스의 clone 메서드 호출하면 `부모 클래스의 clone 메서드 부르도록 해야 한다`고 규약에 써있다. 

<br>

## ❌ 잘못된 clone 메서드 재정의

규약에서 clone 메서드는 super.clone 호출해야 한다고 했다. 

하지만 상위 클래스에서 super.clone이 아니라 그냥 `스스로 객체를 만들어서 복사해서 반환`해 줄 수 있다. 

아래 코드와 같이 하는 것이다. 

```java
public class Parent implements Cloneable{
	...
	@Override
	public Parent clone(){
		Parent clone = new Parent();
		//필드 복사하는 내용
		return clone;
	}
	...
}
```

<br>

이렇게 하고 Parent 클래스에스 clone 메서드 사용해도 컴파일러는 오류 내지 않는다. 

<br>

하지만 `문제점은 상속이 있을 때 생긴다`. 

Parent의 하위 클래스에서 규약을 지켜서 super.clone 메서드 호출하면 `Parent 객체가 만들어진다`. 

그렇게 되면 하위 클래스의 clone 메서드가 제대로 동작하지 않게 되는 것이다. 

<br>

## 🔴 올바른 clone 메서드 재정의 - 기본 타입, 불변 객체

제대로 동작하는 clone 메서드를 가진 상위 클래스 상속해 Cloneable 구현해보자. 

`그냥 규약이 시키는대로 하면 된다.`

그렇다면 clone 메서드에서 먼저 super.clone을 호출한다. 

이렇게 하면 클래스에 정의된 모든 필드는 원본 필드와 똑같은 값을 가진다. 

<br>

이때 `모든 필드가 기본 타입이거나 불변 객체를 참조한다면 clone 메서드 끝`이다. 

왜냐하면 clone 메서드가 문제되던 점이 얕은 복사를 하기 때문이였다. 

<br>

따라서 이 경우는 아래와 같이 clone 메서드 구현하면 된다. 

```java
@Override public 객체타입 clone(){
	try{
		return (객체타입) super.clone();
	} catch (CloneNotSupportedException e){
		throw new AssertionError();  // 이 예외 부분은 일어날 수 없는 일이라고 한다. 
	}
}
```

<br>

이때 super.clone 하고 자신 타입으로 `형변환해서 반환`한다. 

`클라이언트가 형변환 하지 않게 하기 위함`이다. 

super.clone이 반환하는 타입보다 하위 타입이기 때문에 형변환 할 수 있는 것이다. 

이 형변환은 절대 실패하지 않는다. 

<br>

**번외**
>그리고 super.clone 메서드를 try-catch로 감싼 이유는 Object의 clone 예외가 `검사 예외인 CloneNotSupportedException`을 던지도록 선언 되었기 때문이다. 
>
>검사 예외는 try-catch로 예외 처리하거나 throws 키워드를 사용해 예외 처리를 다른 곳으로 넘겨야 한다. 
>
>비검사 예외는 런타임에 발생하는 예외로 프로그램 실행 전 예외 처리 강제하지 않는 것이다. 
>
>Cloneable을 구현하지 않으면 발생하는 예외인데 Cloneable을 구현해야 하는 것은 안다. 
>
>그리고 구현했기 때문에 우리는 super.clone이 성공할 것을 안다. 
>
>괜히 코드만 더 작성한 것이라 `CloneNotSupportedException를 비검사 예외로 했어야 한다`고 한다. 

<br>

## 🟠 올바른 clone 메서드 재정의 - 가변 객체 참조 (배열)

이렇게 간단한 clone 메서드 재정의는 클래스가 `필드에서 가변 객체를 참조하는 순간 큰일 발생`한다. 

<br>

아래 코드처럼 `배열을 필드`로 가지는 클래스가 있다.

```java
public class Stack{
	private Object[] elements;
	private int size=0;
	
	...
	
}
```

<br>

### ❎ 문제점

이때 앞에서 한 clone 메서드 재정의처럼 super.clone 결과 그대로 반환하면 문제 생긴다. 

복제된 인스턴스에서 기본 타입인 size 필드는 올바른 값을 가지지만 배열인 elements 필드는 `원본 인스턴스와 같은 배열을 참`조할 것이다. 

왜냐하면 `clone 메서드는 얕은 복사를 하여 배열의 참조 값을 복사`하기 때문이다. 

원본이나 복제본이나 하나 수정하면 다른 것도 수정되버려서 프로그램이 이상하게 동작하게 된다. 

<br>

>clone 메서드는 `생성자와 같은 효과`를 내야 한다. 
>
>즉 clone 메서드는 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다. 
>
>그렇다면 어떻게 해야 하는지 보자. 

<br>

### ✅ 해결 방법

가장 쉬운 방법은 배열의 `clone 메서드를 재귀적으로 호출`해주는 것이다. 

우선 똑같이 super.clone 메서드를 호출해서 복제된 인스턴스를 받는다. 

그리고 배열 필드의 경우 이 복제된 인스턴스의 배열 필드에 내 배열 필드를 복제해서 넣는다. 

말로 하면 어려우니 코드를 보자. 

<br>

코드로 하면 아래와 같다. 

```java
    // 코드 13-2 가변 상태를 참조하는 클래스용 clone 메서드
    @Override public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```

<br>

즉 ‘내 배열 필드.clone()’ 이렇게 `배열을 복제해서 복제된 인스턴스에 넣어주는 것`이다. 

***`배열의 clone 메서드가 따로 있다.`*** 

배열의 clone 메서드는 런타임 타입과 컴파일타임 타입 모두가 원본 배열과 똑같은 배열을 반환한다. 

따라서 `배열 복제할 때에는 배열의 clone 메서드 사용하라고 권장`한다. 

`배열이 clone 기능 제대로 사용하는 유일한 예`이다. 

<br>

## 🟡 올바른 clone 메서드 재정의 - final 가변 객체 참조

이처럼 배열 필드를 가진 클래스의 경우 배열의 clone 메서드를 한번 더 호출해주면 된다. 

<br>

### ❎ **문제점**

하지만 `배열 필드가 final이라면 문제가 생긴다.`

final 필드에는 새로운 값을 할당할 수 없어서 배열의 clone 메서드 써서 생성한 `복제 배열을 할당할 수 없다.` 

이것은 Cloneable 아키텍처는 ‘가변 객체 잠조하는 필드는 final로 선언하라’는 일반 용법과 충돌한다. 

<br>

### ✅ 해결 방법

이것을 해결하려면 복제할 수 있는 클래스 만들기 위해 `일부 필드에서 final 한정자 제거해야 한다.` 

<br>

## 🟢 올바른 clone 메서드 재정의 - 가변 객체 참조 안에 가변 객체 참조

가변 객체 참조하는 클래스의 clone 메서드 재정의 할 때 `clone 메서드를 재귀적으로 호출하는 것만으로는 충분하지 않을 때도 있다.` 

<br>

예시로 해시 테이블용 clone 메서드를 보자. 

해시 테이블 내부에는 버킷 배열이 있다. 

그리고 버킷은 키-값 쌍을 담는 linked list를 가리킨다. 

아래와 같은 그림인 것이다. 

![image](https://i.namu.wiki/i/d2mLHpOJaleAV9LWDfs2kjlZ3oY1XrX98n1MPOxZmWjzlky7w6Cc7-DAshjMbJ5q0LGXxV7gxKWbTJ-TT9rHaA.webp)

<br>

해시 테이블 클래스를 코드로 표현하면 아래와 같다. 

```java
public class HashTable implements Cloneable{
	private Entry[] buckets = ...;
	
	private static class Entry{
		Entry next;     // 버켓의 linkedlist에서 다음 것을 가리키는 필드
		...
	}

	...
}
```

<br>

코드를 보면 `Entry 타입의 배열`이 있다. 

이것들을 버킷의 역할을 한다. 

그리고 `Entry 타입을 보면 linked list처럼 작동하기 위해 Entry next; `라는 필드를 가지고 있다. 

이 next 필드가 다음 노드를 가리키고 있는 것이다. 

![image](https://file.notion.so/f/f/7d1044b7-24d0-4b32-b24a-1b9061f7bf65/fb338cf1-55db-40ba-905a-3d8684a42266/Untitled.png?id=ebb49e57-5d68-4d06-9d68-fedd59177d86&table=block&spaceId=7d1044b7-24d0-4b32-b24a-1b9061f7bf65&expirationTimestamp=1710604800000&signature=7ypNVdpI0CX4QgUmBcGzGtRmOxdP_7kG3FbRNi96NI0&downloadName=Untitled.png)

<br>

### ❎ 문제점 1

이때 이전에 위에서 하던 것처럼 똑같이 배열의 clone 메서드를 재귀적으로 호출하면 문제가 생긴다. 

배열.clone 하면 buckets 배열과 똑같은 배열이 생길 것이다. 

하지만 이 `복제본 배열은 원본 배열과 같은 linked list를 참조해서 문제가 되는 것`이다. 

아래 그림처럼 되는 것이다. 

![Untitled](https://file.notion.so/f/f/7d1044b7-24d0-4b32-b24a-1b9061f7bf65/6c440a1b-1791-4338-89ee-d3b45becec1b/Untitled.png?id=0ad5f576-2a61-4268-ab02-285e7d4ac551&table=block&spaceId=7d1044b7-24d0-4b32-b24a-1b9061f7bf65&expirationTimestamp=1710604800000&signature=b7k4lXbXVNHzTpBHgikL_5XMe7D_tcqlIXEFxfOdkco&downloadName=Untitled.png)

<br>

### ✅ 해결 방법 1

이를 해결하려면 각 `버킷을 구성하는 linked list를 복사`해야 한다. 

<br>

아래 코드처럼 clone 메서드를 재정의 하면 된다. 

```java
@Override public HashTable clone(){

	try{
		HashTable result = (HashTable) super.clone();
		result.buckets = new Entry[buckets.length];
		for(int i=0; i<buckets.length; i++)
			result.buckets[i] = buckets[i].deepCopy();
			
		return result;
	} catch(CloneNotSupportedException e) {
			throw new AssertionError();
		}
	}
	...
}
```

<br>

이 clone 메서드는 HashTable 클래스 내부의 메서드이다. 

그리고 deepCopy라는 메서드를 Entry 클래스 안에 정의해야 한다. 

아래와 같이 하는 것이다. 

```java
public class HashTable implements Cloneable{
	private Entry[] buckets = ...;
	
	private static class Entry{
		Entry next;     // 버켓의 linkedlist에서 다음 것을 가리키는 필드
		...
		
		Entry deepCopy(){
			return new Entry(next == null ? null : next.deepCopy());
		}
		
	}
	
	@Override public HashTable clone(){
		...
	}
	...
}
```

`Entry 클래스는 깊은 복사를 지원하도록 보강`한 것이다. 

Entry 클래스의 deepCopy 메서드는 `Entry 클래스가 가리키는 linked list 전체를 복사하기 위해 자신을 재귀적으로 호출`하는 것이다. 

<br>

>clone 메서드는 어떻게 재정의 했는지 보자. 
>
>1. 적절한 크기의 새로운 버킷 배열을 할당한다. 
>
>2. 원래 버킷 배열을 순회하면서 비어있지 않은 각 버킷에 대해 깊은 복사를 수행한다. 

<br>

### ❎ 문제점 2

위의 방법은 간단하다. 

그리고 버킷 배열이 너무 길지 않다면 잘 작동한다. 

<br>

하지만 `linked list를 복제하는 방법은 좋지 않다.` 

왜냐하면 재귀 호출을 해서 리스트의 원소 수만큼 stack frame을 소비해 `linked list가 길면 stack overflow를 일으킬 위험`이 있다. 

<br>

### ✅ 해결 방법 2

이 문제를 피하려면 Entry 클래스의 deepCopy 메서드를 재귀 호출 대신 `반복자를 써 순회하는 방향으로 수정`해야 한다.

Entry의 deepCopy 메서드를 아래와 같이 바꾸는 것이다. 

```java
Entry deepCopy(){
	Entry result = new Entry(next);

	for(Entry p = result; p.next != null; p = p.next)
		p.next = new Entry(p.next.next);

	return result;
}
```

<br>

## 🔵 올바른 clone 메서드 재정의 - 복잡한 가변 객체 복제

`복잡한 가변 객체를 복제하는 방법`을 보자. 

1. `super.clone 호출`해 얻은 객체의 모든 필드를 초기 상태로 설정한다. 

2. 원본 객체의 상태를 `다시 생성하는 고수준 메서드들을 호출`한다. 

<br>

원본 객체의 상태를 다시 생성하는 고수준 메서드들을 호출한다는 의미는 예를 들어 배열의 값을 하나하나 복제해 새로운 배열을 반환하는 메서드를 만들고 그것을 호출하라는 의미다. 

<br>

이렇게 고수준 메서드를 활용해 복제하면 `보통 간단하고 우아한 코드 얻는다.` 

하지만 저수준에서 `바로 처리할 때보다 느리다.` 

또한 Cloneable 아키텍처의 기초가 되는 필드 단위 객체 복사를 우회하기 때문에 전체 `Cloneable 아키텍처와는 어울리지 않는 방식`이기도 하다. 

<br>

## ⚠ 주의점

clone 메서드를 재정의 할 때 `주의점`들이 있다. 

1. clone 메서드에서는 재정의 될 수 있는 메서드 호출하지 않아야 한다. 

2. 재정의한 clone 메서드는 throws 절을 없애야 한다. 
3. 상속용 클래스는 Cloneable 구현해서는 안된다. 
4. Cloneable을 구현한 Thread-safe 클래스 작성할 때에는 clone 메서드 역시 동기화 해야 한다. 

<br>

이제 각각 주의점에 대해서 보자. 

<br>

## 1️⃣ 주의점 1. clone 메서드에서는 재정의 될 수 있는 메서드 호출 X.

생성자에서는 `재정의 될 수 있는 메서드 호출하지 말아야 한다.` 

그런데 clone 메서드도 마찬가지다. 

<br>

clone 메서드 안에 `하위 클래스에서 오버라이딩 할 수 있는 메서드를 넣지 말라는 것`이다. 

왜냐하면 clone 메서드에서 super.clone을 호출한다. 

우리는 상위 클래스의 clone 메서드에서 어떤 행동을 하길 기대하고 메서드를 clone 메서드에서 호출하게 한 것이다. 

<br>

>하지만 문제는 `하위 클래스에서 clone 메서드 사용할 때`이다. 
>
>하위 클래스의 clone 메서드에서 super.clone을 호출한다. 
>
>그렇다면 상위 클래스에서 메서드를 호출하는데 `하위 클래스에서 오버라이딩 한 메서드가 호출`이 된다. 
>
>그런데 하위 클래스에서 오버라이딩 한 메서드가 `하위 클래스의 상태를 바꾸도록 오버라이딩` 한 것이다. 
>
>이렇게 되면 문제가 된다. 

<br>

이렇게 되면 하위 클래스는 복제 과정에서 하위 클래스의 상태를 교정할 기회를 잃게 된다. 

따라서 원본과 복제본의 상태가 달라질 가능성이 크다. 

<br>

글로만 보면 이해가 안간다. 코드를 보자. 

코드로 보면 아래와 같다. 

**Parent**

```java
class Parent implements Cloneable {
    int x;

    public Parent(int x) {
        this.x = x;
    }

    @Override
    public Object clone() throws CloneNotSupportedException {
        // 하위 클래스에서 오버라이딩 될 수 있는 메서드 호출
        this.adjustState(); // 잘못된 호출
        System.out.println("Parent clone method called");
        return new Parent(x);
    }

    // 하위 클래스에서 오버라이딩 될 수 있는 메서드
    public void adjustState() {
        System.out.println("Parent's adjustState method called");
    }
    
}
```
<br>

**Child**

```java
class Child extends Parent {
    int y;

    public Child(int x, int y) {
        super(x);
        this.y = y;
    }

    @Override
    public Child clone() throws CloneNotSupportedException {
        System.out.println("Child clone method called");
        return (Child) super.clone();
    }

    @Override
    public void adjustState() {
//        super.adjustState();
        System.out.println("Child's adjustState method called");
        // 복제 과정에서 자신의 상태를 교정
        this.y = 0;
    }
}

```
<br>

**main 메서드**

```java
public class TestClass {
    public static void main(String[] args) {
        try {
            Child original = new Child(10, 20);
            System.out.println("Original: " + original);
            
            Child cloned = original.clone();
            System.out.println("Original: " + original);
            System.out.println("Cloned: " + cloned);
            
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
    }
}
```
<br>

**결과**

```java
Original: Child{x=10, y=20}

Child clone method called
Child's adjustState method called
Parent clone method called

Original: Child{x=10, y=0}
Cloned: Child{x=10, y=0}
```

<br>

보면 Child의 값이 x=10, y=20이고 이것을 clone 해서 같은 값을 가지기를 기대한다. 

하지만 `복제된 것을 보면 x=10, y=0`이 된다. 

왜냐하면 Parent의 clone 메서드에서 adjustState 메서드를 부르는데 `이때 Child의 adjustState 메서드가 호출`이 된다. 

따라서 Original의 y가 0으로 바뀌고 이것이 clone 되어서 x=10, y=0인 값을 가진 복제본이 나온다. 

<br>

## 2️⃣ 주의점 2. 재정의한 clone 메서드는 throws 절을 없애야 함.

Object의 clone 메서드는 CloneNotSupportedException 던진다고 선언했다. 

하지만 재정의한 메서드는 그렇지 않다. 

`public인 clone 메서드에서는 throws 절을 없애야 한다. `

검사 예외를 던지지 않아야 clone 메서드 `사용하기 편하기 때문`이다. 

그렇지 않으면 clone 메서드 사용하는 곳에서 또 throws를 하거나 try-catch문을 사용해야 하기 때문이다. 

<br>

## 3️⃣ 주의점 3. 상속용 클래스는 Cloneable 구현 X.

`상속용 클래스는 Cloneable을 구현해서는 안된다. `

왜냐하면 앞에서 봤듯이 상위 클래스의 clone 메서드를 호출하고 얕은 복사를 하는 등 많은 문제점이 있다. 

<br>

하지만 굳이 구현하고 싶다면 `2가지 방식`이 있다. 

1. Object 클래스의 clone 메서드 모방

2. clone 동작하지 않게 구현하고 하위 클래스에서 재정의 못하게 함

<br>

우선 첫번째 방법은 `Object 클래스의 clone 메서드 방식을 모방`하는 것이다. 

제대로 작동하는 clone 메서드 구현해 protected로 둔다. 

그리고 CloneNotSupportedException 던질 수 있다고 선언하는 것이다. 

이렇게 하면 Cloneable 구현 여부를 하위 클래스에서 선택하도록 해준다. 

<br>

두번째 방법은 `clone 메서드 동작하지 않게 구현하고 하위 클래스에서 재정의하지 못하게 하는 것`이다. 

아래 방식처럼 clone 메서드 재정의 해 놓으면 된다. 

```java
@Override
protected final Object clone() throws CloneNotSupportedException{
	throw new CloneNotSupportedException();
}
```

<br>

## 4️⃣ 주의점 4. Cloneable을 구현한 Thread-safe 클래스 작성시 clone 메서드 역시 동기화.

Cloneable을 구현한 `thread-safe 클래스 작성할 때에는 clone 메서드 역시 적절히 동기화` 해야 한다. 

<br>

>Thread-Safe란 멀티 스레드 프로그래밍에서 일반적으로 어떤 함수나 변수, 혹은 객체가 여러 스레드로부터 동시에 접근이 이루어져도 프로그램의 실행에 문제가 없는 것을 말한다.
>
>하나의 함수가 한 스레드로부터 호출되어 실행 중일 때, 다른 스레드가 그 함수를 호출하여 동시에 함께 실행되더라도 각 스레드에서의 함수의 수행 결과가 옳바르게 나오는 것을 말한다.

<br>

Object의 clone 메서드는 동기화를 신경 쓰지 않았다. 

그러니 super.clone 호출 외에 다른 할 일이 없더라도 `clone을 재정의하고 동기화` 해줘야 한다. 

<br>

## 🖌 요약

요약하자면 Cloneable을 구현하는 `모든 클래스는 clone 메서드를 재정의 해야 한다.` 

이때 접근 제한자는 public으로, 반환 타입은 클래스 자신으로 변경해야 한다. 

clone 메서드에서는 `가장 먼저 super.clone을 호출한 후 필요한 필드를 전부 적절히 수정`해야 한다. 

객체 내부에 숨어있는 모든 가변 객체를 복사하고 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 해야 한다는 것이다. 

이렇게 하려고 주로 각 객체들의 clone을 재귀적으로 호출해서 구현한다. 

하지만 재귀적으로 하는 것이 최선이 아닐 때도 있어 반복자로 순회 해야 할 때도 있다. 

기본 타입 필드와 불변 객체 참조만 가지는 클래스는 따로 필드 수정 안해도 된다. 

하지만 일련번호나 고유 ID 같이 클래스마다 다 달라야 하는 것은 기본 타입이나 불변이라고 수정해야 한다. 

<br>

## 🏭 복사 생성자 & 복사 팩토리

Cloneable 지금까지 보니까 너무 어렵고 복잡하다. 

이 모든 작업이 필요한가 하면 다행히도 이렇게 복잡한 경우는 드물다. 

그래도 Cloneable 구현한 클래스 확장하면 clone 메서드 잘 작동하도록 재정의 해야 한다. 

<br>

`Cloneable 말고 더 쉬운 객체 복사 방식`이 있다. 

바로 `복사 생성자와 복사 팩터리`다. 

<br>

**`복사 생성자`** 는 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다. 

코드는 아래와 같다. 

```java
public Yum(Yum yum){...};
```

<br>

**`복사 팩터리`** 는 복사 생성자를 모방한 정적 팩토리 메서드다. 

코드는 아래와 같다.

```java
public static Yum newInstance(Yum yum){...}; 
```
<br>

복사 생성자와 복사 팩터리는 Cloneable/clone 방식보다 `나은 면이 많다`. 

1. 언어 모순적이고 위험천만한 객체 생성 매커니즘 사용하지 않는다. 
(생성자 쓰지 않고 객체 생성…? → 말이 안됨)

2. 엉성하게 문서화된 규약에 기대지 않는다. 

3. 정상적인 final 필드 용법과 충돌하지 않는다. 

4. 불필요한 검사 예외 던지지 않는다.

5. 형변환도 필요하지 않다. 

6. 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있다. 

<br>

6번을 더 자세히 보자. 

인터페이스 기반 복사 생성자와 복사 팩터리의 더 정확한 이름은 `변환 생성자, 변환 팩터리`다. 

이들 이용하면 클라이언트는 `원본의 구현 타입에 얽매이지 않고 복제본의 타입 직접 선택`할 수 있다. 

예를들어 HashSet 객체 s를 TreeSet 타입으로 복제할 수 있다. 

clone 메서드는 자기 자신 타입만 복제할 수 있었다. 

변환 생성자 사용하면 new TreeSet<>(s); 이렇게 하면 간단히 할 수 있다. 

<br>

## ‼ 결론

Cloneable이 몰고 온 모든 문제 되짚어봤을 때, ***`이것은 사용하면 안된다.`***

새로운 인터페이스를 만들 때 절대 Cloneable 확장하면 안된다. 

새로운 클래스도 Cloneable 구현하면 안된다. 

<br>

final 클래스라면 Cloneable 구현해도 위험 크지 않다. 

하지만 성능 최적화 관점에서 검토한 후 별다른 문제 없을 때만 드물게 허용해야 한다. 

<br>

`기본 원칙은 ‘복제 기능은 생성자와 팩터리 이용하는 것이 최고’ 라는 것`이다. 

단 `배열은 clone 메서드 방식이 가장 깔끔`하다. 

이 규칙의 합당한 예외가 이 배열이다. 

<br>

## 📍 references

- [Cloneable 1](https://catsbi.oopy.io/16109e87-3c7e-4c6e-9816-c86e6b343cdd)
- [Cloneable 2](https://kobalja2020.tistory.com/entry/Java-%EC%9E%90%EB%B0%94-Object-%ED%81%B4%EB%9E%98%EC%8A%A4%EC%9D%98-clone-%EB%A9%94%EC%84%9C%EB%93%9C%EC%99%80-Cloneable-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC)
- [Cloneable 3](https://velog.io/@suky/Java-Cloneable%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B3%A0%EC%B0%B0)
- [얕은 복사, 깊은 복사 1](https://velog.io/@dooboocookie/JAVA-Clonealbe)
- [얕은 복사, 깊은 복사 2](https://zzang9ha.tistory.com/372#google_vignette)
- [marker interface 1](https://kjhoon0330.tistory.com/entry/Java-%EB%A7%88%EC%BB%A4-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC)
- [marker interface 2](https://xonmin.tistory.com/24)
- [marker interface 3](https://recordsoflife.tistory.com/809)
- [검사 예외, 비검사 예외](https://developingman.tistory.com/m/29)
- [Thread-Safe](https://developer-ellen.tistory.com/205)