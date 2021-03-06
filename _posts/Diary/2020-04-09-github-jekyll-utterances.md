---
bg: "Diary.jpg"
layout: post
title:  "Github 블로그를 만들다. Jekyll 그리고 utterances"
crawlertitle: "Github.io 만들기"
summary: "github 블로그 만들기"
date: 2020-04-09 21:06:07 +0900
category: Diary
author: Err0rCode7
---

드디어 첫 포스팅입니다! <br>
블로그를 만들어야지... 만들어야지... 하다가.. 이제서야 드디어 만들게 되었네요~<Br>
얼마 안걸리겠지.. 생각했는데 이상한 곳 진행이 안되어 시간이 꽤나 걸리더군요..<br>

여튼 ! 이번 포스팅에서는 윈도우 10 기반에서 Github 블로그를 만든 내용과 어떻게 진행 했는지에 대해서 포스팅하려고 합니다~<Br>
어떤 순서로 얘기를 진행할지 목차를 알려드리겠습니다.<br>

1. 필요한 기본 설치
2. jekyll 테마 적용
3. jekyll은 어떤식으로 동작하는가 ?
4. 댓글 기능 추가하기, utterances

위과 같은 진행해보도록 하겠습니다~ 그러면 시작하죠!<br>
<br>
<br>

---
#### 1. 필요한 기본 설치
---
간략하게 어떤 것들이 필요하고 준비해야 하는지에 대해서만 다루도록 하겠습니다.

