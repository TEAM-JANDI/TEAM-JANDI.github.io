---
layout: post
title: "[12챕터] Enum"
subtitle: 자바의 정석 스터디 18회차
categories: Java
tags: [java, enum, 자바의정석]
---

## Enum
- 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 혀용하지 않는 타입
- 기본적으로는 클래스이다.
- 상수 하나 당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.
- 밖에서 접근할 수 있는 생성자를 제공하지 않는다.
- 컴파일 시에 타입 안정성이 보장되어야 하므로 new 키워드 사용으로 인스턴스 생성이 불가능하다.

### 간단사용법
```java
// Direction.java
enum Direction {
    EAST,
    NORTH,
    WEST,
    SOUTH;
}
```
```java
// Game.java

// name: enum의 이름을 문자열로 반환.
// ordinal: enum이 정의된 순서를 반환 (0부터 시작.)
// valueOf(String): 해당 String 값에 해당하는 상수를 반환.

public class game() {
    int x = 0;
    int y = 0;
    Direction dir;

    switch(dir) {
        case EAST:
            x = 1;
            y = 0;
            break;
        case NORTH:
            x = 0;
            y = 1;
            break;
        case WEST:
            x = -1;
            y = 0;
            break;
        case SOUTH:
            x = 0;
            y = -1;
            break;
    }

    Direction[] arr = Direction.values();
    
    for(Direction d : arr) {
        System.out.printf("%s=%d%n", d.name(), d.ordinal());
    }
}
```
간단한 코드같은 경우에는 이렇게 작성이 된다.


### 멤버 추가
enum에는 인스턴스 변수가 포함이 되어있을 수 있다.
```java
// Number.java
enum Number {
    ONE(1),
    TWO(2),
    THREE(3);

    private final int value; // 필드 추가
    Number(int value) {this.value = value;} // 생성자

    public int getValue() {
        return value;
    }
}
```
인스턴스 멤버가 추가되는 경우에는 몇 가지 규칙이 생긴다.
1. 마지막 상수 끝에는 ;을 붙인다.
2. 인스턴스 변수가 생기는 것이므로 생성자를 추가해야한다.

### 추상 메서드 추가
```java
// Car.java
enum Car {
    HYUNDAI(400) {
        int option(int number) {
            return number * 300;
        }
    },
    KIA(300) {
        int option(int number) {
            return number * 200;
        }
    },
    DAEWOO(100) {
        int option(int number) {
            return number * 100;
        }
    },
    SSANGYONG(200) {
        int option(int number) {
            return number * 50;
        }
    };

    private final int cost;

    Car(int cost) {
        this.cost = cost;
    }

    abstract int option(int number);
}
```
이렇게 option 추상 메서드를 선언해 각 상수 내부에 구현을 해주는 것이다.

### 알 가치가 있는 것들
- 상수값은 객체의 주소이기에 equals 메서드가 아닌 '=='로도 비교를 할 수 있다. 다만 같은 상수 집합 안에서 비교를 해야하지 다른 집합의 상수끼리는 비교를 할 수 없다.(컴파일 에러)
- 관례적으로 enum의 상수들은 모두 대문자로 표기한다.
- 순서나 인덱싱을 위해서 ordinal 메서드를 쓰는 것은 지양하는 것이 좋다. 상수 순서가 바뀌면 ordinal 값도 변하기 때문이다.
