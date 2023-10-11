---
layout: post
title: "[14챕터] stream의 Lazy Evaluation"
subtitle: 자바의 정석 스터디 22회차
categories: Java
tags: [Java, 자바의정석, stream, LazyEvaluation]
---


##Lazy Evaluation(지연 연산)
Java Stream의 최적화 방법 중 하나가 Lazy Evaluation 이라고 한다.
Lazy Evaluation이 어떻게 동작하는 것이길래 stream을 최적화 해준다는 걸까?

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

Lazy Evaluation의 동작 방식을 몰랐기 때문에, 이 코드가 다음과 같은 순서로 동작할 것이라 생각하였다.
1. 1부터 20까지 중에 2의 배수를 찾는다.
2. 1의 결과 중, 3의 배수를 찾는다.
3. 2의 결과 중, 15 이하인 수를 찾는다.
4. 3의 결과 중, 첫 번째 값을 찾는다.

결과적으로 아래와 같이 출력될 것으로 예상했다.
```text
first filter 1
first filter 2
first filter 3
first filter 4
...
second filter 2
second filter 4
second filter 6
second filter 8
...
third filter 6
third filter 12
third filter 18
6
```
<br>
작업 흐름을 확인하기 위해 코드에 출력문을 추가하고 실행해보았다. 그런데 예상과는 다른 결과를 볼 수 있었다.

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


실제 결과는 아래와 같았다.
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

<br><br>
즉, 작업의 흐름이
```text
filter (1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, ...)
filter (2, 4, 6, 8, 10, 12, ...)
filter (6, 12, 18)
findFirst (6, 12)
//연산 종료.
```
가 아니라,
<br>
```text
1 filter
2 filter > filter
3 filter
4 filter > filter
5 filter
6 filter > filter > filter > findFirst
//연산 종료.
```
인 것이다.

<br>
이러한 결과가 나온 이유가 바로 Lazy Evaluation 때문이라는 것이다.
JVM은 stream 연산을 처리할 때 중간 연산이 나올 때마다 즉각적으로 처리(Eager Evaluation)하지 않고, 중간 연산과 최종 연산을 모두 파악하여 최적화 방식을 정한 뒤에 연산을 진행(Lazy Evaluation)한다.
즉 Lazy Evaluation을 통해 최적화 전략을 정하기 때문에 Lazy Evaluation이 stream을 최적화 한다고 하는 것이다.

###Loop fusion, short circuit

Lazy Evaluation의 최적화 방법으로는 loop fusion, short circuit이 있다.

loop fusion : 위의 출력 값에서 볼 수 있듯(6 filter > filter > filter > findFirst) 요소 하나 당 중간 연산인 filter 3개가 하나로 묶여서 처리되는 것을 볼 수 있다.
이처럼 stream의 파이프라인에서 체이닝된 여러 개의 중간 연산을 하나로 묶는 것을 loop fusion이라 한다.
loop fusion이 없었다면 filter가 동작할 때마다 stream에서 각 개별 요소에 매번 접근해야 했겠으나, 이를 하나로 묶음으로써 요소 당 한 번씩만 접근하면 된다.
다만 언제나 최종 연산이 나오기 전까지의 모든 중간 연산이 fusion되는 것은 아니다. 가령 sorted 메서드에는 전체 요소가 필요하므로, sorted 메서드 전후로 loop fusion이 나뉜다.

short circuit : 위의 코드에서 findFirst 메서드로 인해 첫 번째 요소를 찾은 이후로는 stream의 연산이 종료되었다. 이처럼 필요한 요소를 얻은 후에는 작업을 끝냄으로써 불필요한 연산을 하지 않을 수 있다.

---
참고

https://bugoverdose.github.io/development/stream-lazy-evaluation/