저희는 Github를 이용하여 블로그를 만들어야하기 때문에 반드시! git이 설치되어 있어야 하고 github에 가입이 되어있어야 합니다.<br><br>
git이 어떤거인지 모르시면 빨리 Git부터 알아보구 옵시다 ! -> [Git search](https://www.google.com/search?sxsrf=ALeKk008PpNTvvJP2UZPG6bhpodN3wINuw%3A1586435098389&ei=GhSPXsS5F8HZhwOZ7ZHwBw&q=git&oq=git&gs_lcp=CgZwc3ktYWIQAzIECCMQJzIECCMQJzIECAAQQzIHCAAQFBCHAjIECAAQQzIECAAQQzICCAAyAggAMgQIABBDMgQIABBDOgQIABAKShEIFxINMy0xMzRnMTIzZzMyNkoLCBgSBzMtMWcxZzJQlrMDWJ60A2CatgNoAHAAeACAAcYCiAHwBpIBBzAuMi4wLjKYAQCgAQGqAQdnd3Mtd2l6&sclient=psy-ab&ved=0ahUKEwjErv6jq9voAhXB7GEKHZl2BH4Q4dUDCAw&uact=5)<br>

이제 github에 들어가서 블로그를 넣을 새로운 repository를 만들어 보도록 합시다.
![new repo1](https://user-images.githubusercontent.com/48249549/78896055-d5e7f900-7aaa-11ea-92b7-450ff0ab4b45.png)
![new repo2](https://user-images.githubusercontent.com/48249549/78896681-ccab5c00-7aab-11ea-9eed-e361d823f618.png)

위에 사진처럼 계정네임.github.io 라는 이름으로 새로운 repository를 만들어주시면 됩니다.
만들어 주신후 계정네임.github.io 으로 사이트를 접속해보시면 Readme.md에 대한 내용이 나올겁니다.

Jekyll을 사용하기 위해서는 Ruby가 필요합니다. <br>
[Ruby 설치](https://rubyinstaller.org/downloads/)

![Ruby](https://user-images.githubusercontent.com/48249549/78895512-f6638380-7aa9-11ea-80e1-ded999eda6b7.png)

저는 위 사진에서 화살표 표시가 되어있는 Ruby+Devkit 2.6.6-1 (x64) 버전을 다운받았습니다.

이제 jekyll과 bundler을 다운받아봅시다.
![ruby command](https://user-images.githubusercontent.com/48249549/78897166-b81b9380-7aac-11ea-8ff1-0efb06276460.png){: width="250"}

위에 사진 처럼 ruby command를 실행 시킨후 아래 내용을 입력하여 다운을 받습니다.

```
gem install jekyll bundler
```

이제 미리 적용할 jekyll 테마를 찾아봅시다.<br>
[jekyll 테마](http://jekyllthemes.org/)

구글에 찾아보시면 이 사이트 말고도 다른 사이트들도 있으니 참고하세요. <br>
( 테마에 따라 설정방법이 다를 수 있습니다. )

이렇게 github repo와 jekyll, bundler, jekyll 테마를 준비해주시면 됩니다.
<br>
<br>

---
#### 2. jekyll 테마 적용
---
이제 jekyll 테마를 적용해봅시다.


자신이 관리하기 원하는 디렉토리(폴더)에서 커맨드에 아래 내용을 입력하여 github.io의 로컬 저장소를 만들어줍니다.

![git clone](https://user-images.githubusercontent.com/48249549/78897923-e9e12a00-7aad-11ea-8dbc-ff14f60e47bb.png)


```
git clone "위 사진에서처럼 확인할 수 있는 clone 주소"
cd "설치된 폴더"
```

이제 해당 폴더에서 다운받았던 테마를 압축을 해제하여 넣어줍니다.
그 이후 다음과 같이 커맨드창에 입력해줍니다.

```
bundle install
```

설치가 완료가 되면 계속해서 아래와 같이 입력해줍니다.

```
bundle exec jekyll serve
```

완료 후에 http://127.0.0.1:4000/ 에 들어가보시면 테마를 적용한 홈페이지를 확인해볼 수 있습니다.<br>
이제 Github에도 적용을 해보도록 하죠

커맨드창을 여시고 테마가 있는 디렉터리로 이동하시고 아래와 같이 따라 입력해주세요. <br>

( github에 커밋 푸시하는 작업입니다. 만약 로컬저장소가 github email과 연결이 안되어있다면 이 작업부터 해주세요)

```
git add .
git commit -m "Jekyll 테마 적용"
git push
```

이제 계정네임.github.io 사이트에 접속하셔서 테마가 적용이 된걸 확인하실 수 있습니다.
<br>
<br>

---
### 3. jekyll은 어떤식으로 동작하는가 ?
---
이제 "jekyll 어떤식으로 동작하는가 ?" 에 대해서 다뤄보겠습니다.

#### 3.1 index.html

자신의 블로그 링크에 들어가보면 메인 페이지를 볼 수 있습니다.<br>
이 메인페이지가 보여지는 파일이 바로 index.html 입니다.

index.html에 있는 객체와 내용으로 메인 페이지가 나오게 됩니다.
이 내용은 바로 뒤에 내용에서 같이 이해해보도록 합시다.

#### 3.2 _posts

![_post](https://user-images.githubusercontent.com/48249549/79038446-465e5980-7c14-11ea-8dec-a440d46e0223.png)

본인의 github.io repository를 보면 위에 사진처럼 _posts 폴더를 확인해 볼 수 있습니다.

jekyll은 이 폴더에 yyyy-mm-dd-title.md 형식으로 마크다운을 입력하면 자동적으로 게시가되는 방식입니다. 즉, 이 폴더에서 마크다운을 작성하거나 삭제함으로써 게시글을 올리거나 내릴 수 있습니다.<br>

그렇다면 이 마크다운 파일은 어떤식으로 작성해야 되는걸까요 ? 마크다운 파일 내부를 살펴보도록 하죠

![one of posts](https://user-images.githubusercontent.com/48249549/79038630-c2a56c80-7c15-11ea-8b3d-d1e72ae92365.png)

예시 마크다운 파일을 가져와봤습니다.<br>

마크다운에 상단에 보면 테두리로 여러 객체가 묶여있는 것을 볼 수 있습니다.<br>
이 객체들이 html파일( 테마 )에 적용되어 게시글을 표현한다고 보시면 됩니다. <br>

bg는 배경사진, layout은 말 그대로 마크다운 파일이 어떤 레이아웃으로 포스팅이 될지 정해주는 것입니다.<br>

html 형태의 layout이 있고 틀안에 마크다운에 적은 게시글 내용이 쏘옥 들어간다고 보시면 됩니다. <br>
( 실제로 post.html에 `content` 객체에 들어갑니다. )

layout 파일들은 _layouts 폴더 안에 html로 있으며 이것을 수정하여 자신의 블로그 테마를 수정하거나 여러가지 페이지로 꾸밀 수 있습니다.<br>
( 블로그를 꾸밀 때 본인 테마의 layout 구조가 어떻게 되어 있는지 알고 있어야합니다. html이 어떤식으로 계층화 되어있는지 살펴보는 것을 추천드립니다. )<br>

title부터 date 까지는 게시글에 대한 내용들 입니다.<br>

categories는 블로그 게시자가 항목 별로 나누어서 post를 관리하거나 게시할 수 있는 용도로 사용됩니다.<br>
예를 들어 저의 블로그에서 Project와 Diary를 나눈 것 처럼 이용할 수 있습니다. <br>

tag는 말 그대로 블로그 태그처럼 사용할 수 있는 객체입니다.
author는 게시글의 저자를 나타내주는 객체입니다.

#### 3.2 _config.yml

jekyll은 블로그의 제목과 자신의 개인 정보를 설정하거나 테마에 대해서 설정할 수 있는 파일이 있습니다.

![config file](https://user-images.githubusercontent.com/48249549/79039136-23cf3f00-7c1a-11ea-9aad-6863839a39c3.png)
![config.yml](https://user-images.githubusercontent.com/48249549/79039198-9c360000-7c1a-11ea-9508-af02a23c5092.png)

바로 위에 사진파일입니다. title, email, url 등을 수정하여 다운받은 테마를 본인의 내용으로 꾸밀 수 있습니다.

#### 3.3 active

active 객체는 메뉴바를 관리하는 객체입니다.<br>
제 블로그를 예시로 어떻게 사용했는지 보여드리겠습니다.<br>

![menu ba](https://user-images.githubusercontent.com/48249549/79039280-2a11eb00-7c1b-11ea-8a9b-1a58c1b65641.png)
![active](https://user-images.githubusercontent.com/48249549/79039678-e53b8380-7c1d-11ea-870f-b7658cb6859d.png)

블로그 repository 안에 `Diary.md` `Projects.md` `about.md`을 각각 작성하고 이 파일들에 `active` 를 정의하여 메뉴바를 생성하였습니다.<br>
위에 사진처럼 저의 블로그는 index.html, 위에 마크다운 파일들을 포함하여 총 4개의 메뉴바를 생성한 모습을 보실수 있습니다.<br>

<br>
**위에 내용들을 종합하여 자신만의 레이아웃을 적용하여 블로그를 꾸밀 수 있고 active을 이용하여 메뉴바 생성하거나, 카테고리 객체를 이용하여 자신이 원하는 메뉴를 설정하여 _posts에 마크다운 형태로 글을 작성할 수 있습니다.**

추가로 메뉴를 카테고리 블록으로 구성할 때 참고했던 블로그 링크 남깁니다.
[카테고리 블록](https://devyurim.github.io/development%20environment/github%20blog/2018/08/12/blog-8.html)
<br>
<br>

---
### 4. 댓글 기능 추가하기, utterances
---
이렇게 블로그를 만들면 사람들과 소통을 할 수 있는 공간이 필요하겠죠. <br>
Github issue 기반을 이용해서 댓글을 작성할 수 있는 오픈소스 utterances를 사용해봅시다. <br>

[utterances](https://utteranc.es/) <br>

우선 위에 링크에 접속해봅시다.

![repo_utterances](https://user-images.githubusercontent.com/48249549/79304006-479cc880-7f2b-11ea-89c3-93e636ef6bda.png)

접속을 하면 위에 사진처럼 나옵니다. 내용을 따라서 진행해보도록 합시다. <br>

우선 issue를 사용할 public repository가 필요합니다. <br>
하나 만들어 주시거나 Github.io repo를 그대로 사용하셔도 됩니다. <br>
( 저는 따로 blog-comment라는 repository를 만들어서 관리를 하고있습니다. )<br>

그 다음으로 해당 repository에 utterances 깃허브 앱을 설치해줘야 합니다. <br>
[utterances app](https://github.com/apps/utterances)
링크에 들어가서 빠르게 진행해주시면 됩니다.

이제 `repo:` 에 앱을 설치한 repository를 입력해주시면 됩니다.<br>

![utterance2](https://user-images.githubusercontent.com/48249549/79304396-356f5a00-7f2c-11ea-8bea-a3b127508b35.png)
![utterance3](https://user-images.githubusercontent.com/48249549/79304454-5932a000-7f2c-11ea-9f33-96163317e611.png)

이제 issue가 어떤 방식으로 맵핑 되는 지와 라벨, 테마를 정해봅시다.

위에 사진처럼 issue에 맵핑하는 방법은 여러가지 옵션이 있습니다.

- pathname
    - 포스트의 pathname 으로 이슈를 생성한다.
- page URL
    - 게시글의 URL 전체로 이슈를 매핑한다.
- page title
    - 게시글의 제목으로 이슈를 매핑한다.
- issue number
    - 이슈 번호를 가지고 매핑한다.
- issue title contains specific term
    - 게시글 제목에 특정 단어가 들어가 있는지 체크하여 매핑한다.

본인이 issue를 어떤식으로 관리할지 생각하여 선택해주시면 됩니다. <br>

라벨은 본인의 이슈 뒤에 스티커(?)같은 것을 표시해주는 것입니다. <br>
원하는 단어를 넣어서 구분이 가능하게 사용할 수 있습니다.<br>

이제 적용을 해봅시다.

![utter 사진](https://user-images.githubusercontent.com/48249549/79305984-4f5e6c00-7f2f-11ea-9ffb-a478a37482e7.png)

위에 사진에 있는 부분에 있는 코드를 _layouts/post.html 맨 아랫부분에 적용하시면 바로 댓글처럼 사용하실 수 있습니다.
<br>
<br>
이렇게 Github.io 블로그와 jekyll 그리고 utterances에 대해서 제가 했던 것을 리뷰하면서 정리해봤습니다.<br>
저와 같이 블로그를 시작하는 분들에게 도움이 되었으면 좋겠습니다.
<br>

이상으로 첫 포스팅은 여기에서 마치겠습니다.
