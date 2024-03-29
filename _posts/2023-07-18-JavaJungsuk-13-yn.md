---
layout: post
title: "[13챕터] 프로세스와 스레드의 차이"
subtitle: 자바의 정석 스터디 20회차
categories: Java
tags: [java, thread, 자바의정석]
---


우선 프로세스는 대충 아래와 같은 흐름으로 생성된다.


1. 프로세스 생성 요청 : 사용자가 새 프로그램을 실행하거나, 이미 실행 중인 프로세스가 새로운 프로세스의 생성을 요청. 

2. 프로세스 제어 블록(PCB) 생성 : PCB란 Process Control Block 으로, 여기서 Block이란 쉽게 말해 구조체다. 즉, 프로세스에 대한 중요한 정보를 저장하는 자료구조이다. 운영체제마다 다르지만 주로 프로세스의 상태, 프로그램 카운터, CPU 레지스터, 우선 순위, 메모리 관리 정보 등을 포함한다.

3. 리소스 할당 : 프로세스가 필요로 하는 리소스(메모리 공간, 파일, I/O 디바이스 등)를 할당한다. 

4. 프로세스 코드와 데이터를 메모리에 로드한다. 

5. 프로세스의 초기 상태를 설정한다. 대부분 운영체제의 경우 프로세스는 준비 상태로 설정되어 CPU 스케줄러의 실행 대기열에 추가된다.

6. 운영체제의 스케줄러는 준비 상태의 프로세스 중에서 하나를 선택하여 CPU에 할당한다. 프로세스는 이제 실행 상태로 전환되어 명령어를 실행하게 된다. 

<br>


프로세스는 최소 하나의 스레드(메인 스레드)를 갖는다.

따라서 새로 생성된 프로세스는 이미 메인 스레드가 실행되고 있는 상태이다.

이후 코드에서 추가적으로 다른 스레드들을 생성하면 여러 개의 스레드가 실행되는 것이다.



## 스레드의 메모리 공간

우리 모두 알다시피 하나의 프로세스에서 생성된 스레드들은 운영체제가 처음 프로세스에게 할당한 메모리 공간을 함께 사용한다. 

<br>

더 자세히 설명하면

스레드 각각은 고유의 레지스터 상태와 스택 메모리를 가지지만

힙 메모리, 데이터 영역(전역 변수 포함), 코드 영역 등은 프로세스 내의 모든 스레드와 공유된다.

즉, 지역 변수나 매개변수는 스레드 각각 스택 영역에 저장되지만 new로 할당한 객체나 static 변수, 메소드 등은 공유해서 사용한다는 말이다.

<br>


이렇게 공유하는 메모리로 여러가지 성능적 이점을 취하지만,

단점으로는 여러 동기화 문제(크리티컬 섹션 등)가 야기될 수 있다.

<br>

+) 책에서 나왔던 프로세스 메모리를 초과할 정도로 많은 갯수의 스레드를 만들면 안된다는건, 아마도 스레드별로 주어지는 스택 메모리를 말하는 듯. 스레드당 적어도 몇byte~kb의 스택 메모리는 잡아먹을테니...?


<br>

## CPU 스케쥴링의 기본 단위

현대의 대부분의 운영체제에서 스레드는 CPU 스케쥴링의 기본 단위이다.

<br>

하나의 프로세스 내에서 여러 스레드가 동시에 실행되는 경우,

스레드 단위로 스케줄링하면 한 프로세스 내의 여러 스레드 간에 컨텍스트 전환을 수행할 수 있어 프로세스 전체의 효율이 올라간다. 
즉, 한 프로세스의 스레드 A가 I/O 작업을 기다리는 동안 다른 스레드 B는 계속해서 CPU를 사용할 수 있도록 하는 것이 가능하다.

<br>

그렇다고 하면, 프로세스 단위가 아닌 스레드 단위로 스케줄링이 이루어지는데 PCB만으로 어떻게 이를 해결하는가?

