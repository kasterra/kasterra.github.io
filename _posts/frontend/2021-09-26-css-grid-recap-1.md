---
title: CSS Grid 정리글 - ①
layout: post
subtitle: css에서 2차원 레이아웃을 담당하는 grid에 대해서 알아봅시다.
image: /images/thumbnails/CSSHTML.png
category: frontend
series: css Grid
---

# 들어가며 : CSS Grid가 쓰이는 이유

CSS Flexbox는 웹 페이지 레이아웃을 짜는데 분명 도움이 됩니다. 하지만, flexbox 만으로 모든 레이아웃 문제가 해결되지는 않습니다. 흔히 Flexbox의 가장 큰 한계점이 격자형(grid) 디자인을 만들기 어렵기에 CSS Grid가 탄생했다고 설명하는 경우가 많습니다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="rNwZazm" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/rNwZazm">
  flex cannot grid</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

대부분 첫번째 예제를 보여주면서, flexbox로는 격자형 디자인이 힘들기 떄문에 Grid가 등장했다 라고 설명하곤 합니다. 크게 틀린 설명은 아니지만, Flexbox로 이것저것 만들어 보려고 노력하신 분들이라면, 아래 처럼 격자형 디자인을 만들어 내본 경험이 있을듯 합니다. 일단 제가 그랬거든요.

Flexbox만으로 격자형태를 구성할 수는 있지만, CSS Grid가 격자 형태를 더 만들기 쉬운것은 맞습니다. Flexbox만을 이용해서 grid 형태를 만드려면, Flex container의 크기와, flexbox안에 들어갈 요소들의 크기를 생각해서 wrap 된 후의 결과를 대강 생각하면서 만들어야 합니다. 하지만 이제 소개할 grid를 사용하면, 그러한 계산을 거치지 않고 격자형 화면을 더욱 쉽게 구성할 수 있습니다.

# CSS Grid 시작하기

