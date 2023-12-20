---
layout: post
title: "[14챕터] stream의 Lazy Evaluation"
subtitle: 자바의 정석 스터디 22회차
categories: Java
tags: [Java, 자바의정석, stream, LazyEvaluation]
---


## Stream API
Stream API란 Java 8에서 도입된 기능으로 데이터의 흐름을 선언적으로 처리할 수 있게 해주는 인터페이스이다. 여기서 데이터란 컬렉션, 배열 등을 말한다.  

스트림의 작업 과정은 세 단계로 나눌 수 있다.
1. 컬렉션 지정 
2. 중간연산(파이프라인 구성) 
3. 최종연산(파이프라인 실행 및 결과 생성)

**중간연산** : filter, map, distinct, limit, skip, sorted, peek 등의 작업. 여러 번 호출 가능하다.  
**최종연산** : forEach, collect, reduce, count, findFirst, max 등의 작업. 한 번만 호출 가능하다.


## Lazy Evaluation(지연 연산)
Java Stream은 Lazy Evaluation을 통해 연산을 최적화한다. 
주로 함수형 프로그래밍에서 볼 수 있는 계산 전략 중 하나이며, 지연 연산 혹은 게으른 연산 등으로 불린다. 
연산 코드가 주어졌을 때 바로 연산을 시작하는 것이 아니라, 최종연산이 호출 될 때 연산을 실행하는 방법이다. 


근데 Lazy Evaluation이 어떻게 동작하는 것이기에 연산을 최적화 해준다는 걸까?

1. 2의 배수이면서
2. 3의 배수인 것들 중
3. 15 이하인 것들 중에서
4. 첫 번째 값을 찾는다.

위의 작업을 처리하는 코드를 만들어보자.

```java
List<Integer> nums = Arrays.asList(
        1, 2, 3, 4, 5, 6, 7, 8, 9, 10,
        11, 12, 13, 14, 15, 16, 17, 18, 19, 20);

Integer num = nums.stream()
        .filter(n -> n % 2 == 0)
        .filter(n -> n % 3 == 0)
        .filter(n -> n <= 15)
        .findFirst().get();

System.out.println(num);
```

<br>
어떤 순서로 진행되는지 확인하기 위해서 단계별로 출력문을 추가했다. 

```java
Integer num = nums.stream()
        .filter(n -> {
            System.out.println("first filter " + n);
            return n % 2 == 0;
        })
        .filter(n -> {
            System.out.println("second filter " + n);
            return n % 3 == 0;
        })
        .filter(n -> {
            System.out.println("third filter " + n);
            return n <= 15;
        })
        .findFirst().get();

System.out.println(num);
```

<br>
코드를 실행해보기 전에는 이 코드가 다음과 같은 순서로 동작할 것이라 생각하였다.  

1. 1부터 20까지 중에 2의 배수를 찾는다.
2. 1의 결과 중, 3의 배수를 찾는다.
3. 2의 결과 중, 15 이하인 수를 찾는다.
4. 3의 결과 중, 첫 번째 값을 찾는다.

결과적으로 아래와 같이 출력될 것으로 예상했다.
```text
first filter 1
first filter 2
first filter 3
...
first filter 20
second filter 2
second filter 4
second filter 6
...
second filter 20
third filter 6
third filter 12
third filter 18
6
```

<br>
그러나 실제 결과는 아래와 같았다. 
함수를 쭉 수행한 것이 아니라 요소 별로 연산을 수행했으며, 
필요로 했던 결과값인 6까지만 작업을 수행하고, 이후 요소들은 연산 자체를 하지 않았다.

```text
first filter 1
first filter 2
second filter 2
first filter 3
first filter 4
second filter 4
first filter 5
first filter 6
second filter 6
third filter 6
6
```

<br>
즉, 작업의 흐름이

```text
filter (1, 2, 3, 4, 5, 6, 7, 8, 9 ... 20)
filter (2, 4, 6, 8, 10, 12, ... 20)
filter (6, 12, 18)
findFirst 
//연산 종료.
```
가 아니라,
<br>
```text
1 : filter                                //2의 배수x -> 다음 요소로
2 : filter > filter                       //2의 배수o 3의 배수x -> 다음 요소로
3 : filter                                //2의 배수x -> 다음 요소로
4 : filter > filter                       //2의 배수o 3의 배수x -> 다음 요소로
5 : filter                                //2의 배수x -> 다음 요소로
6 : filter > filter > filter > findFirst  //2의 배수o 3의 배수o 15이하o -> 연산 종료 
```
인 것이다.

<br>
이러한 결과가 나온 이유가 바로 Lazy Evaluation 때문이라는 것이다.
위에서 말했듯, stream은 연산을 처리할 때 중간연산이 나올 때마다 즉각적으로 처리(Eager Evaluation)하지 않고, 
중간연산과 최종연산을 모두 파악한 뒤에 연산을 진행(Lazy Evaluation)한다.

### Loop fusion, short circuit

Lazy Evaluation에 사용된 방법으로는 loop fusion, short circuit이 있다. 이를 이용해 최적화가 이루어진다고 이해하면 될 것 같다. 

loop fusion : 위의 출력 값에서 볼 수 있듯(6 filter > filter > filter > findFirst) 요소 하나 당 중간연산인 filter 3개가 하나로 묶여서 처리되는 것을 볼 수 있다.
이처럼 stream의 파이프라인에서 체이닝된 여러 개의 중간연산을 하나로 묶는 것을 loop fusion이라 한다.
loop fusion이 없었다면 filter가 동작할 때마다 stream에서 각 개별 요소에 매번 접근해야 했겠으나, 이를 하나로 묶음으로써 요소 당 한 번씩만 접근하면 된다.
다만 언제나 최종연산이 나오기 전까지의 모든 중간연산이 fusion되는 것은 아니다. 가령 sorted 메서드에는 전체 요소가 필요하므로, sorted 메서드 전후로 loop fusion이 나뉜다.

short circuit : 위의 코드에서 findFirst 메서드로 인해 첫 번째 요소를 찾은 이후로는 stream의 연산이 종료되었다. 이처럼 필요한 요소를 얻은 후에는 작업을 끝냄으로써 불필요한 연산을 하지 않을 수 있다.

---
참고

https://bugoverdose.github.io/development/stream-lazy-evaluation/
