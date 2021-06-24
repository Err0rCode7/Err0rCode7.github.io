---
bg: "BACKEND.png"
layout: post
title:  "Docker Compose를 이용한 개발 환경구성과 예제"
crawlertitle: "Docker Compose를 이용한 개발 환경구성과 예제"
summary: "Docker Compose를 이용한 개발 환경구성과 예제"
date: 2021-06-24 18:26:00 +0900
category: Backend
author: Err0rCode7
---

#### Docker Compose와 Spring, Mysql
##### 개발 및 배포 환경을 구성하기 위해 Docker Compose를 이용해보았다.

Docker Compose를 구성해서 nginx와 react js를 구성한 영상에서 아주 쉽게 개발환경을 구성하는 것을 보게되었고 내 개발환경도 Nginx, Spring, mysql 환경을 구성해보기로 결정했고 예제를 만들어보았다.

(생각보다 쉽게 되지는 않았다. 하하..😅 M1💻🔨)

## Docker Compose

Docker의 편리함은 다들 익히알고 있을 것이다.

독립적으로 VM 같은 가상머신을 이용하지 않고도 개발 또는 배포 환경을 구성할 수 있는 아주 엄청난 컨테이너 구성이다.

하지만 Nginx, Spring, mysql 구성과 같이 여러 컨테이너를 띄워야할때는 번거로움이 존재한다. 이를 해결하기 위한것이 Docker-compose이다.

> 즉, docker-compose는 여러개의 컨테이너를 정의하고 띄우기 위한 툴이다.

Docker를 이용할 때처럼 `Dockerfile`을 정의하고 각각의 의존성에 맞게 `docker-compose` 파일을 구성해주면 끝이다. 엄청 간단하다.

## Docker-Compose Example

