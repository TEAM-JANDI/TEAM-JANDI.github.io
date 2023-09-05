---
layout: post
title: "[12챕터] 제네릭과 변성(한정적 와일드카드)"
subtitle: 자바의 정석 스터디 18회차
categories: Java
tags: [java, generics, 공변, 반공변, 자바의정석]
---

# 제네릭과 변성
## 변성(Variance)
변성에는 공변성, 반공변성, 무변성(불별성)이 있다. 이들에 대해 간략히 설명하자면 아래와 같다.

**공변성(Covariance)**  
공변성이 있는 경우, Sub가 Super의 하위타입이면  
Sub[]는 Super[]의 하위타입이다.(Java의 배열이 공변성을 가진다.)  
`Collection<Sub>` 는 `Collection<Super>`의 하위타입이다.

**반공변성(ContraVariance)**  
반공변성이 있는 경우, Sub가 Super의 하위타입이면  
Super[]는 Sub[]의 하위타입이다.  
`Collection<Super>`는 `Collection<Sub>`의 하위타입이다.  

**무변성(불변성 Invariance)**  
Sub가 Super의 하위타입이어도(상속관계가 있어도)     
Super[]와 Sub[]는 상속관계가 성립되지 않는다.  
`Collection<Super>`와 `Collection<Sub>`는 상속관계가 성립되지 않는다.(Java의 제네릭이 무변성을 가진다.)  

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
    numberArr[0] = 1.4; //컴파일은 통과하나 런타임 시 RuntimeException(ArrayStoreException) 발생
```
<br>

반면, 제네릭은 무변성을 가진다. 객체 간 상하관계가 적용되지 않는다.  
Number는 Integer의 상위타입이지만, < Number >는 < Integer >의 상위타입이 아니다.

``` java
    List<Number> numberList = new ArrayList<Integer>(); //컴파일에러 -> Required type:List<Number> / Provided:ArrayList<Integer>
```



## 제네릭의 무변성
제네릭은 무변성이기 때문에 동일한 타입만 허용된다.
* 때문에 지정한 타입만 들어올 것이 보장되고(타입 안정성), 이로 인해 형변환을 위한 작업이 필요 없다.
* 다만 다형성을 활용하지 못한다.


MyArrayList 클래스를 보자.  
생산자는 파라미터로 내부 요소를 생산한다.  
clone() 메서드는 파라미터에 내부 요소를 넣는다. 즉, 파라미터가 내부 요소를 소비한다. 

```java
static class MyArrayList<T> {
  Object[] element = new Object[5];
  int index = 0;

  //파라미터로 내부 요소를 '생산'
  public MyArrayList(List<T> in) {
    for (T e : in) {
      element[index++] = e;
    }
  }

  //파라미터가 내부 요소를 '소비'
  public void clone(List<T> out) {
    for (Object o : element) {
      out.add((T) o);
    }
  }
}
```
<br>

제네릭타입으로 < Number >를 주었을 때, 동일한< Number > 타입은 문제가 없다.  
그러나 Number의 하위타입인 Integer를 담거나, Number의 상위타입인 Object에 넣는 것은 불가하다.  
Number 타입만 들어온다는 것이 보장된다는 장점이 있는 반면
다형성을 활용할 수는 없는 것이다. 
```java

  MyArrayList<Number> myList = null;

  // 동일타입일 경우 에러x
  List<Number> numberList1 = Arrays.asList(1, 2, 3);
  myList = new MyArrayList<>(numberList1);

  List<Number> numberList2 = new ArrayList<>();
  myList.clone(numberList2);


  //<Number>와 <Integer>는 상속관계가 성립되지 않는다. 
  List<Integer> integerList = Arrays.asList(1, 2, 3);
  myList = new MyArrayList<>(integerList); // 컴파일에러 -> java: incompatible types

  //<Number>와 <Object>는 상속관계가 성립되지 않는다. 
  List<Object> objectList = new ArrayList<>();
  myList.clone(objectList); // 컴파일에러 -> Required type:List<Number> Provided:List<Object>

