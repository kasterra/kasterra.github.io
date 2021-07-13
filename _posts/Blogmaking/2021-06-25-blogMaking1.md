---
title: 블로그 옮기기 ①:첫걸음 Jekyll 기초
tags: 기타환경 블로그 Jekyll TeXt
katex: True
layout: post
excerpt: 기초의 기초 프론트엔드 지식만 가지고 있는 상태에서, Jekyll 블로그를 운영하기 위한 기초 지식을 더 얹어 봅시다.
image: /images/thumbnails/blogmove1.png
series: 블로그 만들기
category: BLOG
---

# 시작하기 앞서

우선 이 글에서 만들 블로그는, 제가 삽질을 하면서 **아무튼** 만들어낸 블로그를, 만든 과정을 삽질을 털어내고, 개발한 내용을 깔끔하고, 체계적으로 `github`를 활용해 개발 워크플로를 남들한테 보여줄 수 있도록 해보자. 라는것이 이 시리즈의 목표입니다.

<i class="fas fa-info-circle"></i> <strong>Note</strong><br>
만약 `github`를 통한 개발 워크플로 라는게 뭐지 싶으신 분은<br>
<https://www.huskyhoochu.com/issue-based-version-control-201/> 이 글을 한번 읽고 오세요.
{:.info}

우선 제가 삽질을 하면서 만든, 그리고 이 포스팅을 통해서 다시 만들어서, 이번엔 github pages로 서비스에 올리기까지 해볼 블로그의 모습은 아래와 같습니다.
![](/images/blogmaking/블로그메인.png)
그리고, 블로그의 글을 조회하면 나올 화면은 아래와 같습니다.
![](/images/blogmaking/블로그본문.png)

이 두장의 사진이 모든것을 보여주진 않지만, 대강 괜찮게 생긴 블로그를 만드는구나 하는 느낌 정도는 오셨을 것입니다. 이제 저도 여러분과 똑같은 입장이 되어서, 원본이 되는 TeXt 테마를 포크를 하는 것부터 하나 하나 진행해보려고 합니다. 따라올 준비가 되셨길 바랍니다.

# 테마 포크 떠오기

<https://github.com/kitian616/jekyll-TeXt-theme> `Github`에 `TeXt`테마가 올라와 있습니다. 이제 우리는 이걸 포크해와서, 우리의 입맛에 맞게 요리하기를 시작해 봅시다.
![](/images/blogmaking/setting.png)
리포지토리의 `setting`에 들어가서, `본인의 계정 ID 이름`.github.io 로 리포지토리 이름을 변경해 주세요. 이러한 이름을 가진 리포지토리는 `github`에 "이것은 내 개인 블로그입니다" 라는 정보를 알려주는 역할을 합니다.

이제 아까 설정한 리포지토리 이름을 주소창에 입력을하면, 테마의 기본값인 상태로 블로그가 켜질겁니다.
![](/images/blogmaking/first.png)

# 작업 시작

본인이 만약에 딱히 테마를 건들이지 않고, 글만 올리겠다라고 생각한다면, 로컬 컴퓨터에 `ruby`를 설치하고 하는 그러한 과정을 거칠 필요가 없습니다.
<br><br>
하지만, 이 포스팅 시리즈의 목적은, 기본 테마를 기반으로 한, **조금의 양념을 더 추가한** 블로그를 만들어보자 이기 때문에, 로컬에서도 작업을 진행할 필요가 있습니다.
<https://jekyllrb-ko.github.io/docs/>을 참고해서, 각자 본인의 시스템에 맞게 진행하시면 됩니다.

하나씩 차곡 차곡 진행해 봅시다. 우리의 여정을 미리 소개하자면 다음과 같습니다.

<hr>
- [ ] Github 블로그 첫걸음
  - [x] 화면을 띄우기(`본인계정ID`.github.io 접속 확인)
  - [ ] `Jekyll` 기능 및 `TeXt` 자체 기능 탐방
