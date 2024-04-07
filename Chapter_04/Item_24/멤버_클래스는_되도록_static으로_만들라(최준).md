# 2️⃣4️⃣ Item 24: 멤버 클래스는 되도록 static으로 만들라

<br>

## 📌 목차
1. 중첩 클래스란
2. 정적 멤버 클래스
3. 비정적 멤버 클래스
4. 정적 멤버 클래스 VS 비정적 멤버 클래스
5. 익명 클래스
6. 지역 클래스
7. 결론

<br>

## 📚 중첩 클래스란

중첩 클래스는 `다른 클래스 안에 정의된 클래스`를 말한다. 

중첩 클래스는 자신을 감싼 `바깥 클래스에서만 쓰여야 한다`. 

다른 곳에서 쓰인다면 중첩 클래스가 아니라 톱레벨 클래스로 만들어야 한다. 

<br>

중첩 클래스의 종류는 아래 `4가지`가 있다. 

1. 정적 멤버 클래스

2. 비정적 멤버 클래스

3. 익명 클래스

4. 지역 클래스

<br>

2~4번은 `내부 클래스`에 해당한다. 

이 중첩 클래스들을 `언제 사용`하는지 알아보자. 

그러면서 `왜 멤버 클래스는 되도록 static으로 만들라고 하는지` 알아보자. 

<br>

## ♻ 정적 멤버 클래스

정적 멤버 클래스는 `2가지 특징 제외`하고 일반 클래스와 똑같다. 

- 다른 클래스 안에 선언된다.

- 바깥 클래스의 private 멤버에도 접근할 수 있다.

<br>

정적 멤버 클래스는 다른 정적 멤버와 똑같은 접근 규칙을 적용 받는다. 

>`정적 멤버 접근 규칙`은 다음과 같다. 
>
>- public : 다른 클래스나 패키지에서 정적 멤버에 접근 가능
>
>- protected : 동일한 패키지에 속한 클래스, 다른 패키지에 속한 하위 클래스에서 정적 멤버 접근 가능
>
>- package-private : 동일한 패키지에 속한 클래스에서 정적 멤버 접근 가능
>
>- private : 동일한 클래스 내에서만 접근 가능

<br>

### ❓ When?

정적 멤버 클래스는 `바깥 클래스와 함께 쓰일 때만 유용한 public 도우미 클래스로 쓰인다`. 

<br>

예시로 계산기가 지원하는 `연산 종류를 정의하는 열거 타입`이 있다. 

코드로 보면 아래와 같다. 

```java
public class Calculator{
		...
		public enum Operation{
				PLUS("+"){~},
				MINUS("-"){~},
				TIMES("*"){~},
				DIVIDE("/"){~};
				...
		}
		...
}
```

<br>

이렇게 정적 멤버 클래스 Operation 클래스는 `Calculator 클래스와 함께 쓰일 때에만 유용한 public 도우미 클래스`이다. 

Calculator 클래스를 사용하는 곳에서 `Calculator.Operation.PLUS 이렇게 원하는 연산을 쉽게 참조`할 수 있다. 

<br>

## 👨‍👦 비정적 멤버 클래스

비정적 멤버 클래스는 `정적 멤버 클래스에서 static 하나만 뺀 것`이다. 

그런데 의미상으로 차이가 의외로 꽤 크다. 

<br>

비정적 멤버 클래스의 인스턴스는 `바깥 클래스의 인스턴스와 암묵적으로 연결`된다. 

그래서 비정적 멤버 클래스의 인스턴스 메서드에서 `정규화된 this를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조`를 가져올 수 있다. 

여기서 정규화된 this는 `‘바깥 클래스명’.this 형태로 바깥 클래스의 이름을 명시하는 용법`이다. 

<br>

코드로 보면 아래와 같다. 

바깥 클래스 OuterClass가 있다. 

그리고 내부에 비정적 멤버 클래스 InnerClass가 있다. 

```java
public class OuterClass {

    private int num;

    public void setNum(int num) {
        this.num = num;
    }

    public void hello(){
        System.out.println("hello");
    }

    public class InnerClass{

        public void sayOutsideHello(){
            OuterClass.this.hello();
            OuterClass.this.setNum(5);

            System.out.println(OuterClass.this.num);
        }

    }
}
```

<br>

