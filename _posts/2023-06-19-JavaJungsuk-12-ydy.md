---
layout: post
title: "[12회차] 제네릭과 변성(한정적 와일드카드)
subtitle: 자바의 정석 스터디 18회차
categories: Java
tags: [java, generics, 공변, 반공변, 자바의정석]
---

# 제네릭과 변성
## 변성(Variance)
변성에는 공변성, 반공변성, 무변성(불별성)이 있다. 이들에 대해 간략히 설명하자면 아래와 같다.
(예시에서 사용할 배열과 컬렉션을 사용했다.)

**공변성(Covariance)**  
Sub가 Super의 하위타입이면  
Sub[]는 Super[]의 하위타입이다.(Java의 배열이 공변에 해당한다.)  
`Collection<Sub>` 는 `Collection<Super>`의 하위타입이다.

**반공변성(ContraVariance)**  
Sub가 Super의 하위타입이면  
Super[]는 Sub[]의 하위타입이다.  
`Collection<Super>`는 `Collection<Sub>`의 하위타입이다.  

**무변성(불변성 Invariance)**  
Sub가 Super의 하위타입이어도(상속관계가 있어도)     
Super[]와 Sub[]는 상속관계가 성립되지 않는다.  
`Collection<Super>`와 `Collection<Sub>`는 상속관계가 성립되지 않는다.(Java의 제네릭이 반공변에 해당한다.)  

무슨 말인가 싶다.


## 변성에서 배열과 제네릭의 차이
우선, java에서 공변에 해당하는 예시는 배열이고, 무변성에 해당하는 예시는 제네릭이다.
배열은 공변성을 가진다.
다형성을 활용하기 위해 공변성을 가지게 되었다.
``` java
    Number[] numberArr = new Integer[10]; //업캐스팅 가능
    Integer[] integerArr = (Integer[]) numberArr; //다운캐스팅 가능
```
다만 이로인해 아래의 코드는 RuntimeException이 발생할 수 있다.
``` java
    Number[] ints2 = new Integer[10];
    ints2[0] = 1.4; //컴파일은 통과하나 런타임 시 RuntimeException(ArrayStoreException) 발생
```
반면, 제네릭은 무변성을 가진다. 객체간 상하관계가 적용되지 않는다.
Number는 Integer의 상위타입이지만, Collection<Number>는 Collection<Integer>의 상위타입이 아니다.

``` java
    List<Number> numberList = new ArrayList<Integer>(); //컴파일에러 -> Required type:List<Number> / Provided:ArrayList<Integer>
```
메서드를 만들어서 비교해보자.  
배열은 상속관계가 적용되므로 컴파일에러가 발생하지 않는다.  
반면 제네릭은 상속관계가 적용되지 않기 때문에 컴파일에러가 발생한다.
``` java
    //오버로딩 불가, 컴파일에러
    // -> intGeneric(List<Object>)' clashes with 'printGeneric(List<Number>)'; both methods have same erasure
    public static void printGeneric(List<Object> objArr) {...}
    public static void printGeneric(List<Number> objArr) {...}
  ```

## 한정적 와일드카드
제네릭의 무변성으로 인한 문제를 해결하기 위해 한정적 와일드카드를 사용할 수 있다.  
`<? extends T>` : 상한 경계 와일드카드. T와 T의 하위타입만 들어올 수 있다.  
`<? super T>` : 하한 경계 와일드카드. T와 T의 상위타입만 들어올 수 있다.  

파라미터의 값을 내부 요소에 복사하고,
파라미터에 내부 요소를 넣는 기능을 하는 클래스를 만들어보자.

```java
static class MyArrayList<T> {
    Object[] element = new Object[5];
    int index = 0;

//파라미터로 받아온 리스트로 내부 배열의 요소를 생산하는 생산자
//파라미터가 내부 요소를 '생산'하는 데 쓰인다.
    public MyArrayList(Collection<T> in) {
        for (T e : in) {
            element[index++] = e;
        }
    }

//파라미터로 받아온 리스트에 내부 배열을 담는 메서드
//파라미터가 내부 요소를 '소비'한다.
    public void clone(Collection<T> out) {
        for (Object o : element) {
            out.add((T) o);
        }
    }
}
```

무변성을 가진 제네릭 그대로 사용할 경우, 아래와 같이 컴파일 에러가 발생한다.