- [ ] 블로그 포스트 내용이 의도한대로 나오게 하기
    - [ ] `Katex`를 활용한 수식 입력 기능
    - [ ] 잘려서 나오는 목차 목록, 안잘리게 하기
    - [ ] 코드 강조기능 커스텀하기
    - [ ] `github-pages` gem 설치해서, 빌드환경과 테스트 환경 같게 만들기
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
<div style="display:flex; flex-direction:column; align-items:center;"><progress value="1" max="16" style="width:80%"></progress><p>1/16</p></div>
<hr>
갈길이 멀어 보이지만, 차곡 차곡 한단계씩 하면 안될거 없다고 생각합니다. 한번 해봅시다! 아자!

# `Jekyll` 및 `TeXt` 기능 탐방

[이전 포스트]({{page.previous.url}})에서도 간단히 언급을 했는데, 제가 `TeXt`라는 테마를 고른 이유는 여러 기능들이 미리 포함되어 있기 때문입니다.
[공식 문서](https://tianqi.name/jekyll-TeXt-theme/docs/en/quick-start)를 보면서 저는 이 테마에 관해서 이것 저것을 다 학습했습니다.

이걸 다 읽기 귀찮으신 분들을 위해서 간단간단히 요약 해보겠습니다. `Jekyll`의 기능이 아닌 `TeXt`테마에 의존적인 기능인것은, 제목 끝에 `-T`를 붙여서 표시하겠습니다.

## Jekyll 구조

```
├── _data
│   ├── locale.yml
│   ├── navigation.yml
│   └── variables.yml
├── _includes
│   ├── analytics-providers
│   ├── aside
│   ├── comments-providers
│   ├── markdown-enhancements
│   ├── pageview-providers
│   ├── scripts
│   ├── sidebar
│   ├── snippets
│   ├── svg
│   │   ├── icon
│   │   │   ├── social
│   │   │   │   ├── facebook.svg
│   │   │   │   └── ...
│   │   └── logo.svg
│   └── ...
├── _layouts
│   ├── 404.html
│   ├── archive.html
│   ├── article.html
│   ├── base.html
│   ├── home.html
│   ├── none.html
│   └── page.html
├── _sass
├── assets
│   ├── css
│   │   └── blog.scss
│   └── images
├── tools
├── 404.html
├── Gemfile
├── _config.yml
├── about.md
├── archive.html
├── favicon.ico
├── gulpfile.js
├── index.html
├── jekyll-text-theme.gemspec
└── package.json
```

Jekyll을 처음 다운받으면 대충 이러한 구조와 마주할 것입니다. 필요한것만 간략하게 설명하겠습니다. 추가로 설명이 필요하시다면, 댓글 남겨주세요

-   \_data 폴더 : 별로 수정할만한 내용은 없습니다.
    -   `navagation.yml` : 만약에 블로그 상단에 나오는 내용들을 수정하고 싶으시다면 수정하시면 될것 같습니다.
    -   `locale.yml` : `TeXt`테마 내에서 지원하지 않는 다른 외국어 설정을 넣으시고 싶으시면 수정하면 됩니다. 참고로 한국어는 이 테마에서 이미 로케일이 있습니다.
    -   `variables.yml`: 블로그 내에서 yml 변수들을 모아놓은 파일입니다. 기본값을 수정하고 싶으시면 수정하면 될 듯 합니다.
-   \_includes 폴더 : 블로그 테마 내에서 이것저것 포함할 스닙셋들을 모아둔 폴더입니다. 내용물이 많아서, 다 설명은 못드리고, 커스터마이징을 하면서 차례차례 건드리지 않을까 싶습니다.
-   \_layouts 폴더 : 말 그대로 블로그 테마 레이아웃을 렌더링 할 때, 틀이 되는 파일입니다. 만약에 본인이 기본 테마를 좀 더 뒤집고 싶다면 수정하면 될것 같습니다. 이 포스트 내에서는 수정할 일이 없습니다.
-   \_sass 폴더 : sass 서식으로 된 스타일시트들이 이 폴더내에 있습니다. 커스터마이징을 하면서 이것저것 수정할 예정입니다.
-   \_config.yml : [밑](#_configyml에-관해)에서 자세히 설명할 예정입니다.
-   \_posts 폴더 : 포스트를 저장할 폴더입니다. `.md`나 `.html`로 문법에 맞게 작성하면 됩니다. 제목 형식은 `YYYY-MM-DD-제목.확장자`로 작성하면 됩니다.
-   about.md : 블로그 상단의 네비게이션 바에서, about를 클릭하면 나올 파일입니다.
-   index.html : 블로그를 들어가면 맨 처음에 나올 페이지 입니다.

### 블로그 경량화

Jekyll 자체가 정적 페이지를 만드는것이다 보니, 별다른 최적화 호들갑은 떨지 않아도 되나, 파일이 너무많아 정신사나워서 좋을것은 딱히 없으니 지워도 되는 필요없는 파일/폴더 정도는 소개해 두겠습니다.

-   최상위 디렉토리에 있는 텍스트 파일들: 설명서나 라이센스에 해당하는 파일들이 대부분 입니다. `about.md`는 블로그 설명문을 적으실거면 남겨두세요
-   docker 폴더, Dockerfile.dev : `Docker`를 통해서 `Jekyll`을 사용하는데에 필요한 파일인듯 합니다. 이 포스트에서는 `Docker`를 사용하지 않고, 바로 로컬 머신에서 진행할것이기 때문에 필요가 없습니다.
-   .github 폴더 : `github`에서 `issue`를 제출할 때 쓰이는 틀입니다. 필요 없다 생각하시면 지우세요.
-   docs, screenshots, test 폴더 : 폴더 이름 그대로의 역할입니다. `screenshots` 폴더는 지우면 기본의 `about.md`의 몇몇 사진이 제대로 보이지 않을 수 있다는 점 정도는 참고하세요.
-   .travis.yml : `travis CI`라는 배포 자동화 툴에서 사용하는 파일이다만, 여기선 `github action`을 통해서 배포 자동화를 진행해볼 계획이기 때문에 필요가 없다.

## 포스트 작성 시

포스트 작성은 `_posts`폴더 내에 하며, 파일 제목 형식은 `YYYY-MM-DD-제목` 형식을 따릅니다. 예를들어서, 2021년 1월 1일에 작성한 "새해 복 많이 받으세요" 라는 이름의 게시글을 하나 작성한다고 하면,
`2021-01-01-새해 복 많이 받으세요.md` 가 되겠군요(만약 html로 저장한다면 .md가 아닌 .html 이겠죠?).

그리고, 모든 글의 앞에는 (의무는 아니지만), `Jekyll`에서 우리를 도와주기 위해서 사용하는 템플릿 언어인 `Liquid`를 위해서, `Yaml`이라는 형태로 된 간단한 정보를 몇몇개 적어둘 필요가 있습니다.
[머리말 설명(Jekyll 공식 한글 문서)](https://jekyllrb-ko.github.io/docs/front-matter/)를 참조하시면, 조금 더 많은 정보를 얻을 수 있지만, 여기선 간결하게 서술하겠습니다.

문서 **맨 위**에 시작과 끝을 대시기호(-)로 감싼후 내용을 작성하면 됩니다.
거의 필수적으로 작성하는 항목으로는

-   title : 해당 포스트의 제목입니다.
-   excerpt : 발췌문 입니다. 우리가 만들 블로그는 이 기능을 쓰진 않지만, `TeXt`자체 기본 설정에서는 이 값을 많이 씁니다.
-   layout : `Jekyll`이 페이지를 렌더링 해줄때, 어떤 탬플릿을 사용할지 알려주는 옵션입니다. 일반적으로 포스트에는 `article`옵션을 사용합니다.

이 외에도, 추가적으로 원하는 변수를 마음대로 생성할 수도 있습니다. 우리는 이 기능을 이용해서, 여러 기능들을 추가할 것입니다.

## `Liquid` 기본 문법

`Liquid`를 끝까지 이 글에서 파고들 생각은 없습니다. 그냥, Liquid가 뭘 어떤걸 할 수 있는지만 간략하게 설명하도록 하겠습니다. 저는 제 글이 맨 처음 시작했을때의 제 상태와 같은, 약간의 웹 지식을 가지고 깃허브 블로그를 만들겠다고, 덤비는 여러분들에게 도움이 되길 바라지, 두고두고 참고할 문서가 되기까지를 바라진 않습니다.

일반적으로 `Liquid`에선, 제어문 등을 {%raw%}`{%제어문%}`{%endraw%}의 형태로 중괄호와 % 기호로 감싸서 사용하고, 변수의 사용은 {%raw%}`{{변수명}}`{%endraw%}의 형태로 사용한다라고만 알아두시고 이걸 읽으시면, `Liquid`가 정말로 간단하고 직관적임을 느끼시는데 도움이 되지 않을까 싶습니다.

<i class="fas fa-info-circle"></i> <strong>Liquid 문법 사용으로 편집기에 나오는 에러는 진짜 에러가 아니에요!</strong><br>
`Liquid`문법을 HTML안에 넣으면, `vscode`와 같은 일반적인 편집기에서, 얘를 에러로 볼 경우가 있습니다. 하지만, 이건 `Liquid`의 문법을 `vscode`가 읽지 못했기 때문에 생기는 일로, 실제 구동상 문제는 전혀 없습니다. 참고로, liquid 플러그인을 설치해도, liquid 문법만 하이라이팅 해주지, 에러를 잡아주진 않습니다. 플러그인 설명에 에러를 보기 싫으면 HTML 문법 오류 검사를 끄라고 소개하고 있습니다.
{:.info}

좀 더 자세한 `Liquid`문법을 읽고 싶으시다면 공식 독스(<https://shopify.github.io/liquid/>)를 찬찬히 읽어보세요. 친절하게 잘 설명해서 읽기가 편합니다.

### 변수 사용하기

변수를 할당하는것은 {%raw%}`{%assign 변수명 = 넣을_값%}`{%endraw%} 형태로 사용할 수 있습니다. 변수를 꺼내쓸떄는, 위에서도 언급했듯, {%raw%}`{{변수명}}`{%endraw%}의 형태로 사용하시면 됩니다.

### 조건문

`Liquid`는 흐름 제어를 위해서 `if`, `unless`, `elsif`, `else`, `case`, `when`을 지원합니다. 사실 적당히 이름만 봐도 이거다 싶기 때문에 자세한 설명은 길게 하지 않겠습니다.

예시)
{% raw %}

```liquid
{% if product.title == "Awesome Shoes" %}
  These shoes are awesome!
{% endif %}
```

위의 코드는 `product.title`이 `"Awesome Shoes"`이면 These shoes are awesome!이란 글씨를 HTML문서에 남긴다고 보시면 됩니다.

```liquid
{% assign handle = "cake" %}
{% case handle %}
  {% when "cake" %}
     This is a cake
  {% when "cookie", "biscuit" %}
     This is a cookie
  {% else %}
     This is not a cake nor a cookie
{% endcase %}
```

{%endraw%}
C언어 계열의 `switch-case`문과 유사한 문법이라고 보시면 됩니다.

### 반복문

`Liquid`는 반복문 역시 지원합니다. `for`문과, 반복할 인자가 없을때 실행할 `else` 그리고 C계열 언어에서 많이 봤던 `continue`와 `break` 지원합니다. `foreach`라고 불리는, 객체를 순회하며 반복문을 실행하는 문법또한 지원합니다. 이 문법을 통해서, 나중에 글 목록을 페이지에 표시하는데에 활용할 것입니다.
{%raw%}

```liquid
{% for product in collection.products %}
  {{ product.title }}
{% else %}
  The collection is empty.
{% endfor %}
```

위의 코드는 `collection.products`라는 배열 변수를 순회하며, 각 `product`의 `title`변수를 출력하는 코드입니다. `{% else %}` 부분은 `collection.products`가 비어있을 때, 출력되는 문장입니다.
{%endraw%}

## `_config.yml`에 관해

`Jekyll`에서 `_config.yml`은 여러분이 여러분의 테마를 만들거나, 테마를 수정하지 않더라도, 커스터마이징을 원한다면 자주 들락날락할 파일입니다. `JSON`처럼 데이터의 형태를 나타내는 `YAML` 문법으로 작성되어 있으며, 각 테마별로 달려있는 옵션이 다를수도 있습니다. `TeXt`의 `_config.yml`은 기본적으로 어느 부분이 어떤 역할을 하는지 주석으로 표시해 놔서 대략적인 의미 정도는 이해하기에 충분합니다. 한번 쭉 읽어보세요.
이걸 다 이해할 필요는 없습니다. 내가 특정한 기능을 추가하거나 수정하고 싶을 때, 어떤 부분을 건드려야 할 지 검색할 수 있는 능력만 있으면 충분합니다.

## Font Awesome 아이콘 -T

Font Awesome은, 멋진 폰트를 제공하는 서비스가 **아닌** 멋진 아이콘을 제공하는 서비스 입니다. <https://fontawesome.com/>에 들어가서, 상단의 `icons`를 누르시면, 여러 아이콘을 검색할 수 있습니다.
![폰트어섬](/images/blogmaking/fontawesome.png)
Font Awesome을 사용하기 위해서는 스크립트의 한 조각을 font awesome 관련 요소를 사용하려는 페이지에 넣어야 하는데, TeXt는 테마 차원에서 이걸 미리 해놨습니다. 디자인을 하는데에 약간 더 도움을 받을 수 있을것으로 생각됩니다.

## Alert, Tag, Button ,Swiper -T

블로그 포스팅을 하면서, 좀 더 꾸밀 수 있는 요소들을 `TeXt`에서는 지원해 줍니다.

### Alert 기능

Alert에 넣을 글씨
{:.success}
사용법

```markdown
Alert에 넣을 글씨
{:.success}
```

`success` 대신에 `info`, `warning`, `error`를 넣어서 색을 다르게 줄 수 있습니다.

### Tag 기능

`Tag에 넣을 글씨`{:.success}
<br>사용법

```markdown
`Tag에 넣을 글씨`{:.success}
```

여기서도 `success` 대신에 `info`, `warning`, `error`를 넣어서 색을 다르게 줄 수 있습니다.

여담으로 블로그 포스트의 Tag는 이 Tag를 쓰지 않고, Button을 사용합니다.

### Button 기능

Button은 마크다운으로 작성할 수도 있지만, html로도 작성이 가능합니다.

| 타입 | 클래스 이름                                                                                                                                                                                                                                          |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| base | button                                                                                                                                                                                                                                               |
| 타입 | button--primary, button--secondary, button--success, button--info, button--warning, button--error, button--outline-primary, button--outline-secondary, button--outline-success, button--outline-info, button--outline-warning, button--outline-error |
| 모양 | button--pill, button--rounded, button--circle                                                                                                                                                                                                        |
| 크기 | button--md (default), button--xs, button--sm, button--lg, button--xl                                                                                                                                                                                 |

다음은 `markdown`으로 Button 기능을 사용하는 방법입니다.

```markdown
[BUTTON](이동할_링크){위의 표에 있는 옵션들 조합}

예시 :
[BUTTON](#){:.button.button--primary.button--pill}
```

HTML로 작성하는것은 더 간단합니다. `<a></a>`를 하나 만들고, `class`에 위의 표에 명시된 요소들을 적절히 조합해서 넣으면 됩니다.

```html
<a class="button button--primary button--pill" href="">BUTTON</a>
```

결과물 : <br>
<a class="button button--primary button--pill" href="">BUTTON</a>

### Swiper 기능

`TeXt`테마 기능중에 많이 마음에 들었던 것 중 하나입니다.
여러 내용물들을 넘기면서 보려고 할 때, 사용하면 좋은 기능입니다.
터치도 지원하기에, 반응형 웹에 참 어울리죠.

plain HTML에선 당연히 이런건 없기 때문에, JS를 페이지 내에 적절한 형태로 포함시켜야 합니다.

**HTML** :

```html
<div class="swiper swiper-demo">
    <div class="swiper__wrapper">
        <div class="swiper__slide">1</div>
        <div class="swiper__slide">2</div>
        <div class="swiper__slide">3</div>
        <div class="swiper__slide">4</div>
        <div class="swiper__slide">5</div>
        <div class="swiper__slide">6</div>
        <div class="swiper__slide">7</div>
    </div>
    <div class="swiper__button swiper__button--prev fas fa-chevron-left"></div>
    <div class="swiper__button swiper__button--next fas fa-chevron-right"></div>
</div>
```

**CSS** :

```css
.swiper__slide {
    display: flex;
    justify-content: center;
}
```

**JS** :

```javascript
{%- raw -%}
{%- include scripts/lib/swiper.js -%}
{%endraw%}
var SOURCES = window.TEXT_VARIABLES.sources;
window.Lazyload.js(SOURCES.jquery, function() {
  $('.swiper-demo').swiper();
});
```

결과물:

<div class="swiper swiper-demo">
  <div class="swiper__wrapper">
    <div class="swiper__slide">1</div>
    <div class="swiper__slide">2</div>
    <div class="swiper__slide">3</div>
    <div class="swiper__slide">4</div>
    <div class="swiper__slide">5</div>
    <div class="swiper__slide">6</div>
    <div class="swiper__slide">7</div>
  </div>
  <div class="swiper__button swiper__button--prev fas fa-chevron-left"></div>
  <div class="swiper__button swiper__button--next fas fa-chevron-right"></div>
</div>
<style>
  .swiper__slide{
    display:flex;
    justify-content:center;
  }
</style>
<script>
  (function() {
  var SOURCES = window.TEXT_VARIABLES.sources;
  window.Lazyload.js(SOURCES.jquery, function() {
    function swiper(options) {
      var $window = $(window), $root = this, $swiperWrapper, $swiperSlides, $swiperButtonPrev, $swiperButtonNext,
        initialSlide, animation, onChange, onChangeEnd,
        rootWidth, count, preIndex, curIndex, translateX, CRITICAL_ANGLE = Math.PI / 3;

      function setOptions(options) {
        var _options = options || {};
        initialSlide = _options.initialSlide || 0;
        animation = _options.animation === undefined && true;
        onChange = onChange || _options.onChange;
        onChangeEnd = onChangeEnd || _options.onChangeEnd;
      }

      function init() {
        $swiperWrapper = $root.find('.swiper__wrapper');
        $swiperSlides = $root.find('.swiper__slide');
        $swiperButtonPrev = $root.find('.swiper__button--prev');
        $swiperButtonNext = $root.find('.swiper__button--next');
        animation && $swiperWrapper.addClass('swiper__wrapper--animation');
        calc(true);
      }

      function preCalc() {
        rootWidth = $root.width();
        count = $swiperWrapper.children('.swiper__slide').length;
        if (count < 2) {
          $swiperButtonPrev.addClass('d-none');
          $swiperButtonNext.addClass('d-none');
        }
        curIndex = initialSlide || 0;
        translateX = getTranslateXFromCurIndex();
      }

      var calc = (function() {
        var preAnimation, $swiperSlide, $preSwiperSlide;
        return function (needPreCalc, params) {
          needPreCalc && preCalc();
          var _animation = (params && params.animation !== undefined) ? params.animation : animation;
          if (preAnimation === undefined || preAnimation !== _animation) {
            preAnimation = _animation ? $swiperWrapper.addClass('swiper__wrapper--animation') :
              $swiperWrapper.removeClass('swiper__wrapper--animation');
          }
          if (preIndex !== curIndex) {
            ($preSwiperSlide = $swiperSlides.eq(preIndex)).removeClass('active');
            ($swiperSlide = $swiperSlides.eq(curIndex)).addClass('active');
            onChange && onChange(curIndex, $swiperSlides.eq(curIndex), $swiperSlide, $preSwiperSlide);
            if (onChangeEnd) {
              if (_animation) {
                setTimeout(function() {
                  onChangeEnd(curIndex, $swiperSlides.eq(curIndex), $swiperSlide, $preSwiperSlide);
                }, 400);
              } else {
                onChangeEnd(curIndex, $swiperSlides.eq(curIndex), $swiperSlide, $preSwiperSlide);
              }
            }
            preIndex = curIndex;
          }
          $swiperWrapper.css('transform', 'translate(' + translateX + 'px, 0)');
          if (count > 1) {
            if (curIndex <= 0) {
              $swiperButtonPrev.addClass('disabled');
            } else {
              $swiperButtonPrev.removeClass('disabled');
            }
            if (curIndex >= count - 1) {
              $swiperButtonNext.addClass('disabled');
            } else {
              $swiperButtonNext.removeClass('disabled');
            }
          }
        };
      })();

      function getTranslateXFromCurIndex() {
        return curIndex <= 0 ? 0 : - rootWidth * curIndex;
      }

      function moveToIndex(index ,params) {
        preIndex = curIndex;
        curIndex = index;
        translateX = getTranslateXFromCurIndex();
        calc(false, params);
      }

      function move(type) {
        var nextIndex = curIndex, unstableTranslateX;
        if (type === 'prev') {
          nextIndex > 0 && nextIndex--;
        } else if (type === 'next') {
          nextIndex < count - 1 && nextIndex++;
        }
        if (type === 'cur') {
          moveToIndex(curIndex, { animation: true });
          return;
        }
        unstableTranslateX = translateX % rootWidth !== 0;
        if (nextIndex !== curIndex || unstableTranslateX) {
          unstableTranslateX ? moveToIndex(nextIndex, { animation: true }) : moveToIndex(nextIndex);
        }
      }

      setOptions(options);
      init();
      preIndex = curIndex;

      $swiperButtonPrev.on('click', function(e) {
        e.stopPropagation();
        move('prev');
      });
      $swiperButtonNext.on('click', function(e) {
        e.stopPropagation();
        move('next');
      });
      $window.on('resize', function() {
        calc(true, { animation: false });
      });

      (function() {
        var pageX, pageY, velocityX, preTranslateX = translateX, timeStamp, touching;
        function handleTouchstart(e) {
          var point = e.touches ? e.touches[0] : e;
          pageX = point.pageX;
          pageY = point.pageY;
          velocityX = 0;
          preTranslateX = translateX;
        }
        function handleTouchmove(e) {
          if (e.touches && e.touches.length > 1) {
            return;
          }
          var point = e.touches ? e.touches[0] : e;
          var deltaX = point.pageX - pageX;
          var deltaY = point.pageY - pageY;
          velocityX = deltaX / (e.timeStamp - timeStamp);
          timeStamp = e.timeStamp;
          if (e.cancelable && Math.abs(Math.atan(deltaY / deltaX)) < CRITICAL_ANGLE) {
            touching = true;
            translateX += deltaX;
            calc(false, { animation: false });
          }
          pageX = point.pageX;
          pageY = point.pageY;
        }
        function handleTouchend() {
          touching = false;
          var deltaX = translateX - preTranslateX;
          var distance = deltaX + velocityX * rootWidth;
          if (Math.abs(distance) > rootWidth / 2) {
            distance > 0 ? move('prev') : move('next');
          } else {
            move('cur');
          }
        }
        $swiperWrapper.on('touchstart', handleTouchstart);
        $swiperWrapper.on('touchmove', handleTouchmove);
        $swiperWrapper.on('touchend', handleTouchend);
        $swiperWrapper.on('touchcancel', handleTouchend);

        (function() {
          var pressing = false, moved = false;
          $swiperWrapper.on('mousedown', function(e) {
            pressing = true; handleTouchstart(e);
          });
          $swiperWrapper.on('mousemove', function(e) {
            pressing && (e.preventDefault(), moved = true, handleTouchmove(e));
          });
          $swiperWrapper.on('mouseup', function(e) {
            pressing && (pressing = false, handleTouchend(e));
          });
          $swiperWrapper.on('mouseleave', function(e) {
            pressing && (pressing = false, handleTouchend(e));
          });
          $swiperWrapper.on('click', function(e) {
            moved && (e.stopPropagation(), moved = false);
          });
        })();

        $root.on('touchmove', function(e) {
          if (e.cancelable & touching) {
            e.preventDefault();
          }
        });
      })();

      return {
        setOptions: setOptions,
        previous: function(){
          move('prev');
        },
        next: function(){
          move('next');
        },
        refresh: function() {
          calc(true, { animation: false });
        }
      };
    }
    $.fn.swiper = swiper;

});
})();

var SOURCES = window.TEXT_VARIABLES.sources;
window.Lazyload.js(SOURCES.jquery, function() {
$('.swiper-demo').swiper();
});
</script>

이 기능은, 나중에 관련 게시물들을 표시하는데에 사용할 예정입니다. 물론, 이것자체로는 안예쁘니 CSS를 좀 만져야 겠지만요.

# 요약

이번 포스트 에서는

-   테마를 포크해와서
-   Jekyll로 구성된 기본 페이지의 화면을 띄워보고
-   Jekyll과 TeXt 테마의 간단한 기본적인 기능에 대해서 알아보았습니다.

다음 포스트 에서는 **블로그 포스트 화면 내용의 문제점을 개선**해보는 내용을 다뤄보도록 하겠습니다.

끝까지 읽느라 수고하셨습니다.

<hr>
<h3>현재 진행상황</h3>
- [x] Github 블로그 첫걸음
  - [x] 화면을 띄우기(`본인계정ID`.github.io 접속 확인)
  - [x] `Jekyll` 기능 및 `TeXt` 자체 기능 탐방
- [ ] 블로그 포스트 내용이 의도한대로 나오게 하기
    - [ ] `Katex`를 활용한 수식 입력 기능
    - [ ] 잘려서 나오는 목차 목록, 안잘리게 하기
    - [ ] 코드 강조기능 커스텀하기
    - [ ] `github-pages` gem 설치해서, 빌드환경과 테스트 환경 같게 만들기
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
<div style="display:flex; flex-direction:column; align-items:center;"><progress value="2" max="16" style="width:80%"></progress><p>2/16</p></div>
<hr>

<h3>현재 블로그 상태</h3>
![기본 상태](/images/blogmaking/basicText.png)
아직 기본 테마에서 변경한 사항이 없다.
<hr>
<h3>현재 github.io 리포지토리 커밋들</h3>
![커밋 목록](/images/blogmaking/commit1.png)
불필요한 파일만 좀 정리한 모습이다. 나머진 원본에 있던 커밋들이다.