코드를 보면 InnerClass 내부의 sayOutsideHello 메서드에서 `OuterClass.this 를 사용해서 바깥 클래스 참조를 가져온 것`을 볼 수 있다. 

이렇게 사용할 수 있어 `암묵적으로 연결`되어 있다고 하는 것이다. 

<br>

그리고 사용할 때에는 아래와 같이 사용한다. 

```java
    public static void main(String[] args) {
        OuterClass outerClass = new OuterClass();

        OuterClass.InnerClass innerClass = outerClass.new InnerClass();

        innerClass.sayOutsideHello();
    }
```

<br>

사용하는 코드를 보면 `비정적 멤버 클래스는 바깥 인스턴스 없이 생성할 수 없다.` 

따라서 개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있다면 정적 멤버 클래스로 만들어야 한다. 

<br>

비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이의 관계는 `비정적 멤버 클래스가 인스턴스화 될 때 확립되며 더 이상 변경할 수 없다. `

이 관계는 보통 아래처럼 바깥 클래스의 메서드에서 비정적 멤버 클래스의 `생성자를 호출할 때 자동으로 만들어진다`고 한다. 

```java
public class OuterClass{
	
	public void hello(){
		InnerClass innerClass = new InnerClass();
	}
	...
	
	public class InnerClass{
		...
	}

}
```

<br>

그런데 위의 main 메서드에서 보여준 방식처럼 `수동으로 만드는 방법`도 있는데 드물게 사용한다.

```java
    public static void main(String[] args) {
        OuterClass outerClass = new OuterClass();
        OuterClass.InnerClass innerClass = outerClass.new InnerClass();
    }
```

<br>

이렇게 `바깥 클래스와 비정적 멤버 클래스는 관계로 연결`되어 있다. 

이 `관계 정보는 비정적 멤버 클래스의 인스턴스 안에 만들어진다.` 

따라서 `메모리 공간을 차지하고 생성 시간도 더 걸린다. `

<br>

### ❓ When?

비정적 멤버 클래스는 `어댑터를 정의할 때 자주 사용`된다. 

어떤 클래스의 인스턴스 감싸 마치 `다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용`되는 것이다. 

<br>

예시로 2가지가 있다.

1. Map 인터페이스 구현체들은 자신의 컬렉션 뷰를 구현할 때 비정적 멤버 클래스를 사용한다. 

2. Set, List 같은 다른 컬렉션 인터페이스의 구현체들도 자신의 반복자를 구현할 때 비정적 멤버 클래스 사용한다. 

<br>

### 1️⃣

먼저 1번을 이해해보자. 

`컬렉션 뷰`가 뭔지 알아보자. 

>**컬렉션** : 데이터를 저장하고 관리하는 자료구조.
>
>**컬렉션 뷰** : 컬렉션의 내용 표시하고 다룰 수 있는 방법 제공하는 것.

<br>

Map 인터페이스의 구현체들은 내부적으로 키-값 쌍의 컬렉션을 유지하고 있다. 

그리고 `keySet(), values(), entrySet() 메서드를 통해 컬렉션 뷰를 반환`한다. 

아래는 각 메서드들이 반환하는 컬렉션 뷰다. 

>**keySet()** : 키의 컬렉션 뷰 반환
>
>**values()** : 값의 컬렉션 뷰 반환
>
>**entrySet()** : 키-값 쌍의 컬렉션 뷰 반환

<br>

이때 Map은 키들을 Set 컬렉션에 저장하고 있다. 

그리고 이 `Set 컬렉션을 만들 때 내부의 비정적 멤버 클래스를 사용`한다. 

코드로 보면 아래와 같다. 

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    
    ...
   
    public Set<K> keySet() {
        Set<K> ks = keySet;
        if (ks == null) {
            ks = new KeySet();
            keySet = ks;
        }
        return ks;
    }
    
    ...
    
    final class KeySet extends AbstractSet<K> {
	    ...
	  }
	    
}
```

<br>

keySet 메서드로 키의 컬렉션 뷰 반환할 때 keySet이 없다면 비정적 멤버 클래스 KeySet을 만들어 반환하는 것을 볼 수 있다. 

<br>

### 2️⃣

2번도 마찬가지다. 

Set, List 인터페이스의 구현체들이 `Iterator를 반환 받을 때 비정적 멤버 클래스를 만들어 반환`한다. 

코드는 아래와 같다. 

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{

		...
	
    public Iterator<E> iterator() {
        return new Itr();
    }

    /**
     * An optimized version of AbstractList.Itr
     */
    private class Itr implements Iterator<E> {
		    ...
		}
        
}
```