Grid는 간단히 설명하자면, 웹 페이지에서 컨텐츠들이 들어갈 공간을 그리기 위해서 표를 그려서 배치하는 레이아웃 입니다. 웹 페이지를 구성할 때, 헤더는 위에 있고, 주 내용은 가운데에, 사이드바는 왼쪽에... 아래와 같은 레이아웃을 만드는데에 적합한 css 레이아웃이라고 할 수 있겠습니다.
![grid example](https://upload.wikimedia.org/wikipedia/commons/3/3d/CSS_Grid_Holy_Grail_Layout.png)

<div style="display:flex; justify-content:center;"><strong>여러개의 열이 같은 높이를 가지는 holy grail 디자인. 오랜 기간동안 프론트엔드 엔지니어를 괴롭혔다고 한다.</strong></div>

Grid는 flexbox와 유사한 점이 많습니다. 레이아웃에 배치될 각각의 컨텐츠들(grid items)에게 보다는, 그 컨텐츠들을 담고 있는 부모(grid container)에게 들어가는 속성이 좀 더 많습니다.

Flexbox를 사용하려면 부모에게 `display:flex;`를 해줬던것과 같이, Grid를 사용하려면 부모에게 `display:grid;`라는 css 속성을 줘서 시작할 수 있습니다. Flexbox 처럼 `display:inline-grid;`라는 속성 또한 있는데, 이 역시 inline-flex와 flex사이의 관계와 동일합니다. 아이템 배치에는 크게 상관이 없고, 컨테이너가 주변 요소와 어떻게 어우러질지 결정하는 것이거든요.

# 행과 열로 시작하는 Grid

Flexbox를 처음 이해할 떄는 일상생활에서 잘 쓰지 않는 "메인 축"과 "교차 축"이라는 개념을 알 필요가 있었지만, grid는 일상생활에서 오피스 프로그램 등을 통해서 표를 만들어 보셨다면 쉽게 시작할 수 있습니다. 이제부터 알아볼 속성은 컨텐츠들이 배치될 레이아웃을 표처럼 그리는 방법에 대해서 하나하나 알아보겠습니다.

## 행의 개수와 크기 : grid-template-rows

아래아 한글이나 MS Office 등의 프로그램등을 통해서 표를 만들때에는 처음 행의 개수와 열의 개수를 물어봅니다. 그다음에, 각 행과 열의 크기를 마우스나 표 속성등을 이용해서 조정하지요. 반면, CSS Grid에서는 처음 표를 만들때 부터 각 행과 열의 크기를 명시하면서 표를 만듭니다. 행의 길이가 `100px`인 4개의 행을 가진 grid를 작성하려면 아래처럼 작성할 수 있습니다.

```css
display: grid;
grid-template-rows: 100px 100px 100px 100px;
```

아니면 좀 더 깔끔하게 CSS에 내장되어 있는 `repeat()`함수를 사용하여

```css
display: grid;
grid-template-rows: repeat(4, 100px); /*100px를 4번 반복하라*/
```

로도 작성할 수 있습니다.

## 열의 개수와 크기 : grid-template-columns

제대로 된 표라면, 행 또는 열만 있지는 않습니다. 한줄자리 표라도 행 한칸, 열 한칸은 있기 마련이니까요. `grid-template-rows`가 행을 담당하듯, `grid-template-columns`는 열을 담당합니다. 행의 길이가 `100px`이고, 열의 길이가 `60px`인 4행 3열짜리 grid 레이아웃은 아래처럼 작성하면 됩니다.

```css
display: grid;
grid-template-rows: repeat(4, 100px); /*100px를 4번 반복하라*/
grid-template-columns: repeat(3, 60px); /*60px를 3번 반복하라*/
```

## 새로운 단위 fr

CSS에서는 여러 단위를 쓸 수 있습니다. 가장 기본적이고 고정된 단위인 `px`부터, 부모의 크기를 기준으로 하는 `%`, `em`, `rem` 등이 있음을 알고 있을 것입니다. 이번에는 grid에서 유용하게 쓸 수 있는 단위인 `fr`에 대해서 알아보겠습니다.

`fr`은 비율, 분수 등을 나타내는 영단어 fraction에서 온 단위입니다. 전체 길이에서 특정한 "비율"을 차지한다는 뜻입니다. 예를 들어서

```css
grid-template-rows: repeat(3, 1fr);
```

은 행의 길이의 비율을 1:1:1 로 해서 grid container의 너비를 다 쓰겠다 라는 뜻이 됩니다. 이것 자체로는 `%`의 alias 정도인가 정도로밖에 다가오지 않지만, 사실 grid에서 fr이 진가를 발휘하는 분야는 따로 있습니다.

`%`는 **전체 길이**를 기준으로 하는 단위입니다. 행의 길이를 결정한다고 생각해 봅시다. grid container의 높이가 `500px`라고 했을 때,

```css
grid-template-rows: 50px 50% 50px;
```

라고 한다면, 실제로 나타내지는 행의 높이는

```
50px 250px 50px
```

와 같을 것입니다.

반면 `fr`은 **실제로 사용 가능한 길이**를 생각하는 단위입니다. 구체적인 예를 들어보자면,

```css
grid-template-rows: 50px 1fr 3fr 50px;
```

라고 한다면, 50px 두개가 먼저 공간을 차지하고, 사용 가능한 나머지 공간에 1fr과 3fr이 배치되어서, 실제로 나타내지는 행의 높이는

```
50px 100px 300px 50px
```

가 됩니다. `%`를 사용했다면, 할 수 없는 일이지요. 실제로 그렇게 되는지 codepen 예제를 통해서 살펴봅시다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="qBjJYam" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/qBjJYam">
  introducing fr</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

## Grid의 통제를 벗어날 때는? grid-auto-rows / grid-auto-columns

Grid 안에 들어갈 아이템들이 개수가 정해져 있을수도 있지만, 그렇지 않을 수도 있습니다. grid로 만들어진 칸보다 만든 컨텐츠가 많이 와버리면 어떻게 될까요? 행이나 열을 더 늘려서 내용물들을 채워넣는것이 일반적일 것입니다.

css Grid에서 이러한 현상을 제어하기 위해서 위에 제목에도 쓰여져 있는, `grid-auto-rows`, `grid-auto-columns`를 사용합니다. `grid-template-rows`와 `grid-template-columns`로 지정되어 있지 않은 칸의 행과 열의 크기를 결정합니다.

시각적으로 보이는것이 가장 이해하기 쉬운 방법이라 생각해 codepen을 첨부하겠습니다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="QWgZxdK" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/QWgZxdK">
  grid-auto cells</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

흰색 글씨로 쓰여진 숫자들은 `grid-template-rows` `grid-template-columns`로 지정된 칸에 배치된 원소들이고, 연두색으로 표시된 숫자들은 grid의 통제를 받지 않는지라 `grid-auto-rows`와 `grid-auto-columns`가 적용되어 작은 칸에 채워져있는 모습을 볼 수 있습니다.

## 어느 방향으로 넘쳐야해? grid-auto-flow

위에서 `grid-auto-rows`와 `grid-auto-columns`를 설명하면서, Grid의 통제가 벗어났을 때, 밑으로 칸을 늘려서 요소들을 배치하는 모습을 확인하였습니다. 밑으로 넘치는것도 그렇게 나쁘지는 않지만, 옆으로도 넘쳤으면 좋겠다는 생각을 할 수도 있지요. 그럴 때 사용하는 속성이 `grid-auto-flow`입니다.

위에서도 말했듯, 공간이 부족하면 행이나 열을 확장해서 배치를 한다고 하였죠? CSS도 그 상식에 벗어나게 행동하지 않습니다. `grid-auto-flow`는 `row`또는 `column`이라는 값을 취합니다. 기본값은 위에서 살펴봤듯이 행을 확장해서 쓰기 떄문에 `row`죠.

위에서 이미 기본값으로 `grid-auto-flow`가 `row`인 경우를 살펴보았기 떄문에, 위의 코드에서 `grid-auto-flow`를 `column`으로 바꾼 codepen을 첨부하겠습니다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="mdwzKXd" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/mdwzKXd">
  grid-auto cells</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

추가로 `grid-auto-flow`는 `dense`라는 속성을 가질 수 있는데, 이는 dense의 사전적 의미(밀집된) 대로, 넘친 요소들을 밀집되게 배치하려는 방법으로, grid에 빈 공간이 있다면, 그 공간을 우선적으로 넘친 요소한테 주는 방법이라고 할 수 있습니다. 그런데 당연한 얘기겠지만, 빈 공간을 괜히 만든것도 아닐거고, 그 빈 공간에 뒤에 온 요소가 앞으로 끼이고 하면, 우리가 HTML에 작성한 순서와 실제 웹 브라우저에 나오는 순서가 달라져서, 별로 쓰지 않는 속성인것 같습니다.

## `minmax()` 함수

CSS에 내장된 Grid용 함수입니다. 사용처는 `grid-template-rows`, `grid-template-columns`, `grid-auto-rows`, `grid-auto-columns`로 제한됩니다.

함수의 역할은 이름에서도 추론할 수 있듯, 최소/최대값을 설정하는 것입니다. 첫번째 매개변수로는 최소값, 두번째 값은 최대값 입니다. 최소값보다 크거나 같고, 최대값보다 작거나 같게 길이를 조정하는 역할을 담당하는 함수입니다. 당연한 이야기지만, 최대값이 최소값보다 더 작으면 제대로 작동을 하지 않습니다.

매개변수로 가능한 값으로는

- `px`,`em`등의 길이 값 (CSS 자료형 `<length>`)
- `%`값
- `fr`을 사용한 비율 값(얘는 최대값으로만 들어갈 수 있음)
- `max-content` : 컨텐츠가 구겨지지 않을 최소의 넓이.
- `min-content` : 컨텐츠가 유지될 최소의 너비
- `auto`(사실 `max-content`와 동치에요)

아래의 codepen을 통해서 minmax가 미적용/적용된 사례를 살펴봅시다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="mdwzjZq" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/mdwzjZq">
  about minmax</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

위의 그림에는 행의 길이가 `auto`로 적용되어서, 숫자만 딸랑 들어가있는 행들은 숫자 하나만 들어갈 높이만 가지고, 긴 문자열을 가진 행은 문자열을 넣기 충분할 정도로 크기를 키워놨습니다. 두번째 경우를 보면, 숫자만 들어가있는 행은 최대값이라고 적힌 `auto`를 적용했을 때, `50px`보다 크기가 작아지지만, `50px`보다는 같거나 커야하므로, 50px의 높이를 가지게 되었고, 긴 문자열을 가지고 있는 행은, `auto`를 적용했을 때, 최소값에 명시된 `50px`보다 길이가 더 길어지므로, `auto`가 적용된 모습입니다.

## `repeat()` 함수

위에서도 간단히 설명했지만, `repeat()`는 특정한 값을 여러번 반복 할 수 있는 함수입니다. 예상했을지도 모르겠지만, 첫번째 매개변수에는 반복할 횟수, 두번째 매개변수에는 반복할 값을 지정합니다.

첫쨰 매개변수에는 반복할 횟수를 지정하기 떄문에 숫자만 들어간다고 생각할지도 모르겠지만, 사실 다른 값들도 들어갈 수 있습니다. 바로 `auto-fill`과 `auto-fit`입니다. `fr`을 사용하지 않았을 때, grid container의 크기가 달라짐에 따라 행이나 열이 들어가지 않은 그런 공간이 남아있을 수 있겠죠? `auto-fill`과 `auto-fit`은 그 **남는 공간**을 어떻게 활용할지 결정하는 속성입니다.

`auto-fill`은 말 그대로, 남는 공간을 활용해서 최대한 많은 칸을 **채우려고** 하는 것이며, `auto-fit`은 남는 공간을 활용해서 있는 칸을 **맞추려고** 하는 것입니다. 이렇게 말로만 하면 사실 와닫지 않는것이 현실이기에 직접 codepen 예제를 살펴보면서 다루도록 하겠습니다.

<p class="codepen" data-height="300" data-default-tab="result" data-slug-hash="ExXdBar" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/ExXdBar">
  auto-fill vs auto-fit</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

이것으로 보면 `auto-fit`이 빈 공간을 알차게 활용한다는 사실은 알겠는데, `auto-fill`에 관해서는 약간 아리송 할 수 있습니다.
![auto-fill vs auto-fit](/images/frontend/auto-fill-vs-auto-fit.png)
위의 사진은 firefox 개발자 도구를 활용해서 grid 컨테이너가 실제 어떻게 구성되어 있는지 확인해 본 것입니다. auto-fill에 해당하는 컨테이너에 대해서는 아직은 채워져 있지 않지만, 칸이 확보 되어 있음을 알 수 있습니다. `minmax`를 이용해서 column의 넓이를 짜서, 직접 브라우저의 너비를 조정하면서 확인해 보세요.

[위 codepen 큰 페이지에서 보는 링크](https://codepen.io/kasterra-the-bashful/debug/ExXdBar)

너비가 좁을 때에는 둘다 똑같이 행동하다가, 너비가 어느정도 넓어지면, `auto-fill`은 칸이 늘어났다가 줄어들고(새로운 칸이 만들어 지므로) `auto-fit`은 새로운 칸을 만들어도 너비를 0으로 만들어 버리기 때문에 이미 있는 요소들이 쭉쭉 늘어남을 확인할 수 있습니다.

CSS 내장함수이긴 하지만, `repeat`함수는 Grid에 관한 일부 속성, 그 중에서도 아까 다룬 `grid-template-rows`와 `grid-template-columns`에서만 쓰일 수 있습니다.

## 간격 만들기 row-gap column-gap gap

말 그대로 grid에서 간격을 만드는 속성입니다. 이 속성을 적용하지 않고 grid를 짜면 서로 여유 없이 따닥따닥 붙어 있는데, `row-gap`이나 `column-gap`을 통해서 각각 행,열에 간격을 둘 수 있고, `gap`속성을 통해서 행,열 모두에 여유공간을 할당할 수도 있습니다.

# 마치며

Flexbox와는 다르게, grid는 속성이 좀 많아서 한 게시물로만 정리하기가 까다로운듯 합니다. 다음 게시물에는 이렇게 만들어진 grid에 요소들을 배치하는 요소들에 대해서 마저 알아보도록 하겠습니다. 끝까지 읽어주셔서 감사합니다.

# 참고한 글

- 웹 개발자의 영원한 친구 mdn
- <https://css-tricks.com/introduction-fr-css-unit/>
- <https://www.webdesignerdepot.com/2018/09/grid-vs-flexbox-which-should-you-choose/>
