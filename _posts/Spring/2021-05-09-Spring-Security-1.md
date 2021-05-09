---
bg: "Review.jpg"
layout: post
title:  "Spring Security - 1"
crawlertitle: "Spring Security - 1"
summary: "Spring Security - 1"
date: 2021-05-09 23:20:00 +0900
category: Spring
author: Err0rCode7
---

#### Spring Security
##### Spring Security에 대해 공부하고 정리한 내용입니다.

## What is a Spring Security ?

Spring Security란 보안 솔루션을 제공해주는 Spring 기반의 스프링 하위 프레임워크이다.

Spring Security에서 제공해주는 보안 솔루션을 사용하면 개발자가 보안 관련 로직을 짤 필요가 없어 굉장히 간편하다.

🔗[인증과 인가](https://err0rcode7.github.io/backend/2021/05/06/%EC%9D%B8%EC%A6%9D%EA%B3%BC%EC%9D%B8%EA%B0%80.html)의 개념이 부족하다면 공부하고오는 것을 추천한다.

## 필터에서 동작한다.

Spring에서 클라이언트의 요청은 `Filter`를 타고 `Servlet`(Spring MVC에서 DispatcherServlet)으로 도달하게 되는데, Spring Security는 `Servlet` 도달하기 이전에 서블릿 필터 기반에서 동작을 한다. 따라서 단일 HTTP request에 대한 핸들링을 한다.

<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/117572859-d3bdf700-b10f-11eb-89cf-e8862c383516.png">
<p style="font-weight:bold" align="center">Spring Security 동작 위치</p>
</p>

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
    // do something before the rest of the application
    chain.doFilter(request, response); // invoke the rest of the application
    // do something after the rest of the application
}
```

`Filter` 는 위에와 같이 체이닝의 형태로 여러 필터를 통과하여 `Servlet`에 도달한다.

Spring에서는 `Filter`를 `DelegatingFilterProxy`를 이용해서 추가할 수 있는데 Spring Security는 `DelegatingFilterProxy`를 이용해서 Spring 컨테이너에 생성된 `FilterChainProxy` bean을 이용하여 SecurityFilter를 구성하는 `SecurityFilterChain`을 만든다. (springSecurityFilterChain 이름으로 생성된 빈을 ApplicationContext에서 찾아 위임을 요청)

<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/117573539-72982280-b113-11eb-8a1d-f8ba7a5dd1b0.png">
<p style="font-weight:bold" align="center">Spring Security와 SecurityFilterChain</p>
</p>

실질적으로 `FilterChainProxy`에서 보안을 처리한다. 결과적으로 `SecurityFilterChain`을 통해 URL에 따른 인증과 인가 등의 보안 처리가 된다.

위에 사진을 보면 `/api/messages/` URL의 요청이 온다면 `SecurityFilterChain` 0번째 필터를 타게되고 `/messages/` URL의 요청이 온다면 필터 체인을 0번째부터 쭉 찾다가 마지막 남은 `SecurityFilterChain` n번째 필터에 타게된다.

## 인증과 인가 예외처리

`SecurityFilterChain` 중 `ExceptionTranslationFilter` 는 잘못된 권한에 대한 `AccessDeniedException` 과 `AuthenticationException` 예외의 응답을 다룬다.

<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/117573789-d111d080-b114-11eb-88c9-04c305288d1b.png">
<p style="font-weight:bold" align="center">인증과 인가 예외처리</p>
</p>

```java
// ExceptionTranslationFilter pseudocode

try {
    filterChain.doFilter(request, response); 
} catch (AccessDeniedException | AuthenticationException ex) {
    if (!authenticated || ex instanceof AuthenticationException) {
        startAuthentication(); 
    } else {
        accessDenied(); 
    }
}
```

`ExceptionTranslationFilter pseudocode`를 보면 단번에 이해가 쉽다. `ExceptionTranslationFilter`에서 다음 필터체인으로 넘겨줄때(`doFilter`) `try-catch`로 이후 필터에서 일어나는 인가와 인증에 대한 예외를 받는 방식이다.

## 인증(Authentication)

Spring Security의 인증에 사용되는 아키텍처 컴포넌트을 살짝 살펴보면 다음과 같다.

- **SecurityContextHolder**

  Spring Security가 누가 인증됐는 지의 세부정보들을 저장하는 곳이다.

- **SecurityContext**

  `SecurityContextHolder`로 부터 얻게된 현재 인증된 유저의 `Authentication`를 포함하고 있는 애플리케이션 경계이다.

- **Authentication**

  한 유저가 인증하기 위해 제공한 credential(password 같은 자격을 의미) 또는 SecurityContext로부터 현재 유저를 제공하기 위한 `AuthenticationManager`의 인풋 값이다.

- **GrantedAuthority**

  Authentication를 통해 인증된 주체(principal)에 대한 권한

- **AuthenticationManager**

  Spring Security의 필터에서 인증을 수행하는 API

- **ProviderManager**

  `AuthenticationManger`의 가장 일반적인 구현체

- Request Credentials with **AuthenticationEntryPoint**

  클라이언트로부터 credentials(자격)을 요청할 때 사용된다.

- **AbstractAuthenticationProcessingFilter**

  인증을 위해 기반이 되는 필터. 높은 수준의 인증 흐름과 조각들이 어떻게 함께 작동하는지 나타나있다.

## SecurityContextHolder, SecurityContext

`SecurityContextHolder`는 Spring Security 인증 모델의 중심이며 `SecurityContext`를 포함하고있다.

`SecurityContext`는 `SecurityContextHolder`로 부터 얻을 수 있으며 인증(Authentication) 객체를 포함한다.

<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/117574868-2f8d7d80-b11a-11eb-9cf1-b8950952f6c2.png">
<p style="font-weight:bold" align="center">SpringContextHolder</p>
</p>

값이 채워진 경우 `SpringContextHolder`는 현재 인증된 사용자로 사용된다.

```java
SecurityContext context = SecurityContextHolder.createEmptyContext(); 
Authentication authentication =
    new TestingAuthenticationToken("username", "password", "ROLE_USER"); 
context.setAuthentication(authentication);

SecurityContextHolder.setContext(context); 
```

위에 예제처럼 빈 `SecurityContext`를 만들고 아주 간단한 인증 토큰을 생성해서 `SecurityContext`에 넣어준다음 이 Context를 `SpringContextHolder`에 넣음으로써 SpringSecurity는 인가를 위한 정보로 사용할 것이다.

따라서 인증된 주체(principal)에 대한 정보를 얻고싶다면 아래와 같이 `SecurityContextHolder`를 사용하면된다.

```java
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
String username = authentication.getName();
Object principal = authentication.getPrincipal();
Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
```

`SecurityContextHolder`는 default로 `ThreadLocal`를 이용하여 인증된 유저의 세부사항들을 저장한다. 이는 인자로 메소드에 값을 넘겨주지 않아도 SecurityContext는 항상 같은 스레드에서 메소드를 호출할 수 있다는 것을 의미한다.

SpringSecurity에서는 `ThreadLocal`을 안전하게 사용하기 위해서 `FilterChainProxy`가 `SecurityContext`를 항상 지우는 것을 보장한다.

#### 이어서 2편에서 진행하겠습니다.

## Reference

[https://docs.spring.io/spring-security/site/docs/current/reference/html5/#servlet-hello](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#servlet-hello)

