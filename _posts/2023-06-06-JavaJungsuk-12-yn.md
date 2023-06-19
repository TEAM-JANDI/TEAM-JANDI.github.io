---
layout: post
title: "[12챕터] 제네릭과 와일드카드의 차이점은 무엇일까"
subtitle: 자바의 정석 스터디 18회차
categories: Java
tags: [java, generics, wildcard, 자바의정석]
---

678쪽에 나온 내용인

스태틱 메서드에 타입 매개변수 T를 사용할 수 없다는게 무슨말일까.. 

1. 제네릭 메서드로 사용하면 사용할 수 있지 않나? 
2. 이것 때문에 와일드 카드를 만들었다는게 이해가 안됨.
3. 제네릭 클래스가 아니어도 제네릭 메서드는 사용할 수 있는데..그럼 와일드 카드가 필요한 경우는 무엇일까..?

위 궁금증에 의해 정리를 해보았다.
<br>

## 제네릭과 와일드 카드의 차이점

제네릭은 타입 매개변수를 사용하여 타입 안정성을 제공하는 반면, 와일드카드는 다양한 타입을 수용하거나 제한하기 위해 사용된다. 제네릭은 타입을 파라미터화하는 방식이고, 와일드카드는 타입을 불특정하게 나타내는 방식이다.

…그래도 납득이 안된다.

왜 납득이 안되냐면 와일드카드를 넣는 예제들을 보면 전부 제네릭이나 오브젝트로도 커버가 가능한데, 왜 굳이 와일드카드라는 애를 사용했는지가 이해가 안된다. 뭔가 더 장점이 있으니까 사용을 하는 걸텐데, 그게 뭘까.

이것 저것 예시를 찾아보았다.

Optional 내부 코드를 보면

```java
private static final Optional<?> EMPTY = new Optional<>(null);

public static<T> Optional<T> empty() {
      @SuppressWarnings("unchecked")
      Optional<T> t = (Optional<T>) EMPTY;
      return t;
}
```

위와 같은 코드가 있다. 

EMPTY는 사실상 Object Empty = new Optional<>(null); 로 선언되어도 흐름상 아무 차이는 없다.

그러나 Optional<?> 로 선언하여 EMPTY가 항상 Optionl 타입의 객체만을 갖는다는 것을 명시해주면 이후에 코드 파악이 더 쉬워진다.
<br> <br>

# List<T> 와 List<?> 의 차이점

List<T> : T 라는 타입의 리스트라는 것을 나타냄

List<?> : 어떠한 타입의 리스트이긴 한데, 그 타입이 뭔지는 모른다는 것을 나타냄. 또한 List<?>으로 선언하면 null만 add 할 수 있고 읽기만 가능해진다. 

예를 들면 아래 코드는 컴파일 에러가 난다. 

```java
static void fromArrayToCollection(Object[] a, Collection<?> c) {
    for (Object o : a) {
        c.add(o); // compile-time error
    }
}
```

왜냐하면 와일드카드를 사용하면 add가 불가능하기 때문에.

그럼 이 경우에 Collection<Object> 와 Collection<T> 중에 더 적합한 것은 무엇일까?

Collection<Object>를 사용하면 호출시에 형변환이 일어날 수 있으므로 Collection<T>가 더 적합하다고 할 수 있다.

<br> <br>
# 그럼 와일드 카드는 대체 어느 경우에 사용하는가?

아래와 같은 코드가 있다.

```java
interface Collection<E> {
    public boolean containsAll(Collection<?> c);
    public boolean addAll(Collection<? extends E> c);
}
```

이 코드는 generic을 사용해도 무방하다.

```java
interface Collection<E> {
    public <T> boolean containsAll(Collection<T> c);
    public <T extends E> boolean addAll(Collection<T> c);
}
```

그러나 이 코드를 잘 들여다보면,

containsAll과 addAll 에서 타입 파라미터 T는 한번만 사용되며, 반환 타입은 타입 파라미터에 의존하지 않는다. 즉 타입 파라미터 T가 오직 다형성과 다양한 타입의 매개변수를 받기 위해 사용되고 있다는 것을 알 수 있다. 

바로 이러한 경우가 와일드 카드를 사용하는 것이 더 적합한 경우이다. 즉, 와일드 카드는 여러 다른 타입의 객체를 처리하는 메서드를 작성할 때 사용되면 적합하고 제네릭은 타입에 따라 다르게 동작하는 메서드를 작성할 때 유용하다. 위의 예시에서 containsAll과 addAll은 타입에 따라 다르게 동작하지 않으며 다양한 타입의 컬렉션을 처리해야하기 때문에 둘 중 와일드 카드가 적합하다.

물론 제네릭을 사용해서 처리할 수도 있지만, 와일드 카드를 사용하는 게 좀더 의도가 명확해지고 코드를 작성하기 편리할 때도 있기 때문에 와일드 카드를 사용하는 것이 적합할 때는 와일드 카드를 사용하자. 

```java
// 제네릭을 사용한 경우
class Collections {
    public static <T, S extends T> void copy(List<T> dest, List<S> src) {
    /* 
			S extends T 는 오직 매개변수를 받기 위해서만 사용되었다. 
			즉, 와일드카드로 대체하기 좋은 상황이라는 뜻이다.
    */
	  ...
}
```

```java
// 와일드 카드를 사용한 경우
class Collections {
    public static <T> void copy(List<T> dest, List<? extends T> src) {
    ...
}
```

참고 문서 : https://docs.oracle.com/javase/tutorial/extra/generics/methods.html


<br>
그 외 12챕터 관련 참고할만한 코드 예제

제네릭 람다 예제

```java
interface MyFunction<T> {
    T apply(T t);
}

public class GenericLambdaExample {
    public static void main(String[] args) {
        MyFunction<Integer> addOne = (num) -> num + 1;
        Integer result = addOne.apply(5);
        System.out.println(result); // 출력: 6

        MyFunction<String> concatenate = (str) -> str + " World";
        String message = concatenate.apply("Hello");
        System.out.println(message); // 출력: Hello World
    }
}
```

제네릭 비동기 콜백 인터페이스 예제
```java
interface Callback<T> {
    void onComplete(T result);
}

class AsyncWorker {
    public <T> void doAsyncWork(T data, Callback<? super T> callback) {
        // 비동기 작업 수행
        // 작업이 완료되면 결과를 전달하고 콜백 메서드 호출
        T result = // 작업 결과;
        callback.onComplete(result);
    }
}

class Main {
    public static void main(String[] args) {
        AsyncWorker worker = new AsyncWorker();

        Callback<Number> callback = new Callback<Number>() {
            @Override
            public void onComplete(Number result) {
                System.out.println("작업이 완료되었습니다. 결과: " + result);
            }
        };

        worker.doAsyncWork(10, callback);
    }
}
```