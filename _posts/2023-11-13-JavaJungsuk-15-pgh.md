---
layout: post
title: "[15챕터] File"
subtitle: 자바의 정석 스터디 24회차
categories: Java
tags: [java, 자바의정석, 입출력 I/O, File]
---
 
일단 본인은 Java 백엔드 개발자로 시작은 했으나 현재 프로젝트에 투입된 역할로 인해 소홀히 하고 공부도 예전보다는 많이 안 하고 있다.
그 중에서도 예전에도 잘 습득이 되지 않은 부분을 포스팅하면서 정리하고자 한다.
바로 File이다.

## File
일단은 File 클래스부터 알아보자.
> java.io 패키지에서 제공하며 파일이나 폴더에 대한 제어를 하는 데에 사용되는 클래스이다.

### 개요
기본적으로는 File 인스턴스를 생성하여서 다루는데 파일 혹은 디렉터리일 수도 있다.
이는 시스템 혹은 os에 따라 문자열의 규칙은 다를 수 있지만 문자열로 이뤄진 파일 혹은 디렉터리인 것은 공통적이다.
예를 들어, UNIX 플랫폼에서 절대경로의 접두부가 항상 있지만(/) 상대 경로에는 접두부가 없다.
하지만, MS 플랫폼에서 드라이브 문자와 경로 이름이 절대적인 경우 드라이브를 포함한 경로 이름의 접두부가 뒤에 올 수도 있다.

### 생성자
생성자는 다음과 같다.

|생성자|설명|
|:---:|:---:|
|File(String fileName)|주어진 문자열을 이름으로 갖는 파일을 위한 인스턴스 생성된다. 디렉터리도 같은 방법이다. 경로는 프로그램이 실행되는 위치가 경로로 간주 된다.|
|File(URI uri)|지정된 uri로 파일을 생성한다.|
|File(String pathName, String fileName)|파일의 경로와 이름을 각각 문자열로 분리해서 지정하는 생성자|
|File(File pathName, String fileName)|파일의 경로와 이름을 분리하여 지정하는 생성자. 이 때는 경로가 File 인스턴스인 경우이다.|

### 메서드
메서드는 일단 개수가 많다. 생성, 수정, 삭제, 조회, 확인용 메서드 등 각 역할마다 하는 일이 다른 메서드들이 있다.
그렇기 때문에 자주 사용하는 메서드들은 외우되, 혹시 모르니 공식 사이트에서 찾아보는 것도 나쁘지 않다.

일단은 대표적인 메서드들이다.
|메서드|설명|
|:---:|:---:|
|boolean createNewFile()|아무런 내용이 없는 새로운 파일을 생성한다. 이미 존재하면 생성되지 않는다.|
|booolean delete()|해당 파일을 삭제시킨다.|
|boolean mkdir()|파일에 지정된 경로로 디렉토리를 생성한다.|
|boolean renameTo(File fileName)|지정된 파일(fileName)으로 이름을 변경한다.|
|String getAbsoluteFile()|파일의 절대경로를 String으로 반환한다.|
|String getCanonicalFile()|파일의 정규경로를 String으로 반환한다.|
|String getPath()|파일의 경로를 String으로 반환한다.|
|boolean exists()|파일이 존재하는지 검사한다.|
|boolean isFile()|파일인지 확인한다.|
|boolean isDiredtory()|디렉토리인지 확인한다.|
|boolean isHidden|파일 속성이 숨김인지 확인한다.|
|long length()|파일의 크기를 반환한다.|
|boolean canRead()|읽을 수 있는 파일인지 확인한다.|
|boolean canWrite()|쓸 수 있는 파일인지 확인한다.|
|boolean canExecute()|실행할 수 있는 파일인지 확인한다.|
