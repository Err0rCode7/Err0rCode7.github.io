---
bg: "Review.jpg"
layout: post
title:  "Spring Aop와 Java Dynamic Proxy, CGLib"
crawlertitle: "Spring Aop와 Java Dynamic Proxy, CGLib"
summary: "Spring Aop와 Java Dynamic Proxy, CGLib"
date: 2021-04-30 01:22:00 +0900
category: Spring
author: Err0rCode7
---

#### Spring Aop은 어떻게 되어있을까 ?
##### 스프링의 3대 프로그래밍 모델 중 AOP에 대해 정리하는 글입니다.

## AOP

AOP는 Aspect-Oriented Programming의 약자이고 번역하면 관점 지향 프로그래밍이다.

Spring DI가  의존성(new)에 대한 주입이라면 스프링 AOP는 로직(code) 주입이라고 할 수 있다.

<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/116520775-1f052800-a90e-11eb-870d-829c4527c9b8.png">
<p style="font-weight:bold" align="center">AOP</p>
</p>

그림을 보면 입금, 출금, 이체 모듈에서 로깅, 보안, 트랜잭션 기능이 반복적으로 나타나는 것을 볼 수 있다. 프로그램을 작성하다 보면 이처럼 다수의 모듈에 공통적으로 나타나는 부분이 존재하는데, 바로 이것을 횡단 관심사(cross-cutting concern)라고 한다.

DB 연동 프로그램을 작성한 적이 있다면 insert 연산이든 update 연산이든 항상 반복해서 등장하는 다음과 같은 코드르 본 적이 있을 것이다.

```java
DB 커넥션 준비
Statement 객체 준비
  
try {
  DB 커넥션 연결
  Statement 객체 세팅
  insert / update / delete / select 실행
} catch .. {
  예외 처리
} catch .. {
  예외 처리
} finally {
  DB 자원 반납
}
```

위의 의사 코드를 보면 `insert / update / delete / select 실행` 을 제외하면 모두 어떤 데이터베이스 연산을 하든 공통적으로 나타나는 코드다. 이를 바로 횡단 관심사라고 한다. 그리고 `insert / update / delete / select 실행` 부분을 핵심 관심사라고 한다.

> 코드 = 핵심 관심사 + 횡단 관심사

핵심 관심사는 모듈별로 다르지만 횡단 관심사는 모듈별로 반복되어 중복해서 나타나는 부분이다. 반복/중복을 처리하기 위한 것이 바로 AOP이다. 결과적으로 자연스럽게 SRP(단일 책임 원칙)이 적용된다.

위에 말했듯이 AOP는 로직(code) 주입이라고 할 수 있다. **그렇다면 객체 지향에서 로직을 주입한다면 어디에 주입할 수 있을까 ?**

객체 지향에서 로직(코드)가 있는 곳은 다연히 메서드 안쪽이다. 그럼 메서드에서 코드를 주입할 수 있는 곳은 몇 군데일까?

<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/116521854-748e0480-a90f-11eb-983e-83b8a5cabc6c.png">
<p style="font-weight:bold" align="center">메서드에서 코드 주입이 가능한 위치</p>
</p>

위 그림에서 볼 수 있듯이 Around, Before, After, AfterReturning, AfterThrowing으로 총 5군데다. 그림에서는 메서드 시작, 메서드 종료 후라고 보이는데 메서드 시작 직후, 메서드 종료 직전으로 생각해도 된다.

## AOP 용어

AOP에서 사용되는 5가지의 용어가 있다.