```

## 제네릭의 오버로딩
불가하다.  
컴파일러는 컴파일 시, 제네릭타입이 없는 버전과의 호환을 위해 타입이레이저로 제네릭타입을 지운다.  
때문에 제네릭타입이 상이한 것으로는 오버로딩이 되지 않는다.

배열과 제네릭의 오버로딩을 메서드를 통해 비교해보자.

``` java
    //배열은 오버로딩 가능
    public static void printArr(Object[] objArr) {}
    public static void printArr(Number[] numArr) {}
    public static void printArr(Integer[] intArr) {}

    //제네릭타입으로는 오버로딩 불가, 컴파일에러 발생
    public static void printGeneric(List<Object> objArr) {} 
    //컴파일에러 -> printGeneric(List<Object>)' clashes with 'printGeneric(List<Number>)'; both methods have same erasure
    public static void printGeneric(List<Number> objArr) {}
    public static void printGeneric(List<Integer> objArr) {}
  ```
<br><br>
무변성이라서 상속관계도 적용 못해, 타입이레이저가 지워서 오버로딩도 못해.  
제네릭을 쓰면서 다형성을 활용하고 싶으면 어떻게 할 수 있을까?


## 한정적 와일드카드
제네릭의 무변성으로 인한 문제를 해결하기 위해 한정적 와일드카드를 사용할 수 있다.  
`<? extends T>` : 상한 경계 와일드카드. T와 T의 하위타입만 들어올 수 있다.  
`<? super T>` : 하한 경계 와일드카드. T와 T의 상위타입만 들어올 수 있다.

제네릭에 변성을 주기 위해 한정적 와일드카드를 사용해보자.  
MyArrayList 클래스의 생산자에 상한 경계 와일드카드를, clone() 메서드에 하한 경계 와일드카드를 주었다.

```java
static class MyArrayList<T> {
    Object[] element = new Object[10];
    int index = 0;

    //<? extends T> -> 공변성 부여
    public MyArrayList(List<? extends T> in) {
        for (T e : in) {
            element[index++] = e;
        }
    }

    //<? super T> -> 반공변성 부여
    public void clone(List<? super T> out) {
        for (Object o : element) {
            out.add((T) o);
        }
    }
}
```
### 공변성
상한 경계 와일드카드로 공변성을 줄 수 있다.  
생산자의 파라미터를 < ? extends T > 타입으로 선언함으로써 공변성이 부여되어 < Number >가 < Integer >의 상위타입이 되었다. (정확히는 < ? extends Number > 가 < Integer > 의 상위타입)
```java
MyArrayList<Number> myList;

List<Integer> integerList = Arrays.asList(1, 2, 3);
myList = new MyArrayList<>(integerList);
```
<br>
그러나 < ? extends T >로 받은 데이터는 해당 데이터를 읽는 것만 가능하다. 해당 데이터에 쓰기를 시도하면 컴파일 에러가 발생한다. 
T의 하위타입이 들어온다는 것은 알겠으나 그것이 정확히 어떤 타입인지는 알 수 없으므로 컴파일러가 에러를 발생시키기 때문이다. 

```java
// 에러x
public void covarianceRead(List<? extends T> in) {
  for (T e : in) {
    element[index++] = e;
  }
}

// 에러o
public void covarianceWrite(List<? extends T> in) {
  in.add(0); //컴파일에러 -> Required type:capture of ? extends T / Provided:int
}

```

<br>

형변환 시에는 T 자리에 넣은 타입으로 형변환 하는 것이 안전하다. 가령 T의 타입이 Number라면 파라미터로 들어올 수 있는 타입 중 가장 최상위타입이 Number이므로 Number로 형변환하는 것이 안전하다는 것이다. 


```java
public void covarianceCast(List<? extends Number> in) {
  //in 타입이 Double 일 때
  Number n = (Number) in.get(0); //에러x
  Double d = (Double) in.get(0); //에러x
  Integer i = (Integer) in.get(0); //ClassCastException 발생
}

