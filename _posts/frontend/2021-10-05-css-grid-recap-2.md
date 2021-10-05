---
title: CSS Grid 정리글 - ②
layout: post
subtitle: grid layout에 아이템들을 배치해 봅시다
image: /images/thumbnails/CSSHTML.png
category: frontend
series: css Grid
---

# 들어가며 : css grid에 실제로 요소를 배치하기

[저번 글](/css-grip-recap-1)에선 css grid 레이아웃을 어떻게 만드는지에 대해서 알아 보았습니다. 이번 글에서는 저번 글에서 예고한 대로, 그렇게 만들어진 레이아웃에 어떻게 요소를 배치하는지 알아보겠습니다.

기본적으로 별다른 옵션을 주지 않아도, 순서대로, 같은 크기로 순서대로 배치가 되긴 합니다. 하지만 그것만으로 충분하지 않을 때도 많지요. 헤더, 사이드바 nav, 본문, 그리고 footer 를 배치하는데, 순서대로 같은 크기로 배치하고 싶지는 않을듯 합니다. 4 x 4 크기의 grid 에서, 맨 윗줄은 header로, 맨 아랫줄은 footer, 중간 둘째줄의 맨 오른쪽 두칸은 nav... 이런식으로 하길 원할 수 있으니까요.

이번 포스팅에서는 그러한 순서대로가 아닌 레이아웃을 구성하는 법과, 할당된 칸 안에서 요소를 정렬하는 법까지 다루어 보겠습니다.

# 기본적인 아이템 배치 : `grid-row/column-start/end`

기본적으로 grid에 아이템을 배치하는것은 시작하는 행/열의 번호와, 끝나는 행/열의 번호를 지정하는 것으로 설명 할 수 있습니다. grid 레이아웃에서 행/열의 번호 지정은 아래 그림과 같이 이루어 집니다.

![grid 라인 넘버](/images/frontend/grid-line-number.png)

<div style="display:flex; justify-content:center;">grid 레이아웃의 행/열 라인 번호 매기는 법</div>

위의 그림은, 지난 포스팅에서 다루었던 예시 중 하나를 firefox의 개발자 모드를 이용해서, inspect를 해본 결과입니다. F12를 눌러서, 그리드 컨테이너 옆에 나오는 `grid` 버튼을 누르면 위와 같이 확인 할 수 있습니다.

![firefox 에서 그리드 inspect](/images/frontend/inspector-guide-grid.png)

핵심을 말해보자면 이렇습니다. grid container 에서 행/열의 번호는 각각 왼쪽과 위쪽에서부터 1번부터 차례 차례 올라갑니다. 상당히 직관적이지요. 아무튼 이 번호들로 아이템들을 배치하는 방법이, 제목에도 써놓은 `grid-rows-start`, `grid-rows-end`, `grid-columns-start`, `grid-columns-end` 입니다. 제목에 이걸 다 넣기에는 너무 길어질 것 같아서 임의로 줄였습니다.

문법의 사용법은, 각 css 속성의 이름에서 느껴지는 것과 동일합니다. 시작하는 행/열의 번호와, 끝나는 행/열 번호를 지정해서, 배치하면 됩니다. 서두에서 말했던, header,main,nav,footer를 4 x 4 grid에 직접 배치하는 예제를 살펴봅시다.

<p class="codepen" data-height="300" data-default-tab="result" data-slug-hash="GREaXpa" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/GREaXpa">
  grid item placement</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

여기서 grid에 배치하는 부분만 따서 보면

```css
.header {
  grid-row-start: 1;
  grid-row-end: 2;
  grid-column-start: 1;
  grid-column-end: 5;
}
.main {
  grid-row-start: 2;
  grid-row-end: 4;
  grid-column-start: 1;
  grid-column-end: 4;
}
.nav {
  grid-row-start: 2;
  grid-row-end: 4;
}
.footer {
  grid-column-start: 1;
  grid-column-end: 5;
}
```

입니다. `.nav`와 `.footer`의 사례를 보면 알 수 있듯이 반드시 4가지 속성 모두를 명시해서 할 필요는 없긴 합니다. start와 end를 따로 적지 않고, 하나로 적는 방법도 있습니다. 위의 css는 아래의 css와 기능상 동일합니다.

```css
.header {
  grid-row: 1/2;
  grid-column: 1/5;
}
.main {
  grid-row: 2/4;
  grid-column: 1/4;
}
.nav {
  grid-row: 2/4;
}
.footer {
  grid-column: 1/5;
}
```

이 방법은, 이해가 안되지는 않으나, 이것이 실제로 어떻게 그려질지는, 머리속이든, 실제로든 grid 레이아웃을 그린다음에 css 명세대로 그림을 그려봐야 하기 때문에 약간 비직관적입니다.

