---
bg: "Review.jpg"
layout: post
title:  "Spring과 디자인 패턴 - 1"
crawlertitle: "Spring과 디자인 패턴"
summary: "Spring과 디자인 패턴 - 1"
date: 2021-05-03 15:22:00 +0900
category: Spring
author: Err0rCode7
---

#### 스프링이 사랑한 디자인 패턴 - 1
##### [자바 객체지향의 원리](https://wikibook.co.kr/java-oop-for-spring/) 책을 읽고 내용을 정리하는 글입니다.

객체 지향 특성은 도구이고 설계 원칙은 도구를 올바르게 사용하는 방법이라고할 수 있다. 그리고 디자인 패턴은 레시피에 해당한다.

디자인 패턴은 실제 개발 현장에서 비즈니스 요구 사항을 프로그래밍으로 처리하면서 만들어진 다양한 해결책 중에서 많은 사람들이 인정한 베스트 프랙티스를 정리한 것이다. 디자인 패턴은 당연히 객체 지향 특성과 설계 원칙을 기반으로 구현돼있다.

스프링 역시 다양한 디자인 패턴을 활용하고 있는데 스프링을 이해하는 데 크게 도움될 디자인 패턴들을 살펴보자.

## 어댑터 패턴(Adapter Pattern)

어댑터를 번역하면 변환기(converter)라고 할 수 있다. 변환기의 역할은 서로 다른 두 인터페이스 사이에 통신이 가능하게 하는 것이다. 주변에서 가장 흔히 볼 수 있는 변환기로는 충전기가 있다. 휴대폰 충전기의 경우 휴대폰을 직접 전원 콘센트에 연결할 수 없기 때문에 충전기가 변환기 역할을 한다.

DB 관련 프로그램을 작성해 본 사람이라면 다양한 데이터베이스 시스템을 공통의 인터페이스인 ODBC 또는 JDBC를 이용해 조작할 수 있다는 사실을 알고 있을 것이다. 바로 ODBC/JDBC가 어댑터 패턴을 이용해 다양한 데이터베이스 시스템을 단일한 인터페이스로 조작할 수 있게 해주기 때문이다.

JDBC는 SOLID에서 개방 폐쇄 원칙(OCP)를 설명할 때도 예로 들었던 내용이다. 결국 어댑터 패턴은 개방 폐쇄 원칙을 활용한 설계 패턴이라고 할 수 있다.

예제 코드를 통해 어댑터 패턴을 이해해 보자

```java
public class ServiceA {
  void runServiceA() {
    System.out.println("SErviceA");
  }
}

public class ServiceB {
  void runServiceB() {
    System.out.println("SErviceB");
  }
}

public class Client {
	public static void main(String[] args) {
    ServiceA sa1 = new ServiceA();
    ServiceB sb1 = new ServiceB();
    
    sa1.runServiceA();
    sb1.runServiceB();
  }
}
```

위에 코드를 살펴보면 `ServiceA`와 `ServiceB` 의 각 메서드가 비슷한 일을 하는 것을 볼 수 있다. 하지만, 메서드 명이 다르다.  Adapter 패턴을 이용해서 A와 B의 메서드를 `runService` 메서드로 변환 할 수 있다.

```java
// ServiceA, ServiceB class 생략

public class AdapterServiceA {
	ServiceA sa1 = new ServiceA();
  void runService() {
		sa1.runServiceA();
  }
}

public class AdapterServiceB {
	ServiceA sb1 = new ServiceB();
  void runService() {
		sb1.runServiceB();
  }
}

public class ClientWithAdapter {
  public static void main(String[] args) {
		AdapterServiceA asa1 = new AdapterServiceA();
		AdapterServiceB asb1 = new AdapterServiceB();
    
    asa1.runService();
    asb1.runService();
  }
}
```

이렇게 하는 이유가 무엇일까? 시퀀스 다이어그램을 살펴보면 이해가 더 잘될 것이다.

<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/116843409-d05fd280-ac1a-11eb-9bee-723752936472.png">
<p style="font-weight:bold" align="center">Adapter Pattern Sequence Diagram</p>
</p>

클라이언트 입장에서는 원하는 메서드를 요청하면 Adapter 내부적으로 변환된 요청이 되도록 된다.

위에 충전기 예시를 다시 가져와보면, 지금 작성한 예제는 스마트 폰을 충전하기 위해서 220v를 C 타입 Adapter를 이용해서 충전한 것 과 같다. 만약 해외에 간다면 110v를 C 타입으로 변경해주는 Adapter가 필요할 것이다.

