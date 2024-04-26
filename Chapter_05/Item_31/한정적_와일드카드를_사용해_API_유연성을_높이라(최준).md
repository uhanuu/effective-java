# 3️⃣1️⃣ Item 31 : 한정적 와일드카드를 사용해 API 유연성을 높이라

<br>

## 📌 목차
1. 불공변 
2. 불공변의 불편함 1
3. 해결책
4. 불공변의 불편함 2
5. 해결책
6. 와일드카드 타입 사용 공식
7. Tip
8. 타입 매개변수 vs 와일드카드
9. 결론 

<br>

## 🚫 불공변

매개변수화 타입은 `불공변`이다. 

서로 다른 타입 Type1, Type2 있을 때 List< Type1>과 List< Type2>는 하위 타입도 상위 타입도 아닌 `아무 관계 없는 타입`이다. 

그리고 Type1의 하위 타입 Type2가 있을 때 List< Type1>는 List< Type2>의 `상위 타입이 아니다`. 

예시를 보면 더 이해가 된다. 

<br>

List< String>은 List< Object>의 하위 타입이 아니다. 

왜냐하면 List< Object>에는 어떤 객체도 넣을 수 있다. 

하지만 List< String>에는 문자열만 넣을 수 있다. 

하위 타입이라면 상위 타입의 일을 할 수 있어야 하는데 List< String>은 List< Object>가 하는 일을 제대로 수행하지 못한다. 

따라서 하위 타입이 될 수 없는 것이다. 

<br>

## 🚨1️⃣ 불공변의 불편함 1

그런데 이런 불공변 방식은 불편한 점이 있다. 

<br>

`Stack< E> 클래스에 Iterable< E> 타입의 원소를 push`하는 pushAll 메서드 추가한다고 생각해보자. 

Iterable은 for-each 루프 또는 Iterator를 사용하여 반복하거나 반복할 수 있는 개체 컬렉션을 나타내는 인터페이스이다. 

따라서 `Iterable< E>는 E 타입의 값을 반복할 수 있는 타입`인 것이다. 

그래서 ArrayList, Queue, Set 이런 컬렉션에 E 타입 값들 담아서 한번에 stack에 넣고 싶은 것이 pushAll 메서드다. 

<br>

pushAll 메서드는 아래와 같다. 

```java
public void pushAll(Iterable<E> src){
		for(E e : src)
				push(e);
}
```

<br>

이 메서드는 컴파일 된다. 

하지만 완벽하지는 않다. 

왜냐하면` Iterable< E> src의 원소 타입 E가 Stack< E>의 원소 타입 E와 같다면 잘 작동`한다. 

<br>

하지만 Stack< Number>인데 pushAll(Iterable< Integer>) 하면 어떻게 되는가?

Stack< Number>에 Integer 타입을 넣으려고 하는 것이다. 

`Integer는 Number의 하위 타입이니 잘 동작할 것 같다.` 

`그리고 이렇게 되기를 원한 것이다. `

<br>

하지만 실제로는 매개변수화 타입이 불공변이기 때문에 오류 메시지가 뜬다. 

Stack< Number>이기 때문에 pushAll의 `파라미터도 Iterable< Number>` src로 되는 것이다. 

그런데 `Iterable< Number>와 Iterable< Integer>는 불공변`이기 때문에 다르다. 

따라서 파라미터로 받을 수 없고 오류가 발생하는 것이다. 

이것이 `불공변 방식의 불편한 점`이다. 

<br>

## ✅ 해결책

이런 불공변의 불편한 점을 해결해주는 방법이 있다. 

바로 `한정적 와일드카드 타입`이라는 특별한 매개변수화 타입인 것이다. 

<br>

>한정적 와일드카드 타입(bounded wildcard types)은 자바의 제네릭(generics)을 사용할 때 `타입 매개변수의 범위를 제한`하는 방법이다.
>
>한정적 와일드카드를 사용함으로써, 제네릭 타입의 사용을 더 유연하게 만들 수 있다. 
>
>한정적 와일드카드를 사용하면 특정 타입을 기준으로 상한 범위와 하한 범위를 지정함으로써 호출 범위를 확장 또는 제한할 수 있다. 
>
>한정적 와일드카드에는 `상한 경계 와일드카드`(Upper Bounded Wildcard)와 `하한 경계 와일드카드`(Lower Bounded Wildcard)가 있다
>
>주로 `extends 키워드나 super 키워드를 사용`하여 타입의 상한(upper bound) 또는 하한(lower bound)을 지정한다.