<br>

Itr이라는 비정적 멤버 클래스를 가지고 있는 것이다. 

<br>

## 🆚 정적 멤버 클래스 VS 비정적 멤버 클래스

멤버 클래스에서 `바깥 인스턴스에 접근할 일이 없다면 무조건 정적 멤버 클래스`로 만들자. 

<br>

비정적 멤버 클래스로 만들면 위에서 본 것처럼 `바깥 인스턴스로의 숨은 외부 참조`를 갖게 된다. 

그러면 다음과 같은 `문제점`들이 생긴다. 

- 바깥 인스턴스 참조를 저장하려면 `시간과 공간이 소비`된다.

- 가비지 컬렉션이 바깥 인스턴스 수거하지 못하는 `메모리 누수` 생길 수 있다.

- 바깥 인스턴스 참조 눈에 보이지 않아 `문제 원인 찾기 어려워 심각한 상황 초래`한다.
  
<br>

비정적 멤버 클래스가 보이지 않는 숨은 바깥 인스턴스 참조를 가지고 있어서 가비지 컬렉션이 바깥 인스턴스 수거하지 못하는 것이다. 

<br>

### 🔒 private 멤버 클래스일 때 정적 멤버 클래스 VS 비정적 멤버 클래스

Map 인터페이스의 구현체를 보자. 

많은 Map 인터페이스 구현체들은 각각 `키-값 쌍을 표현하는 엔트리 객체`들을 가지고 있다. 

이런 엔트리 객체들은 Map 인터페이스 구현체들의 `멤버 클래스`일 것이다. 

모든 엔트리가 Map과 연관되어 있지만 `엔트리를 사용하는 메서드들은 Map 직접 사용하지 않는다.` 

따라서 바깥 클래스인 Map 인터페이스 구현체와 `내부 클래스인 엔트리 멤버 클래스에 관계가 없다.` 

<br>

이럴 때 `private 정적 멤버 클래스를 사용`한다. 

private 정적 멤버 클래스는 흔히 `바깥 클래스가 표현하는 객체의 한 부분 즉 구성요소를 나타낼 때 쓴다.` 

엔트리 멤버 클래스를 비정적 멤버 클래스로 표현하는 것은 낭비다. 

왜냐하면 `바깥 클래스와 관계가 없기 때문`이다. 

그리고 엔트리 멤버 클래스 선언할 때 실수로 static 빠뜨려도 Map은 여전히 동작한다. 

하지만 `모든 엔트리가 바깥 맵으로의 참조 갖게 되어 공간, 시간 낭비`하게 된다. 

<br>

따라서 이렇게 바깥 클래스의 한 부분 나타낼 때에는 private 정적 멤버 클래스 사용하는 것이다. 

<br>

***이러한 이유들 때문에 `멤버 클래스는 되도록 static으로 만들라고 하는 것`이다.***

<br>

## 🙅‍♂️ 익명 클래스

익명 클래스에는 `이름이 없다`. 

익명 클래스는 바깥 클래스의 멤버도 아니다. 

멤버와 달리 `쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다`. 

익명 클래스는 코드의 어디서든 만들 수 있다. 

바깥 클래스의 `static 아닌 메서드에 익명 클래스 있을 때만` **바깥 클래스의 인스턴스 참조**할 수 있다. 

바깥 클래스의` static 메서드에 익명 클래스 있으면` **익명 클래스는 상수 변수 이외의 static 멤버 가질 수 없다.** 

이때 익명 클래스는 상수 표현을 위해 초기화 된 final 기본 타입과 문자열 필드만 가질 수 있다. 

<br>

익명 클래스는 응용하는데 `제약`이 많다. 

- `선언한 지점에서만` 인스턴스 만들 수 있다.

- instanceOf 검사나 `클래스 이름 필요한 작업 수행할 수 없다`.

- `여러 인터페이스 구현할 수 없다`.

- `인터페이스를 구현하는 동시에 다른 클래스 상속할 수 없다`.

- 익명 클래스 사용하는 클라이언트는 그 `익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없다`.

- 표현식 중간에 등장해 10줄 이하로 짧지 않으면 `가독성이 떨어진다`.

