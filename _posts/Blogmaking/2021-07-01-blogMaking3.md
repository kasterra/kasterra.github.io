---
title: 블로그 옮기기 ③:본격 블로그 메인화면 꾸미기
tags: 기타환경 블로그 Jekyll TeXt
katex: True
layout: post
excerpt: 밋밋한 블로그 화면을 화사하게 꾸며봅시다.
image: /images/thumbnails/blogmove3.png
series: 블로그 만들기
---

# 이번 포스트의 목표

이번 포스트에서는 아직 밋밋한 블로그에 좀 더 화사한 느낌을 불어넣고, 기본적인 TeXt 테마의 파비콘으로 설정된 밋밋한 블로그를, 우리의 개성을 나타낼 수 있는 블로그로 탈바꿈 시키는 시간을 가져볼까 합니다.

저는 디자인 센스가 좋은 예술가는 못되서, 여러 외부 서비스들을 적극적으로 도입했습니다.

<hr>
<h3>현재 진행상황</h3>
- [x] Github 블로그 첫걸음
  - [x] 화면을 띄우기(`본인계정ID`.github.io 접속 확인)
  - [x] `Jekyll` 기능 및 `TeXt` 자체 기능 탐방
- [x] 블로그 포스트 내용이 의도한대로 나오게 하기
    - [x] `Katex`를 활용한 수식 입력 기능
    - [x] 잘려서 나오는 목차 목록, 안잘리게 하기
    - [x] 코드 강조기능 커스텀하기
    - [x] `github-pages` gem 설치해서, 빌드환경과 테스트 환경 같게 만들기
- [ ] 그냥 글 목록만 나오는 메인 화면, 좀더 예쁘게 만들기
    - [ ] 블로그 대문이 예쁘게 나오게 만들기
    - [ ] 포스트 커버 이미지가 나오게 만들기
    - [ ] 내 블로그 로고 만들어서 장식하기
- [ ] 원래 테마를 넘어서 : 새로운 기능들 추가
  - [ ] 일기장에서 블로그로. 댓글 기능 달기
  - [ ] 관련 게시글 기능 만들기
  - [ ] 시리즈 기능 추가
- [ ] 블로그 관리 기타
  - [ ] `Github action`을 통한 배포 자동화
  - [ ] `Google Search Console` 사용하기
  - [ ] 광고 달기
  - [ ] 개인 도메인 연결하기

<div style="display:flex; justify-content:center"><strong>진행률</strong><br></div>
<div style="display:flex; flex-direction:column; align-items:center;"><progress value="6" max="16" style="width:80%"></progress><p>6/16</p></div>
<hr>
여하튼, 이제 네번째 대주제를 완성할 시간입니다. 기억하실지 모르겠지만, 이 시리즈의 맨 첫번째 포스팅에서 소개한 블로그의 모습과, 지금 우리가 보는 `github.io` 블로그와는 많이 차이가 있습니다.
<div style="display:flex; justify-content:center">
    <img src="/images/blogmaking/블로그메인.png">
</div>
커다란 커버 이미지도 나오고, 제목도 큼지막하게 나오고, 그리고 위에 나오는 네비게이션 바의 내용들도 한국어로 나옵니다.

이번 포스트에서는 지금 우리의 밋밋한 흰색 블로그를 화사한 민트색 블로그로 만들어보는 시간을 가지겠습니다. 우선 작업하기 전에, 우리가 해야할 일들을 github에 이슈로 등록하고, 작업을 시작합시다.

# 우리의 블로그의 정보를 넣자!

지금 현재의 `github.io` 블로그는 이름도 `Your Site Title`이고, 위의 여러가지들도 모두 영어로 나옵니다. 우리 블로그는 영어를 쓰기보단, 한국어를 더 많이 쓰니, 이런 설정보다는 한국어로 좀 나오는게 자연스럽죠.

이러한 설정을 반영시키기 위해서 `_config.yml`을 수정합시다.

**`_config.yml` 수정 후**

