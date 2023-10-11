---
layout: post
title: "[14챕터] 반복문과 스트림"
subtitle: 자바의 정석 스터디 22회차
categories: Java
tags: [java, 자바의정석, stream, 반복문]
---


## 개요
나는 이 포스팅을 통해서 반복문과 stream이라는 것에 대해서 정리를 하고자 한다.

stream이란 것은 왜 생겨난 것일까?<br/>
따지고 보면 그냥 반복문을 쓰면 될 것이지 왜 하필 stream이라는 것을 만들었나 생각한다.

Java 8 이전에는 프로그램들이 싱글코어만을 가지고 사용하는 것들이 대부분이였다.<br/>
그러나, 시대가 지나면서 멀티코어를 필요로 하는 프로그램들이 생겨나기 시작했고 기존에 있던<br/>
멀티스레드를 활용하면 관리나 안정성에 있어서 불안한 점이 있었다.

이를 해결하기 위해서 Java8에서 Stream API를 제공하였다.

## Stream이란 무엇일까?
일단 기본적으로 stream은 데이터 소스가 무엇이던 간에 같은 방식으로 데이터를 다루는데 자주 사용되는 메서드들을 정의해 놓는다.<br/>
이로써 코드의 재사용성을 높일 수 있는 장점이 생겼다. 기본적으로 이렇게 쓰인다.

```java
// 배열 방식
String[] strArr = {"1", "2", "3"};
// ArrayList 방식
List<String> strList = Arrays.asList(strArr);
// stream 방식
Stream<String> strStream1 = Arrays.stream(strArr);
Stream<String> strStream2 = strList.stream();
```

stream은 다음과 같은 특징을 지니고 있다.

1. 데이터 소스를 변경하지 않는다.<br/>
	데이터 소스로부터 읽는 작업을 할 뿐, 데이터 소스를 변경하지 않는다. <br/>값뿐만 아니라 정렬까지도 유지된다.<br/>
2. 일회용이다.<br/>
	Iterator처럼 모두 읽고 나면 다시 사용할 수 없는 일회용이다. <br/>다시 사용하기 위해서는 stream을 다시 생성할 수 있다.<br/>
3. 작업을 내부에서 반복 처리한다.<br/>
	내부에서 반복 처리한다는 것은 반복문을 내부에 숨길 수 있다는 것을 의미한다. <br/>그렇다면 이 내부에서 반복하면 이점은 무엇일까?<br/>
	일반적인 반복문은 병렬성을 스스로 관리해야하는 반면, 데이터 표현과 하드웨어를 활용한 병렬성 구현을 자동으로 선택한다. 
4. 연산<br/>
	중간 연산(결과가 stream인 연산), 최종 연산(결과가 stream이 아닌 연산) 등 다양한 연산을 이용하여 복잡한 연산을 처릿할 수 있다.<br/>
	단, 최종 연산이 수행되기 전까지 중간 연산이 실행되지 않는 지연 연산을 이용하기에 이 점은 유의해야 한다.<br/>
5. 병렬 처리가 쉽다.<br/>
	fork&join 프레임워크로 이용하여 자동적으로 연산을 병렬적으로 수행한다. <br/>단지 parallel() 메서드를 호출하여 자동적으로 수행하라고 지시하기만 하면 된다.<br/>
	이를 이용해 멀티코어에서 수행되는 작업에서 멀티쓰레드를 통해 수행시간을 감소시킬 수 있다.
	

## 반복문과 stream의 성능

### 배열의 요소 더하기
4가지 코드를 준비한다.

1. for문
2. 향상된 for문
3. stream forEach
4. stream reduce

그리고 배열의 사이즈별로 테스트를 해보겠다. (10, 1000, 100000)

```java
import java.util.ArrayList;
import java.util.List;

public class Main {
    private static int sum = 0;

    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
				// 사이
        for(int i = 0; i < 10; i++) {
            list.add(i);
        }

        // for문
        long startTime = System.nanoTime();
        for(int i = 0; i < list.size(); i++) {
            sum += list.get(i);
        }
        long endTime = System.nanoTime();
        System.out.println("time = " + (endTime - startTime) + " ns");

        sum = 0;
        // 향상된 for문
        startTime = System.nanoTime();
        for(int i : list) {
            sum += i;
        }
        endTime = System.nanoTime();
        System.out.println("time = " + (endTime - startTime) + " ns");

        sum = 0;
        // stream forEach
        startTime = System.nanoTime();
        list.stream().forEach(item -> sum += item);
        endTime = System.nanoTime();
        System.out.println("time = " + (endTime - startTime) + " ns");

        sum = 0;
        // stream reduce
        startTime = System.nanoTime();
        list.stream().reduce(Integer::sum);
        endTime = System.nanoTime();
        System.out.println("time = " + (endTime - startTime) + " ns");
    }
}
```
크기가 10일 때
>time = 5200 ns<br />
time = 13900 ns<br />
time = 793000 ns<br />
time = 616100 ns<br />

