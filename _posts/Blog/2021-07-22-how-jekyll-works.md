---
title: Jekyll이란 무엇이며 어떻게 동작하는가
tags: Jekyll
layout: post
subtitle: github.io 블로그를 만들때 쓰는 Jekyll. 정확히 어떻게 돌아가는지 알아봅시다.
image: /images/thumbnails/jekyll.png
category: blog
gists: True
---

# 들어가며

github.io 라는 주소를 가진 블로그를 만들고 싶다는 생각에, 인터넷의 여러 글들을 뒤적이면서, 얄팍한 웹 지식으로 블로그를 만들긴 했습니다.

하지만, 블로그를 만들고 시간이 약간 지나니, Jekyll의 힘으로 블로그를 돌리고 있는 것인데, Jekyll이 어떤 식으로 동작하는지 전혀 모르고 쓰고 있는 자신을 발견하게 되어서, 이번 기회에 정확한 코드의 동작은 모를지라도, 어떠어떠한 구조로 이 블로그가 표시되는구나 정도는 알고 싶어서, 이 글을 쓰게 되었습니다.

맨 처음 제 머리속에 있던 Jekyll은,

> 뭔지 잘 모르겠지만, 일단 `markdown`으로 글을 쓰면 그걸 페이지로 보여준대. 와! 너무 신기하다.

정도였지만, 지금 이 글을 쓰는 목적은 그 **"신기한"** 과정이 어떻게 돌아가는지 정도의 대략의 감을 잡고 싶었습니다.
더 나아가서, 정확히 뭔지는 모르지만, 정적 사이트 생성기라는 jekyll과 경쟁하는 다른 프로그램들과의 차별점 같은것도 대략적으로 알아보고 싶었던 마음도 있었습니다.

