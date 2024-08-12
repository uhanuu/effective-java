# 아이템54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

## Null 반환

- 컬렉션이 비어있을 때 null로 처리하고자 한다면 아래와 같이 할 수 있다. 

```java
return list.isEmtpy() ? null : new ArrayList<>(list);
```

- 이거의 가장 큰 단점은 위의 값을 return하는 메소드를 사용하는 곳에서 null 방지를 해줘야 한다는 점입니다.
- 이렇게 쓰는 사람들의 의견은 빈 컬렉션을 반환하는 것이 비용이 들기 때문에 null로 처리해야한다는 의견입니다.

### 빈 컬렉션을 만드는 것의 문제점이 아닌 이유

- 성능 저하의 주범이라고 확인되지 않는 이상 빈 컬렉션을 반환하는 것은 성능면에서 크게 차이가 나지 않습니다.
- 빈 컬렉션과 배열을 굳이 새로 할당하지 않고 만들수 있다.

## 빈 컬렉션과 배열을 굳이 새로 할당하지 않는 방법

- static final로 처리하면 됩니다.

```java
private static final EMPTY_LIST = new ArrayList<>(); 
```

- 불변이기 때문에 공유해도 안전하다는 장점이 있습니다.
- 아울러 우리가 이렇게 만들필요가 없는게 이미 Java에는 빈 컬렉션이 구현되어 있습니다.

```java
Collections.emptyMap();
Collections.empryList();

public static final <T> List<T> emptyList() {
    return (List<T>) EMPTY_LIST;
}
```

### 배열은?

```java
return list.toArray(new Integer[]);
```

```java
private static final Integer[] EMPTY_ARRAY = new Integer[0];

public Integer[] getEmtpy() {
	return list.toArray(EMPTY_ARRAY);
}
```

- 단순히 성능을 개선할 목적이라면 toArray에 넘기는 배열을 미리 할당하는 건 추천하지 않습니다.  

```java
return list.toArray(new Integer[list.size()];
```

## 결론

- 비어 있는 컬렉션을 반환할떄 null이 아닌 빈 컬렉션을 반환하기
- null을 발환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어나며 성능이 좋은 것도 아닙니다.  
