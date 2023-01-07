---
layout: post
title: shape-outside와 clip-path와 함께하는 CSS shape
subtitle: 세모난 DIV? 그게 가능해?
category: frontend
image: /images/thumbnails/CSS.png
---

# 들어가며

요즘은 프론트엔드의 정수라고 할 수 있는 뷰(View)에 대한 관심이 다시 생겼습니다. 어떤 사이트를 들어가도 특정한 애니메이션 효과들을 보면서, "이건 어떻게 만들었지?" 하는 질문을 스스로에게 하면서, 직접 CSS 코드들을 inspect 하면서 CSS 감각을 되살리는 그런 나날을 보냈습니다.

그렇게 보던 중, 제 시선을 사로잡는 것이 있었는데, 바로 동그라미에서 쭉 펼쳐져서 하나의 DIV가 되는 그런 애니메이션 이었습니다. [이 블로그](https://johnny-mh.github.io/post/gatsby-%EB%B8%94%EB%A1%9C%EA%B7%B8%EC%97%90-%EA%B2%80%EC%83%89%EC%9D%84-%EB%B6%99%EC%97%AC%EB%B3%B4%EC%9E%90)에서 우측 상단에 있는 검색 버튼을 클릭하면 일반적으로 뜨는 그런 심심한 모달이 아닌, 동그라미에서 네모로 바뀌는 뭔가 동적인 그런 새로운 느낌을 주는 것이었기에, CSS 코드를 살펴보았고, `clip-path`라는 것을 사용해서 `transition`을 사용한 애니메이션임을 보게 되었습니다.

근데, 이 내용이 상당히 흥미로운 것들이 있고, 제가 잘 몰랐던 CSS 관련 사항들이 많았기에, 이렇게 블로그 글로 한번 정리를 해보려고 합니다. 본 포스트의 많은 사진자료는 [freeCodeCamp의 글](https://www.freecodecamp.org/news/css-shapes-explained-how-to-draw-a-circle-triangle-and-more-using-pure-css/)에서 가져왔음을 밝힙니다.

# 실제 계산되는 모습을 정의하는 `shape-outside`

일반적으로 HTML의 원소들은 네모네모하게 생겼습니다만, 이 속성을 사용하면 네모네모에서 벗어날 수 있습니다. `shape-outside`의 형식 문법(Formal syntax)는 아래와 같다고 합니다.

```
shape-outside =
  none                              |
  [ <basic-shape> || <shape-box> ]  |
  <image>

<shape-box> =
  <box>       |
  margin-box

<image> =
  <url>       |
  <gradient>

<box> =
  border-box   |
  padding-box  |
  content-box

<url> =
  url( <string> <url-modifier>* )  |
  src( <string> <url-modifier>* )
```

여기서 처음 보는게 `basic-shape`인데, 이 `basic-shape`를 활용해서, 사각형 뿐만 아닌, 원형, 타원형, 다각형, 그리고 path 까지 외형으로 그려낼 수 있습니다. 해당 속성을 이용해서 별도의 JS 없이도 글자들이 원형으로 밀려나는 등의 효과를 만들어 낼 수 있습니다! 아래처럼 글자가 밀려나는 효과를 `shape-outside`와 기본적인 CSS `margin`으로 나타낼 수 있습니다.

![image](https://www.freecodecamp.org/news/content/images/2020/01/circle2.png)

대강의 원리를 설명하자면 이런 느낌이지요

![image](https://developer.mozilla.org/en-US/docs/Web/CSS/basic-shape/shapes-reference-box.png)

연한 보라색이 원래 div의 사이즈 입니다. 그리고 진한 보라색 만큼이 `shape-outside`를 이용해서 원형으로 잘라낸 div가 되고, 여기에서 margin을 줘서, 청색 실선 만큼 차지하게 됩니다. 따라서 글자가 원형으로 밀려나는 것이지요.

# 보이는 모습을 정의하는 `clip-path`

그래픽 작업을 좀 해보신 분이라면 '마스크' 라는 개념을 아실것 입니다. 특정한 영역만큼만 보여주고, 나머지 부분은 잘라서 보이지 않게 하는 효과를 말하지요. `clip-path`는 요소에 마스크를 씌우는 기능을 합니다. `clip-path`가 마스크를 씌울 수 있는 방법은 이 역시 형식 문법을 참조하면 나름의 감각을 찾을 수 있습니다.

```
clip-path =
  <clip-source>                        |
  [ <basic-shape> || <geometry-box> ]  |
  none

<clip-source> =
  <url>

<geometry-box> =
  <shape-box>  |
  fill-box     |
  stroke-box   |
  view-box

<url> =
  url( <string> <url-modifier>* )  |
  src( <string> <url-modifier>* )

<shape-box> =
  <box>       |
  margin-box

<box> =
  border-box   |
  padding-box  |
  content-box
```

이 친구는 직접 체험할 수 있는 예시를 아래에 남겨두겠습니다. 마우스 클릭을 통해서 path들을 마음대로 조작할 수 있습니다.

<p class="codepen" data-height="1000" data-default-tab="result" data-slug-hash="abZxoOM" data-user="stoumann" style="height: 1000px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/stoumann/pen/abZxoOM">
  CSS clip-path Editor</a> by Mads Stoumann (<a href="https://codepen.io/stoumann">@stoumann</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

# 다양한 도형을 그릴 수 있는 CSS의 `<basic-shape>` 자료형

아까 두가지 CSS 속성을 소개하면서 `<basic-shape>`라는 것을 사용해서 여러 도형들을 그려낼 수 있다고만 하고 넘어갔습니다. 이번에 소개할 `<basic-shape>` 자료형은, 순수 CSS 문법으로 여러 도형들을 그릴 수 있는 자료형들입니다. 여기에서는 기본 도형 함수(basic shape function)들을 쓸 수 있고, 종류는 아래에 나열한 것과 같습니다.

- `inset`
- `circle`
- `ellipse`
- `polygon`
- `path`

이 CSS 함수들을 이용해서, 원래는 네모난 div의 크기 안에서 새로운 형태들을 그려낼 수 있습니다. 이제 자세히 알아봅시다.

## 사각형을 덧그리자! `inset`

`inset` 함수를 이용해서, 사각형을 그려낼 수 있습니다. `border` 속성의 shorthand로 값을 4가지 다가 아니라, 한개나 두개만 설정할 수 있듯이, `inset`또한 그러합니다.

```css
shape-outside: inset(20px); // 위, 아래, 왼쪽, 오른쪽 다 20px
shape-outside: inset(
  20px 5px 30px 10px
); //위 오른쪽 아래쪽 왼쪽 순서대로 20 5 30 10
```

그리고 `round`라는 구분자를 사용해서, `border-radius`처럼 곡률을 줄 수 있습니다.

```css
#square {
  float: left;
  width: 100px;
  height: 100px;
  clip-path: inset(20px 5px 30px 10px round 50px);
  background: lightblue;
}
```

이렇게 스타일시트를 작성하고 square 라는 `id`를 가진 HTML 요소를 생성해 보세요!

## 원을 덧그리자! `circle`

`inset`을 하고 50% 만큼 `round`를 줘서 원을 생성할 수도 있지만, 바로 원을 짠 하고 생성하는 방법이 바로 `circle` 입니다. 반지름을 함수에 제공해서 해당 반지름 크기만큼의 원을 그려낼 수 있습니다.

```css
#circle {
  float: left;
  width: 300px;
  height: 300px;
  margin: 20px;
  shape-outside: circle();
  clip-path: circle();
  background: lightblue;
}
```

여기서 따로 `circle` 안에 아무것도 넣지 않았는데, 이렇게 되면 `div` 만큼의 원(지름이 가로,세로와 같은)이 나오게 됩니다. 만약 특정한 크기를 원한다면 첫번째 파라미터로 필요한 만큼의 크기(예 : `50px` `10%` 등...)를 넣으면 됩니다.

`circle` 또한 두번째 파라미터를 받습니다. `at` 라는 키워드로, 원의 중심을 옮길 수 있습니다.

```css
#circle {
  float: left;
  width: 150px;
  height: 150px;
  margin: 20px;
  shape-outside: circle(50% at 30%);
  clip-path: circle(50% at 0%);
  background: lightblue;
}
```

![image](https://www.freecodecamp.org/news/content/images/2020/01/circle2.png)

## 타원을 덧그리자! `ellipse`

원을 그려내는 `circle`과 크게 다를 것 없지만, 이 친구는 두개의 파라미터를 받습니다. X 방향으로, Y 방향으로의 반지름 두개를 받지요.

```css
#ellipse {
  float: left;
  width: 150px;
  height: 150px;
  margin: 20px;
  shape-outside: ellipse(20% 50%);
  clip-path: ellipse(20% 50%);
  background: lightblue;
}
```

![image](https://www.freecodecamp.org/news/content/images/2020/01/ellipse.png)

## 다각형을 그리자! `polygon`

각 점 위치의 순서쌍들을 나열해서 다각형을 그려 낼 수 있습니다. 아래의 예시는 T 자 모양 DOM을 만들어 내는 예시입니다.

```css
#polygon {
  float: left;
  width: 150px;
  height: 150px;
  margin: 0 20px;
  shape-outside: polygon(
    0 0,
    100% 0,
    100% 20%,
    60% 20%,
    60% 100%,
    40% 100%,
    40% 20%,
    0 20%
  );
  clip-path: polygon(
    0 0,
    100% 0,
    100% 20%,
    60% 20%,
    60% 100%,
    40% 100%,
    40% 20%,
    0 20%
  );
  background: lightblue;
}
```

![image](https://www.freecodecamp.org/news/content/images/2020/01/polygon_t.png)

## SVG path를 써보자! `path`

SVG path에 대해서는 그 주제 하나만으로 글을 써도 될 정도로 내용이 상당히 방대합니다. 해당 내용을 전부 다루기에는 이 지면이 넘쳐 버리기 때문에 [mdn 링크](https://developer.mozilla.org/en-US/docs/Web/CSS/path)로 대체합니다.

## 번외 : 이미지 그자체 `images`

이미지 형태 그 자체를 DOM의 형태로 사용할 수 있습니다.

```html
<img src="src/moon.png" id="moon" />
```

```css
#moon {
  float: left;
  width: 150px;
  height: 150px;
  shape-outside: url("./src/moon.png");
}
```

결과
![image](https://www.freecodecamp.org/news/content/images/2020/01/moon2.png)

# 마치며

상당히 쌈박한 CSS 속성을 발견했습니다. 네모네모한 웹문서에서 벗어나서 뭔가 동글동글한 그러한 형태를 만들 수 있다는것은 정말 좋은 일이지 않을까요? 이러한 속성을 활용해서 [clip-path로 애니메이션 만들기](https://css-tricks.com/animating-with-clip-path/) 글을 읽으면 더욱 아름다운 웹 앱을 만드는데 많은 도움이 되지 않을까 싶습니다. 그럼 이만~!