Adapter 패턴을 적용해서 메서드를 사용한 클라이언트(`ClientWithAdapter class`)는 변환기를 통해 `runService()`라는 동일한 메서드명으로 두 객체의 메서드를 호출하게 되었다. 이로써 클라이언트 입장에서는 변환기를 통해 동일한 메서드를 사용하는 것으로 느끼게 된다.

> Adapter Pattern 이란,
>
> 호출당하는 쪽의 메서드를 호출하는 쪽의 코드에 대응하도록 중간에 변환기를 통해 호출하는 패턴

## 프록시 패턴(Proxy Pattern)

프록시는 대리자, 대변인이라는 뜻을 가진 단어다. 대리자/대변인이라고 하면 다른 누군가를 대신에 그 역할을 수행하는 존재를 말한다.

프록시 패턴이 적용되지 않은 예제를 작성해보겠다.

```java
public class Service {
  public String runSomething() {
    return "서비스 짱 !!";
  }
}

public class ClientWithNoProxy {
  public static void main(String[] args) {
    // 프록시를 이용하지 않은 호출
    Service service = new Service();
    System.out.println(service.runSomething());
  }
}
```

이번에는 프록시 패턴이 적용된 경우를 살펴보자. 프록시 패턴의 경우 실제 서비스 객체가 가진 메서드와 같은 이름의 메서드를 사용하는데, 이를 위해 인터페이스를 사용한다. 인터페이스를 사용하면 서비스 객체가 들어갈 자리에 대리자 객체를 대신 투입해 클라이언트 쪽에서 실제 서비스 객체를 통해 메서드를 호출하고 반환값을 받는지, 대리자 객체를 통해 메서드를 호출하고 반환값을 받는지 전혀 모르게 처리할 수도 있다.

```java
public interface IService {
  String runSomething();
}

public class Service implements IService {
  public String runSomething() {
    return "서비스 짱 !!";
  }
}

public class Proxy implements IService {
  IService service1;
  
  public String runSomething() {
    System.out.println("호출에 대한 흐름 제어가 주목적, 반환 결과를 그대로 전달");
		
    service1 = new Service();
    return service1.runSomething();
  }
}

public class ClientWithProxy {
  public static v한id main(String[] args) {
    // 프록시를 이용한 호출
    IService proxy = new Proxy();
    System.out.println(proxy.runSomething());
  }
}
```

위에 작성된 것을 보면 `IService` 인터페이스를 구현한 `Proxy` 객체가 `runSomething()` 메소드를 대신 호출한다. 따라서 클라이언트 입장에서는 어떤 `IService` 구현체가 동작하는 지 전혀 모르게된다. 이게 바로 프록시 패턴이다.

시퀀스 다이어그램으로 한번 더 이해해보자

<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/116844680-b9bb7a80-ac1e-11eb-89db-43bde148283c.png">
<p style="font-weight:bold" align="center">Proxy Pattern Sequence Diagram</p>
</p>

Proxy 패턴 객체에 감춰져 request가 이루어지는 것을 볼 수 있다.

Proxy 패턴의 중요 포인트를 짚어 보자.

- 대리자는 실제 서비스와 같은 이름의 메서드를 구현한다. 이때 인터페이스를 사용한다.
- 대리자는 실제 서비스에 대한 참조 변수를 갖는다.(합성)
- 대리자는 실제 서비스의 같은 이름을 가진 메서드를 호출하고 그 값을 클라이언트에게 돌려준다.
- 대리자는 실제 서비스의 메서드 호출 전후에 별도의 로직을 수행할 수도 있다.

> Proxy Pattern 이란,
>
> 제어 흐름을 조정하기 위한 목적으로 중간에 대리자를 두는 패턴

## 데코레이터 패턴(Decorator Pattern)

데코레이터는 도장/도배업자를 의미한다. 여기서는 장식자라는 뜻을 가지고 생각을 해보겠다. 데코레이터 패턴이 원본에 장식을 더하는 패턴이라는 것이 이름에 잘 드러나 있다.

데코레이터 패턴은 프록시 패턴과 구현 방법이 같다. 다만 프록시 패턴은 클라이언트가 최종적으로 돌려 받는 반환값을 조작하지 않고 그대로 전달하는 반면 데코레이터 패턴은 클라이언트가 받는 반환값에 장식을 덧입한다.

간단히 예제를 살펴보자

```java
public interface IService {
  public abstract String runSomething()
}

public class Service implements IService {
  public String runSomething() {
    return "서비스 Good!!!";
  }
}

public class Decorator implements IService {
  IService service;
  
  public String runSomething() {
    System.out.println("호출에 대한 장식이 주목적이다. 클라이언트에게 반환 결과에 장식을 더하여 전달.");
    
    service = new Service();
    return "진짜 " + service.runSomething();
  }
}

public class ClientWithDecorator {
  public static v한id main(String[] args) {
    // 프록시를 이용한 호출
    IService decorator = new Decorator();
    System.out.println(decorator.runSomething());
  }
}
```

