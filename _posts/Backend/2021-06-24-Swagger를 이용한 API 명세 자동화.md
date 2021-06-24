---
bg: "BACKEND.png"
layout: post
title:  "Swagger를 이용한 API 명세 자동화"
crawlertitle: "Swagger를 이용한 API 명세 자동화"
summary: "Swagger를 이용한 API 명세 자동화"
date: 2021-06-24 21:42:00 +0900
category: Backend
author: Err0rCode7
---

#### Swagger Annotation을 사용한 Api docs
##### Swagger annotation을 직접 달아보고 사용해보자.

혼자 개발이 아닌 여러명이서 개발을 진행할 때 꼭 필요한 것은 Api 명세이다. 예를 들어 프론트 엔드팀이 Api를 사용할 때, 모바일 개발팀이 Api를 사용할 때를 생각해볼 수 있다. Api 서버에서 어떤 Spec을 갖으며 어떤 데이터를 내려주는 지 필수적으로 공유해야한다.

하지만 이 작업은 매우 매우 귀찮은 작업이다. 이런 번거로운 작업을 해결하기 위한 방법으로 Swagger를 이용해서 Api 문서 자동화를 해보자.

## Swagger

Swagger는 간단한 설정으로 지정한 URL들을 HTML화면으로 확인할 수 있게 해주는 프로젝트이다.

과거 했던 프로젝트에 적용을 해보았고 [Github](https://github.com/Err0rCode7/openstudio-back)에서 볼 수 있다.

간단한 설명파일 생성과 Swagger Annotation을 이용하면 쉽게 등록할 수 있다. 우선 의존성부터 받아보자.

```groovy
// build.gradle

implementation group: 'io.springfox', name: 'springfox-swagger-ui', version: '2.9.2'
implementation group: 'io.springfox', name: 'springfox-swagger2', version: '2.9.2'
```

versiond은 3.0.0 까지 나왔는데 최신 버전에서 not found 오류가 발생하여 2.9.2 버전의 의존성을 사용했다.

이제 Swagger Annotation 설정 빈을 등록해야한다.

```java
@EnableSwagger2
@Configuration
public class Swagger2Config  {

    @Bean
    public Docket apiV1() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.codetogether.openstudio.controller.apis.v1"))
                .paths(PathSelectors.ant("/api/v1/**"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("OpenStudio API")
                .version("v1")
                .description("OpenStudio Api 문서입니다.")
                .license("jeon4708@gmail.com")
                .licenseUrl("https://github.com/err0rCode7/")
                .build();
    }
}
```

`@Configuration`을 통해 설정 빈을 등록해야하고 `@EnableSwagger2`를 통해 스웨거를 활성화한다.

`Docket`은 하나의 Document로 api 기본정보, api 패키지 위치, URL 경로 등을 등록할 수 있다.

Api v1을 기준으로 하나의 Docket을 빈으로 등록하여 자동으로 api 명세를 만들도록 했다.

읽으면 어떤 내용인지 바로 알 수 있을 정도로 되어있기 때문에 다른 자세한 설명은 생략한다.

## 스프링 시큐리티와 같이 쓸 때 주의 ‼️

Spring Security와 Swagger를 같이 적용할 때 주의해야할 것이 있는데 static 파일들의 요청을 시큐리티 필터에서 막아버리면 안된다.

이때 [두 가지 방법](https://ravthiru.medium.com/springboot-security-configuration-using-httpsecurity-vs-websecurity-1a7ec6a23273)을 생각해보았는데,

1.  `HttpSecurity`에서 `antMatcher(AUTH_WHITELIST)` 를 이용해서 인가의 대상을 회피시키는 방법
2. `WebSecurity`에서 `ignoring().antMatchers(AUTH_WHITELIST)`를 통해 필터 자체를 거치지 않게 하는 방법이 있다.

필터 자체를 거치지 않아 비용을 줄이는 2번 방법을 추천한다. 아래와 같이 Swagger static 파일들을 요청시 시큐리티 필터를 무시할 수 있도록 해주면된다.

```java
// SpringConfig 클래스 내부

...

private static final String[] AUTH_WHITELIST = {
  // -- Static resources
  "/css/**",
  "/images/**",
  "/js/**",
  // -- Swagger UI v2
  "/v2/api-docs",
  "/configuration/ui",
  "/swagger-resources/**",
  "/configuration/security",
  "/swagger-ui.html",
  "/webjars/**",
  // -- Swagger UI v3 (Open API)
  "/v3/api-docs/**",
  "/swagger-ui/**"
};

@Override
public void configure(final WebSecurity webSecurity) {
  webSecurity.ignoring().antMatchers(AUTH_WHITELIST);
}

...
```

## 애노테이션을 달아보자

위의 내용과 Swagger 애노테이션을 적용하고 `host:port/swagger-ui.html`를 확인하면 다음처럼 볼 수 있다.
(`host:port/v2/api-docs` 를 통해 Json으로도 볼 수 있다.)

<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/123259868-ebbad000-d52f-11eb-82a8-c6a370968736.png">
<p style="font-weight:bold" align="center">swagger-ui.html</p>
</p>

애노테이션을 적용한 Api Controller 일부를 가져와 설명을 진행하겠다.

```java
@Api(tags = {"Member API"})
@RequiredArgsConstructor
@RequestMapping("/api/v1/members/")
@RestController
public class MembersController {

    private final MemberService memberService;

    @ApiOperation(value = "자신의 정보 확인", notes = "세션 값을 통해 자신의 정보를 확인하는 API입니다.")
    @GetMapping("me")
    public MemberResponseDto getMember(
            @ApiIgnore
            @LoginUser SessionUser user) {
        if (user == null) {
            throw new NoSuchSessionUserException("존재하지 않는 세션의 유저입니다.");
        }
        return memberService.findByEmail(user.getEmail());
    }
	...
}
```

여기서 Swagger 애노테이션을 뽑아 설명을 진행하면 다음과 같다.

- `@Api(tags = {"Member API"})`

  Api의 태그를 지정해주는 것이다. `swagger-ui.html` 사진을 보면 바로 이해가 될 것이다. (여러개의 태그를 지정할 수 있다.)

<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/123263734-422a0d80-d534-11eb-875b-2c782fb33d75.png">
<p style="font-weight:bold" align="center">ApiOperation</p>
</p>

- `@ApiOperation(value = "자신의 정보 확인", notes = "세션 값을 통해 자신의 정보를 확인하는 API입니다.")`

  `ApiOperation`은 하나의 Api에 대해 정보를 정의하는 애노테이션이다.

- `@ApiIgnore`

  `ApiIgnore`은 매개변수로 들어간 변수를 무시해주는 애노테이션으로 위에 코드에서는 `SessionUser`에 대한 정보를 빼기위해 사용한다.

이렇게 Swagger 애노테이션을 아주 간단히 Api 명세를 만들어볼 수 있으며 Api 직접 실행해볼 수 도 있다.

<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/123264011-8fa67a80-d534-11eb-97f4-f1f6633700b3.png">
<p style="font-weight:bold" align="center">Api 직접 실행 해보기</p>
</p>

또한 매개변수로 들어간 Dto(모델)들의 형태도 보여준다.

<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/123264150-b49aed80-d534-11eb-81cb-2ed936e2715d.png">
<p style="font-weight:bold" align="center">Models</p>
</p>


유지보수 측면에서 다른 트레이드 오프가 있을 수 있지만 이렇게 간단히 Api 명세를 만들어 볼 수 있다는 건 큰 장점인 것 같다.

앞으로 프로젝트에서 적극 이용해볼 생각이고 여러분에게도 추천한다.