<br>

위에서 원하는 것은 Stack< Number>에 Number의 하위 타입인 Integer 타입 넣으려고 하는 것이다.  

그런데 pushAll 메서드의 파라미터가 Iterable< Number>라 Iterable< Integer>는 들어갈 수 없었던 것이다. 

pushAll 메서드의 파라미터가 Iterable< E>가 아니라 `Iterable< E의 하위타입>`이어야 우리가 원하는 것이 된다. 

<br>

Iterable< E의 하위타입> 이렇게 할 수 있는 것이 한정적 와일드카드 타입인 것이다. 

바로 `Iterable< ? extends E>` 이렇게 하면 우리가 원하는 것을 할 수 있다. 

모든 타입을 대신할 수 있는 와일드카드 타입을 타입 매개변수로 하는데 `와일드카드 타입의 최상위 타입을 정의해서 상한 경계를 설정`하는 것이다. 

이렇게 `< ? extends E>를 상한 경계 와일드카드`라고 한다. 

<br>

>그래서 메서드 파라미터로 받을 때에는 E나 E의 하위 타입이면 모두 받을 수 있다. 
>
>하지만 메서드 파라미터로 E의 상위 타입은 넣을 수 없다. 
>
>그리고 꺼내서 사용할 때에는 E나 E의 상위 타입으로만 꺼낼 수 있는 것이다. 
>
>E의 하위 타입으로는 꺼낸 것을 받을 수 없다. 
>
>그리고 넣을 때에는 어떤 타입을 넣어도 오류가 발생한다. 
>
>왜냐하면 E의 하위 타입이 여러 개일 수 있는데 어떤 타입인지 모르기 때문이다. 
>
>그리고 상위 타입은 애초에 넣을 수 없기 때문이다. 

<br>

`메서드 파라미터로 받는 예시`

```java
    public static void main(String[] args) {
        Stack< Number> stack = new Stack< >();

        Iterable< Integer> test1 = new ArrayList< >();
        Iterable< Number> test2 = new ArrayList< >();
        Iterable< Object> test3 = new ArrayList< >();

        stack.pushAll(test1);
        stack.pushAll(test2);
        stack.pushAll(test3);    < <  컴파일 에러 발생
    }
```

<br>

`꺼내서 사용할 때 예시`

```java
    public void pushAll(Iterable< ? extends Number> src){
        for (Integer number : src) {        < <  컴파일 에러 발생

        }

        for (Number number : src) {

        }

        for (Object number : src) {

        }
    }
```

<br>

`넣으려고 하는 예시`

```java
public void test(ArrayList< ? extends Number> dst){
        Integer integer = 3;
        Number number = 3;
        Object object = 3;

        dst.add(integer);      < <  컴파일 에러 발생
        dst.add(number);       < <  컴파일 에러 발생
        dst.add(object);       < <  컴파일 에러 발생
}
```

<br>

이렇게 pushAll(Iterable< ? extends E> src)로 바꾸게 되면 Stack은 물론 Stack을 사용하는 클라이언트 코드도 말끔하게 컴파일이 된다. 

즉 모든 것이 타입 안전하다는 뜻이다. 

<br>

## 🚨2️⃣ 불공변의 불편함 2

다음 불공변의 불편함을 보자. 

<br>

popAll 메서드를 작성하려고 한다. 

popAll 메서드는 Stack 안의 모든 원소를 주어진 컬렉션으로 옮겨담는 메서드다. 

코드는 아래와 같다. 

```java
public void popAll(Collection< E> dst){
		while(!isEmpty()){
				dst.add(pop());
		}
}
```

<br>

이 메서드도 `컴파일 되지만 완벽하지는 않다`. 

왜냐하면 주어진 컬렉션의 원소 타입이 스택의 원소 타입과 `일치할 때만 컴파일되고 문제없이 동작`한다. 

<br>

Stack< Number>의 원소를 Object용 컬렉션에 옮기려고 한다. 

popAll(Collection< Object>) 이렇게 하려는 것이다. 

`Object는 Number의 상위 타입이기 때문에 잘 될 것 같지만 역시 컴파일 오류가 발생`한다. 

Collection< Object>는 Collection< Number>의 하위 타입이 아니라고 하며 컴파일 오류가 발생한다. 

역시` 불공변 때문에 생기는 문제`이다. 

<br>

## ✅ 해결책

