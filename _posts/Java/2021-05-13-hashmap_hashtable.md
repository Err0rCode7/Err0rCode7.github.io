---
bg: "Review.jpg"
layout: post
title:  "HashMap vs Hashtable"
crawlertitle: "HashMap vs Hashtable"
summary: "HashMap vs Hashtable"
date: 2021-05-15 12:00:00 +0900
category: Java
author: Err0rCode7
---

#### HashMap vs Hashtable

##### 왜 우리는 HashMap을 사용할까?

## HashMap vs Hashtable

[🔗 equals, hashcode 글 hashcode](https://err0rcode7.github.io/java/2021/04/21/equals_hashCode.html)를 같이 읽으면 도움이 됩니다.

### HashMap

- 키에 대한 해시값을 사용하여 값을 저장하고 조회하며, 키-값 쌍의 개수에 따라 동적으로 크기가 증가하는 associate array(예를 들면 Map, Dictionary, Symbol Table)
- **Java Collections Framework**에 속한 구현체 클래스이다.
- Map 인터페이스를 구현하고 있다.
- **Amortized Constant Time을 위해 해시 충돌 가능성을 줄이는 방식으로 작동한다. -> 보조 해시 함수 사용**
- key, value 에 null을 허용한다.
- `synchronized`가 선언되어있지않다. 따라서 **thread-safe 하도록 사용해야 할 때는 ConcurrentHashMap을 사용해야 한다.**

### HashTable

- **JDK 1.0 부터 있었던 Java API**
- Map 인터페이스를 구현하고 있다. -> 기능적으로 HashMap과 동일하다.
- HashTable의 현재 가치는 JRE 1.0, JRE 1.1 환경을 대상으로 구현한 Java 애플리케이션이 잘 동작할 수 있도록 하위 호환성을 제공하는 것이다. -> HashMap과 성능과 기능을 비교하는게 큰 의미가 없다.
- key, value에 null 을 허용하지 않는다.
- **주요 메소드에 `synchronized`가 선언되어있다.**

## Reference

[https://d2.naver.com/helloworld/831311](https://d2.naver.com/helloworld/831311)

[https://jdm.kr/blog/197](https://jdm.kr/blog/197)