```java
    MyArrayList<Number> myList = null;

    Collection<Integer> collInteger = Arrays.asList(1, 2, 3);
    myList = new MyArrayList<>(collInteger);//컴파일에러. collInteger의 제네릭타입을 Number로 변경해주어야 컴파일에러가 발생하지 않는다.

    MyArrayList<Number> myList2 = null;

    Collection<Number> collNumber = Arrays.asList(1, 2, 3);
    myList = new MyArrayList<>(collNumber);//타입 일치시킨 후 생성 성공

    Collection<Object> objList = new LinkedList<>();
    myList2.clone(objList);//컴파일 에러. objList의 제네릭타입을 Number로 변경해주어야 컴파일에러가 발생하지 않는다.
```

제네릭에 변성을 주기 위해 한정적 와일드카드를 사용해보자.

```java
static class MyArrayList<T> {
    Object[] element = new Object[5];
    int index = 0;

//파라미터가 내부 요소를 '생산'하는 데 쓰인다.
//<? extends T> -> 공변성 부여
    public MyArrayList(List<? extends T> in) {
        for (T e : in) {
            element[index++] = e;
        }
    }

//파라미터가 내부 요소를 '소비'한다.
//<? super T> -> 반공변성 부여
    public void clone(List<? super T> out) {
        for (Object o : element) {
            out.add((T) o);
        }
    }
}
```
### 공변성
공변성을 갖게 된 생성자는 제네릭 타입 간 상속관계를 갖게 되어, 아래의 코드에서 컴파일에러를 발생시키지 않는다.

```java
MyArrayList<Number> myList;

// <? extends T> 를 통해 공변성이 부여됨.
List<Integer> collInteger = Arrays.asList(1, 2, 3, 4, 5);
myList = new MyArrayList<>(collInteger);
```

그러나 공변성을 가진 제네릭으로 받은 데이터는 읽기만 가능하고 쓰기는 불가능하다.(쓰기 시도 시 컴파일 에러 발생)
`<T>`에서 T의 하위타입이 들어온다는 것 외에 정확한 어떤 타입이 들어오는지는 알 수 없으므로 컴파일러가 에러를 발생시키기 때문이다.

```java
public void covariantTest(List<? extends T> in) {
    for (Object o : element) {
        in.add(o);//컴파일에러 발생
    }
}
```


반공변성을 갖게 된 메서드는 아래의 코드에서 컴파일에러를 발생시키지 않는다.  
`<? superT>`를 통해 반공변성이 부여되어, `Obejct`가 `Number`의 상위타입일 때, `Collection<Object>`가 `Collection<Number>`의 하위타입이 되었다.  
<span style="color:gray">
사실 반공변에 대해 찾아보다가, 반공변에서 의미하는 '하위타입'이 무슨 의미인지 이해하기 어려웠다. 나름대로 정리해보면, 
다형성을 활용 할 때 메서드 파라미터와 같은 타입이거나 그 하위타입이면 해당 파라미터 자리에 들어갈 수 있다는 맥락에서 '하위타입' 이라고 표현하는 것 같다.
</span>
```java
// <? super T> 를 통해 반공변성이 부여됨.
MyArrayList<Number> myList2 = null;

List<Integer> collNumber = Arrays.asList(1, 2, 3);
myList2 = new MyArrayList<>(collNumber);

List<Object> objList = new ArrayList<>();
myList2.clone(objList);
```

그러나 반공변을 주어 받은 데이터는 쓰기만 가능하고 읽기는 불가능하다.

```java
public void Contravariant(List<? super T> out) {
    for (T e : out) { //컴파일에러 발생
        element[index++] = e;
    }
}
```

공변성과 마찬가지로 T와 T의 상위타입이 들어온다는 것은 알겠으나 정확한 타입을 모르기 떄문에, 타입 안정성을 위해 컴파일러가 에러를 발생시킨다.  
`<? super T>`를 `Object`로 바꾸면 컴파일에러는 발생하지 않으나, 모든 타입에 개방되기 때문에 타입안정성을 해치게 된다.


## PECS
공변성과 반공변성을 언제 부여해야 할지 헷갈린다면, PECS 공식을 기억하면 좋다.
- Producers : Extend
- Consumers : Super  
MyArrayList 클래스에 있던 주석이 이를 의미한 것이다.  
행위의 주체를 제네릭타입 파라미터로 생각하고 이해하면 된다.
```java
//Producers : Extend
//파라미터가 내부 요소를 '생산'하는 데 쓰인다.
  public MyArrayList(Collection<T> in) {
      for (T e : in) {
          element[index++] = e;
      }
  }

//Consumers : Super
//파라미터가 내부 요소를 '소비'한다.
  public List<Number> clone(Collection<T> out) {
      for (Object o : element) {
          out.add((T) o);
      }
      return (List<Number>) out;
  }
```