<br>

### ❓ When?

`즉석에서 작은 함수 객체나 처리 객체를 만드는데` 익명 클래스를 주로 사용했다. 

하지만 이것은 자바가 람다를 지원하기 전에 사용하던 것이다. 

`이제는 위의 경우 람다를 사용`한다. 

<br>

그리고 또 익명 클래스는 `정적 팩터리 메서드를 구현할 때` 많이 사용한다. 

아래와 같이 사용하는 것이다. 

아래의 new AbstractList<>() {~~~}; 부분이 익명 클래스다. 

```java
public class IntArrays {
    static List<Integer> intArrayAsList(int[] a) {
        Objects.requireNonNull(a);

        // 다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능하다.
        // 더 낮은 버전을 사용한다면 <Integer>로 수정하자.
        return new AbstractList<>() {
            @Override public Integer get(int i) {
                return a[i];  // 오토박싱(아이템 6)
            }

            @Override public Integer set(int i, Integer val) {
                int oldVal = a[i];
                a[i] = val;     // 오토언박싱
                return oldVal;  // 오토박싱
            }

            @Override public int size() {
                return a.length;
            }
        };
    }
}
```

<br>

## 🌏 지역 클래스

지역 클래스는 `블록 내부에 클래스를 정의하는 경우`다. 

대표적으로 메서드 내부에서 클래스 정의하는 경우가 있다. 

마치 `메서드 내의 지역 변수처럼 사용`된다. 

지역 클래스를 포함하는 메서드 안에서만 객체를 생성 할수 있다. 

메서드의 실행이 끝나면 해당 지역 클래스가 메모리에서 사라지게 된다.

`지역 변수처럼 사용되기 때문에 메서드 나가면 메모리에서 사라지고 메서드 밖에서 사용할 수 없는 것`이다. 

<br>

지역 클래스는 지역 변수를 선언할 수 있는 곳이면 실질적으로 어디서든 선언할 수 있고, 유효 범위도 지역 변수와 같다. 

다른 `3개 중첩 클래스와의 공통점도 하나씩 가지고 있다`. 

- `멤버 클래스` : 멤버 클래스처럼 이름이 있고 반복해서 사용할 수 있다.

- `익명 클래스` : 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스 참조할 수 있으며 정적 멤버는 가질 수 없고 가독성 위해 짧게 작성해야 한다.

<br>

### ❓ When?

지역 클래스는 4가지 중첩 클래스 중 `가장 드물게 사용`된다. 

`이벤트 리스너 구현, 스레드 구현 같은 곳에서 사용`될 수 있다고 한다. 

<br>

스레드 구현의 예시 코드이다. 

```java
public class Example {
    public void startThread() {
        // Runnable을 구현한 지역 클래스 정의
        class MyRunnable implements Runnable {
            @Override
            public void run() {
                // 쓰레드가 실행할 코드
                System.out.println("Thread is running...");
            }
        }
        
        // 쓰레드 생성 및 시작
        Thread thread = new Thread(new MyRunnable());
        thread.start();
    }
}

```

<br>

하지만 `이렇게보다는 익명 클래스를 이용해 많이 구현`한다. 

`익명 클래스는 지역 클래스의 한 형태`이다. 

<br>

## ‼ 결론

중첩 클래스에는 `4가지가 있으며 각각의 쓰임이 다르다`. 

메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기엔 너무 길다면 멤버 클래스로 만든다. 

<br>

`멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면 비정적`으로 만들자. 

바깥 인스턴스 참조하지 않으면 정적으로 만들자. 

바깥 인스턴스 참조하지 않는데 비정적으로 만들면 `객체 생성 시간도 오래 걸리고 메모리도 차지하며 메모리 누수도 생길 수 있다`. 

***따라서 `item 24에서 멤버 클래스는 되도록 static으로 만들라는 것이다`.*** 

<br>

중첩 클래스가 `한 메서드 안에서만 쓰이면서 그 인스턴스 생성하는 지점이 단 한 곳이고 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있다면 익명 클래스로 만들자`. 

`그렇지 않다면 지역 클래스`로 만들자. 

<br>

## 📍 references

- [지역 클래스 1](https://sjh836.tistory.com/145)
- [지역 클래스 2](https://m.blog.naver.com/aroaroro/220567157714)