[Jekyll의 documentation 페이지](https://jekyllrb.com/philosophy/)에는 jekyll의 철학이 나와 있습니다. 대략적으로 요약하면 아래와 같습니다.

1. No magic
   - 사용자는 밑에서 돌아가는 프로세스에 대해서 쉽게 이해 할 수 있어야 한다.
2. It "just work"
   - 별다른 초기 설정 없이 그냥 돌아가야 한다.
3. Content is the king
   - 웹에서 퍼블리싱 하는것은 신경쓰지 말고, 컨텐츠에만 집중할 수 있게 하자.
4. Stability
   - 하위호환 보장 등등
5. Small & extensible
   - 90% 이상이 쓰는 기능만 기본으로 넣었고, 나머지는 플러그인으로 쓰게 하자.

1번에 No magic 이라고 써져있고, 밑에서 돌아가는 프로세스에 대해서 쉽게 이해할 수 있어야 한다지만, 공식 문서 페이지인 [https://jekyllrb.com/docs/](https://jekyllrb.com/docs/)에는, **사용방법**은 잘 기술해 놓았지만, `jekyll build`를 누르면 어떤 일이 생기는지는 적어놓지 않았더군요.

그래서 이제 No magic을 표방하는 jekyll이니만큼, 쉽게 이해할 수 있는 jekyll이 하는 일들에 대해서 알아봅시다.

# Jekyll이 하는 일

이 부분은 해당 주제로 구글링을 하다가 찾은 [좋은 medium 글(영문)](https://nielsdehoog.medium.com/how-jekyll-works-under-the-hood-bd5c819a5aa8)을 기초해서 작성 했습니다.

우리가 `jekyll build`를 콘솔창에 타이핑하면, 실행되는 루비 코드는 아래와 같다고 합니다.

아래의 코드는 [jekyll의 repo](https://github.com/jekyll/jekyll/blob/master/lib/jekyll/site.rb)에 있는 `site.rb`라는 파일의 일부분 입니다.

```ruby
    def initialize(config)
    # some ruby codes blah blah...

      reset
      setup

    # and another ruby codes blah blah...
    end

    # lots of stuffs.... Some Ruby codes blah blah....

    def process
      return profiler.profile_process if config["profile"]

      reset
      read
      generate
      render
      cleanup
      write
    end
```

참으로 간결합니다. 구조를 보여주기 위해서 다른 루비 코드들을 생략했지만, 대략적인 구조가 어떻게 돌아가는지는 이것으로 충분하다고 생각합니다.

`process` 부분을 보면 6단계로 jekyll의 작동이 구성됨을 한눈에 볼 수 있습니다.

## 1. Reset

이 단계에서는 캐시를 청소하고, 내부 변수들을 다시 초기화 합니다.

내부 변수라고 함은, 웹 사이트 내에서 쓰이는, 레이아웃, 카테고리, 포스트, 정적 파일 배열 등이 있겠습니다.

## 2. Read

이 단계에서는 우리 사이트를 구성하기 위한 요소들을 읽어서, Jekyll 내부에서 쓸 수 있는 자료구조로 변환하는 과정을 거칩니다.

우리의 웹사이트를 구성하는 주요 자료구조를 종류별로 구분한다면 아래와 같습니다.

### 문서(Document)

블로그 포스트는 문서(Document)로 표현됩니다.

### 페이지(Page)

우리의 웹사이트를 구성하는 각각의 웹페이지 입니다. 블로그 포스트가 **아닌** HTML 파일이나 markdown 파일은, front matter의 내용을 반영하여 페이지로 구성됩니다.

### 정적 파일(StaticFile)

front matter를 포함하지 않는 파일은 모두 정적파일로 분류됩니다.

정적파일은 따로 렌더링 엔진의 처리를 거치지 않고, 목적지 폴더로 복사됩니다.

## 3. Generate

우리의 웹사이트를 위한 자료들이 처리되고 생성된 후에, 우리는 generate로 진입하게 됩니다. 이 단계에서는 우리의 웹사이트를 위해서 미리 정의된 `generator`들을 거치면서, generate 를 하게 됩니다.

`generator`는 웹사이트에 추가적인 컨텐츠를 만들기 위해서 사용될 수 있는 객체입니다. `generator`의 한 예시로, [Flickr photostream에 있는 사진 각각으로 블로그 포스트를 만들어 주는 `generator`](https://github.com/lawmurray/indii-jekyll-flickr)가 있습니다. Jekyll에서는 기본값으로 지정된 generator가 없으므로, 이 단계는 여러분이 여러분의 사이트를 위해서 generator plugin(jekyll plugin)을 설정했을때에만 작동한다고 보면 되겠습니다.

## 4. Render

문서(document)와 페이지는 render 단계에서 전처리가 이루어 집니다. 전체 과정중에서 **핵심**과정이라고 할 수 있습니다. 렌더링 과정은 전체 문서와 페이지를 훑으며, 후술한 과정을 수행합니다.

### 1. Liquid 렌더링

Jekyll에서는 템플릿 엔진으로 Liquid를 채택했음을 알고 있을것 입니다. 이 단계에서는 템플릿 등에 있는 liquid 태그와 변수들을 파싱해서 결과물을 문서에 반영합니다. 예를 들어서

{%- raw -%}

```liquid
{{"# Title" | markdownify}}
```

{%endraw%}
는

```html
<h1>Title</h1>
```

의 형태로 렌더링 됩니다.

### 2. converter들로의 가공

이 과정에서 Markdown 등으로 쓴 문서의 내용을 HTML로 변환합니다. 이 과정에서 `converter plugin`을 사용할 수 있습니다.

이전 단계에서 렌더링 된 결과물들이 파일명 확장자에 알맞는 converter들을 거치면서 처리됩니다.

### 3. 출력물들을 layout에 배치

페이지나 문서에 `layout`에 관한 정보가 front matter등으로 명시되어 있다면, 해당 컨텐츠를 레이아웃 안에 렌더링 하는 과정을 거칩니다.

이 과정은 읽고 바로 이해가 되지 않을수도 있기에, 예시를 들겠습니다.

Hello World라는 포스트를 `_posts` 디렉토리 안에 적었다고 합시다. 해당 포스트에는 `layout: post`를 front matter에 명시해 두었습니다. 그러면 렌더러는 다음의 과정을 거칩니다.

1. `post.html`을 `_layout`디렉토리에서 불러옵니다.
2. `post.html`을 Hello world 포스트에 설정되어 있는 콘텐츠 매개변수에 맞게끔 Liquid로 렌더링을 합니다.
3. `post.html`이 layout 매개변수를 가지고 있는지 확인합니다. 만약 그렇다면, 해당 과정에서 얻은 출력을 새로운 과정의 입력으로 사용해서 1번부터 과정을 반복합니다.

## 5. Cleanup

이제 우리의 생성된 사이트는 디스크에 기록될 준비가 완료되었습니다. 하지만, 기록을 하기 전에, 목적지 디렉토리를 정리를 하는 과정이 필요합니다. 이미 존재하는 파일중에 현재 빌드 프로세스의 부분이 아닌 파일들은 이 단계에서 제거됩니다.

## 6. Write

생성된 사이트의 각각 파일은 목적지 디렉토리에 쓰여지기 시작합니다. 정적 파일들은 상술한대로, 단순히 복사만 됩니다. 디렉토리가 각 파일마다 필요하다면 이때 생성됩니다.

## 7. Print status

`--profile`옵션을 켜고 빌드할 경우에, 해당 단계에서는 liquid 렌더러에서 해당 파일을 렌더링 하기 위해서 걸렸던 렌더링 시간등을 기록한 표를 출력합니다. 아래는 이 블로그를 `--profile` 옵션을 켜고 빌드해본 결과입니다.

```
Filename                                  | Count |    Bytes |  Time
------------------------------------------+-------+----------+------
_layouts/post.html                        |    29 | 1904.25K | 0.517
feed.xml                                  |     1 |   53.38K | 0.123
_includes/footer.html                     |    48 |  105.19K | 0.096
_includes/head.html                       |    48 |  209.03K | 0.085
_layouts/category.html                    |    10 |  316.59K | 0.075
_includes/author.html                     |    29 |   34.35K | 0.052
_includes/header.html                     |    48 |   60.09K | 0.045
_layouts/compress.html                    |     9 |  239.06K | 0.030
_layouts/default.html                     |     7 |  135.45K | 0.030
_includes/minutes-to-read.html            |    29 |    0.14K | 0.029
_includes/series.html                     |    29 |   23.46K | 0.027
_includes/links.html                      |    48 |   35.81K | 0.024
_includes/menu.html                       |    48 |   23.06K | 0.023
_includes/recommendation.html             |    29 |   17.31K | 0.022
_layouts/minimal.html                     |     1 |   43.69K | 0.019
_layouts/home.html                        |     1 |  106.83K | 0.017
_layouts/tags.html                        |     1 |   24.94K | 0.017
_includes/share.html                      |    29 |   22.27K | 0.016
_includes/pagination-post.html            |    29 |    1.53K | 0.014
_includes/date.html                       |    38 |    1.50K | 0.012
_layouts/main.html                        |     1 |  125.89K | 0.005
_includes/stats.html                      |    48 |   18.52K | 0.004
_includes/svg-icons.html                  |    48 |  515.39K | 0.004
_includes/new-post-tag.html               |     9 |    0.35K | 0.004
_includes/search.html                     |    48 |   13.17K | 0.004
_layouts/search.html                      |     1 |    6.81K | 0.002
sitemap.xml                               |     1 |    7.38K | 0.002
_includes/toc.html                        |    29 |    1.16K | 0.002
_includes/time-bar.html                   |    29 |    7.94K | 0.002
_layouts/author.html                      |     1 |    0.68K | 0.002
_includes/extra-js.html                   |    48 |    0.00K | 0.001
_includes/extra-css.html                  |    48 |    0.00K | 0.001
_layouts/contact.html                     |     1 |    2.38K | 0.001
_includes/comments.html                   |    29 |    6.34K | 0.001
_includes/subscription.html               |    29 |    1.47K | 0.001
_includes/loader.html                     |     9 |   74.76K | 0.001
_posts/Blog/2021-06-25-blogMaking1.md     |     1 |   29.66K | 0.000
_includes/read-icon.html                  |     9 |    4.56K | 0.000
_posts/Blog/2021-07-03-blogmaking4.md     |     1 |   13.00K | 0.000
_layouts/page.html                        |     5 |    4.88K | 0.000
_posts/Blog/2021-07-01-blogMaking3.md     |     1 |   18.29K | 0.000
staff.html                                |     1 |    0.23K | 0.000
_posts/Blog/2021-06-27-blogMaking2.md     |     1 |   25.25K | 0.000
_layouts/404.html                         |     1 |    0.37K | 0.000
_layouts/message-sent.html                |     1 |    0.36K | 0.000
_posts/Blog/2021-07-14-move-blog-again.md |     1 |    5.05K | 0.000
_drafts/how-jekyll-works.md               |     1 |    8.20K | 0.000

```

# 마치며

이번 포스팅에서는 아무 생각 없이 사용했을 수도 있는 jekyll이 실제로 어떻게 돌아가는지 알아보았습니다. Jekyll의 표어중 하나였던 No magic 이라는 말 대로, 어떻게 jekyll이 돌아가는지 정도를 알아볼 수 있었습니다.

나중에 기회가 된다면 Jekyll 뿐만 아니라, 다른 경쟁 정적 사이트들과 jekyll의 비교를 하는 포스팅도 작성해볼까 합니다.

끝까지 읽어주셔서 정말로 감사합니다.