```yaml
## => Site Settings
##############################
text_skin: forest # "default" (default), "dark", "forest", "ocean", "chocolate", "orange"
highlight_theme: tomorrow-night-eighties # "default" (default), "tomorrow", "tomorrow-night", "tomorrow-night-eighties", "tomorrow-night-blue", "tomorrow-night-bright"
url     : # the base hostname & protocol for your site e.g. https://www.someone.com
baseurl : # does not include hostname
title   : 카스테라의 공부방
description: > # this means to ignore newlines until "Language & timezone"
  기술 블로그 카스테라의 공부방 입니다.

## => Language and Timezone
##############################
lang: ko
timezone: KR

## => Author and Social
##############################
author:
  type      : # "person" (default), "organization"
  name      : Kasterra
  url       :
  avatar    : # path or url of avatar image (square)
  bio       : 배울것이 아직 많아, 열심히 배우는 학생입니다.

생략....
```

뭐 파일을 찬찬히 읽어보고, 본인에게 필요한 정보를 잘 채워넣으면 됩니다.

<div style="display:flex; justify-contents:center;">
    <img src="/images/blogmaking/demo2.png">
</div>
이제 다시 로컬에서 블로그를 빌드해보면, 우리가 넣은 설정이 반영됨을 볼 수 있습니다.

# 블로그 대문 꾸미기

## 블로그 제목 꾸미기

우리의 블로그 대문은, 대문치고는 좀 심심합니다. 포스트를 읽을떄는 위에 나오는 네비게이션바가 요란스럽지 않아야, 내용에 집중 할 수 있어서 좋다고 생각하지만, 메인 페이지는 좀 제목하고 블로그 좌우명 같은것이 요란하게 나오는것이 어필에 도움이 된다고 개인적으로 생각합니다.

현재 우리의 블로그의 첫화면을 담당하고 있는 `index.html`은 아래와 같습니다.

```html
---
layout: home
# articles:
#   excerpt_type: html
---
```