[Example 코드](https://github.com/err0rCode7/docker-compose-example)에서 예제 전체내용을 확인해볼 수 있다. 

**이번 포스팅 이후에 여러개의 서비스(Web app)을 추가할 예정이니 커밋 내용(`last commit: 1687f706f50464f169133d53394f136610ef0754`)을 잘 확인하길 바란다.**

<p align="center">
<img src="https://user-images.githubusercontent.com/48249549/123220823-47259780-d509-11eb-84e2-777367c9ff37.png">
<p style="font-weight:bold" align="center">컨테이너 구성</p>
</p>


시작하기 전에 구성을 먼저 살펴보자. 하고자하는 예제는 다음과 같은 구성이다.

예제로 Spring_app에서는 Mysql DB를 사용하여 Api를 구성하고 Nginx는 ssl을 이용하여 리버스 프록시로 Spring_app을 연결하는 구성을 하고자한다.

따라서 Nginx는 Spring_app에 의존하고 Spring_app은 Mysql에 의존한다. 이를 생각하며 `docker-compose`를 구성해보자.

### 사전 준비물

- Docker version 20.10.7
- docker-compose version 1.29.2
- Jdk 11 (커스텀을 하고 싶다면 ?!)

## 간단한 Spring App 만들기

`Spring Boot, Spring Data Jpa, lombok annotation, thymeleaf`를 이용해서 유저의 내용을 저장해볼 수 있는 아주아주 간단한 구성을 만들어보았다. 코드는 아래와 같고 자바는 11버전을 사용했다.

```java
// Member.java
@NoArgsConstructor
@Getter
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String name;

    public Member(String name) {
        this.name = name;
    }
}
```

```java
// MemberController.java
@RequestMapping("/members")
@RequiredArgsConstructor
@RestController
public class MemberController {

    private final MemberService memberService;

    @GetMapping("/")
    public List<ResponseMemberDto> getMembers() {
        return memberService.findAll();
    }

    @PostMapping(value = "/", consumes = MediaType.APPLICATION_FORM_URLENCODED_VALUE)
    public String saveMemberForFormRequest(@ModelAttribute RequestMemberDto requestMemberDto) {
        memberService.save(requestMemberDto);
        return "Ok";
    }

    @PostMapping("/")
    public String saveMember(@RequestBody RequestMemberDto requestMemberDto) {
        memberService.save(requestMemberDto);
        return "Ok";
    }
}
```

```java
// MemberRepository.java
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

```java
// MemberService.java
@RequiredArgsConstructor
@Service
public class MemberService {

    private final MemberRepository memberRepository;

    @Transactional(readOnly = true)
    public List<ResponseMemberDto> findAll() {
        return memberRepository.findAll().stream()
                .map(ResponseMemberDto::new)
                .collect(Collectors.toList());
    }

    @Transactional()
    public void save(RequestMemberDto requestMemberDto) {
        memberRepository.save(requestMemberDto.toEntity());
    }
}
```

```java
// RequestMemberDto.java
@NoArgsConstructor
@Getter
@Setter
public class RequestMemberDto {
    private String name;

    public RequestMemberDto(String name) {
        this.name = name;
    }

    public Member toEntity() {
        return new Member(this.name);
    }
}

// ResponseMemberDto.java
@Getter
public class ResponseMemberDto {

    private Long id;
    private String name;

    public ResponseMemberDto(Long id, String name) {
        this.id = id;
        this.name = name;
    }

    public ResponseMemberDto(Member member){
        this.id = member.getId();
        this.name = member.getName();
    }
}
```

`host:port/members/` URL로 Get 요청을하면 모든 멤버를 볼 수 있고 Post 요청으로는 멤버를 저장할 수 있는 구성이다.

또한, `host:port/`에 접근하면 멤버를 저장하는 폼을 갖고있는 html을 볼 수 있다.

유심히 봐야할 부분은 아래의 `application.yml` 파일이다.

```yml
spring:
  profiles:
    active: prod

---

spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: jdbc:mysql://mysql_db:3306/myapp
    username: test
    password: test
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: update
    database-platform: org.hibernate.dialect.MySQL5Dialect
    generate-ddl: true
    show-sql: true

---
spring:
  config:
    activate:
      on-profile: local
  datasource:
    url: jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
    username: sa
    password:
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: update
#    database-platform: org.hibernate.dialect.MySQL5Dialect
    generate-ddl: true
    show-sql: true
```

`profile` 구성은 굳이 하지않아도 되는데 mysql을 사용하려면 `spring.profiles.active`가 prod로 되어있어야한다. 유심히 봐야해야할 곳은 `datasource`에서 `mysql URI` 구성과 username, password이다.

`jdbc:mysql://mysql_db:3306/myapp` 이부분에서 `mysql_db` 부분은 `docker-compose.yml`에 구성한 container 이름을 적어야한다. 또한 뒤에 `myapp`은 사용할 DB 명이된다.

또한 mysql을 test라는 유저로 접속하고자 하기 때문에 구성할 때 test라는 유저를 만들어줘야하며 myapp이라는 DB를 구성해야한다.

위에 구성과 함께 Dockerfile을 다음과 같이 구성해주었다.

```dockerfile
// myapp.dockerfile
// jdk 11 이미지를 사용한다.
FROM adoptopenjdk/openjdk11

// 현재 디렉토리에 있는 파일들을 컨테이너 내부 환경 spring-app 디렉토리로 복사한다.
ADD . /spring-app
// 컨테이너 내부에서 "cd spring-app"를 한 것과 같다.
WORKDIR /spring-app
```

이로써 Jdk11 이미지를 사용하여 구성된 컨테이너에 작성한 Spring app이 복사된 환경이 구성된다.

## Mysql 구성

mysql은 8.0.22 버전을 사용했다.

mysql의 구성은 간단하다. mysql 이미지를 받아서 그 위에서 spring-app에서 사용할 DDL(DB 정의),  schema를 정의해주면 된다.

```dockerfile
// mysql.dockerfile

// platform은 m1 칩셋 환경에서 x86 환경으로 이미지를 다운받기위해 사용했다.
// 지우게되면 빌드하는 과정에서 메타데이터를 불러오지 못하는 문제가 있다.
FROM --platform=linux/x86_64 mysql:8.0.22

// 필요한 DDL, Schema 정의 파일을 복사한다.
ADD ./mysql-init-files /docker-entrypoint-initdb.d
```

 `docker-entrypoint-initdb.d` 디렉토리에 초기 실행이 필요한 sql을 복사를 해두면 mysql 컨테이너가 띄워지면서 sql문을 실행해준다.

따라서 mysql-init-files 디렉토리에 sql을 정의해두었다.

(mysql 8 버전에서는 utf-8을 4바이트로 사용하는 utf8mb4을 설정할 수 있으며 collation으로 utf8mb4 값을 어떻게 비교할지 설정할 수 있다. 여기 예제에서는 하지 않았다.)

```sql
// mysql/mysql-init-files/init.sql

CREATE DATABASE IF NOT EXISTS `myapp`;
use `myapp`;

DROP TABLE IF EXISTS `member`;

CREATE TABLE IF NOT EXISTS `member`(
		member_id bigint not null,
		name varchar(255),
		primary key (member_id)
);
```

mysql에 User를 만드는 부분은 Docker-compose에서 환경변수로 컨테이너에 전달하여 만들도록하겠다.

또한 docker 컨테이너 환경은 컨테이너가 종료되면 제거가 되기때문에 디스크에 볼륨을 맵핑하여 DB내용을 저장해야한다. 이 또한 Docker-compose에서 진행하도록하겠다.

## Nginx 구성

우리가 구성할 Nginx는 ssl을 사용해야하기 때문에 mysql 처럼 ssl 키와 인증서를 만드는 초기화가 필요하다.

mysql에서 `docker-entrypoint-initdb.d` 디렉토리에 넣어준것 과 달리 초기화 쉘 스크립트를 작성해서 이미지를 빌드하는 과정에 실행할 수 있도록 구성을 했다.

```dockerfile
// nginx 이미지를 사용
FROM nginx

// init 쉘 스크립트를 복사
ADD ./nginx-init-files/init.sh .

// 쉘스크립트 권한 부여 및 실행
RUN chmod 777 init.sh
RUN sh init.sh

// background로 nginx 실행 -> 실행 후 꺼진다. 아래에서 내용을 얘기하겠다.
# CMD [ "service", "nginx", "start" ]
// nginx를 foreground로 실행
CMD [ "nginx", "-g", "daemon off;" ]

```

nginx를 background로 실행하게되면 nginx가 실행되고 프로세스가 종료되어 컨테이너가 내려가는 문제가 있었다. 이를 해결하기위해 docker-compose 구성에서 `tty: true`를 사용하거나 위에 처럼 foreground로 nginx를 실행하여 잡아두어야한다.

Nginx에서 ssl환경과 리버스 프록시를 구겅하기위해 nginx 설정을 손을 봐야하는데, docker-compose에서 볼륨을 이용하여 구성했다. 따라서 dockerfile을 보고서는 알 수 없는데, 간단히 설정파일만 보고 뒤에서 다시 mysql 볼륨 내용과 같이 다루겠다.

```
# nginx/conf.d/defalt.conf

# 사용할 server zone으로 domain 또는 ip를 나열해준다.
upstream spring_app_server {
	# host를 docker name으로 해줘야함.
	server spring_app:8080;
}

server {
	listen		80 default_server;
	listen		[::]:80 default_server;

	return 301 https://$host$request_uri;
}

server {
	listen		443 ssl;

	ssl_certificate	/etc/ssl/certs/localhost.dev.crt;
	ssl_certificate_key	/etc/ssl/private/localhost.dev.key;

	location / {
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header Host $host;
		proxy_set_header X-NginX-Proxy true;
		proxy_pass http://spring_app_server;
	}
}

```

간단히 보면 80 포트와 443 포트를 기다리고 있도록 구성했다. 또한 리버스 프록시에 사용할 내용으로  spring_app을 가리키는 server zone을 만들었다. 이때, 꼭 docker 컨테이너 명으로 해주어야한다.

80 포트를 기본으로 listen하고 있는 구성에서는 80포트를 통해 들어오면 301 상태 코드를 보내어 https로 리다이렉션 되도록했다.

Https(443 port)를 기본으로 listen하고 있는 구성에서는 ssl의 인증서와 키를 셋팅해주고 들어오는 요청을 리버스 프록시로 spring_app 컨테이너로 던져준다.

## docker-compose.yml 구성

개별적인 dockerfile을 다 살펴보았고 `docker-compose.yml` 파일을 살펴보자

docker-compose 구성은 yml(야믈 == yaml) 파일로 구성해야하고 공백으로 space(' ')를 사용해야한다. (탭은 사용이 안된다.)

docker-compose는 아래와 같이 실행, 제거를 할 수 있다.

```sh
# docker-compose 실행, 현재 디렉토리에 docker-compose.yml이 있어야한다.
docker-compose up
# compose된 컨테이너를 중지한다.
docker-compose stop
# compose된 컨테이너를 중지 및 삭제한다.
docker-compose down
```

`docker-compose.yml` 전체 내용은 아래와 같이 구성했다.

```yaml
version: '3'

services:
  nginx:
    build:
      context: ./nginx
      dockerfile: ./nginx.dockerfile
    volumes:
      - "./nginx/conf.d:/etc/nginx/conf.d"
      - "./nginx/log/:/var/log/nginx/"
    ports: 
      - 80:80
      - 443:443
    restart: always
    networks: 
      - spring_app_net
    depends_on: 
      - spring_app
    # command: nginx -g daemon off;

  mysql_db:
    build:
      context: ./mysql
      dockerfile: ./mysql.dockerfile
    container_name: mysql_db
    environment:
      MYSQL_ALLOW_EMPTY_PASSWORD: "yes"
      MYSQL_DATABASE: myapp
      MYSQL_USER: test
      MYSQL_PASSWORD: test
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - "./mysql/myapp/:/var/lib/mysql"
    ports:
      - "3306:3306"
    networks:
      - spring_app_net
    cap_add:
      - SYS_NICE

  spring_app:
    build:
      context: ./myapp
      dockerfile: ./myapp.dockerfile
    container_name: spring_app
    expose: 
      - 8080
    # ports:
    #   - 8080:8080
    depends_on:
      - mysql_db
    # restart: on-failure
    restart: always
    networks: 
      - spring_app_net
    command: ./gradlew bootRun

networks: 
    spring_app_net:
```

- `version`: docker-compose 구성의 버전을 적어준다.
- `services`: 사용하고자 하는 컨테이너 구성을 나열하는 곳이다. 위와같이 각각을 계층적으로 나열하면 된다.
- `networks`: docker에서는 컨테이너별로 개별적 네트워크 구성을 할 수 있다. 또한 같이 사용할 수 있는데 이 networks를 정의하지 않으면 default로 정의가 되어 나중에 다른 컨테이너를 네트워크 구성에 넣을 수 없다.

이제 각각을 살펴보자.

### nginx service

```yaml
  nginx:
    build:
      context: ./nginx
      dockerfile: ./nginx.dockerfile
    volumes:
      - "./nginx/conf.d:/etc/nginx/conf.d"
      - "./nginx/log/:/var/log/nginx/"
    ports: 
      - 80:80
      - 443:443
    restart: always
    networks: 
      - spring_app_net
    depends_on: 
      - spring_app
    # command: nginx -g daemon off;
```

- `services` 아래에는 사용하고자 하는 서비스 이름(`nginx:`)을 적는데 이걸 그대로 사용하면 컨테이너 이름으로 `docker__` 라는 prefix와 뒤에 숫자가 postfix로 붙는다. 서비스 내부에`container_name:`을 이용하여 컨테이너 명을 지정할 수 있다.

- `build:` 이미지로 만들 `dockerfile`을 정의하는 곳인데, 위에 내용은 nginx 디렉토리에 nginx.dockerfile을 빌드해서 이미지로 사용할 것이다. 라는 내용이다

- `volumes:` `:`를 기준으로 좌측은 현재 파일시스템, 우측은 컨테이너 내부의 내용이된다.
  현재 파일 시스템의 볼륨을 컨테이너 내부에서 바라보게(맵핑) 하는 방법이다. 이를 이용해 nginx 컨테이너가 설정파일을 컨테이너 외부 파일시스템을 바라보게했다. `- "내용"` 같은 형태로 여러개 나열이 가능하다.
- `ports:` 현재시스템포트:컨테이너포트 방식으로 적는다. 포트를 열어준다. 이 또한 여러개 나열이 가능하다.
- `restart:` nginx가 종료되었을 때 행해지는 내용으로 `always` 이면 항상 재실행한다는 것이다. `on-failure`로도 사용이 가능한데 컨테이너가 오류를 반환하면서 꺼지면 재실행한다는 내용이다.
- `networks:` 컨테이너 네트워크 구성을 정의하는 곳이다.
- `depeonds_on:` 서비스의 의존성을 정의하는 것으로 어떤 이미지의 컨테이너를 먼저 띄울지 결정한다.
- `command:` 컨테이너 띄워지고 실행할 명령을 정의한다.

### mysql_db service

```yaml
  mysql_db:
    build:
      context: ./mysql
      dockerfile: ./mysql.dockerfile
    container_name: mysql_db
    environment:
    #  MYSQL_ALLOW_EMPTY_PASSWORD: "yes"
      MYSQL_DATABASE: myapp
      MYSQL_USER: test
      MYSQL_PASSWORD: test
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - "./mysql/myapp/:/var/lib/mysql"
    ports:
      - "3306:3306"
    networks:
      - spring_app_net
    cap_add:
      - SYS_NICE
```

(겹치는 것은 생략해서 설명을 진행)

- `container_name:` 컨테이너 명을 정의한다. 이 내용이 spring의 DB URI 설정 값이 반영되고 `nginx upstream`에 반영 되는 것을 위에서 얘기했었다.

- `environment:` 환경 변수 값을 지정하는 것인데, docker를 이용해서 mysql 환경을 구성하면 환경변수로 값을 셋팅할 수 있다.

  `MYSQL_ALLOW_EMPTY_PASSWORD` 루트 비밀번호을 빈 값으로 허용하는지에 대한 여부

  `MYSQL_DATABASE` 컨테이너의 이미지를 시작 하면서 지정한 DB를 생성

  `MYSQL_USER, MYSQL` mysql 사용자 아이디와 비밀번호 지정

  `MYSQL_ROOT_PASSWORD` root 비밀번호 지정

- `volumes:` DB 컨테이너를 구성할 때 중요한 부분이다. 위에서 말했듯, 파일 시스템의 볼륨을 컨테이너안에서 사용할 수 있도록 해주는 것이다. DB의 내용은 사라지면 안되므로 파일시스템에 저장해야한다.

- `cap_add: capability를 등록하는 것이다 `SYS_NICE` 컨테이너가 프로세스의 nice 값을 올릴 수 있도록 허용하는 것인데, 프로세스의 스케줄링 우선순위를 변경하는 것을 가능하도록한다.

### spring_app service

```yaml
  spring_app:
    build:
      context: ./myapp
      dockerfile: ./myapp.dockerfile
    container_name: spring_app
    expose: 
      - 8080
    # ports:
    #   - 8080:8080
    depends_on:
      - mysql_db
    # restart: on-failure
    restart: always
    networks: 
      - spring_app_net
    command: ./gradlew bootRun
```

위에서 거의 다 설명이 되었기 때문에 하나만 expose만 설명하겠다.

- `expose:` `ports:` 같은 경우 컨테이너 외부와 내부를 이어주는 포트포워딩이라고 할 수 있는데, expose 같은 경우 컨테이너 간의 포트를 열어주는 것을 얘기한다. 따라서 nginx가 8080 포트로 spring_app에 접근할 수 있게 되는 것이다. 하지만 컨테이너 외부에서 브라우저로 `localhost:8080` 접근하는 것은 불가능하다. 따라서 nginx의 리버스 프록시로만 spring_app에 접근할 수 있게 된다.

## 의존성의 또 다른 문제

전체적으로 `depends_on`을 살펴보면 `nginx -> spring_app -> mysql_db` 로 의존되는 것을 볼 수 있다. `docker-compose`는 이 의존성에 맞게 컨테이너를 실행한다.

 **`mysql_db` 컨테이너를 실행하고 `spring_app` 컨테이너가 실행되지만 `spring_app` 컨테이너가 먼저 띄워지게 될 수도 있다 이로인해 `spring_app`의 jdbc는 db에 연결할 수 없어 애플리케이션이 종료되게 될 텐데, 이를 해결해야한다.**

예제 코드에서는 위와 같은 문제가 발생하면  `restart`덕분에 애플리케이션이 애플리케이션을 재실행하게 돼서 문제는 해결될 것이다. 또는 `spring_app` 컨테이너가 더 늦게 띄워져서 문제가 없을 것이다.

하지만 이를 쉘 스크립트로 tcp 연결을 해보는 방식으로 근본적으로 문제를 막는 [wait-for-it](https://github.com/vishnubob/wait-for-it) 이나 [curl을 이용한 방법](https://keepgrowing.in/tools/how-to-make-one-docker-container-wait-for-another/), nc 명령어를 이용해보는 방법을 생각해보것을 추천한다.

[나의 과거 wait-for-it을 사용한 예제 링크](https://github.com/Err0rCode7/CapsuleTime-Docker)

마지막으로 [docker-compose의 다양한 예제가 있는 docker 깃허브](https://github.com/docker/awesome-compose)를 추천하면서 내용을 마치겠다.

## References

[https://github.com/docker/awesome-compose] (https://github.com/docker/awesome-compose)

[wait-for-it](https://github.com/vishnubob/wait-for-it)

[나의 과거 wait-for-it을 사용한 예제 링크](https://github.com/Err0rCode7/CapsuleTime-Docker)

[curl을 이용한 wait](https://keepgrowing.in/tools/how-to-make-one-docker-container-wait-for-another/),