프로세스가 아닌 각 스레드의 정보가 필요할텐데...?

여기서 알게된 것이 TCB이다.

<br>

## TCB 

PCB가 프로세스를 관리하는 구조체였듯이, TCB는 쓰레드의 실행 상태를 저장하고 복원하는 역할을 하는 구조체이다.

TCB는 보통 PCB보다 적은 데이터를 가지며, 해당 Thread에 대한 정보와 PCB를 가리키는 포인터를 갖는다.

ex) 리눅스 기준 TCB는 24개의 필드를 가지며, PCB는 106개의 필드를 갖는다.

<br>

보통 TCB는 커널 레벨에서 Context Switching의 기본 단위가 되며, 같은 프로세스 내에서의 컨텍스트 스위칭은 TCB 정보만 저장하면 된다.

하지만 다른 프로세스 간의 컨텍스트 스위칭이 이루어질 때는 PCB와 TCB 모두 저장해야 한다.

<br>

## 스레드 생성 부하

스레드를 새로 생성할 때는 부하가 생긴다. 

1. 시간적 부하: 스레드를 생성하면 TCB를 초기화하고, 스택 메모리를 할당하고, 스케줄러에 스레드를 등록하는 등의 작업을 해야합니다. 이 모든 작업은 CPU 시간을 소비한다.

2. 메모리 부하: 각 스레드의 고유 스택 메모리와 TCB는 메모리를 차지한다. 당연히 스레드가 많아질수록 더 많은 메모리가 필요하므로, 생성할 때마다 메모리에 부하가 온다.

3. 스케줄링 부하: 운영체제는 실행 가능한 스레드 중에서 어떤 스레드를 먼저 실행할지 결정하는 스케줄링 작업을 수행하는데, 스레드가 많아질수록 스케줄링에 더 많은 시간과 리소스가 필요해진다. 또한, 스레드 간에 컨텍스트 스위칭을 자주 일어나면, 이로 인한 오버헤드도 커진다.

따라서 스레드를 많이 생성하면, 이런 여러 가지 부하 때문에 시스템의 성능이 저하될 수 있다. 

<br>

아래 예시는 내 뇌피셜이긴 하지만,

1요청당 1스레드 방식으로 수행하는 톰캣을 생각해보면

요청마다 스레드를 생성-제거한다면 요청이 많아질수록 이로 인한 부하가 심해질 것이다.

또한 요청이 들어올때마다 무작정 스레드를 생성해버리면 요청이 몰리면 자칫 엄청나게 많은 수의 스레드가 생성될 수도 있다.

<br>

이를 효율적으로 관리하기 위해 필요한 만큼의 스레드를 미리 생성해두고 재사용하는 방식을 스레드 풀이라 한다.

<br>

## 스레드풀

<br>

스레드 풀의 장점은 위에서 추측할 수 있듯 아래와 같다.

1. 필요한 갯수의 스레드를 미리 생성하여 스레드 생성/제거 오버헤드 감소

2. 너무 많은 수의 스레드가 생성되는 것을 방지하여 스레드가 사용하는 메모리나 스케줄링 오버헤드를 제어 가능

<br>

스레드의풀의 동작 흐름은 아래와 같다.
1. 시작할 때 일정 수의 스레드를 미리 생성하고 **대기 상태**로 둠
2. 작업큐에 수행 가능한 작업이 들어오면 **대기 상태** 중인 스레드 중 하나를 깨워서 작업 수행
3. 스레드가 작업을 완료하면, 그 스레드는 다시 **대기 상태**로 돌아감

<br>

**대기 상태**의 스레드는 CPU 자원을 소비하지 않는다!

따라서 활동 중인 다른 스레드가 CPU를 사용할 수 있다. 

실제로 웹 서버, 데이터베이스, 게임 서버 등과 같이 많은 병렬 작업을 수행해야 하는 애플리케이션에서는 스레드 풀을 자주 사용한다고 한다.