이번에도 `한정적 와일드카드 타입으로 해결`을 할 수 있다. 

이번에는 popAll의 입력 매개변수의 타입이 Collection< E>가 아니라 `Collection< E의 상위 타입>`이어야 한다. 

<br>

이렇게 Collection< E의 상위 타입> 할 수 있는 것이 `Collection< ? super E>`이다. 

Collection< ? super E>는 위의 해결책과는 반대인 `하한 와일드카드 타입`이다. 

상한 경계와 반대로 `super를 사용해 와일드카드의 최하위 타입을 정의하여 하한 경계를 설정`하는 것이다. 

? super E 형태의 와일드카드는, E 타입이거나 E의 상위 타입(supertype)인 어떤 타입도 될 수 있다는 것을 의미한다. 

이런 형태는 주로 출력을 처리할 때 사용되며, 주어진 타입이나 그 상위 타입의 객체에 데이터를 쓰는 데 사용된다. 

<br>

>그래서 메서드 파라미터로 받을 때에는 E나 E의 상위 타입이면 모두 받을 수 있다. 
>
>하지만 메서드 파라미터로 E의 하위 타입은 넣을 수 없다. 
>
>그리고 꺼내서 사용할 때에는 E의 상위 타입으로만 꺼낼 수 있는 것이다. 
>
>E나 E의 하위 타입으로는 넣거나 꺼낸 것을 받을 수 없다. 
>
>그리고 넣을 때에는 E나 E의 하위 타입만 넣을 수 있다. 
>
>왜냐하면 넣는 것이 최소 E 타입이고 아니면 E의 상위 타입이기 때문에 E나 E의 하위 타입들은 넣을 수 있다. 
>
>하지만 E의 상위 타입인 경우 어떤 상위 타입인지 알 수 없어 넣을 수 없다. 

<br>

`메서드 파라미터로 받는 예시`

```java
        Collection< Integer> test11 = new ArrayList< >();
        Collection< Number> test22 = new ArrayList< >();
        Collection< Object> test33 = new ArrayList< >();

        stack.popAll(test11);        < <  컴파일 에러 발생
        stack.popAll(test22);
        stack.popAll(test33)
```

<br>

`꺼내서 사용할 때 예시`

```java
    public void test(Collection< ? super Number> dst){
        for (Object o : dst) {

        }

        for (Number o : dst) {       < <  컴파일 에러 발생

        }

        for (Integer o : dst) {      < <  컴파일 에러 발생

        }
    }
```

<br>

`넣으려고 할 때 예시`

```java
    public void test(Collection< ? super Number> dst){
        Integer integer = 3;
        Number number = 3;
        Object object = 3;

        dst.add(integer);
        dst.add(number);
        dst.add(object);      < <  컴파일 에러 발생
  }
```

<br>

이렇게 하면 Stack과 클라이언트 코드 모두 깔끔히 컴파일 된다. 

<br>

## 📝 와일드카드 타입 사용 공식

위에서 불편한 점과 해결책에서 주는 메시지는 아래와 같다. 

- 유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입 사용하라

- 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을게 없다.

<br>

`언제 와일드카드 타입을 써야 하는지` 잘 모르겠다. 

와일드카드 타입을 언제 사용하면 좋은지 기억하는데 `좋은 공식`이 있다. 

<br>

### `PECS : producer-extends, consumer-super`

<br>

>매개변수화 타입 T가 `생산자` : < ? `extends` T>
>
>매개변수화 타입 T가 `소비자` : < ? `super` T>

이렇게 사용하라는 공식이다. 

<br>

Stack 클래스 예시에서 pushAll 메서드의 파라미터 Iterable< ? extend T> src는 Stack이 사용할` T 인스턴스를 생산 = 꺼내주니까 extends를 사용`한 것이다. 

변수명에서도 알 수 있듯이 원소를 꺼내는, 생산하는 source = src인 것이다. 

<br>

Stack 클래스 예시에서 popAll 메서드의 파라미터 Collection< ? super T> dst는 Stack으로부터 `T 인스턴스를 소비 = 받으니까 super를 사용`한 것이다. 

변수명에서도 알 수 있듯이 원소를 받는, 소비하는 destination = dst인 것이다. 

<br>

## ❇ Tip

와일드카드 타입을 사용할 때 알아두면 좋을 `tip`에 대해서 보자. 

- `반환 타입`에는 한정적 와일드카드 타입을 `사용하면 안된다`.
    - 클라이언트 코드에서도 와일드카드 타입을 써야 하기 때문이다.