그리고 우리가 원하는 디자인에 부합하는 이미지는 `TeXt`에서 샘플로 내놓은 페이지 디자인 형태중 하나인 [Article Header Overlay with Background Fill](https://tianqi.name/jekyll-TeXt-theme/page/article-header-overlay-background-fill-immersive-translucent-header.html)입니다.

주의해야 할 점이 한가지 있다면, 위에서 설명하는 것은 `layout`이 `article`이고, 우리의 `index.html`은 `layout`이 `home`이기 때문에, 저기에 있는걸 그대로 붙여넣기 하면, 제목이 안보이는 문제가 생깁니다.
이렇게 되는 이유는 추후에, `Jekyll`을 제대로 파보는 시리즈를 작성할 때 적어보고, 일단 지금은 레이아웃이 달라서 그런가보다 는 식을 대충 넘어가면 될 것 같습니다.

```yaml
---
layout: home
mode: immersive
article_header:
  type: overlay
  theme: dark
background_color: "#7cc7bb"
show_title: true
title: 카스테라의 공부방
excerpt: 이 블로그는 알고리즘, PS, 웹개발 이야기를 다룹니다
---
```

![폭이 넓은 성공](/images/blogmaking/demo3.png)
성공 했습니다만, 대문 이미지가 좀 많이 넓은것 같아서, CSS 작업을 좀 해야겠습니다. 이 레이아웃을 사용할 곳은 지금 생각해보면 아마도, 홈 레이아웃 말고는 없을것 같아서, `home.html`에 그냥 CSS 요소를 넣겠습니다.

```css
.overlay {
  min-height: auto !important;
  height: 18rem !important;
}
h1 {
  margin-top: 2rem;
}
```

![폭이 적당한 성공](/images/blogmaking/demo4.png)
우리가 원하는 첫 페이지의 디자인이 절반쯤 완성됐습니다. 이제, 포스트들이 나오는 디자인을 수정해야할 차례입니다.

## 포스트 목록 디자인 꾸미기

`index.html`에 우리는 딱히 포스트를 불러와라는 코드를 직접 삽입하지 않았지만, 포스트의 목록을 보여주고 있습니다. 그리고, 지금은 포스트가 충분히 많지 않아서 보이지는 않는데, `TeXt`테마에는 pagination이 적용되어 있고, 밑의 페이지 목록도 꽤나 볼만하게 나오게 됩니다. 나중에 Jekyll을 제대로 탐구하는 글을 쓰면서 정리를 할것입니다만, 간략하게 소개해보겠습니다.

`Jekyll`에서 `layout`이 작동하는 방식은 다음과 같습니다. Jekyll에서 쓰일 수 있는 가상의 html파일인 `something.html`을 생각해 봅시다.
**somecontent.html**의 예시

```html
---
layout: something
---

content of the page
```

와 같은 페이지가 있을떄, 이 페이지가 최종적으로 렌더링 되는 결과를 간단하게 요약해보겠습니다.

**\_layout/something.html**

```html
content of layout part 1 {%-raw-%}{{content}}{%endraw%} content of layout part 2
```

가 someting 레이아웃의 내용일 때,

최종적으로 나오는 **somecontent.html**

```html
content of layout part 1 content of the page content of layout part 1
```

이런식으로 작동합니다. 만약, `something`이라는 레이아웃이 또 다른 레이아웃 `anotherthing`을 사용하고 있었다 하면, `anotherthing` 레이아웃의 {%-raw-%}{{content}}{%endraw%} 부분에 `something`레이아웃과, `someconent`의 내용까지 포함되어 들어가는 식이죠.

이런식으로 블로그 레이아웃을 쭉쭉 읽어나가다 보면, `articles`레이아웃 내에서, `article-list.html`이라는 파일을 불러와서 포스팅의 목록들을 표시하고 있음을 볼 수 있습니다. `TeXt`테마 내에는 [다양한 포스팅 표시 방법이 존재](https://tianqi.name/jekyll-TeXt-theme/samples.html)하는데, 파일 내에서 페이지를 렌더링하는 모습을 보면, 여러 옵션을 받아서, 그 옵션에 맞게 페이지에 html 요소들을 넣는 방식입니다.

이중에서, cover image를 보여주는 레이아웃을 수정해서, 우리가 원하는 레이아웃을 만들어 봅시다.

저는 `velog` 스타일로 썸네일을 표시하고 싶었습니다. 데스크톱 화면에서는 커다란 이미지가 가운데 정렬되어 있고, 그보다 화면이 작은 기기에서는 각 기기 화면 사이즈에 맞게 꽉 들어찬 이미지를 말이죠.
이를 위해서는 다음과 같은것을 할 필요가 있습니다.

1. `item__image`에 `flex`를 넣어서, 이미지 가운데 정렬 하고, 이미지에도 글로 가는 링크 넣기
2. `@media`사용해서 화면 크기가 작은 기기에도 대응하게 하기

우선 옵션에 맞게 flex를 부여할 수 있도록 파일들을 수정해 봅시다.

`show_cover` 에 'thumbnail'이라는 값을 넣었을 때, 위에서 요구했던게 작동하도록 코드를 작성하였습니다.

수정된 **article-list.html** 일부분
{%-raw-%}
```html
{%- if _article.cover and include.show_cover and include.show_cover != 'thumbnail'-%}
          {%- include snippets/get-nav-url.html path=_article.cover -%}
          {%- assign _article_cover = __return -%}
          <div class="item__image">
          {%- if include.cover_size == 'lg' -%}
            <img class="image image--lg" src="{{ _article_cover }}" />
          {%- elsif include.cover_size == 'sm' -%}
            <img class="image image--sm" src="{{ _article_cover }}" />
          {%- else -%}
            <img class="image" src="{{ _article_cover }}" />
          {%- endif -%}
          </div>
        {%- endif -%}
        <div class="item__content">
          <header>
            <a href="{{ _article_url }}">
            {%- if include.show_cover =='thumbnail' and _article.cover -%}
              <div class="item__image thumbnail"><img src="{{ _article_cover }}" alt="cover of artilce"></div>
            {%- endif -%}
              <h2 itemprop="headline" class="item__header">{{ _article.title }}</h2>
            </a>
          </header>
```
{%endraw%}

이제 `item__image thumbnail`에 해당하는 CSS요소를 꾸며줍시다. `item`과 `image`둘다 각각의 CSS 파일이 있는데, 저는 `item.scss`에 css 요소를 더 추가하는 쪽으로 가겠습니다.

**\_sass/common/components/\_item.scss**

```scss
.item__image{
  생략...
  &.thumbnail{
    @include flexbox();
    justify-content: center;
    & > img{
      width:100%;
      height:100%;
    }
    @include media-breakpoint-up(lg){
      width:100%;
      height:50%;
      & > img{
        width:70%;
      }
    }
    margin-bottom:1rem;
    margin-right: map-get($spacers, 3);
  }
  생략...
}
```

코드 작성이 끝났습니다. 이제, 포스트 목록이 예쁘게 나오는지 확인해 봅시다.

![섬네일성공](/images/blogmaking/demo5.png)
성공했습니다.

우리의 성공을 자축하는 마음으로, 진행률 체크리스트도 갱신해 봅시다.
<hr>
<h3>현재 진행상황</h3>
- [x] Github 블로그 첫걸음
  - [x] 화면을 띄우기(`본인계정ID`.github.io 접속 확인)
  - [x] `Jekyll` 기능 및 `TeXt` 자체 기능 탐방
- [x] 블로그 포스트 내용이 의도한대로 나오게 하기
    - [x] `Katex`를 활용한 수식 입력 기능
    - [x] 잘려서 나오는 목차 목록, 안잘리게 하기
    - [x] 코드 강조기능 커스텀하기
    - [x] `github-pages` gem 설치해서, 빌드환경과 테스트 환경 같게 만들기
- [ ] 그냥 글 목록만 나오는 메인 화면, 좀더 예쁘게 만들기
    - [x] 블로그 대문이 예쁘게 나오게 만들기
    - [x] 포스트 커버 이미지가 나오게 만들기
    - [ ] 내 블로그 로고 만들어서 장식하기
- [ ] 원래 테마를 넘어서 : 새로운 기능들 추가 
  - [ ] 일기장에서 블로그로. 댓글 기능 달기
  - [ ] 관련 게시글 기능 만들기
  - [ ] 시리즈 기능 추가
- [ ] 블로그 관리 기타
  - [ ] `Github action`을 통한 배포 자동화 
  - [ ] `Google Search Console` 사용하기
  - [ ] 광고 달기
  - [ ] 개인 도메인 연결하기

<div style="display:flex; justify-content:center"><strong>진행률</strong><br></div>
<div style="display:flex; flex-direction:column; align-items:center;"><progress value="8" max="16" style="width:80%"></progress><p>8/16</p></div>
<hr>
드디어 절반에 도달했습니다.

# 블로그에 개성 불어넣기
웹 사이트의 개성을 들어내는 데에는 웹사이트의 기능이나, 디자인도 중요하게 작용하지만, 웹사이트의 로고도 정말 중요하게 작용한다고 생각합니다. 넷플릭스, 구글, 네이버 등등 우리가 많이 쓰는 서비스들의 로고는 우리의 뇌리에 강하게 각인되어 있습니다. 우리의 블로그가 그정도 브랜드파워를 가지기 소원하는 마음에서, 로고를 만들고 이를 방문자들이 볼 수 있게 해봅시다.
## 로고 만들기
이 글 서두에 잠깐 언급했듯, 저는 디자인을 잘하는 사람은 아닙니다. 그래서 이 블로그 로고를 디자인 하는데는 저보다 더 디자인 센스가 좋은 사람들이 만들어놓은 서비스를 활용했습니다.
[freelogodesign](https://www.freelogodesign.org/)이라는 서비스를 활용했습니다. 다양한 레이아웃 주제를 제공해서, 본인의 취향껏 고를 수 있는점이 참 마음에 들었습니다.
<div style="display:flex; justify-content:center"><img src="/images/blogmaking/freelogo.png"></div>

무료 플랜으로는 200px * 200px 의 이미지밖에 사용을 못하지만, 페이지를 간단히 장식할 목적으로는 충분하다고 생각합니다.

<div style="display:flex; justify-content:center"><img src="/images/blogmaking/kaslogo.png"/></div>
저는 이런 로고를 만들었습니다. 상당히 마음에 드는군요. 블로그 각자 입맛에 맞도록 로고를 만들면 될 것 같습니다.

## 파비콘으로 가공하고 적용하기
[TeXt 공식 문서](https://tianqi.name/jekyll-TeXt-theme/docs/en/logo-and-favicon)에서 로고와 파비콘에 대한 설명이 있어, 이 설명대로 따라가겠습니다.
파비콘으로 만들 이미지 파일을 제출하는데, 개인적인 조언이라면, 손톱만큼의 크기로 보이는것이 파비콘 이기에, 많은 요소를 넣지 않는것이 좋습니다. 제 경우에는, kasterra 라는 글씨를 뺴고, 행성 그림만 나오게 하여 제출해서 사용하였습니다.

아이콘을 패키지를 다운받고, 이를 적절한 곳에 블로그에 배치해 줍시다.
`browserconfig.xml`과 `site.webmanifest`또한 교체하여 주고, 페이지에서 나온 html 코드는 `_includes/head/favicon.html`에 교체해서 넣어줍시다.

이렇게 바꾸어 놔도, 블로그에 처음 들어왔을때 로고 부분은 은행나뭇잎 그대로일 것입니다. 이건 `_includes/logo.svg`를 교체해서 넣으면 됩니다. 

그렇게 바꿔놓고 들어와보니, 또 문제가 보입니다.
<div style="display:flex; justify-content:center"><img src="/images/blogmaking/demo6.png"/></div>

로고가 너무 작게 보인다는 것인데, 이 문제는 ` /_sass/component/header.scss`의 67,68번째 줄을 고치는 것으로 해결합니다.

```scss
.header__brand {
  @include flexbox();
  @include align-items(center);
  & > svg {
    width: map-get($base, font-size-h4) * 3;
    height: map-get($base, font-size-h4) * 3;
    margin-right: map-get($spacers, 3);
    vertical-align: middle;
    @include media-breakpoint-down(md) {
      width: map-get($base, font-size-h4) * 1.2;
      height: map-get($base, font-size-h4) * 1.2;
    }
  }
  & > a {
    display: inline-block;
    font-size: map-get($base, font-size-h4);
    @include media-breakpoint-down(md) {
      font-size: map-get($base, font-size-h4-small);
    }
  }
}
```
<div style="display:flex; justify-content:center"><img src="/images/blogmaking/demo7.png"/></div>

이제 문제가 모두 해결된것 같습니다.
<hr>
<h3>현재 진행상황</h3>
- [x] Github 블로그 첫걸음
  - [x] 화면을 띄우기(`본인계정ID`.github.io 접속 확인)
  - [x] `Jekyll` 기능 및 `TeXt` 자체 기능 탐방
- [x] 블로그 포스트 내용이 의도한대로 나오게 하기
    - [x] `Katex`를 활용한 수식 입력 기능
    - [x] 잘려서 나오는 목차 목록, 안잘리게 하기
    - [x] 코드 강조기능 커스텀하기
    - [x] `github-pages` gem 설치해서, 빌드환경과 테스트 환경 같게 만들기
- [x] 그냥 글 목록만 나오는 메인 화면, 좀더 예쁘게 만들기
    - [x] 블로그 대문이 예쁘게 나오게 만들기
    - [x] 포스트 커버 이미지가 나오게 만들기
    - [x] 내 블로그 로고 만들어서 장식하기
- [ ] 원래 테마를 넘어서 : 새로운 기능들 추가 
  - [ ] 일기장에서 블로그로. 댓글 기능 달기
  - [ ] 관련 게시글 기능 만들기
  - [ ] 시리즈 기능 추가
- [ ] 블로그 관리 기타
  - [ ] `Github action`을 통한 배포 자동화 
  - [ ] `Google Search Console` 사용하기
  - [ ] 광고 달기
  - [ ] 개인 도메인 연결하기

<div style="display:flex; justify-content:center"><strong>진행률</strong><br></div>
<div style="display:flex; flex-direction:column; align-items:center;"><progress value="9" max="16" style="width:80%"></progress><p>9/16</p></div>
<hr>

# 요약
이번 시간에는 블로그에 우리의 개성을 불어넣는 작업을 해봤습니다. 여태까지의 블로그 글은 이전 `velog`에서 쓴 것이거나, 임시로 땜빵한 블로그에서 썼던 것인데, 이제 지금 우리가 github pages로 서비스 하는 블로그에서 작업을 할 수 있을만큼 블로그가 화사해 졌습니다. 

<h3>현재 블로그 상태</h3>
![블로그 상태](/images/blogmaking/frontpageComplete.png)

기존 TeXt 기본 상태보다 훨씬 화사하고 보기 좋은것 같습니다.

