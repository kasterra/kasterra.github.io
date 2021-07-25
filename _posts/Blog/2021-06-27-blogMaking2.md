---
title: 블로그 옮기기 ②:포스트 창 개선하기
tags: 기타환경 블로그 Jekyll TeXt
katex: True
layout: post
subtitle: TeXt의 기본 테마의 포스팅 화면에서 아쉬운점을 개선해봅시다.
image: /images/thumbnails/blogmove2.png
series: 블로그 만들기
category: blog
---

# 이번 포스트의 목표

이번 포스트 에서는, 우리가 테마만 가져온 블로그 틀에 실제로 글을 하나 올려보고, 그 글이 나오는 형태들을 개선해보는 것이 목표입니다. CSS 파일들을 수정하는것이 이 포스트 내용의 대부분이 되지 않을까 싶습니다.

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

이번 포스트 에서는 두번째 대주제를 완성시킬 예정입니다.

# 블로그에 포스트 등록해보기

블로그에 포스트를 작성하는 방법은 이전 포스트에서도 말했듯, `_post`폴더에, `.md`나 `.html` 형식으로 글을 쓰면 됩니다. 예전에 `Velog`를 쓰면서 [마크다운과 Katex 수식으로 썼던 글](https://drive.google.com/file/d/19rLqs6GxSbNVfks9LCcxEkeu9_OmpP2z/view?usp=sharing)을 이 Jekyll 블로그에 올려서, `Velog`처럼 잘 나오는지 확인해 보고자 합니다. 이 글 내용은, 지금 이 블로그에서도 볼 수 있는, 유량그래프 관련한 글입니다. 원래 `velog`에서는 문제없이 보였던 글입니다. [원본 velog글 링크](https://velog.io/@kasterra/%EC%9C%A0%EB%9F%89-%EA%B7%B8%EB%9E%98%ED%94%84-%EC%97%90%EB%93%9C%EB%AA%AC%EB%93%9C-%EC%B9%B4%ED%94%84-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98)

아무튼 이제 이 글을 우리의 블로그에 올려보고, 이것이 잘 작동하는지 확인해 봅시다. 물론, 로컬에서 빌드 작업을 진행하겠습니다. 지금부터는 블로그의 여러 내용을 이것저것 수정해볼 것이기 떄문에, 로컬에서 작업을 다 하고, 확인이 된 후에 커밋을 하는쪽이 좀더 커밋 목록이 깔끔해 지는 방법이 아닐까 싶습니다.

우선, 제목을 `YYYY-MM-DD-제목.md` 형식에 맞추어 바꿔봅시다. 이 글을 작성한 시간은 2021년 6월 26일 이니, `2021-06-26-테스트.md`로 이름을 바꾸고, 이전 포스트의 머릿말 설명에 맞게, 적당히 한번 작성해 봅시다.

```markdown
---
title: 테스트용
layout: post
excerpt: 발췌문 텍스트
tags: 태그1 태그2 tag1 tag2
---
```

라는 내용을 글 맨앞에 추가해 줍시다.
<br>즉 아래와 같은 모습이 나오는 것입니다.

![포스트 하나](/images/blogmaking/testArticle1.png)

이제 빌드를 해서, 로컬에서 작동하는지 알아봅시다. jekyll을 실행할 환경이 지금 본인이 잡고 있는 PC라면,

```bash
$ bundle exec jekyll serve
```

를 실행해서, `localhost:4000`에 들어가면, 다음과 같은 화면을 볼 수 있을겁니다.
![데모 하나](/images/blogmaking/demo1.png)
우리가 머릿말에 넣은 정보들이 잘 나오는군요. 이제 들어가서 글이 잘 나오는지 확인해 볼까요?

# 문제 발견

![에러 하나](/images/blogmaking/error1.png)
![코드 강조](/images/blogmaking/code-highlight1.png)

문제가 생긴게 눈에 띕니다.

1. 우선 첫째로, 오른쪽에 있는 **목차 위젯이 내용이 잘려서** 나옵니다. 화면 크기를 늘려도 계속 잘려있는걸 보면, 보기에 불편한건 사실입니다.
2. 둘째로는 `$` 기호로 감싸서 작성한 `Katex` 문법으로 작성된 **수식이 렌더링 되지 않고**, 그냥 적은 그대로 나옵니다.
3. 코드 하이라이팅이 되긴 한데, 일반 글자보다 좀 작고, 코드가 눈에 확 들어오지 않아서, 눈이 좀 아픕니다.

이제 문제가 생긴것을 확인했으니, github의 레포지토리에 issue로 등록합시다.
![이슈 등록](/images/blogmaking/issue1.png)

# ① : 목차 위젯 안잘리게 하기

첫번째 문제는, 간단한 HTML과 CSS를 안다면, 어차피 정적 웹페이지는 HTML하고 CSS 규칙으로 렌더링 되는거니까, CSS를 좀 바꾸면 되지 않을까? 하는 생각이 듭니다.
<br><br> 브라우저의 개발자 도구를 사용해서, 우측 사이드의 HTML 코드를 살펴보고, 각 요소별로 스타일링이 어떻게 되있는가를 확인해 봅시다.

```html
<div class="col-aside d-print-none js-col-aside">
    <aside class="page__aside js-page-aside" style="left: 0px; top: 0px;">
        <div class="toc-aside js-toc-root">
            "대충 리스트 목록. 너무 길어져서 임의로 생략"
        </div>
    </aside>
</div>
```

맨 상위 `div`와 `aside`의 너비를 inspect 해보면, 제 화면 기준으로 220px로 되어있고, 이를 개발자 창에서 임의로 `auto`로 수정해본 결과, 글자가 잘리지 않고 잘 나오는 것을 확인했습니다.
![인라인수정](/images/blogmaking/inlineEdit1.png)
빨간색으로 강조 표시한 부분들을 보세요. 문제가 해결된것이 보이죠?
<br><br>
크롬의 개발자 도구는 참 편리합니다. 위의 이미지를 보면, 어떠한 스타일이, 어떠한 파일의 몇째줄에서 왔다는것을 보여주고 있습니다. 여기서 알 수 있는 정보는 `_page.scss`라는 파일의 27번째 줄에서 이 서식을 적용했다는 사실입니다. 그렇다면, 우리의 블로그 파일을 뒤져서 `_page.scss` 파일의 해당 내용을 찾으면 이 문제를 해결할 수 있겠네요! `visual studio code`의 편리한 기능을 한번 써봅시다. `Ctrl+p` 단축키를 vscode 상에서 입력하면, 열려있는 폴더 내에서 파일을 찾을 수 있는 기능이 있습니다.
![파일검색1](/images/blogmaking/filesearch1.png)
우리가 수정해야할 `_page.scss`에 도달했습니다. 이제, 브랜치를 만들어서, 파일을 수정하고, 이를 커밋해 봅시다.

```bash
$ git checkout -b 1-sidebar
```

`1-sidebar`라는 브랜치를 만들어 이동했습니다. 이제 코드의 내용을 바꿔봅시다.

```scss
.page__main {
    height: 100%;
    color: $text-color;
    .col-aside {
        display: none;
        & > aside {
            position: absolute;
            width: map-get($layout, aside-width);
            @include overflow(hidden);
        }
    }
}
중략... .has-aside {
    .col-aside {
        position: relative;
        display: block;
        width: map-get($layout, aside-width);
        & > aside {
            &.fixed {
                position: fixed;
                -webkit-font-smoothing: subpixel-antialiased;
            }
        }
        @include media-breakpoint-down(lg) {
            display: none;
        }
    }
}
```

이 두 부분의 width 부분을 auto로 교체해주면 됩니다.

이제 수정을 완료했으니 커밋을 해봅시다.

```bash
$ git add . # 변경 내용을 스테이징
$ git commit -m "fixed #1" #fixed #이슈번호 이런식으로 하면 github에서 이걸 인식한다.
$ git push --set-upstream origin 1-sidebar #따로 위에서 upstream을 지정하지 않아 지금 지정
```

이제 커밋이 되었고, 원격 저장소에도 이것이 반영되었습니다.

이제 master브랜치에 병합하면 자동적으로 issue가 닫히고, 이것이 project에도 반영됩니다.

이렇게 끝이날줄 알았는데, 빌드 실패가 떴습니다. 세상에! 로컬에서는 잘 됐는데, 이상하군요.
{:.error}

## 개인 삽질 : prettier가 저질러버린 오류

Github pages에서 출력한 오류를 읽어보니, `main.scss`에서 문제가 생겼답니다. ![빌드에러](/images/blogmaking/gherror1.png)
`_main.scss`가 문제가 생겼다던데, `_main.scss`는 건드린적이 없었지 않는가? `_main.scss`를 확인해보니, 얘는 여러 scss파일을을 import 하는 역할을 하고 있었고, `_page.scss`도 포함하고 있었습니다.

`prettier`가 저지른 짓은 이것이었습니다. `_page.scss`의 126번째 줄 내용입니다.

```scss
@include transform(translate(-map-get($layout, sidebar-width), 0));
```

`map-get`으로 받아온 값을 음수로 처리해서 사용하겠다는건데, 얘를 `-map-get($layout, sidebar-width),0);`으로 만들어 버린겁니다. 세상에.
빠르게 구글링을 해서 나온 결과인 <https://www.geeksforgeeks.org/sass-negative-variable-value/>를 참조해서,

```scss
@include transform(translate(-(map-get($layout, sidebar-width), 0)));
```

로 괄호로 감싸서 수정했더니 빌드가 이상없이 되는것을 확인했습니다.

로컬에서 되는지 확인을 분명히 하고 올리는데, 이상하게 로컬에선 돌아가는데 `github pages`환경에서는 에러가 떴네요.... 로컬에서는 되는데 깃허브 페이지에서 안되요라고 구글링을 해봤는데, 별 수익이 없었습니다.

## `github-pages` gem 설치

주변 사람들에게, 로컬에서는 잘 돌아갔는데, 빌드 환경에서 안돌아갔다. 어떻게 해야하느냐를 물어봤을 떄, 저는 너무나도 당연한 답변을 들었습니다.

테스트 해보는 로컬 환경이랑, 빌드 환경이랑 똑같이 맞춰야 되지 않겠니?
{:.info}

그래서 github pages blog 관련 검색어로 구글링을 해봤더니. `github pages`공식 문서에, `github-pages`라는 `gem`을 설치해서, 로컬에서 테스트를 할 수 있다고 하는 내용을 확인했습니다.
<https://github.com/github/pages-gem> 공식 리포지토리의 `readme.md`를 참고해서 설치를 진행해 봤습니다.

`Gemfile`을 실행해서 읽어보니,

```ruby
#gem 'github-pages', group: :jekyll_plugins
```

라고 주석이 쳐져있어서, 주석을 해제해 줬습니다.

```ruby
gem 'github-pages', group: :jekyll_plugins
```

그리고

```bash
$ gem update bundler # github-pages 리포지토리의 권장사항
$ bundler update
$ bundle install
```

을 실행하여, 다시 그 오류를 고의로 내서 같은 에러가 나오는지 확인했습니다.

```bash
$ bundle exec jekyll serve
<아무튼 에러 내용 생략>
/var/lib/gems/2.7.0/gems/jekyll-sass-converter-1.5.2/lib/jekyll/converters/scss.rb:123:in
`rescue in convert': (header-height: 5rem, header-height-sm: 3rem, content-max-width: 950px, sidebar-width: 250px, sidebar-header-height: 3rem, aside-width: 220px) isn't a valid CSS value. on line 127 (Jekyll::Converters::Scss::SyntaxError)
```

로 `github-pages`에서 봤던 그 오류를 그대로 재현한것을 확인했습니다. 이제, 우리의 로컬 환경은 github-pages와 거의 같다고 생각해도 되겠군요.
gemfile을 변경했으니, 이것도 커밋을 작성해서 날려줍시다.

```bash
$ git add .
$ git commit -m "installed github-pages gem"
$ git push
```

문제를 해결했으니 진행상황에도 반영해 봅시다.

<hr>
<h3>현재 진행상황</h3>
- [x] Github 블로그 첫걸음
  - [x] 화면을 띄우기(`본인계정ID`.github.io 접속 확인)
  - [x] `Jekyll` 기능 및 `TeXt` 자체 기능 탐방
- [ ] 블로그 포스트 내용이 의도한대로 나오게 하기
    - [ ] `Katex`를 활용한 수식 입력 기능
    - [x] 잘려서 나오는 목차 목록, 안잘리게 하기
    - [ ] 코드 강조기능 커스텀하기
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
<div style="display:flex; flex-direction:column; align-items:center;"><progress value="4" max="16" style="width:80%"></progress><p>4/16</p></div>
<hr>

# ② : `Katex` 수식 렌더링 되도록 하기.

`Katex`를 `Jekyll`로 만든 페이지에서 표시하고 싶어하는 사람이, 구글에 검색을 해보니 많아서 정말로 다행이었습니다.
<https://stackoverflow.com/questions/50890702/running-katex-on-jekyll>
나와있는대로 작업을 쭉 진행해 봅시다.

우선 issue를 해결할땐, 그 해당 issue 만을 해결하는 `branch`를 작성해서 작업했었죠?

```bash
$ git checkout -b 2-katex
```

`ctrl+p`를 눌러, `head.html`을 검색해서. 맨 끝부분에 위의 게시물의 첫번째 답변에 있는 코드를 그대로 복사-붙여넣기 합시다.

```html
생략....
<!--KaTeX-->
<link
    rel="stylesheet"
    href="https://cdn.jsdelivr.net/npm/katex@0.12.0/dist/katex.min.css"
    integrity="sha384-AfEj0r4/OFrOo5t7NnNe46zW/tFgW6x/bCJG8FqQCEo3+Aro6EYUG4+cU+KJWu/X"
    crossorigin="anonymous"
/>
<script
    defer
    src="https://cdn.jsdelivr.net/npm/katex@0.12.0/dist/katex.min.js"
    integrity="sha384-g7c+Jr9ZivxKLnZTDUhnkOnsh30B4H0rpLUpJ4jAIKs4fnJI+sEnkvrMWph2EDg4"
    crossorigin="anonymous"
></script>
<script
    defer
    src="https://cdn.jsdelivr.net/npm/katex@0.12.0/dist/contrib/auto-render.min.js"
    integrity="sha384-mll67QQFJfxn0IYznZYonOWZ644AWYC+Pt2cHqMaRhXVrursRwvLnLaebdGIlYNa"
    crossorigin="anonymous"
></script>
<script>
    document.addEventListener("DOMContentLoaded", function () {
        renderMathInElement(document.body, {
            // ...options...
        });
    });
</script>
```

그리고, `_config.yml` 맨 뒤에 이거를 붙여넣어 봅시다.

```yml
kramdown:
    math_engine: katex
```

이렇게 한 후에, 글의 머릿말에 `katex: True`를 추가하면 katex렌더링이 된다고 합니다.

그렇게 추가한 후 `bundle exec jekyll serve`를 실행해서 테스트 해봅시다.

<i class="fas fa-exclamation-triangle"></i> <strong>주의</strong><br>
몇몇 기술적 문제 때문에 `_config.yml`을 수정하는 사항은 기존에 실행했던 `jekyll serve`에 반영되지 않습니다. 즉, `jekyll serve`명령을 틀고 작업을 하면서 실시간으로 결과물을 확인하고 있었다면, `ctrl+c`를 눌러 명령을 잠시 껐다가, 다시 켜서 확인해 보셔야 제대로 적용된 모습을 볼 수 있습니다.
{:.warning}

딱히 에러는 생기지 않지만, 예전처럼 똑같이, `Katex`가 렌더링 되지 않았습니다. 그리고, 여전히 이상한 표도 보이고 개판 5분전인건 똑같군요.

이 문제는, `velog`에서는 `$`한개로만 감싸는 인라인 수식 기능을 지원하지만, 이 코드는 그렇지 않기 때문입니다. (참고자료 : <https://stackoverflow.com/questions/27375252/how-can-i-render-all-inline-formulas-in-with-katex>)

이 문제는 `$` 한개로 둘러싸인 인라인 수식을 `$`두개로 둘러싸인것으로 변경해서 해결하면 됩니다. 표가 나오는 문제는, `|`기호가 `markdown`내에서 표를 만드는데 쓰여서 생기는 문제고, `\|`이런식으로 escape 해주면 되는데, 이 작업은 정규식을 활용해서 저는 진행했습니다.

vscode의 치환 기능중에 정규식 사용 옵션을 이용해서 수식을 둘러싼`$` 를 `$$`로 변경하는 방법은

```
찾을 텍스트 : (\$[^$]+\$)
변경할 텍스트 : $$$1$$
```

를 활용해서 진행했습니다.작업이 번거롭다면 [이 링크](https://drive.google.com/file/d/1ysAbZt-yzgpiR4pE0vuwSm--6APu5ZJU/view?usp=sharing)에서 수정된 md파일을 받아가세요.

이제 다시 확인해보면,
![렌더링 성공](/images/blogmaking/katex1.png)

정상적으로 katex가 렌더링 됨을 확인할 수 있습니다. 에러 없이 빌드가 되니, 커밋을 작성하고, 원격에 올립시다.

![커밋 둘](/images/blogmaking/commit2.png)
성공적으로 반영되었습니다.

목표를 하나 더 성공적으로 완수했습니다.

<hr>
<h3>현재 진행상황</h3>
- [x] Github 블로그 첫걸음
  - [x] 화면을 띄우기(`본인계정ID`.github.io 접속 확인)
  - [x] `Jekyll` 기능 및 `TeXt` 자체 기능 탐방
- [ ] 블로그 포스트 내용이 의도한대로 나오게 하기
    - [x] `Katex`를 활용한 수식 입력 기능
    - [x] 잘려서 나오는 목차 목록, 안잘리게 하기
    - [ ] 코드 강조기능 커스텀하기
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
<div style="display:flex; flex-direction:column; align-items:center;"><progress value="5" max="16" style="width:80%"></progress><p>5/16</p></div>
<hr>

# ③ : 코드 하이라이팅 기능 가시성 개선하기

## 폰트 종류 및 크기 변경

현재 코드 하이라이팅이 되는것을 확인할 수 있으나, 코드가 나오는 공간의 글자 폰트가 좀 더 컸으면 좋겠고, 개인적으로 선호하는 폰트인 `IBM PLEX MONO`를 폰트로 쓰고 싶습니다. 한글은 `나눔고딕코딩`을 이용해서 출력하기를 원하고요.

이것 또한 css 파일이 어디있는지 확인하기 위해서, 개발자 도구를 사용해 봅시다.

<div style="display:flex; justify-content:center">
  <img src="/images/blogmaking/code-font-setting1.png" alt="코드 영역 폰트 설정 확인"/>
</div>
`<code>`에서 쓰이는 `font-size`와 `font-family`를 운좋게도 `main.css`에 있음을 한눈에 찾아냈습니다.

블로그의 파일들을 찾아보니, `main.css`는 없지만, `_main.scss`는 있어서 파일 내용을 확인해 보았습니다.
{%- raw -%}

```scss
@import "skins/{{ site.text_skin | default: site.data.variables.default.text_skin }}",
    // "skins/chocolate",
    // "skins/dark",
    // "skins/default",
    // "skins/forest",
    // "skins/ocean",
    // "skins/orange",
    "skins/highlight/{{ site.highlight_theme | default: site.data.variables.default.highlight_theme }}",
    // "skins/highlight/tomorrow",
    // "skins/highlight/tomorrow-night",
    // "skins/highlight/tomorrow-night-eighties",
    // "skins/highlight/tomorrow-night-blue",
    // "skins/highlight/tomorrow-night-bright",
    중략..... "common/reset", 하략.....;
```

{% endraw %}
웹 개발을 간단히 공부해봤으면 `reset.css`나 그 비슷한것의 존재를 알 것입니다. `reset.scss`을 먼저 찾아보고, 거기에 `code`에 관련된 옵션이 있는지 찾아봅시다.
**\_reset.scss** 발췌

```scss
pre,
code {
    font-family: map-get($base, font-family-code);
}

code {
    font-size: map-get($base, font-size-xs);
    line-height: map-get($base, line-height-sm);
}
```

135번째 줄부터의 내용인데, `map-get`을 이용해서 변수를 받아옴을 알 수 있습니다. 이 변수들이 있는곳은 `_variable.scss`입니다.  
**\_variable.scss** 발췌

```scss
$base: (
    font-family: (
        -apple-system,
        BlinkMacSystemFont,
        "Segoe UI",
        Helvetica,
        Arial,
        sans-serif,
    ),
    font-family-code: (
        Menlo,
        Monaco,
        Consolas,
        Andale Mono,
        lucida console,
        Courier New,
        monospace,
    ),
    font-size-root: 16px,
    font-size-root-sm: 14px,
    font-size-xl: 1.5rem,
    font-size-lg: 1.25rem,
    font-size: 1rem,
    font-size-sm: 0.85rem,
    font-size-xs: 0.7rem,
    하략....,
);
```

저는 `code`의 `font-size`가 본문의 크기와 같은 `1rem`이기를 바라고, `font-family-code`에 `Google Font`에서 가져온 웹폰트를 넣어보겠습니다.

**\_reset.scss** 수정본

```scss
pre,
code {
    font-family: map-get($base, font-family-code);
}

code {
    font-size: map-get($base, font-size);
    line-height: map-get($base, line-height-sm);
}
```

**\_variable.scss** 수정본

```scss
@import url("https://fonts.googleapis.com/css2?family=IBM+Plex+Mono&family=Nanum+Gothic+Coding&display=swap");
$base: (
    font-family: (
        -apple-system,
        BlinkMacSystemFont,
        "Segoe UI",
        Helvetica,
        Arial,
        sans-serif,
    ),
    font-family-code: (
        "IBM Plex Mono",
        "Nanum Gothic Coding",
        monospace,
    ),
    font-size-root: 16px,
    font-size-root-sm: 14px,
    font-size-xl: 1.5rem,
    font-size-lg: 1.25rem,
    font-size: 1rem,
    font-size-sm: 0.85rem,
    font-size-xs: 0.7rem,
    하략...,
);
```

<div style="display:flex; justify-content:center">
  <img src="/images/blogmaking/code-font-setting2.png" alt="코드 영역 폰트 설정 확인"/>
</div>
정상적으로 적용이 됨을 확인할 수 있습니다.
정상적으로 빌드가 됨을 확인하였으니, 커밋합시다.
```bash
$ git add .
$ git commit -m "changed code area's font and its size"
```

## 배경색 변경

배경색을 변경하는것은 별다른 scss 파일들을 헤집을 필요가 없습니다. `TeXt`테마의 문서를 봤으면 알겠지만, `TeXt`의 코드 강조 테마는 6가지의 색 테마를 지원합니다. 그중 본인이 선호하는것을 골라 `_config.yml`에 변경시키면 끝입니다.

수정 전 **\_config.yml**

```yaml
highlight_theme: default # "default" (default), "tomorrow", "tomorrow-night", "tomorrow-night-eighties", "tomorrow-night-blue", "tomorrow-night-bright"
```

이 줄을 수정하면 됩니다. 저는 `tomorrow-night-eighties` 로 하겠습니다.

수정 후 **\_config.yml**

```yaml
highlight_theme: tomorrow-night-eighties # "default" (default), "tomorrow", "tomorrow-night", "tomorrow-night-eighties", "tomorrow-night-blue", "tomorrow-night-bright"
```

<div style="display:flex; justify-content:center">
  <img src="/images/blogmaking/code-color-setting.png" alt="코드 영역 색 테마 설정 확인"/>
</div>

정상적으로 된것을 확인할 수 있습니다. 이 역시 커밋하고, 이슈를 닫으면 되겠습니다.

```bash
$ git commit -m "changed code area's background. closes #3"
```

목표를 또 하나 완수했습니다.

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

# 요약

이번 포스트 에서는 블로그 포스트 창의 내용을 우리의 입맛에 맞게 커스텀 해보는 시간을 가졌습니다. 이 블로그 글을 쓸 때, 함께 작업을 하고 있는듯한 느낌을 주기 위해서 노력을 했는데, 그렇게 잘 느껴졌는지 모르겠습니다. 이거 쓰는데 이틀이 걸릴줄은 몰랐습니다. 여하튼 끝까지 읽느라 고생하셨습니다. 혹시 질문이 생기시면 주저없이 댓글 달아주세요. 확인하는대로 답변 드리도록 노력하겠습니다.

<h3>현재 블로그 상태</h3>
![블로그 상태](/images/blogmaking/postComplete.png)
예전보다 좀더 예쁘게 수식도 잘 나오고, 코드도 좀더 확 띕니다.
<h3>현재 github.io 리포지토리 커밋들</h3>
![커밋 목록](/images/blogmaking/commit3.png)
혹시 글을 읽으시다가 궁금한 점이 있으시다면 주저없이 댓글 달아주세요.
