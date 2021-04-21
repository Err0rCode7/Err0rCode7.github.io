---
bg: "Review.jpg"
layout: post
title:  "equals(), hashCode()"
crawlertitle: "equals(), hashCode()"
summary: "equals(), hashCode()"
date: 2021-04-21 15:40:00 +0900
category: Java
author: Err0rCode7
---

#### equals와 hashCode 함수
equals와 hashCode는 모두 Java 객체의 최상위 객체인 Object 클래스에 정의 되어있는 메소드이다. 그렇기 때문에 Java의 모든 객체는 Object 클래스에 정의된 equals와 hashCode 함수를 상속받고있다.

##### What is equals method?

equals 메소드는 2개의 객체의 동등을 비교할 때 사용된다. Object 클래스에서 equals가 구현된 방법은 2개의 객체의 참조 값이 동일한지를 확인한다.

즉, 다음과 같다.

```java
public boolean equals(Object obj) {
  return (this == obj)
}
```

String 클래스의 경우 equals 메소드가 오버라이드 되어있다. 동일한 문자열을 갖는 String 인스턴스를 두 개를 생성하고 equals 메소드를 이용해보면 `true`	가 나오게 되는데, 참조하는 객체를 비교하는 것이 아닌 객체가 갖는 문자열을 비교하는 코드로 equals이 오버라이드됐음을 알 수 있다. 즉, String 클래스에서는 동등을 문자열(객체 내부 value)로 정의한다.

두 객체를 `a==b`와 같이 비교한다면 동일성 비교가 되고 `a.equals(b)`와 같이 비교한다면 동등성 비교가 된다.



##### What is hashCode method?

hashCode는 런타임에 객체의 유일한 integer값을 반환하는 메소드로 일반적으로 Object 클래스에서는 heap에 저장된 객체의 메모리 주소를 integer 값으로 반환하는 방식으로 구현되어있다.

이 메소드는 해시를 사용할 때의 이점을 위해 제공된다. 즉, Hash 관련 Collections(HashSet, HashMap, HashTable)을 이용할 때 hashCode를 기준으로 key를 찾는 것에 이용된다.

hashCode 메소드의 일반 규약은 다음과 같다.

1. 변경되지 않은 한 객체의 hashCode 메소드를 호출한 결과는 항상 똑같은 integer 값이어야 한다.
   - 객체가 변경됐더라도 equals 메소드가 참고하는 정보가 변경되지 않았다면 hashCode 값은 달라지지 않는다.
2. equals 메소드가 같다고 판별한 두 객체의 hashCode 호출 결과는 똑같은 integer 값이어야한다.
3. 그러나 `java.lang.Object.equals` 메소드가 다르다고 판별한 두 객체의 hashCode 값이 반드시 달라야 하는 것은 아니다.
   - 단, 같지 않은 객체들이 각기 다른 hashCode 값을 가지면 해시 테이블 성능이 향상된다는 점 을 기억하자.

어떻게 Hash Collections 에서 hashCode 메서드로 동등성을 비교하는지 살펴보면 아래와 같이 진행된다.

1. hashCode 메소드를 실행해서 리턴된 해시코드 값이 같은지를 본다. 해시 코드값이 다르면 다른 객체로 판단하고, 해시 코드값이 같으면 다음을 진행한다.
2. equals 메소드로 다시 비교를 한다. 이 두 개가 모두 맞아야 동등 객체로 판단한다. 즉, 해시코드 값이 다른 엔트리끼리는 동치성 비교를 시도조차 하지 않는다.

해시 값이 같은 것이 있다면 버킷 안에 다른 객체가 있는 경우 equals 메서드가 사용되는 것이다.

- HashTable에 put 메서드로 객체를 추가하는 경우, 값이 같은 객체가 이미 있다면(equals()가 true) 기존 객체를 덮어쓰고 값이 같은 객체가 없다면 해당 entry를 LinkedList에 추가한다.

- HashTable에 get 메서드로 객체를 조회하는 경우, 값이 같은 객체가 있다면(equals()가 true) 그 객체를 리턴한다. 값이 같은 객체가 없다면 null을 리턴한다.

<details>
  <summary> 👉 접기 / 펼치기 👈</summary>
  <div markdown="1">
##### 해시 분포와 해시 충돌


동일하지 않은 어떤 객체 X와 Y가 있을 때, 즉 `X.equals(Y)`가 거짓일 때 `X.hashCode() != Y.hashCode()` 가 같지 않다면, 이때 사용하는 해시 함수는 완전한 해시 함수라고 한다.

Boolean같이 서로 구별되는 객체의 종류가 적거나, Integer, Long, Double 같은 Number 객체는 객체가 나타내려는 값 자체를 해시값으로 사용할 수 있기 때문에 완전한 해시 함수 대상으로 삼을 수 있다. 하지만 String이나 POJO(plain old java object)에 대하여 완전한 해시 함수를 제작하는 것은 사실상 불가능하다.

