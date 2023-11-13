---
layout: post
title: "[15챕터] 입출력 I/O"
subtitle: 자바의 정석 스터디 24회차
categories: Java
tags: [java, 자바의정석, 입출력 I/O, 직렬화]
---


##직렬화
### 직렬화란?
객체를 바이트스트림 혹은 문자스트림 형태로 변환하는 과정을 말한다.

### 직렬화가 필요한 이유
메모리상에만 있는 데이터(객체)를 영속화 하거나 외부에 전송하고자 할 때,
메모리의 주소값이 아닌 해당 객체가 가진 값 자체를 저장/전송할 필요가 있다.
이를 위해 객체의 구조, 데이터 타입, 값 등에 대한 정보를 데이터스트림 형식으로 변환하는 작업이 필요하다.

### 직렬화의 종류
문자스트림 형식의 직렬화로는 cvs, xml, json 등이 있다.  
바이트스트림 형식의 직렬화로는 .jpg, .pdf 등이 있으며 java의 직렬화도 여기에 속한다.


### java의 Serializable
- 'java 시스템 간' 데이터 교환이 필요할 때 사용할 수 있다.
- 데이터 타입이 자동으로 맞춰지기 때문에 복잡한 구조의 클래스라도 역직렬화 하면 기존 형태 그대로 사용 가능하다.
- serialVersionUID에 의해 버전관리를 한다. 클래스 구조가 달라지면 버전도 달라지며, 버전이 다르면 역직렬화 시 예외가 발생한다.

마침 얼마 전 업무 중에  serialVersionUID를 본 적 있었고
그 당시 '직렬화라는 거에 쓰는 거구나. 직렬화에 왜 필요하지?' 라는 의문을 갖고 있었기에
이참에 직접 테스트해보았다.

1. 클래스 수정하지 않고 직렬화 및 역직렬화
```java
//Serializable 구현하여 직렬화 필요한 클래스임을 명시
public class TestClass implements Serializable {
    private int age;
    private String name;

    public TestClass(int age, String name) {
        this.age = age;
        this.name = name;
    }

    @Override
    public String toString() {
      ...
    }
}
```
```java
//Serialize
public static void main(String[] args) {
    TestClass t1 = new TestClass(1, "kimHello");
    try {
        //test.ser 이라는 파일 생성
        FileOutputStream fos = new FileOutputStream("test.ser");
        ObjectOutputStream oos = new ObjectOutputStream(fos);
        
        //t1을 바이트스트림으로 변환, test.ser에 저장
        oos.writeObject(t1);

        oos.close();
        fos.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

```java
//Deserialize
public static void main(String[] args) {
    TestClass t2 = null;
    try {
        FileInputStream fis = new FileInputStream("test.ser");
        ObjectInputStream ois = new ObjectInputStream(fis);
        //파일을 읽어와서 java 객체로 변환
        t2 = (TestClass) ois.readObject();

        ois.close();
        fis.close();
    } catch (IOException | ClassNotFoundException e) {
        e.printStackTrace();
    }
    System.out.println("t2 = " + t2);
}

/*
결과
t2 = TestClass{age=1, name='kimHello'}
*/
```  
역직렬화가 잘 되었다.


2. 직렬화 > 클래스 수정 > 역직렬화  
   이번엔 TestClass 객체를 직렬화 한 뒤, 인스턴스 변수의 타입을 수정하고 역직렬화 해보겠다.
```java
public class TestClass implements Serializable {
      
    private Integer age; //직렬화 시 : private int age; 역직렬화 시 : private Integer age; 
    private String name;
    
    //이하 동일
}

/*
결과  
java.io.InvalidClassException: com.example.how.serialization.TestClass; 
 local class incompatible: 
 stream classdesc serialVersionUID = 924594703824691821, 
 local class serialVersionUID = -8442131895083547755
 */

```
직렬화 했을 때 저장되었던 serialVersionUID와 역직렬화 시의 serialVersionUID가 달라 InvalidClassException이 발생했다.


serialVersionUID를 수동으로 관리하는 방법도 있는데,  
serialVersionUID가 동일할 경우 변수가 추가되는 등 클래스 구조가 달라도 예외가 발생하지 않는다. (이래도 되나 싶다.)  
반면 클래스 구조가 동일해도 serialVersionUID가 다르면 예외가 발생한다.

<br>
궁금했던 주제였기 때문에 이런 저런 테스트를 해보긴 했으나,  
개인적으로는 자바 시스템 간에만 사용이 가능하고 
serialVersionUID가 동일할 경우 클래스 구조가 달라도 예외가 발생하지 않는다는 점 등이 너무 불편하게 느껴졌다. 
JSON이라는 대중적인 대체제가 있는 상황에서 굳이 java에서 제공하는 직렬화를 사용할 것 같지는 않다.

---

참고  