- 클래스 사용자가 와일드카드 타입을 신경 써야 한다면 `API에 문제가 있을 가능성`이 크다.

- 자바 7 이전에는 와일드카드 타입 사용할 때 `반환 타입을 명시적 타입 인수로 명시`해야 한다.
    - 컴파일러의 타입 추론 능력이 강력하지 못했기 때문에 명시적 타입 인수를 추가해줘야 한다.

- Comparable, Comparator 인터페이스는 언제나 소비자이므로 Comparable< E>보다는 `Comparable< ? super E>를 사용하는 낫다`.
    - Comparable, Comparator는 E 인스턴스를 소비하고 선후 관계를 뜻하는 정수(음수, 0, 양수)를 생산한다.

<br>

## 🆚 타입 매개변수 vs 와일드카드

`타입 매개변수와 와일드카드에 공통되는 부분`이 있다. 

그래서 메서드를 정의할 때 둘 중 어느 것을 사용해도 괜찮을 때가 많다. 

<br>

swap 메서드를 타입 매개변수, 와일드카드 각각으로 정의한 코드를 보자.

```java
public static < E> void swap(List< E> list, int i, int j);
public static void swap(List< ?> list, int i, int j);
```

<br>

어떤 선언이 낫고 더 나은 이유는 무엇인가?

`public API라면 간단한 두번째가 낫다. `

<br>

따라서 `타입 매개변수를 와일드카드로 바꾸는 것`이 좋다. 

그렇다면 어떻게 바꾸는지 보자. 

<br>

### 1️⃣ 1단계

메서드 선언에 타입 매개변수가 한번만 나오면 `와일드카드로 대체`하자. 

비한정적 타입 매개변수라면 비한정적 와일드카드로 바꾸면 된다. 

한정적 타입 매개변수라면 한정적 와일드카드로 바꾸면 된다. 

<br>

### 2️⃣ 2단계

와일드카드 타입의 `실제 타입을 알려주는 private 도우미 메서드 따로 작성`해 활용하자. 

왜 이런 것을 하는가 하면 `와일드카드 타입을 사용하면 null외에는 어떤 값도 넣을 수 없기 때문`이다. 

위의 swap 메서드에서 보면 List< ?>이기 때문에 null외에는 어떤 값도 List에 넣을 수 없다. 

<br>

이럴 때 런타임 오류 낼 가능성 있는 형변환이나 로 타입을 사용하지 않고도 해결하는 방법이 있다. 

바로 와일드카드 타입의 실제 타입을 알려주는 private 도우미 메서드 따로 작성해 활용하는 것이다. 

<br>

swap의 경우 도우미 메서드는 아래와 같다. 

```java
// 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
private static < E> void swapHelper(List< E> list, int i, int j){
		// 로직 구현...
}
```

<br>

도우미 메서드 swapHelper 메서드는 `리스트가 List< E>임을 알고 있는 것`이다. 

즉 이 리스트에서 꺼낸 값의 타입이 항상 E이고 E의 타입 값이라면 이 리스트에 넣어도 안전함을 알고 있다. 

따라서 이제 `와일드카드 타입을 사용한 메서드에도 null 이외의 값을 넣을 수 있는 것`이다. 

<br>

swap 메서드 내부에서는 더 복잡한 제너릭 도우미 메서드를 이용했다. 

하지만 덕분에 외부에서는 와일드카드 기반의 멋진 선언을 유지할 수 있다. 

swap 메서드 호출하는 클라이언트는 복잡한 swapHelper 존재 모른채 그 혜택을 누리는 것이다. 

<br>

## ‼ 결론

조금 복잡하더라도 `와일드카드 타입을 적용하면 API가 훨씬 유연`해진다. 

그러니 널리 쓰일 라이브러리를 작성한다면 반드시 와일드카드 타입을 적절히 사용해줘야 한다. 

그리고 `한정적 와일드카드 타입을 사용할 때에는 PECS 공식`을 기억하자.

<br>

## 📍 references

- [iterable 1](https://velog.io/@jaybon/%EC%9E%90%EB%B0%94-Iterable)
- [iterable 2](https://devlog-wjdrbs96.tistory.com/84)
- [한정적 와일드카드 타입 1](https://mangkyu.tistory.com/241)
- [한정적 와일드카드 타입 2](https://parkmuhyeun.github.io/woowacourse/2023-06-25-generics/)