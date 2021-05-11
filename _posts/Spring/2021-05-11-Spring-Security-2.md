---
bg: "Review.jpg"
layout: post
title:  "Spring Security - 2"
crawlertitle: "Spring Security - 2"
summary: "Spring Security - 2"
date: 2021-05-11 16:20:00 +0900
category: Spring
author: Err0rCode7
---

#### Spring Security - 2
##### Spring Security에 대해 공부하고 정리한 내용입니다.

## Authentication

**Authentication**은 Spring Security에서 두가지 목적을 갖고있다.

- `AuthenticationManager`의 입력으로 한 유저가 인증하기위해 제공한 credentials을 제공하기위함. 이 경우 `isAuthenticated()`의 결과는 false다.
- 현재 인증된 사용자를 나타낸다. 현재 `Authentication`이 SecurityContext로 부터 얻을 수 있다면 인증된 사용자라고 나타낼 수 있다.

**Authentication**은 다음을 포함하고 있다.

- principal - 유저를 식별한다. 유저 이름과 패스워드와 함께 인증이 시도 될 때, principal은 보통 `UserDetails`의 인스턴스이다.
- credentials - 보통 패스워드를 나타낸다. 유저가 인증된 이후에는 노출이 되지 않도록 제거된다.
- authorities - `GrantedAuthority`는 유저가 권한이 있다는 높은 레벨의 허가를 나타낸다. 예를 들어 roles or scopes를 얘기한다.

## GrantedAuthority

위에 Authentication에서 보았듯이 `GrantedAuthority`는 유저가 권한이 있다는 높은 레벨의 허가를 나타낸다.

`GrantedAuthority`는 `Authetication.getAuthorities` 메소드를 통해 얻을 수 있다. 이 메소드는 `GrantedAuthority`의 컬렉션을 제공한다.

## AuthenticationManager

**AuthenticationManager**는 Spring Security 필터가 인증을 수행하는 방법을 정의한 API다. 반환 된 `Authentication`은 `AuthenticationManager`를 실행한 컨트롤러(Spring Security 필터들)에 의해 `SecurityContextHolder`에 설정이된다.

`AuthenticationManager`를 사용하지 않고 Spring Security 필터에서 말고 직접 `Authentication`을 `SecurityContextHolder`에 설정할 수 있다.

가장 일반적인 AuthenticationManager의 구현체는 `ProviderManager` 이다.

## ProviderManager

**ProviderManager** 가장 일반적인 AuthenticationManager의 구현체이다.

<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/117766457-95891a80-b26a-11eb-955a-7224343d6bf5.png">
<p style="font-weight:bold" align="center">ProviderManager 동작</p>
</p>

`ProviderManager`는 `AuthenticationProvider` 리스트로 위임을 한다. 각각의 `AuthenticationProvider`는 인증이 성공, 실패를 나타낼 수 있고 결정할 수 없으면 다음 `AuthenticationProvider`에게 결정하도록 내려보낸다. 만약 어떠한 `AuthenticationProvider`가 인증을 할 수 없으면 `AuthenticationException`의 `ProviderNotFoundException`와 함께 인증 실패를 알릴 것이다.

여기서 결정할 수 없다는 것은 각 `AuthenticationProvider`는 특정 유형의 인증을 수행하는 방법을 갖고 있고 이에 맞지 않다는 것을 얘기한다. 예를 들어 한 `AuthenticationProvider`는 유저 이름과 패스워드로 검증을 할 수 있는 반면, 다른 `AuthenticationProvider`는 다른 방식으로 검증을 한다. 즉, 다양한 인증 방식을 지원하면서 그 중 하나의 방식으로 인증을 할 수 있다.

## AuthenticationProvider

위에서 보았듯이 다수의 `AuthenticationProvider`들은 `ProviderManger`에 속해 있고 각각의 `AuthenticationProvider`는 특정 유형의 인증을 수행한다.

예를 들어 `DaoAuthenticationProvider`는 유저 이름과 패스워드로 인증을 지원하고 반면에 `JwtAuthenticationProvider`는 JWT 토큰으로 인증을 지원한다.

## Request Credential with `AuthenticationEntryPoint`

`AuthenticationEntryPoint`는 유저에게 credential을 요청하는 HTTP 응답을 보내는 것에 사용된다.

인증할 수 있는 정보를 클라이언트가 같이 보낸다면 HTTP 응답으로 crendential 요청을 할 필요가 없지만 클라이언트가 인증되지 않은 상태로 자원을 요청한다면 접근을 인가할 수 없다. 이 경우에 `AuthenticationEntryPoint`를 구현함으로 클라이언트에게 credential을 요청하는 것이다. `AuthenticationEntryPoint`에 page를 redirect하거나 WWW-Authenticate를 응답 헤더에 포함하는 방법 등으로 클라이언트에게 요청을 한다.

## AbstractAuthenticationProcessingFilter

유저의 credential을 인증하기 위한 기반 필터이다.

<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/117768742-c1f26600-b26d-11eb-8aaf-ac428a95d786.png">
<p style="font-weight:bold" align="center">ProviderManager 동작</p>
</p>

1. 유저가 credential을 보냈을 때 `AbstractAuthenticationProcessingFilter`가 인증 될 `HttpServletRequest`에 `Authentication`을 만든다. `Authentication`의 타입은 `AbstractAuthenticationProcessingFilter`의 subclass의 의존해서 만들어진다.

   예를 들어, `UsernamePasswordAuthenticationFilter`는 `HttpServletRequest`에 제출된 유저이름과 패스워드로  `UsernamePaswordAuthenticationToken`을 만든다.

2. `AuthenticationManager`에게 인증을 위해 `Authentication`이 전달된다.

3. 만약 인증이 실패라면 인증 실패 처리가 된다.

   - `SecurityContextHolder`는 비워진다.
   - `RememberMeService.loginFail`이 실행된다. Remember me가 설정되지 않았다면 아무것도 하지 않는다.
   - `AuthneticationFailureHandler`가 실행된다.

4. 만약 인증이 성공이라면 인증 성공 처리가 된다.

   - `SessionAuthenticationStrategy` 에 새 로그인이 알려진다.
   - `Authentication`이 `SecurityContextHolder`에 설정되고 이후에 `SecurityContextPersistencFilter`가 `SecurityContext`를 `HttpSession`에 저장한다.
   - `RememberMeService.loginSuccess`가 실행된다. Remember me가 설정되지 않았다면 아무것도 하지 않는다.
   - `ApplicationEventPublisher`는 `InteractiveAuthenticationSuccessEvent`를 발행한다.
   - `AuthenticationSuccessHandler`가 실행된다.

## Reference

[https://docs.spring.io/spring-security/site/docs/current/reference/html5/#servlet-authentication-authentication](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#servlet-authentication-authentication)

