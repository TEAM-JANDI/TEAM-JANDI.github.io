---
layout: post
title: "[13챕터] volatile"
subtitle: 자바의 정석 스터디 22회차
categories: Java
tags: [java, 자바의정석, 쓰레드, volatile]
---

## Volatile
- java 변수를 캐시가 아닌 main memory에서 저장하겠다는 키워드이다.
- 그렇기 때문에 main memory에서 변수를 읽어들이고 수정도 main memory에서 일어난다.

### 필요 이유
- 싱글 코어 프로세서에서는 작업이 일어난다 하더라도 캐시가 하나이기 때문에 상관이 없다.
- 멀티 코어 프로세서는 코어가 여러 개이기 때문에 캐시도 여러 개가 된다.