# 직관적인 배치 : `grid-template-areas` & `grid-area`

`grid-template-areas`는 정말로 직관적인 배치 방법을 제공합니다. 정말 css만 읽고, 어떤식으로 배치가 될지 바로 알 수 있거든요. 그리드 레이아웃의 ascii-art 라는 별명 또한 가지고 있습니다. 직접 `grid-template-areas`가 사용된 예를 보면, 공감할 수 있으리라 생각합니다.

```
$$$_____$$$$$$$_$$$$$$$_$$$_______$$$_$$$$$$$$$$
$$$____$$$____$$$____$$$_$$$_____$$$__$$$_______
$$$____$$$_____$_____$$$_$$$_____$$$__$$$_______
$$$_____$$$_________$$$___$$$___$$$___$$$$$$$$__
$$$______$$$_______$$$_____$$$_$$$____$$$_______
$$$_______$$$_____$$$______$$$_$$$____$$$_______
$$$$$$$$$___$$$$$$$_________$$$$$_____$$$$$$$$$$

```

<div style="display:flex; justify-content:center">ASCII 아트의 한 예(출처 : https://fsymbols.com/text-art/)</div>

위와 같은 레이아웃 배치를, `grid-template-areas`에서는 다음과 같이 할 수 있습니다.

```css
.container {
  /*grid 컨테이너!*/
  display: grid;
  /*행/열 그리는 과정 생략*/
  grid-template-areas:
    "header header header header"
    "main main main nav"
    "main main main nav"
    "footer footer footer footer";
}
```

별다른 지식 없이 보더라도, 해당 명세를 보고, "header는 맨 위에 배치되고, nav는 오른쪽 구석에..." 이런 느낌이 확 오지 않나요? 마치 아스키 아트로, 현실 세계의 그림을 표현한것과 같은 느낌이 들지요.

하지만, 이런 직관적인 `grid-template-areas`는 단독으로 사용할 수 없습니다. `grid-template-areas`에 쓰여있는 `header`라던지, `main` 이라는지 하는 것들이 어떤것인지 `grid-template-areas`한테 알려줄 필요가 있기 때문입니다. 단순히 class 이름과 같으니, 적당히 알아먹지 않을까 하고 코드를 작성하면, 원하는대로 정렬이 되지 않음을 직접 확인 할 수 있습니다.

`grid-template-areas`가 확인할 수 있게, 배치될 요소에 이름을 붙여주는 작업을 `grid-area`가 수행 합니다. `grid-area`에 지정해 줄 값은 **문자열이 아니**라는 점에 주목 하세요.

```css
.header {
  grid-area: header;
}
.main {
  grid-area: main;
}
/*이하 생략*/
```

해당 예제에셔는 css class 이름과 grid-area 이름을 같게 했지만, 이 둘 사이에는 어떠한 관련도 없기 떄문에, 아무 제한 없이 할 수 있다는 점을 꼭 알아두시기 바랍니다.

그리고, 기본적인 배치는, 위에서 설명한 행/열의 시작과 끝을 지정해주는 방식으로 되기 때문에, ㄱ자 배치 등은 불가능 하다는 점도 알아두셨으면 좋겠습니다.

# 아이템들을 정렬하기 : justify-items & align-items

이 부분 부터는 CSS Flexbox와 유사한 점이 꽤나 많습니다. 컨테이너에 적용을 하는 것으로, 컨테어너 안에 있는 아이템들을 일관성있게 정리하는 것들이거든요.

사용 가능한 속성을 정리해 보겠습니다.

- auto
  - 부모의 `justify-items`, `align-items` 값을 그대로 사용합니다.
- normal
  - grid에서는 하술할 `stretch`와 같습니다. flexbox에서는 무시되어 사용할 수 없는 속성입니다.
- **stretch**
  - 기본값. 아이템들의 크기를 늘려서, 칸의 크기에 맞춥니다.
- start
  - 컨테이너의 시작하는 모서리 부분에 맞추어 정렬합니다
- center
  - 컨테이너의 중간부분 모서리에 맞추어 정렬합니다
- end
  - 컨테이너의 끝나는 모서리 부분에 맞추어 정렬합니다

`justify-items`는 가로방향에 대해서, `algin-items`는 세로방향에 대해서 아이템들을 각 칸에 맞추어 정렬합니다. flexbox에서 `align-items`를 교차축 방향으로 아이템들을 정렬하는것을 생각하면 좋을 듯 합니다.

아래는 `justify-items` 그리고 `align-items`를 사용한 codepen 예제입니다. 각 제목에 붙인 속성 외에는 전부 기본값으로 설정되있다는 점을 착안해서 아래의 예제를 살펴 주시기 바랍니다.

<p class="codepen" data-height="300" data-default-tab="result" data-slug-hash="KKqjWqN" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/KKqjWqN">
  justify-items align-items</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

## 단축 속성 : place-items

위의 두가지 속성을 한줄에 적고 싶을 때 사용 할 수 있는 속성입니다.

```css
place-items: center start; /*justify-items align-items 순서대로 작성*/
```

# 개별 아이템에 대해서 정렬 : justify-self & align-self

flexbox에서도 `align-self`가 있었습니다. 특정한 아이템 하나만 콕 집어서, `align-items`를 따로 해주는듯한 효과를 내는 그것 맞습니다. 특정 아이템 하나에 대해서 따로 `justify-item`이나 `align-item`을 해 주고 싶을때 사용할 수 있는 속성이며, 사용 가능한 값도, `justify-items`나 `align-items`와 같습니다.

## 단축 속성 : place-self

`justify-self`와 `align-self`를 동시에 한줄로 쓸 수 있는 속성입니다.

```css
place-self: center start; /*justify-self align-self 순서대로 작성*/
```

# 아이템 그룹 정렬 : justify-content & align-content

css grid에 적용되는 이 두 속성을 쉽게 이해하기 위해서는, flexbox에서 사용되었던 **`align-content`**를 잠시 떠올려보면 도움이 될 것 같습니다. flexbox에서 wrap이 발생 했을 때, wrap 된 줄을 조정하는 속성이었습니다. grid layout 에서 `justify-content`와 `align-content`는 grid의 아이템들의 너비와 높이의 합이 컨테이너의 너비와 높이보다 작을 때, 어떻게 아이템 그룹들을 정렬할지를 다루는 속성 입니다. 그러니까 **컨테이너 안에 있는 grid 자체를 옮긴다** 라고 생각하면 편합니다.

사용 가능한 속성은 아래와 같습니다.

- stretch
  - 아이템들의 너비/높이를 칸에 맞춰서 늘립니다.
- start
  - 아이템 그룹들을 `justify-content`면 왼쪽 정렬, `align-content`면 위쪽 정렬합니다.
- center
  - 아이템 그룹들을 가운데 정렬합니다.
- end
  - 아이템 그룹들을 `justify-content`면 오른쪽 정렬, `align-content`면 아랫쪽 정렬합니다.
- space-between
  - 아이템들의 사이(between)에 균일한 간격을 넣어서 정렬합니다. flexbox 때 다루었던 그것과 동일합니다.
- space-around
  - 아이템들의 둘레(around)에 균일한 간격을 넣어서 정렬합니다. flexbox 떄 다루었던 그것과 동일합니다.
- space-evenly
  - 아이템들의 양 끝에 완전히 동일한(evenly)간격으로 정렬합니다. flexbox 때 다루었던 그것과 동일합니다.

아래는 `justify-content` 그리고 `align-content`를 사용한 codepen 예제입니다. 각 제목에 붙인 속성 외에는 기본값으로 설정했다는 점을 착안하여 확인해 주시기 바랍니다.

<p class="codepen" data-height="300" data-default-tab="result" data-slug-hash="jOwjmzK" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/jOwjmzK">
  justify-items align-items</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

실제 예제로 봤을 때, grid 자체가 옮겨지는 느낌이 들지 않나요? 확실하게 확인하고 싶다면, 이 [링크](https://cdpn.io/kasterra-the-bashful/debug/jOwjmzK/PNAvYXVxWGOr)로 들어가서 개발자 모드를 활용해서 inspect 를 해보는것도 그렇게 나쁘지는 않을 듯 합니다.

## 단축속성 : place-content

`justify-content`와 `align-content`를 한줄에 쓸 수 있는 속성입니다.

```css
place-content: center start; /*justify-content align-content 순서대로 작성*/
```

# 시각적인 배치 순서 : order

HTML을 변경하지 **않고** 시각적인 배열 순서를 결정할 수 있습니다. 숫자값이 들어가며, 낮은 숫자일수록 먼저 배치됩니다. flexbox의 그것과 동일하게, 웹 접근성 측면에서 좋다고 말하기 어려운것은 매한가지 입니다. `order:정수`의 형태로 사용합니다.

# z축 정렬 : z-index

`position`에서의 `z-index`와 동일하게 z 축 정렬또한 할 수 있습니다.

# 마무리하며

이상 두개의 글에 걸쳐서, CSS Grid 레이아웃에 대해서 알아보았습니다. 레이아웃을 단순히 아는 것으로 예쁜 디자인이 나오지는 않지만, 본인이 원하는 디자인을 만드는데에, 필요한 지식이 되었으면 좋겠다는 마음에서, 이렇게 글을 마무리 해보고 싶었습니다.

끝까지 읽어주셔서 감사하고, 혹시나 잘못된 부분이나 궁금한 점이 있다면 댓글로 남겨주시면, 빠른 시일내에 답변 드리겠습니다.