크기가 1000일 때 
>time = 179800 ns<br />
time = 176300 ns<br />
time = 713400 ns<br />
time = 652100 ns<br />

크기가 100000일 때
>time = 1170600 ns<br />
time = 1303100 ns<br />
time = 2496600 ns<br />
time = 2776000 ns<br />

크기가 10인 경우에는 for문이 확실히 시간이 적게 걸린다. 그리고 forEach문이 가장 오래 걸린다.<br/>
크기가 1000인 경우에는 for문과 향상된 for문이 엎치락뒤치락거렸고 stream끼리 엎치락뒤치락거렸다.<br/>
크기가 100000인 경우에는 for문이 시간이 가장 적게 거렸고 reduce가 가장 오래 걸린다.<br/>

### 변환
그럼 변환 작업을 보자. 배열을 ArrayList와 Stream으로 각각 전화시키고 반대 작업도 해보자.<br/>
이 또한 배열의 크기를 10, 1000, 100000 나눠서 해본다.<br/>

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Stream;

public class Main {
    public static void main(String[] args) {
        String[] strArr = new String[10]; // 배열의 크기

        // 배열 -> ArrayList
        long startTime = System.nanoTime();
        List<String> strList = Arrays.asList(strArr);
        long endTime = System.nanoTime();
        System.out.println("time = " + (endTime - startTime) + " ns");

        // 배열 -> Stream
        startTime = System.nanoTime();
        Stream<String> strStream1 = Arrays.stream(strArr);
        endTime = System.nanoTime();
        System.out.println("time = " + (endTime - startTime) + " ns");

        // ArrayList -> 배열
        startTime = System.nanoTime();
        strList.toArray();
        endTime = System.nanoTime();
        System.out.println("time = " + (endTime - startTime) + " ns");

        // Stream -> 배열
        startTime = System.nanoTime();
        strStream1.toArray();
        endTime = System.nanoTime();
        System.out.println("time = " + (endTime - startTime) + " ns");
    }
}
```
크기가 10일 때,
>time = 15900 ns<br />
time = 25100 ns<br />
time = 13500 ns<br />
time = 1247600 ns<br />

크기가 1000일 때,
>time = 14300 ns<br />
time = 25700 ns<br />
time = 14000 ns<br />
time = 1225600 ns<br />

크기가 100000일 때,
>time = 13800 ns<br />
time = 67800 ns<br />
time = 143400 ns<br />
time = 3303400 ns<br />

크기가 커지면 커질수록 변환하는 시간이 길어진다.(배열 -> ArrayList 제외)<br/>
대체적으로 stream 변환하는 작업이 시간이 더 길었고 특히나 stream -> 배열은 급격하게 늘었다.<br/>

## stream 작업 시간이 나는 이유
일단 바이트코드 상에도 길이 차이가 난다.<br/>
위 더하기 연산을 예로 들겠다.<br/>

```bytecode
LINENUMBER 16 L10
    IINC 4 1
    GOTO L7


LINENUMBER 34 L22
    ALOAD 1
    INVOKEINTERFACE java/util/List.stream ()Ljava/util/stream/Stream; (itf)
    INVOKEDYNAMIC accept()Ljava/util/function/Consumer; [
      // handle kind 0x6 : INVOKESTATIC
      java/lang/invoke/LambdaMetafactory.metafactory(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
      // arguments:
      (Ljava/lang/Object;)V, 
      // handle kind 0x6 : INVOKESTATIC
      Main.lambda$main$0(Ljava/lang/Integer;)V, 
      (Ljava/lang/Integer;)V
    ]
    INVOKEINTERFACE java/util/stream/Stream.forEach (Ljava/util/function/Consumer;)V (itf)
```

첫 번째 문단은 for문에 해당하는 bytecode이고<br/>
그 밑은 stream forEach에 해당하는 bytecode이다. <br/>
보듯이 처리하는 것들이 많아 실행이 느리다. 또한, for문은 인덱스 기반으로 도는 메모리 접근이기에 빠르다.<br/>

그러면, 굳이 쓰는 이유가 무엇일까?<br/>
1. 메서드를 이용하여 코드로 일일이 구현해야 하는 기능을 간단하게 해결해 줄 수 있다.<br/>
2. 멀티 코어 환경에서 멀티스레드를 이용하는 경우에는 더 빠르게 처리할 수 있다.<br/>
3. 가독성이 더 좋다. 다만 이거는 개인적인 차이라 생각한다.<br/>

다음 포스팅은 직렬 stream과 병렬 stream의 성능을 비교해볼까 한다. <br/>

## 참고
https://12bme.tistory.com/461