적은 연산만으로 빠르게 동작할 수 있는 완전한 해시 함수가 있다고 하더라도, 그것을 HashMap에서 사용할 수 있는 것은 아니다. HashMap은 기본적으로 각 객체의 hashCode() 메서드가 반환하는 값을 사용하는 데, 결과 자료형은 int다. 32비트 정수 자료형으로는 완전한 자료 해시 함수를 만들 수 없다. 논리적으로 생성 가능한 객체의 수가 232보다 많을 수 있기 때문이며, 또한 모든 HashMap 객체에서 O(1)을 보장하기 위해 랜덤 접근이 가능하게 하려면 원소가 232인 배열을 모든 HashMap이 가지고 있어야 하기 때문이다.

따라서 HashMap을 비롯한 많은 해시 함수를 이용하는 associative array 구현체에서는 메모리를 절약하기 위하여 실제 해시 함수의 표현 정수 범위 `|N|`보다 작은 M개의 원소가 있는 배열만을 사용한다. 따라서 다음과 같이 객체에 대한 해시 코드의 나머지 값을 해시 버킷 인덱스 값으로 사용한다.

```java
int index = X.hashCode() % M;
```

이렇게 사용을 하면 서로 다른 객체가 1/M의 확률로 같은 해시 버킷을 사용하게 된다. 이는 해시 함수가 얼마나 해시 충돌을 회피하도록 잘 구현되었느냐에 상관없이 발생할 수 있는 또 다른 종류의 해시 충돌이다. 이를 해결하는 방식은 대표적으로 두 가지가 있는데, 하나는 Open Addressing과 다른 하나는 Separate Chaning이다. 이 둘 외에도 해시 충돌을 해결하기 위한 다양한 자료 구조가 있지만, 거의 모두 이 둘을 응용한 것이라고 볼 수 있다.

<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/115504986-c43b5300-a2b3-11eb-9a14-aadd7d776c4f.png">
<p style="font-weight:bold" align="center">Open Addressing과 Separate Chaning</p>
</p>

Java HashMap에서 사용하는 방식은 Separate Chaning이다. Open Addressing은 데이터를 삭제할 때 처리가 효율적이기 어려운데, HashMap에서 remove() 메서드는 매우 빈번하게 호출될 수 있기 때문이다. 게다가 HashMap에 저장된 키-값 쌍 개수가 일정 개수 이상으로 많아지면, 일반적으로 Open Addressing은 Separate Chaining보다 느리다. Open Addressing의 경우 해시 버킷을 채운 밀도가 높아질수록 Worst Case 발생 빈도가 더 높아지기 때문이다. 반면 Separate Chaining 방식의 경우 해시 충돌이 잘 발생하지 않도록 '조정'할 수 있다면 Worst Case 또는 Worst Case에 가까운 일이 발생하는 것을 줄일 수 있다

Java 7까지는 데이터의 개수가 많아지면 Seperate Chaning에서 링크드 리스트를 사용했지만 Java 8에서부터는 하나의 해시 버킷에 8개의 키-값 쌍이 모이면 링크드 리스트를 트리로 변경하여 트리를 이용해 저장한다.
  </div>
</details>

##### equals() 와 hashCode()를 같이 재정의해야 하는 이유

equals()와 hashCode() 중 하나만 재정의하면 어떤 문제가 일어나는 지 살펴보자.

hashCode()를 재정의하지 않으면 같은 값 객체라도 해시 값이 다를 수 있는 것의 문제가 생기게된다. 따라서 HashTable에서 해당 객체가 저장된 버킷을 찾을 수 없게된다.

반대로 equals()를 재정의하지 않으면 hashCode()가 만든 해시값을 이용해 객체가 저장된 버킷을 찾을 수는 있지만 해당 객체가 자신과 같은 객체인지 값을 비교할 수 없기 때문에 null을 리턴하게 된다. 따라서 이또한 원하는 객체를 찾을 수 없다.

##### 결론

객체의 정확한 동등 비교를 위해서는 특히 Hash 관련 컬렉션 프레임워크를 사용할 때에서는, equals() 와 hashCode() 메소드를 모두 재정의해서 두 가지 모두가 논리적 동등을 비교 가능하도록 해야한다.

##### Reference

https://d2.naver.com/helloworld/831311
https://johngrib.github.io/wiki/Object-hashCode/
https://jisooo.tistory.com/entry/java-hashcode%EC%99%80-equals-%EB%A9%94%EC%84%9C%EB%93%9C%EB%8A%94-%EC%96%B8%EC%A0%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B3%A0-%EC%99%9C-%EC%82%AC%EC%9A%A9%ED%95%A0%EA%B9%8C