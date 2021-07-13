---
layout: post
title: 테마 의존적 기능 모듈화
excerpt: 블로그 이사를 위한 이사짐
category: BLOG
image: /images/thumbnails/jekyll.png
---
# 이 생각을 하게 된 계기
[7월 4일날 쓴 짧은 글](https://kasterra.github.io/2021/07/04/personal.html)을 읽으면 알겠지만, 2주동안 열심히 jekyll을 만졌지만, 이것보다 훨배는 더 예쁜 테마에다가 그냥 _posts 폴더만 이사해서 돌려보고 얻은 충격이 이만저만이 아니라, 여러 생각들을 했습니다.

한 생각중 하나는, `github-pages`로 호스팅 하는 `jekyll`블로그의 장점을 최대한 살리기 위해서 언제든 더 마음에 드는 멋진 테마를 발견한다면 글을 옮길 수 있도록, 테마 의존적 기능을 어느 테마에서나 쓸 수 있도록, 이사할 떄 챙길 짐처럼 미리 싸둬야 겠다는 생각이었습니다. 그래서 이제 짐을 싸둘려고 합니다.

블로그에 관한 정보를 알아볼 떄마다 점점 추가해 나갈 계획입니다.

# alert 기능 - `TeXt` 
`TeXt`를 골랐던 이유 중 하나입니다. 기존 블로그에서 작업을 했을때 중간에 강조하고 싶은 정보(주의 사항) 등이나, 좀 더 강조가 되었으면 좋겠는 정보들을 표현할 때는 인용구 문법(>)을 이용해서 했는데, alert는 4가지의 색을 따로 제공하여 조금 더 필요에 맞게 사용할 수 있는 점이 있어서 좋았습니다.

<i class="fas fa-info-circle"></i> <strong>2021년 7월 5일 추가</strong><br>
지금 이 게시물을 읽고 있다면, 이걸 그대로 가져가는 것도 그렇게 나쁜 판단은 아니지만, 더욱 범용적인 대안이 있음을 말씀드리고 싶습니다.
저같은 디자인이 젬병인 사람들을 위해서, 디자인을 잘하는 분들이 만든 [Bulma](https://bulma.io/)라는 서비스가 있습니다. 반응형이고, 모던한 자태를 뿜어내는 100% `css` 혹은 `sass` 파일들로 구성되어있는 디자인 파일 묶음입니다. 얼마나 많은 요소들을 담은지 직접 공식 문서를 통해서 확인을 해보셨으면 좋겠네요<br>
`TeXt`를 고른 이유였던 다양한 디자인 요소 존재가 필요없어 지는 순간이 와버렸군요. 기본 Bulma 테마가 마음에 안든다면, 마음에 드는 테마를 [여기서](https://jenil.github.io/bulmaswatch/) 고르면 됩니다.
{:.info}

**alert.scss** : `main.scss`등에 include 해서 사용하면 됨
```scss
p.success {
  padding: 0.5rem 1rem;
  background-color: rgba($green, 0.1);
  border: 1px solid $green;
  border-radius: 0.4rem;
}

p.info {
  padding: 0.5rem 1rem;
  background-color: rgba($blue, 0.1);
  border: 1px solid $blue;
  border-radius: 0.4rem;
}

p.warning {
  padding: 0.5rem 1rem;
  background-color: rgba($yellow, 0.1);
  border: 1px solid $yellow;
  border-radius: 0.4rem;
}

p.error {
  padding: 0.5rem 1rem;
  background-color: rgba($red, 0.1);
  border: 1px solid $red;
  border-radius: 0.4rem;
}
```

색을 나타내는 변수 등은, 블로그 테마를 보고 적당히 고르면 될 것 같습니다. 
# 아이콘들 - Font awesome
웹 프론트 작업을 약간이라도 해보셨으면 font-awesome의 아이콘이 꽤 쓸만함을 알 것입니다. 아래의 코드를 `head`에 삽입해 주면 됩니다.
```html
<script src="https://kit.fontawesome.com/42fb684910.js" crossorigin="anonymous"></script>
```

# Katex - Velog
흔히 jekyll테마 블로그에는 mathjax로 수식 렌더링을 도와주는 테마들이 있습니다만, 렌더링 속도가 `Katex`가 더 빠르다고 하여, 저는 쭉 `Katex`를 사용했습니다. 
이를 적용하는 방법은 아래와 같습니다.

**_config.yml**에 추가해야 할 내용
```yaml
kramdown:
  math_engine: katex
```

`include/head.html` 등에 추가할 내용
```html
  <!--KaTeX-->
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.12.0/dist/katex.min.css" integrity="sha384-AfEj0r4/OFrOo5t7NnNe46zW/tFgW6x/bCJG8FqQCEo3+Aro6EYUG4+cU+KJWu/X" crossorigin="anonymous">
  <script defer src="https://cdn.jsdelivr.net/npm/katex@0.12.0/dist/katex.min.js" integrity="sha384-g7c+Jr9ZivxKLnZTDUhnkOnsh30B4H0rpLUpJ4jAIKs4fnJI+sEnkvrMWph2EDg4" crossorigin="anonymous"></script>
  <script defer src="https://cdn.jsdelivr.net/npm/katex@0.12.0/dist/contrib/auto-render.min.js" integrity="sha384-mll67QQFJfxn0IYznZYonOWZ644AWYC+Pt2cHqMaRhXVrursRwvLnLaebdGIlYNa" crossorigin="anonymous"></script>
  <script>
      document.addEventListener("DOMContentLoaded", function() {
          renderMathInElement(document.body, {
              // ...options...
          });
      });
  </script>
```