-  Pointcut

  Aspect 적용 위치 지정자라고 표현할 수 있다.

  ```java
  // Aspect Bean
  @Aspect
  public class MyAspect {
    @Before("execution(⛓* runSomething()⛓)")
    public void before(JointPoint joinPoint) {
      System.out.println("AOP !");
    }
  }
  ```

  ⛓ 사이가 바로 Pointcut이다. 우선 `@Before("execution(* runSomething())")` 의 의미부터 살펴보자면 지금 선언하고 있는 메서드 (public void before)를 ⛓로 감싸져 있는 부분의 메서드가 실행되기 전(`@Before`)에 실행하라는 의미이다. (⛓는 구분하기 위해 적어놓은것 뿐이다.)

  결국 Pointcut이라고 하는 것은 횡단 관심사를 적용할 타깃 메서드를 선택하는 지시자(메서드 선택 필터)인 것이다. 이를 줄여서 표현하면 Pointcut이란 `"타깃 클래스의 타깃 메서드 지정자"` 라고 할 수 있다. 그럼 위에서는 Aspect 적용 위치 지정자라고 표현했을까?

  스프링 AOP만 보자면 Aspect를 메서드에만 적용할 수 있으니 타깃 메서드 지정자라는 말이 틀리지 않다. 그렇지만 Aspectj 처럼 스프링 AOP 이전부터 있었고 지금도 유용하게 사용되는 다른 AOP 프레임워크에서는 메서드 뿐만 아니라 속성 등에도 Aspect를 적용할 수 있기에 그것들까지 고려한다면 Aspect 적용 위치 지정자(지시자)가 맞는 표현이다. Pointcut을 메서드 선정 알고리즘이라고도 한다.

  타깃 메서드 지정자에는 익히 잘 알려진 정규식과 Aspectj 표현식 등을 사용할 수 있다. 간단하게 소개하면 다음과 같다.

  > [접근제한자패턴] 리턴타입패턴 [패키지&클래스패턴.]메서드이름패턴(파라미터패턴) [throws 예외패턴] \

  여기서 대괄호 []는 생략이 가능하다는 의미이다. 필수 요소는 리턴 타입 패턴, 메서드 이름 패턴, 파라미터 패턴 뿐이다. `*`은 어떠한 것이라도 좋다라는 의미이다.

- JointPoint

  연결 가능한 지점이라고 표현할 수 있다.

  Pointcut은 JointPoint의 부분 집합이다. Pointcut의 후보가 되는 모든 메서드들이 JointPoint, 즉 Aspect 적용이 가능한 지점이 된다. 그래서 삼단 논법을 적용하면 JoinPoint란 `"Aspect 적용이 가능한 모든 지점을 말한다"` 라고 결론 지을 수 있다.

- Advice

  Advice는 pointcut에 적용할 로직, 즉 메서드를 의미한다. 여기에 더해 언제라는 개념까지 포함하고 있다. 결국 Advice란 Pointcut에 언제, 무엇을 적용할지 정의한 메서드다.

  ```java
  // Aspect Bean
  @Aspect
  public class MyAspect {
    @Before("execution(* runSomething())")
    public void before(JointPoint joinPoint) {
      System.out.println("AOP !");
    }
  }
  // 위에서 아래의 덩어리 표현이 Advice 인 것이다.
    @Before
    public void before(JointPoint joinPoint) {
      System.out.println("AOP !");
    }
  // pointcut 전에 before() 메소드를 실행하라 !
  ```

- Aspect

  AOP에서 Aspect는 여러 개의 Advice와 여러 개의 Pointcut의 결합체를 의미하는 용어다. 수식으로 표현하면 아래와 같다.

  > Aspect = Advice들 + Pointcut들

  Advice는 `언제`, `무엇을`을 의미하는 것이었고 Pointcut은 `어디에`를 의미하는 것이었다. 결국 Aspect는 `언제 + 어디에 + 무엇을`이 된다.

- Advisor

  Advisor는 다음과 같은 수식으로 표현할 수 있다.

  > Advisor = 한 개의 Advice + 한 개의 Pointcut

  Advisor는 스프링 AOP에서만 사용하는 용어이며 다른 AOP 프레임워크에서는 사용하지 않는다. 또 스프링 버전이 올라가면서 이제는 쓰지 말라고 권고하는 기능이기도 하다. 스프링이 발전해 오면서 다수의 Advice와 다수의 Pointcut을 다양하게 조합해서 사용할 수 있는 방법, 즉 Aspect가 나왔기 때문에 하나의 Advice와 하나의 Pointcut만을 결합하는 Advisor가 필요가 없어졌기 때문이다.

## Spring AOP

스프링에서 AOP를 적용한 것을 살펴보자

