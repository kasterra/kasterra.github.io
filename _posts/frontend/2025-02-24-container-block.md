---
title: CSS 레이아웃의 기준! 컨테이닝 블록
subtitle: 문제없이 퍼블리싱을 하기 위해서 알고 있어야 할 것들
layout: post
category: frontend
image: /images/thumbnails/CSS.png
---

# 들어가며

취준을 위해서 이것저것 들을 요즘 하고 있습니다. [실행 컨텍스트 글](https://kasterra.github.io/exec-context-1/)부터, [클로저 글](https://kasterra.github.io/closure-basics/), [그리고 최근에 작성했던 React 글](https://kasterra.github.io/react-knowledge-avail-in-interview-1/) 까지, 면접 대비를 위해 여러 개념들을 정리하고 있습니다.

인터넷에 올라와있는 여러 면접 예상 질문들과, 모의 면접 스터디를 통해서 제가 명확히 설명하지 못하는 그러한 지식들을 복기하면서, 면접에서 물어본다면 당황하지 않고, 자신있게 말할 수 있었으면 좋겠다는 소망을 가지고 이 글을 시작해 보고자 합니다.

이번 글에서는 마크업을 스타일링 할 때, 배치의 '기준'이 되는 여러 요소들과 그 요소들에서 비롯된 브라우저의 레이아웃 '특성'들에 대해서 알아보겠습니다.

# 컨테이닝 블록

CSS에서 크기를 정하는 단위 중 하나인 `%`를 통해서, '기준 요소'의 크기에 비례해서 설정할 수 있다는 사실을 알고 있을겁니다. 근데 이 '기준 요소' 라는 것이 정말로 계층상 바로 위에 있는 요소인 '부모'가 아니고, 정확히는 '컨테이닝 블록' 입니다.

그리고 이 컨테이닝 블록은 `position:fixed`나 `position:absolute`의 `top`, `bottom`,`left`,`right`등의 속성으로 위치를 지정할때의 기준점이 되기도 합니다.

## 컨테이닝 블록의 기준

mdn에 여러 기준들이 있지만, 실제로 자주 봐 왔던 상황 위주로 다시 정리해 보고자 합니다.

1. `position`이 `static`, `relative`, `sticky`중 하나이면, 가장 가까운 조상 블록 컨테이너(`inline-block`, `block` 등등) 또는 가장 가까우면서, 서식 맥락(Format Context)를 형성하는 조상 요소의 컨텐츠 영역 경계를 따라 형성
   - 가장 흔하게 보는, 가장 가까운 블록 요소인 부모
2. `position`이 `absolute`일 경우, 가장 가까운, **`position`이 `static`이 아닌 조상**의 내부 여백 영역
   - 퍼블리싱 입문 강의 등에서는 '가장 가까운 `relative` 조상'을 강조 하는 경우가 많지만, 엄연히, **`static`이 아닌 조상**임.
3. `position`속성이 `fixed`일 경우, 컨테이닝 블록은 뷰포트나 페이지 영역이 됨.
4. `position`속성이 `absolute`거나 `fixed`일 경우, `transform` 속성이 `none`이 아닌 가장 가까운 요소. 또는 `filter`속성이 `none`인 가장 가까운 요소
   - 애니메이션을 다룰 때 기존 알고 있던 기준과는 다르게 엉뚱한게 기준이 되는 것 처럼 보여서 '브라우저 버그인가' 하고 당황할 수 있는 부분

## 컨테이닝 블록을 찾아 크기 계산하기

[mdn 컨테이닝 블록 페이지](https://developer.mozilla.org/ko/docs/Web/CSS/CSS_display/Containing_block)에 있는 예제 문제를 풀어봅시다.

마크업은 공통의 형태로

```html
<body>
  <section>
    <p>문단입니다!</p>
  </section>
</body>
```

로 되어 있고, CSS만 바뀌는 식으로 문제 5개가 주어집니다. 렌더링 될 결과를 예측해보고 실제 렌더링 결과를 확인해 보세요.

Codepen을 사용해서 임베드 하였기 때문에 Edit on codepen 버튼을 클릭해서 실제 결과를 확인하면서 글을 읽으면 더욱 도움이 될 듯 합니다.

### 1번 문제

`p` 요소의 크기를 예측해 보세요

<div class="codepen" data-height="300" data-default-tab="css" data-slug-hash="ZYEOjZo" data-pen-title="Containing-block-eg-1" data-user="kasterra-the-bashful"  data-prefill='{"title":"Containing-block-eg-1","tags":[],"scripts":[],"stylesheets":[]}'>
  <pre data-lang="html">&lt;body>
  &lt;section>
    &lt;p>문단입니다!&lt;/p>
  &lt;/section>
&lt;/body></pre>
  <pre data-lang="css">body {
  background: beige;
}

section {
display: block;
width: 400px;
height: 160px;
background: lightgray;
}

p {
width: 50%;
height: 25%;
margin: 5%;
padding: 5%;
background: cyan;
}</pre></div>

<script async src="https://public.codepenassets.com/embed/index.js"></script>

`p` 요소에는 별도의 `position`과 `display` 속성이 지정되어 있지 않아, static block 요소라고 볼 수 있습니다. 가장 가까운 `block` 요소가 `<section>` 이므로, `p`의 크기의 기준은 `<section>`의 크기가 되어서, 다음과 같이 예측할 수 있습니다.

```css
p {
  width: 50%; /* == 400px * .5 = 200px */
  height: 25%; /* == 160px * .25 = 40px */
  margin: 5%; /* == 400px * .05 = 20px */
  padding: 5%; /* == 400px * .05 = 20px */
  background: cyan;
}
```

### 2번 문제

`p`의 `width`를 예상해보고, `height`가 50% 로 지정이 되면 얼마의 크기가 될 지 예상해 봅시다.

<div class="codepen" data-height="300" data-default-tab="css" data-slug-hash="EaxyeYG" data-pen-title="Containing-block-eg-2" data-user="kasterra-the-bashful"  data-prefill='{"title":"Containing-block-eg-2","tags":[],"scripts":[],"stylesheets":[]}'>
  <pre data-lang="html">
&lt;body>
  &lt;section>
    &lt;p>문단입니다!&lt;/p>
  &lt;/section>
&lt;/body>
</pre>
  <pre data-lang="css">body {
  background: beige;
}

section {
display: inline;
background: lightgray;
}

p {
width: 50%;
height: 200px;
background: cyan;
}

</pre></div>
<script async src="https://public.codepenassets.com/embed/index.js"></script>

`p`의 컨테이닝 블록은 바로 위 부모가 블록 요소가 아니기 때문에 `body`가 됩니다. body의 `width`는 기본값인 `auto`가 뷰포트 너비와 같고, `height`는 자식의 컨텐츠 크기만큼만 할당이 되기 때문에, 정답은 아래와 같습니다.

```css
p {
  width: 50%; /* == body 너비의 절반 */
  height: 200px; /* 참고: 백분율 값이었으면 0 */
  background: cyan;
}
```

[freecodecamp의 글](https://www.freecodecamp.org/korean/news/html-vs-body-jeonce-peiji-keugiyi-neobiwa-nopireul-seoljeonghaneun-bangbeob/)을 참조하면 아마 body의 `width`와 `height`에 대한 수수께끼를 푸는데에 도움이 조금이나마 될 듯 합니다.

### 3번 문제

`p` 요소의 `width`, `height`, `margin`, `padding`이 몇 px일지 예상 해 봅시다.

<div class="codepen" data-height="300" data-default-tab="css" data-slug-hash="pvobOWO" data-pen-title="Containing-block-eg-3" data-user="kasterra-the-bashful"  data-prefill='{"title":"Containing-block-eg-3","tags":[],"scripts":[],"stylesheets":[]}'>
  <pre data-lang="html">&lt;body>
  &lt;section>
    &lt;p>문단입니다!&lt;/p>
  &lt;/section>
&lt;/body>
</pre>
  <pre data-lang="css">body {
  background: beige;
}

section {
position: absolute;
left: 30px;
top: 30px;
width: 400px;
height: 160px;
padding: 30px 20px;
background: lightgray;
}

p {
position: absolute;
width: 50%;
height: 25%;
margin: 5%;
padding: 5%;
background: cyan;
}</pre></div>

<script async src="https://public.codepenassets.com/embed/index.js"></script>

`p`의 `position`이 `absolute`이고, 바로 위 부모인 `section`의 `position`이 `static`이 아닌 값이기 때문에, `p`의 컨테이닝 블록은 `section`이 됩니다. 따라서, % 단위로 표시한 단위들의 기준은 `section`의 크기를 기준으로 계산하면 됩니다.

```css
p {
  position: absolute;
  width: 50%; /* == (400px + 20px + 20px) * .5 = 220px */
  height: 25%; /* == (160px + 30px + 30px) * .25 = 55px */
  margin: 5%; /* == (400px + 20px + 20px) * .05 = 22px */
  padding: 5%; /* == (400px + 20px + 20px) * .05 = 22px */
  background: cyan;
}
```

추가로 이 예제를 통해 `position:absolute`에 대한 잘못 생각 할 수 있는 부분 또한 확인할 수 있습니다.

많은 경우에 `absolute`는 기준 요소에 대해서 `top`, `left`, `right`, `bottom`등의 속성을 통해서 배치를 하는 경우가 많지만, 이러한 위치값에 대한 속성이 명시가 되어 있지 않은 상태라면, 컨테이닝 블록을 기준하여 레이아웃이 잡힙니다. 이것을 직관적으로 알고 싶다면, 주어진 codepen의 css에서 p의 `top`속성 값을 `30px`만큼 줬다가 뺐다가를 해보면 나름 직관적으로 다가오지 않을까 하고 생각이 듭니다.

![top 30px 적용하고 top 60px 적용](/images/frontend/bfc-container-block-margin-merge/absolute-elem-flow.gif)

기존 코드에 top 30px를 적용했지만 결과물은 바뀌지 않았습니다. 혹시 갱신이 안된것이 아닐까 하고 생각할 수 있으니 top 60px를 적용했을 때는 확실히 변경이 됨을 알 수 있습니다.

컨테이닝 블록인 `<section>` 요소의 `padding-top`이 `30px` 만큼 적용이 되어 있으므로, 기본값인 `top:auto` 일 때와 `top:30px`으로 바뀌었을 때의 시각적으로 보이는 것이 같게 되는 것 이지요

### 4번 문제

`p` 요소의 `width`, `height`, `margin`, `padding`이 몇 px일지 예상 해 봅시다.

<div class="codepen" data-height="300" data-default-tab="css" data-slug-hash="ZYEOMMZ" data-pen-title="Containing-block-eg-4" data-user="kasterra-the-bashful"  data-prefill='{"title":"Containing-block-eg-4","tags":[],"scripts":[],"stylesheets":[]}'>
  <pre data-lang="html">&lt;body>
  &lt;section>
    &lt;p>문단입니다!&lt;/p>
  &lt;/section>
&lt;/body>
</pre>
  <pre data-lang="css">body {
  background: beige;
}

section {
width: 400px;
height: 480px;
margin: 30px;
padding: 15px;
background: lightgray;
}

p {
position: fixed;
width: 50%;
height: 50%;
margin: 5%;
padding: 5%;
background: cyan;
}

</pre></div>
<script async src="https://public.codepenassets.com/embed/index.js"></script>

`p`의 `position`이 `fixed`이기 때문에, 컨테이닝 블록은 뷰포트이고, 요소의 크기는 브라우저 크기에 따라서 달라지게 됩니다.

```css
p {
  position: fixed;
  width: 50%; /* == (50vw - (세로 스크롤바 너비)) */
  height: 50%; /* == (50vh - (가로 스크롤바 높이)) */
  margin: 5%; /* == (5vw - (세로 스크롤바 너비)) */
  padding: 5%; /* == (5vw - (세로 스크롤바 너비)) */
  background: cyan;
}
```

### 5번 문제

`p` 요소의 `width`, `height`, `margin`, `padding`이 몇 px일지 예상 해 봅시다.

<div class="codepen" data-height="300" data-default-tab="css" data-slug-hash="dPyXqBN" data-pen-title="Containing-block-eg-5" data-user="kasterra-the-bashful"  data-prefill='{"title":"Containing-block-eg-5","tags":[],"scripts":[],"stylesheets":[]}'>
  <pre data-lang="html">&lt;body>
  &lt;section>
    &lt;p>문단입니다!&lt;/p>
  &lt;/section>
&lt;/body>
</pre>
  <pre data-lang="css">body {
  background: beige;
}

section {
transform: rotate(0deg);
width: 400px;
height: 160px;
background: lightgray;
}

p {
position: absolute;
left: 80px;
top: 30px;
width: 50%;
height: 25%;
margin: 5%;
padding: 5%;
background: cyan;
}

</pre></div>
<script async src="https://public.codepenassets.com/embed/index.js"></script>

컨테이닝 블록의 기준을 설명하면서 언급한, `transform`이 들어갔을 때, 당황할 수 있는 부분 중 하나입니다. `<section>`의 `position`은 기본값인 `static`이지만, `transform`이 `none`이 아니기에, `<section>`이 컨테이닝 블럭이 되어서, 아래와 같은 결과가 나오게 됩니다.

```css
p {
  position: absolute;
  left: 80px;
  top: 30px;
  width: 50%; /* == 200px */
  height: 25%; /* == 40px */
  margin: 5%; /* == 20px */
  padding: 5%; /* == 20px */
  background: cyan;
}
```

### 번외 문제

`.fixed-box`는 분명히 `position:fixed`이지만 뷰포트에 대해서 고정되어 있지 않습니다. 왜 그럴까요?

<div class="codepen" data-height="300" data-default-tab="css,result" data-slug-hash="LEYZBpr" data-pen-title="Untitled" data-user="kasterra-the-bashful"  data-prefill='{"title":"position-fixed-what-the-heck","tags":[],"scripts":[],"stylesheets":[]}'>
  <pre data-lang="html">&lt;div class="container">
  &lt;div class="fixed-box">&lt;/div>
&lt;/div>

&lt;div id="container2">&lt;/div></pre>

  <pre data-lang="css">.container {
  width: 300px;
  height: 900px;
  background: lightgray;
  transform: scale(1); /* 컨테이닝 블록을 변경하는 속성 */
}

.fixed-box {
  position: fixed;
  top: 20px;
  left: 20px;
  width: 100px;
  height: 100px;
  background: red;
}

#container2{
  width:200px;
  height:500px;
  background:brown;
}</pre></div>
<script async src="https://public.codepenassets.com/embed/index.js"></script>

이 역시 `transfrom`이 적용되어 컨테이닝 블록이 일반적인 경우와 바뀐 경우입니다. 보통의 경우는 `position:fixed;`의 경우에는 뷰포트가 컨테이닝 블록이 되지만, `.container`에 `transform`이 적용되어 동작이 달라진 것이라고 볼 수 있습니다.

# 마치며

기술 면접 대비를 하면서, 사실 내가 정확하게 알지 못했던 것들을 제대로 된 지식으로 교정하는 좋은 경험을 하는 것 같습니다. 면접장에서 면접관들 표정이 이상해지는 것을 보는것 보다는, 집에서 혼자 당황하고 면접장에서는 개선된 모습을 보여주는 것이 더 낫지 않을까 싶네요.

적다보니 글이 상당히 길어졌지만, 끝까지 읽어 주셔서 참 감사합니다!!