//...
MyArrayList<Number> myList = new MyArrayList<>(Arrays.asList());
List<Double> doubleList = Arrays.asList(1.0, 2.0, 3.0);
myList.covarianceCast(doubleList);
```


### 반공변성
하한 경계 와일드카드로 반공변성을 줄 수 있다.  
< ? super T > 를 통해 반공변성이 부여되어 < Object >가 < ? super Number >의 하위타입이 되었다.
```java
MyArrayList<Number> myList = new MyArrayList<>(Arrays.asList());

List<Object> objectList = new ArrayList<>();
myList.clone(objectList);
```
<span style="color:gray">
반공변에서 의미하는 '하위타입'이 무슨 의미인지 이해하기 어려웠다. Object가 어떻게 Number의 하위타입이 되냐고. 나름대로 정리해보면, 
다형성을 활용 할 때 파라미터와 같은 타입이거나 그 하위타입이면 해당 파라미터 자리에 들어갈 수 있다는 맥락에서, < ? super Number >의 자리에 < Object >가 들어갈 수 있으니 < Object >를 < ? super Number >의 '하위타입' 이라고 표현하는 것 같다.
</span>

<br>

< ? super Number > 로 파라미터를 세팅할 경우  
파라미터로 Number 이상의 타입만 넣을 수 있으며,
메서드 내에서 해당 파라미터를 사용 할 때는 형변환 시 Object로 하는 것이 안전하고 (Number의 상위타입 중 뭐가 올지 모르니), 삽입은 하위타입으로만 가능하다.   
< ? super Object > 로 하면? 컴파일에러는 발생하지 않으나, 모든 타입에 개방되기 때문에 타입안정성을 해치게 된다.

```java
public void contravariantCast(List<? super Number> out) {
  //형변환
  Object o = out.get(0);
  Number n = (Number) out.get(0); // out의 타입이 < Object > 이면 ClassCastException 발생

  //삽입
  out.add(1);
  out.add(1.0);
  //out.add(new Object()); //컴파일에러 -> Required type:capture of ? super Number / Provided:Object
}

//...
  
  MyArrayList<Number> myList = new MyArrayList<>(Arrays.asList(1, 2, 3));

  // 1. Number의 상위타입을 넣을 때 (< Number >의 하위타입) -> 컴파일 에러는 안 나지만 runtime 시 ClassCastException 발생할 수 있음
  List<Object> obejctList = Arrays.asList("1", "2", "3");
  myList.contravariantCast(numberList2);

  // 2. Number의 하위타입을 넣을 때 ( <Number >의 상위타입)
  List<Double> doubleList = Arrays.asList(1.0, 2.0, 3.0);
  myList.contravariantCast(doubleList3); //컴파일에러

```




## PECS
공변성과 반공변성을 언제 부여해야 할지 헷갈린다면, PECS 공식을 기억하면 좋다.
- Producers : Extend
- Consumers : Super  
행위의 주체를 제네릭타입 파라미터로 생각하고 이해하면 된다. 

```java
//Producers
//파라미터가 내부 요소를 '생산'하는 데 쓰인다.
  public MyArrayList(Collection<? extends T> in) {
      for (T e : in) {
          element[index++] = e;
      }
  }

//Consumers
//파라미터가 내부 요소를 '소비'한다.
  public void clone(Collection<? super T> out) {
      for (Object o : element) {
          out.add((T) o);
      }
  }
```

---
출처  
자바 제네릭의 공변성 & 와일드카드 완벽 이해 : <https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%A0%9C%EB%84%A4%EB%A6%AD-%EC%99%80%EC%9D%BC%EB%93%9C-%EC%B9%B4%EB%93%9C-extends-super-T-%EC%99%84%EB%B2%BD-%EC%9D%B4%ED%95%B4>  
프로그래밍 초식 - 지네릭 변성(java) (최범균) : <https://www.youtube.com/watch?v=PtM44sO-A6g> 
