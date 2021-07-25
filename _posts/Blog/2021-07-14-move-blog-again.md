---
title: 블로그 2차이사 기록
tags: 기타환경 블로그
katex: True
layout: post
subtitle: 다른 테마가 예뻐서 옮겨보려다가 실패한 기록입니다.
image: /images/thumbnails/jekyll.png
category: blog
---

# 시작하며

예전에 밑도끝도 없이 github pages 블로그를 `jekyll`로 만들어 보자 해서, `TeXt`라는 테마가 예쁘게 보여서 시작했다가, 나름 제 입맛에 맞게 꾸미려고 했던 노력들을 했고, 그 노력들을 이 블로그에도 담았습니다.

<a href="/blogMaking0/"><i class="fas fa-arrow-circle-right"></i> TeXt 블로그 만든 기록들</a>

다 만들고 가장 크게 느낀점은, 이렇게 난리를 떨었는데, 다른 테마를 적용한 블로그들이 몇몇개 예뻐 보여서, 심심풀이로 `_posts`폴더와 제 블로그의 `images`폴더만 옮겨서 대강 적용시켜 봤는데, 기존 블로그보다 훨씬 화사해서, 디자인 젬병인 내가 아무리 난리쳐봐야, 디자인 고수분들의 실력을 따라잡을 수 없을것만 같아 Jekyll 테마를 [jekyll-themes 사이트](https://jekyll-themes.com/)에서 찾아보기로 했습니다.

제가 만들었던 예전 기록을 봤으면 알겠지만, 저는 커버 이미지가 크게 나오는 카드 형식을 선호했기 때문에, card 카테고리를 가진것 위주로 살펴 보았다가, 처음 봤을 때 가장 마음에 들었던 테마는 Jasper 2 였습니다.

# Jasper 2

![](https://raw.githubusercontent.com/jekyllt/jasper2/master/assets/screenshot-desktop.jpg)
**[공식 repo](https://github.com/jekyllt/jasper2)에 있던 소개 사진**

`Jasper 2`는 Ghost라는 다른 블로깅 툴의 테마중 하나인 `Casper`를 포팅해온 것이라고 밝히고 있습니다. 일단 다른 블로그에서 가져온 것이라 디자인 자체의 미려함은 보장되었으나, 저의 블로그에 아쉽게도 맞지 않은 부분들이 있었습니다.

## 태그가 여러개가 되면 안예쁘다

Jasper 2의 나름의 상징성인 카드로 되어있는 대문 위에 있는 글자는, 태그입니다. 그런데, 이게 태그가 여러개가 되면, 여러줄로 나와서, 형태를 무너트립니다.
![태그가 여러개래요](/images/blogmaking/multiTag jasper.png)

## 한글 태그가 100% 기능을 발휘 못한다

그리고, 한글 태그가 보이는것은 제대로 보이나, 해당 태그를 가진 페이지들을 보는 화면으로 제대로 넘어가지 않았습니다. 정확한 원인은 아직도 정확히는 모르겠으나, 태그들을 만들 때, latin 문자들로 `slugify`하는것이 이와 관련이 있지 않을까 싶습니다.
{% raw %}

```html
<nav class="pagination" role="pagination">
    {% if paginator.previous_page %}
        {% if paginator.previous_page == 1 %}
            <a class="newer-posts" href="{{ site.baseurl }}tag/{{ page.tag | slugify: "latin" }}" title="Previous Page">&laquo; Newer Posts</a>
        {% else %}
            <a class="newer-posts" href="{{ site.baseurl }}tag/{{ page.tag | slugify: "latin" }}/page{{ paginator.previous_page }}/" title="Previous Page">&laquo; Newer Posts</a>
      {% endif %}
    {% endif %}
    <span class="page-number"> Page {{ paginator.page }} of {{ paginator.total_pages }} </span>
    {% if paginator.next_page %}
        <a class="older-posts" href="{{ site.baseurl }}tag/{{ page.tag | slugify: "latin" }}/page{{ paginator.next_page }}/" title="Next Page">Older Posts &raquo;</a>
    {% endif %}
</nav>
```

{% endraw %}
하지만 테마의 html 코드를 뒤적거리며 `slugify`를 하는 부분들을 하지 않도록 설정해 보았으나, 원하는대로 되지가 않더라고요. 여기서 저는 손을 땠습니다. 저는 jekyll 사용법은 예전에 블로그를 만든다고 허우적 거릴 때 배운걸로 간단한 것은 할줄 알지만, 루비 코드로 된 더 섬세한 기능들은 사용을 하지 못하거든요...

# sleek

![](https://github.com/janczizikow/sleek/raw/master/sleek.jpg)
sleek은 반면 여러 태그들을 적용하기에 좋은 테마였습니다. 카드 형식이고, 원한다면 여러 태그를 달아도 카드의 크기가 바뀌어서 보기 흉하게 된다거나 하는 일들이 없습니다. sleek을 적용한 블로그의 예는 [gwanwoodev 님의 블로그](https://gwanwoodev.github.io/) 입니다.

여러 태그들이 한 글에 존재하지만 심미성을 해치지 않고 있는것이 인상적입니다. 하지만 아쉽게도 더이상의 유지보수는 없다고 하여 장기적인 관점에서 별로라고 생각해서 다른 블로그를 찾아 나섰습니다.
![](/images/blogmaking/sleeknomore.png)

# jekflix - 지금 사용중!

jekflix는 이때까지 제가 본 `jekyll` 블로그 테마중에서 최고라고 자신할 수 있습니다.

-   넷플릭스를 그대로 옮겨온듯한 미려한 디자인
    -   대문짝만한 Hero 제공
    -   포스트를 읽을 때 영상을 보는것처럼 시간 진행 바를 제공
    -   포스트를 다 읽거나 나갈 때, 추천 포스트 제공
-   각 언어별 세팅을 할 수 있도록 번역 지원
-   너무나도 자세한 문서화. 해당 repo의 Github wiki에 찾아보면 다 나옴
-   카테고리 기능과 태그 기능을 동시에 지원하여, 본인의 게시글 분류 스타일에 최적화

등등의 사유로 사용하게 되었습니다. 현재 매우 만족하고 있습니다.