```java
public interface Person {
  void runSomething();
}

public class Boy implements Person {
  public void runSomething() {
    System.out.println("컴퓨터로 게임을 한다.");
  }
}

public class Start {
  public static void main(String[] args) {
    // aspect bean setting
    ApplicationContext context = new ClassPathXmlApplicationContext("....xml")
    
    Person remeo = context.getBean("boy", Person.class);
    
    romeo.runSomething();
  }
}

// Aspect Bean
@Aspect
public class MyAspect {
  @Before("execution(public void Boy.runSomething())")
  public void before(JointPoint joinPoint) {
    System.out.println("AOP !");
  }
}
```

위와 같이 Aspect bean을 xml로 설정하면 Boy클래스의 `runSomething()` 메소드 실행 직전에 `before` 메소드가 실행된다.

> 어떻게 runSomething 메소드 전에 before 메소드가 실행되는 것일까?

Spring AOP는 Proxy 를 이용한다. [Spring 4.0.1 docs](https://docs.spring.io/spring-framework/docs/4.0.1.RELEASE/spring-framework-reference/htmlsingle/#aop-understanding-aop-proxies)



<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/116526040-20d1ea00-a914-11eb-8a6b-559de4ee1e0f.png">
<img src="https://user-images.githubusercontent.com/48249549/116526306-6c849380-a914-11eb-8577-4ceb50490067.png">
<p style="font-weight:bold" align="center">Proxy와 Spring AOP</p>
</p>



proxy를 통해 OCP를 지키면서 타겟이 되는 메소드를 실행 하기 전후에 원하는 기능을 확장할 수 있게된다.

그렇다면 Proxy는 무엇일까?

일반적으로 부르는 Proxy는 실제 Target의 기능을 수행하면서 기능을 확장하거나 추가하는 실제 **객체**를 의미한다.

> Proxy 와 Proxy Pattern은 다르다.
>
> Proxy Pattern은 실제로 Target에 대한 기능을 확장하거나, 추가하지 않는다. 그저 클라이언트가 타깃에 접근하는 방식을 변경해주는 역할을 한다.

프록시 클래스는 많은 것들을 원래 코드를 수정하지 않고 편리하게 구현할 수 있다.

아래는 일부 예시이다.

- 메소드가 시작하고 끝날 때 로그를 남긴다.
- argument에 대한 추가적인 확인을 수행한다.
- 원래 클래스의 행동을 모킹한다.
- 비싼 자원에 대한 lazy 접근을 실행한다.

Spring AOP는 사용자의 특정 호출 시점에 IoC 컨테이너에 의해 AOP를 할 수 있는 Proxy Bean을 생성해준다. 동적으로 생성된 Proxy Bean은 타깃의 메소드가 호출되는 시점에 부가기능을 추가할 메소드를 자체적으로 판단하고 가로채어 부가기능을 주입해준다. 이처럼 호출 시점에 동적으로 위빙을 한다 하여 런타임 위빙(Runtime Weaving)이라 한다.

따라서 Spring AOP는 런타임 위빙의 방식을 기반으로 하고 있으며, Spring에선 런타임 위빙을 할 수 있도록 상황에 따라 JDK Dynamic Proxy와 CGLib 방식을 통해 Proxy Bean을 생성 해준다.

## 두 가지 AOP Proxy가 생성되는 상황

Spring은 AOP Proxy를 생성하는 과정에서 자체 검증 로직을 통해 타깃의 인터페이스 유무를 판단한다.

만약 타깃(횡단 기능(Advice)이 적용될 객체)이 하나 이상의 인터페이스를 구현하고 있는 클래스라면 JDK Dynamic Proxy의 방식으로 생성되고 인터페이스를 구현하지 않은 클래스라면 CGLib의 방식으로 AOP 프록시를 생성해준다.

## JDK Dynamic Proxy 방식

JDK Dynamic Proxy란 Java의 리플렉션 패키지에 존재하는 Proxy라는 클래스를 통해 생성된 Proxy 객체를 의미한다.

리플랙션의 Proxy 클래스가 동적으로 Proxy를 생성해준다하여 JDK Dynamic Proxy 라 불린다. 이 클래스를 사용하여 Proxy를 생성하기 위해선 몇 가지 조건이 있지만 핵심은 타깃의 인터페이스를 기준으로 Proxy를 생성해준다는 것이다.

Spring AOP는 JDK Dynamic Proxy 방식을 디폴트로 사용한다.

JDK Dynamic Proxy를 사용하여 Proxy 객체를 생성하는 방법은 간단하다.

```java
Object proxy = Proxy.newProxyInstance(ClassLoader       // 클래스로더
                                    , Class<?>[]        // 타깃의 인터페이스
                                    , InvocationHandler // 타깃의 정보가 포함된 Handler
```

단순히 리플랙션의 Proxy 클래스의 newProxyInstance 메소드를 사용하면 된다. JDK Dynamic Proxy가 이 파라미터를 가지고 Proxy 객체를 생성해주는 과정은 다음과 같다.

1. 타깃의 인터페이스를 자체적인 검증 로직을 통해 ProxyFactory에 의해 타깃의 인터페이스를 상속한 Proxy 객체 생성
2. Proxy 객체에 InvocationHandler를 포함시켜 하나의 객체로 반환

여기서 핵심은 무엇보다 인터페이스를 기준으로 Proxy 객체를 생성해준다는 점이다. 따라서 구현체는 인터페이스를 상속받아야하고, @Autowired를 통해 생성된 Proxy Bean을 사용하기 위해선 반드시 인ㄴ터페이스의 타입으로 지정해줘야 한다.

```java
@Controller
public class UserController{
  @Autowired
  private MemberService memberService; // <- Runtime Error 발생...
  ...
}

@Service
public class MemberService implements UserService{
  @Override
  public Map<String, Object> findUserId(Map<String, Object> params){
    ...isLogic
    return params;
  }
}
```

위와 같이 `UserService` 인터페이스 타입이 아닌 `MemberService` 타입으로 @Autowired를 하게 된다면 Runtime Exception이 발생하게된다.

## 인터페이스 기준 그리고 내부적인 검증 코드

다른 관점에서 보자면 JDK Dynamic Proxy는 Proxy 패턴의 관점을 구현한 구현체라 할 수 있다.

이 Proxy 패턴은 접근제어의 목적으로 Proxy를 구성한다는 점도 중요하지만, 무엇보다 사용자의 요청이 기존의 타깃을 그대로 바라볼 수 있도록 타깃에 대한 위임코드를 Proxy 객체에 작성해줘야 한다. 생성된 Proxy 객체의 타깃에 대한 위임코드는 바로 InvocationHandler에 작성해줘야 한다.

따라서 사용자의 요청이 최종적으로 생성된 Proxy의 메소드를 통해 호출할 때 내부적으로 invoke에 대한 검증과정이 이뤄진다. 결과적으로 코드는 다음과 같다.

```java
public Object invoke(Object proxy, Method proxyMethod, Object[] args) throws Throwable {
  Method targetMethod = null;
  // 주입된 타깃 객체에 대한 검증 코드
  if (!cachedMethodMap.containsKey(proxyMethod)) {
    targetMethod = target.getClass().getMethod(proxyMethod.getName(), proxyMethod.getParameterTypes());
    cachedMethodMap.put(proxyMethod, targetMethod);
  } else {
    targetMethod = cachedMethodMap.get(proxyMethod);
  }

  // 타깃의 메소드 실행
  Ojbect retVal = targetMethod.invoke(target, args);
  return retVal;
}
```

이 과정에서 검증과정이 이뤄지는 까닭은 다름이 아닌 Proxy가 기본적으로 인터페이스에 대한 Proxy만을 생성해주기 때문이다. 따라서 개발자가 타깃에 대한 정보를 잘못 주입할 경우를 대비하여 JDK Dynamic Proxy는 내부적으로 주입된 타깃에 대한 검증코드를 형성하고 있다.

## CGLib(Code Generator Library)

그렇다면 CGLIB은 무엇일까?

CGLib은 Code Generator Library의 약자로, 클래스의 바이트코드를 조작하여 Proxy 객체를 생성해주는 라이브러리이다.

Spring은 CGLib을 사용하여 인터페이스가 아닌 타깃의 클래스에 대해서도 Proxy를 생성해주고 있다.

CGLib은 Enhancer라는 클래스를 통해 Proxy를 생성할 수 있다.

```java
Enhancer enhancer = new Enhancer();
         enhancer.setSuperclass(MemberService.class); // 타깃 클래스
         enhancer.setCallback(MethodInterceptor);     // Handler
Object proxy = enhancer.create(); // Proxy 생성
```

Handler인 MethodInterceptor를 통해 원하는 메소드를 검증하고 실행하는 Proxy를 만들 수 있다.

CGLib은 타깃의 클래스를 상속받아 Proxy를 생성해준다. 이 과정에서 CGLib은 타깃 클래스에 포함된 모든 메소드를 재정의하며 Proxy를 생성해준다. 이 때문에 CGLib은 Final 메소드 또는 클래스에 대해 재정의를 할 수 없으므로 Proxy를 생성할 수 없다는 단점이 있다. 하지만 CGLib은 바이트 코드로 조작하여 Proxy를 생성해주기 때문에 성능에 대한 부분이 JDK Dynamic Proxy보다 좋다.

## invoke의 차이, 성능의 차이

성능의 차이의 근본적인 이유는 CGLib은 타깃에 대한 정보를 제공 받기 때문이다.

따라서 CGLib은 제공받은 타깃 클래스에 대한 바이트 코드를 조작하여 Proxy를 생성하기 때문에 Handler안에서 타깃의 메소드를 호출할 때 다음과 같은 코드가 형성된다.

```java
public Object invoke(Object proxy, Method proxyMethod, Object[] args) throws Throwable {
  Method targetMethod = target.getClass().getMethod(proxyMethod.getName(), proxyMethod.getParameterTypes());
  Ojbect retVal = targetMethod.invoke(target, args);
  return retVal;
}
```

1. 메소드가 처음 호출되었을 때 동적으로 타깃의 클래스의 바이트 코드를 조작
2. 이후 호출시엔 조작된 바이트 코드를 재사용

CGLib은 성능이 좋긴하지만, Spring은 JDK Dynamic를 기반으로 Proxy를 생성해주고 있다.

## 권장하지 않았던 CGLib

<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/116579470-619a2500-a94d-11eb-9e04-dccf4eaa3995.png">
<p style="font-weight:bold" align="center"><a src="https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop">Spring AOP Reference</a></p>
</p>

CGLib은 3 가지의 한계가 존재한다.

- net.sf.cglib.proxy.Enhancer 의존성 추가
- default 생성자
- 타깃의 생성자 두 번 호출

첫 번째는 Spring에서 기본적으로 지원하지 않는 방식이었기 때문에 별도로 의존성을 추가하여 개발을 했고 그 다음으로는 CGLib 구현을 위해선 반드시 파라미터가 없는 default 생성자가 필요했다. 생성된 CGLib Proxy의 메소드를 호출하게 되면 타깃의 생성자가 2번 호출된다는 단점도 있다.

## Spring Boot가 선택한 CGLib

하지만 어느 시점부터 Spring Boot에선 CGLib을 방식으로 Proxy를 생성해주고 있다. 위에 3가지 문제가 해결이 되었기 때문이다.

CGLib의 의존성을 추가하여 개발해야 된다는 점은 Spring 3.2 버전부터 CGLib을 Spring Core 패키지에 포함시켜 더이상 의존성을 추가하지 않아도 개발할 수 있게 되었다.

그 다음 4버전에선 Objensis 라이브러리의 도움을 받아 default 생성자 없이도 Proxy를 생성할 수 있게 되었고, 생성자가 2번 호출되던 상황도 같이 개선이 되었다.

결과적으로 기존의 CGLib이 가지고 있던 대부분의 한계들이 개선이 되어, Spring에선 성능이 좋은 CGLib으로 Proxy를 생성하게 되었다. 

## Reference

[자바 객체지향의 원리](https://wikibook.co.kr/java-oop-for-spring/)

[https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-introduction-proxies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-introduction-proxies)

[https://www.baeldung.com/cglib](https://www.baeldung.com/cglib)

[https://gmoon92.github.io/spring/aop/2019/04/20/jdk-dynamic-proxy-and-cglib.html](https://gmoon92.github.io/spring/aop/2019/04/20/jdk-dynamic-proxy-and-cglib.html)

[https://velog.io/@hanblueblue/Spring-Proxy-1-Java-Dynamic-Proxy-vs.-CGLIB](https://velog.io/@hanblueblue/Spring-Proxy-1-Java-Dynamic-Proxy-vs.-CGLIB)