결과적으로 프록시 패턴처럼 `service.runSomething()`의 결과가 그대로 나오는 것이 아니라 `"진짜 "` 라는 장식이 더해져서 전달된다.

데코레이터 패턴의 중요 포인트를 짚어보자.

- 장식자는 실제 서비스와 같은 이름의 메서드를 구현한다. 이때 인터페이스를 사용한다.
- 장식자는 실제 서비스에 대한 참조 변수를 갖는다.(합성)
- 장식자는 실제 서비스의 같은 이름을 가진 메서드를 호출하고, 그 반환값에 **장식을 더해** 클라이언트에게 돌려준다.
- 장식자는 실제 서비스의 메서드 호출 전후에 별도의 로직을 수행할 수도 있다.

> Decorator Pattern 이란,
>
> 메서드 호출의 반환값에 변화를 주기 위해 중간에 장식자를 두는 패턴



프록시 패턴과 데코레이터 패턴은 중간에 인터페이스를 두고 흐름을 바꾸거나 장식을 추가하는 것이기 때문에 OCP를 만족하는 것을 볼 수 있다. 또한, 인터페이스에 의존하기 때문에 DIP도 만족을 한다 !

## 싱글턴 패턴(Singleton Pattern)

싱글턴 패턴이란 인스턴스를 하나만 만들어 사용하기 위한 패턴이다. 커넥션 풀, 스레드 풀, 디바이스 설정 객체 등과 같은 경우 인스턴스를 여러 개 만들게 되면 불필요한 자원을 사용하게 되고, 또 프로그램이 예상치 못한 결과를 낳을 수 있다. 싱글턴 패턴은 오직 인스턴스를 하나만 만들고 그것을 계속해서 재사용한다.

싱글턴 패턴을 적용할 경우 의미상 두 개의 객체가 존재할 수 없다. 이를 구현하려면 객체 생성을 위한 new에 제약을 걸어야하고, 만들어진 단일 객체를 반환할 수 있는 메서드가 필요하다. 따라서 필요한 요소를 생각해 보면 다음 세 가지가 반드시 필요하다.

- new를 실행할 수 없도록 생성자에 private 접근 제어자를 지정한다.
- 유일한 단일 객체를 반환할 수 있는 정적 메서드가 필요하다.
- 유일한 단일 객체를 참조할 정적 참조 변수가 필요하다.

```java
public class Singleton {
  static Singleton singletonObject; // 정적 참조 변수
  
  private Singleton() { }; // private 생성자
  
  public static Singleton getInstance() {
    if (singletonObject == null) {
      singletonObject = new singleton();
    }
    
    retrun singletonObject;
  }
}
```

유일한 단일 객체를 저장하기 위한 정적 참조 변수, new를 실행할 수 없도록 생성자를 private 설정, 유일한 단일 객체를 반환할 수 있는 정적 메서드를 이용해서 위와 같이 싱글톤 클래스를 만들 수 있다.

싱글턴은 하나의 단일 객체를 여러 부분에서 사용하기위해 사용된다. 그렇기 때문에 속성을 갖지 않게 하는 것이 정석이다. 단일 객체가 속성을 갖게 되면 하나의 참조 변수가 변경한 단일 객체의 속성이 다른 참조 변수에 영향을 미치기 때문이다. 이는 전역/공유 변수를 가능한 한 사용하지 말라는 지침과 일맥상통한다. 다만, 읽기 전용 속성을 갖는 것은 문제가 되지 않는다. 더불어 단일 객체가 다른 단일 객체에 대한 참조를 속성으로 가진 것 또한 문제가 되지 않는다. **이는 스프링의 싱글턴 빈이 가져야 할 제약조건이기도 하다.**

기억해 둬야하는 싱글턴 패턴의 특징은 다음과 같다.

- private 생성자를 갖는다.
- 단일 객체 참조 변수를 정적 속성으로 갖는다.
- 단일 객체 참조 변수가 참조하는 단일 객체를 반환하는 `getInstatnce()` 정적 메서드를 갖는다.
- 단일 객체는 쓰기 가능한 속성을 갖지 않는 것이 정석이다.

> Singleton Pattern 이란,
>
> 클래스의 인스턴스, 즉 객체를 하나만 만들어 사용하는 패턴



**2편에서 템플릿 메서드 패턴, 팩토리 메서드 패턴, 전략 패턴, 템플릿 콜백 패턴이 이어서 진행됩니다.**