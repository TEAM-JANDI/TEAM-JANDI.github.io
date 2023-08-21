---
layout: post
title: "[13챕터] volatile"
subtitle: 자바의 정석 스터디 22회차
categories: Java
tags: [java, 자바의정석, 쓰레드, volatile]
---

## volatile
- java 변수를 캐시가 아닌 main memory에서 저장하겠다는 키워드이다.
- 그렇기 때문에 main memory에서 변수를 읽어들이고 수정도 main memory에서 일어난다.

### 필요 이유
- 싱글 코어 프로세서에서는 작업이 일어난다 하더라도 캐시가 하나이기 때문에 상관이 없다.
- 멀티 코어 프로세서는 코어가 여러 개이기 때문에 캐시도 여러 개가 된다.
- 캐시 간의 데이터 이격이 있기 때문에 변수를 읽어들이는 데에 있어서 불일치 문제가 있을 수 있다.

### 사용 방법
```java
volatile boolean flag = false;
```
이와 같이 변수 앞에 volatile을 붙여서 선언을 하면 된다.

같은 효과를 내기 위해서 sychronized 키워드를 쓸 때도 있다.
다만 차이점은 synchronized는 메서드 단에서 사용하고 쓰레드가 synchronized 블럭을 들어가고 나올 때 캐시와 메모리간의 동기화가 이뤄진다
```java
public synchronized void test() {
  boolean flag = false;
}
```

### 단점
- 어찌 되었든 변수의 읽고 쓰는 작업을 캐시가 아닌 main memory에서 진행하기 때문에 비용이 크기에 영향이 있다.

### 주의할 점 - 원자화
- jvm은 데이터를 4 byte 단위로 처리하기 때문에 그보다 큰 long, double을 처리할 때 에는 한 번에 처리를 못 한다.
- 이 때 다른 쓰레드에서 해당 변수를 가지고 작업이 될 수 있기 때문에 변수를 앞에서 volatile을 붙인다. 이를 원자화라 한다. 즉, 작업을 더 이상 할 수 없는 단위로 나눈다는 것이다.
- synchronized 블럭도 일종의 원자화로 이를 통해서 동기화를 해주는 것인데 다만, volatile은 변수를 읽고 쓰는 것을 원자화하는 것일 뿐 동기화를 구현한 것은 아니다.
- 때문에 동기화를 원한다면 synchronized 블럭을 상용해야한다.
```java
volatile long balance;

synchronized int getBalance() {
  return balance;
}

synchronized void withdraw(int money) {
  if(balance >= money){
    balance -= money; 
  }
}
```
