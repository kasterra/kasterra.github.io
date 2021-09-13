---
title: CSS background 정리글
layout: post
subtitle: 배경색을 칠하는 여러 방법에 대해서 정리하였습니다.
image: /images/thumbnails/CSS.png
category: frontend
---

# 들어가며

HTML(또는 pug)과 CSS(또는 scss)만으로 클론 코딩 연습을 하다가, 아래와 같은 그라데이션 효과를 적용한 div를 만들어야 할 일이 생겼습니다.

<div style="display:flex;justify-content:center;aligin-items:center">
<div style="background: linear-gradient(
      to bottom,
      rgba(0, 0, 0, 0.3) 2%,
      transparent,
      transparent,
      transparent,
      transparent,
      transparent
    )
    #e7473c; width:200px; height:200px"></div>
</div>

여러 자료를 뒤져보면서 linear-gradient와 색 하나를 다른 레이어에 얹어서 하는 방식으로 작성을 할 수 있다는것을 알아냈지만, 이왕 알아본 김에 확실하게 관련 개념들을 정리를 해보고자 이 글을 쓰고자 합니다.

# 단색 배경 : background-color

`background-color` 속성은 CSS `<color>` 자료형을 사용하여 **단색** 배경을 지정하는데에 사용합니다. 자식 요소 등이 다른 `background-color` 속성을 명시하고 있지 않은 한, 자식 요소에도 적용이 됩니다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="OJgbEmB" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/OJgbEmB">
  background-color</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

<i class="fas fa-exclamation-triangle"></i> 주의 : `background-color` 에서는 여러 색을 쓸 수 없습니다!<br>
혹시 색을 반반 쪼개서 넣는 등의 그런 효과들을 원한다면 후에 설명할 `background-image`부분을 참고 바랍니다.
{:.info}

## `<color>` 자료형

CSS에서 사용하는 단색을 나타내는 자료형 입니다. 사용 가능한 방법에는 다음과 같은것이 있습니다. 사실 `background-color` 속성을 사용해 본적이 있으면 친숙한 형태들도 많을듯 합니다.

- 키워드 사용(blue, transparent 등등) [w3링크](https://www.w3.org/TR/css-color-3/) [나무위키 링크](https://namu.wiki/w/%ED%97%A5%EC%8A%A4%20%EC%BD%94%EB%93%9C?from=CSS%20%EC%83%89%EC%83%81%EB%AA%85#s-5.1)
- RGB 좌표계 사용 : #16진수, `rgb()`, `rgba()`를 사용해서 색을 나타냅니다.
- HSL(HSV) 좌표계 이용 : `hsl()` `hsla()`를 사용해서 나타낼 수 있습니다. [HSL(HSV)좌표계 : 위키백과](https://ko.wikipedia.org/wiki/HSV_%EC%83%89_%EA%B3%B5%EA%B0%84)
- LAB 좌표계 이용 : `lab()`을 사용해서 나타낼 수 있습니다.[LAB좌표계 : 위키백과](https://ko.wikipedia.org/wiki/CIELAB_%EC%83%89_%EA%B3%B5%EA%B0%84)
- `color()` 함수를 사용해서 미리 정의된 색 공간 또는, 사용자 정의 색 공간을 사용할 수 있다고 합니다.

# 이미지 배경 : background-image

`background-image`는 CSS에서 `<image>`자료형을 사용해서 배경으로 지정할 수 있는 문법입니다. `<image>`자료형은 다음과 같이 정리할 수 있습니다.

- 원래 크기를 가진 이미지 : jpg,png 등의 [래스터 포맷](https://ko.wikipedia.org/wiki/%EB%9E%98%EC%8A%A4%ED%84%B0_%EA%B7%B8%EB%9E%98%ED%94%BD%EC%8A%A4)
- 여러개의 크기를 가진 이미지 : 하나의 파일에 이미지의 여러 버전을 가지고 있는 ico 파일 등
- 원래 크기는 없지만, 종횡비는 가지고 있는 파일 : svg등의 [벡터 포맷](https://ko.wikipedia.org/wiki/%EB%B2%A1%ED%84%B0_%EA%B7%B8%EB%9E%98%ED%94%BD%EC%8A%A4)
- 원래 크기도 없고, 종횡비도 없는 경우 : CSS 그라디언트 등

박스에 배경 이미지를 적용한다고 할 때, 박스의 크기에 정확히 맞는 이미지만을 사용하지 않을 수도 있습니다. 이럴 경우, 큰 이미지의 경우에는 축소되지 않으므로 이미지의 일부만 표시가 되고, 작은 이미지의 경우는, 박스를 채우기 위해서 바둑판 식으로 배열이 됩니다.

## 배경 이미지 반복 제어

박스 보다 작은 이미지를 사용하면서, 기본적으로 적용되는 이미지 반복 속성을 원치 않을 수도 있습니다. 이럴 경우에는 `background-repeat` 속성에 다음 값들을 사용해서 배경 이미지 반복을 제어할 수 있습니다.

- no-repeat : 배경이 반복되지 않도록
- repeat-x : 수평 방향으로만 반복
- repeat-y : 수직 방향으로만 반복
- repeat : **기본 값** 수직, 수평 양 방향으로 반복

## 배경 이미지 크기 조정

박스 보다 큰 이미지를 사용할 때, 이미지 크기를 좀 줄여서 사용하고 싶을 수도 있습니다. `background-size` 속성을 통해서 배경 이미지의 크기를 조정할 수 있습니다.

- 키워드 값
  - cover : 이미지가 찌그러지지 않는 한도에서 제일 크게 설정. 종횡비가 맞지 않는다면 잘라서 넣는다.
  - contain : 이미지가 잘리거나 찌그러지지 않는 한도에서 제일 크게 설정
- 단일 값 : 이미지 너비를 지정. 이 경우, 높이는 auto가 됨
  - 예시 : background-size : 50% 등
- 두개 값 : 첫번째 값은 이미지 너비, 두번쨰 값은 이미지 높이 **콤마를 넣지 않는다**

<i class="fas fa-question-circle"></i> `background-size`값에 콤마를 넣으면 안되는 이유?<br>
`background-color`를 제외한 배경 속성에서는, 배경을 여러개 넣을 수 있습니다. 배경 이미지를 여러개 넣었을 때, 각각의 크기를 명시하기 위해서, 콤마로 구분을 해서 사용합니다.
{:.info}

## 배경의 시작점 지정 `background-origin`

`background-origin` 속성을 이용해서, 배경의 시작점을 지정할 수 있습니다. 좌표계의 원점을 설정한다고 생각해도 되겠네요.

`background-origin` 속성에서는 3가지 값을 사용해서, 배경을 배치할 수 있습니다.

- border-box : 배경을 테두리를 포함한 박스에 배치합니다.
- padding-box : 배경을 padding등을 제외한 안쪽 여백 박스에 배치합니다.
- content-box : 배경을 콘텐츠 박스에 상대적으로 배치합니다.

## 배경 이미지 배치 `background-position`

`background-position` 속성을 사용해서, 배경 이미지의 초기 위치를 선정할 수 있습니다. 이 속성값에는 값으로 1개에서 4개까지의 값이 올 수 있습니다.

- 1개 값을 사용할 때 :
  - `top`, `bottom`, `left`, `right`, `center` 등의 키워드 : 해당 키워드 방향으로 정렬합니다. 다른 축(left, right 경우에는 x축, top, bottom 경우에는 y축)에 대해서는 한 가운데로 정렬됩니다. center는 x축, y축 기준으로 모두 가운데에 정렬됩니다.
  - `<length>`또는 `<percentage>` 값 : 해당 값만큼 왼쪽 끝 모서리를 기준으로 X축을 기준으로 옮깁니다. Y축 좌표는 50%로 설정됩니다.
- 2개 값을 사용할 때 : 한 값은 X좌표를 다른 한 값은 Y좌표를 의미합니다.
  - 값 중에 하나가 `top`, `bottom`, `left`, `right`일 경우에 다른 한 값이 `left`나 `right`라면 이 값은 X축에 대한 정보를 제공한 것이니, 다른 한 값은 Y축에 대한 정보를 제공한것이 되고, `top`이나 `bottom`이라면 그와 반대로 해석됩니다. (ex : top left)
  - 값 중에 하나가 `<length>`나 `<percentage>`값을 제공하고, 다른 값이 `left`나 `right`라면, 해당 값은 맨 위 변을 기준으로 Y축만큼의 방향을 제공하는 것이고, `top`이나 `bottom`이라면 해당 값은 왼쪽 변을 기준으로 X 축 만큼의 방향을 제공하는 것이 됩니다. 만일 둘다 `<length>` `<percentage>`값을 제공한다면, 순서대로 X Y 좌표입니다.
  - 당연한 이야기이지만, 값들 중 하나가 `top`이나 `bottom`이면, 나머지 값은 `top`이나 `bottom`이 되서는 안되고, `left`와 `right`에 관해서도 그러합니다. 그러므로 top top이나 left right 같은 속성은 유효하지 않습니다.
  - `background-position`의 기본값은 top left 또는 0% 0% 입니다.
- 3개 값을 사용할 때 : 앞의 값 두개는 키워드이고 세번째 값은 앞의 두개의 값의 오프셋(offset) 입니다.
  - 첫번째 값은 `top`, `left`, `bottom`, `right`, `center`로 올 수 있습니다. 당연하겠지만, `left`나 `right`면 나머지 값은 X축에 대한 정보... 뭐 이런거 이제 생략하겠습니다.
  - `<length>`나 `<percent>`값. 두번째나 세번째에 올 수 있으며, 직전에 온 값에 대한 프리셋으로 작용합니다. 세번째에 오면 두번째 값에 대해서... 뭐 그런식이죠
    - 이러한 이유로 `right top 20%`와 `top right 20%`의 결과는 다르게 나옵니다. 전자는 top에 대한 프리셋이라, 좌표가 (100%,20%)정도로 나타내지고, 후자는 (100%-20%, 0%) 정도로 표시 할 수 있겠네요
  - `<length>`나 `<percent>`값은 아까 말했듯 앞에 나와있는 값들에 대한 오프셋 이므로, 키워드 한개에 `<length>`나 `<percent>`값 두개는 유효한 값이 아닙니다.
- 4개 값을 사용할 때 : 1 3 번째 값은 X Y 를 나타내고, 2 4 번째 값은 앞에 명시한 X Y에 대한 오프셋 입니다.
  - 1 3 번째 값으로는 `top`, `left`, `right`, `bottom`을 사용할 수 있습니다. 굳이 X Y 순서를 맞추지 않고 써도 상관 없기 때문에, `bottom 50px right 100px` 이런식으로 써도 되고, `right 35% bottom 45%` 이런식으로 작성해도 유효한 문법입니다.
  - 2 4 번째 값으로는 `<length>`나 `<percentage>`값을 사용해서, 각각 1 3 번째 값의 오프셋이 되어 줍니다.

## 배경 첨부(attachment) `background-attachment`

배경이 적용된 컨텐츠에 스크롤이 적용되면 배경은 어떻게 움직여야 할까요? 스크롤이 되어도 고정되길 원할 수 있고, 아니면 스크롤과 함께 움직이길 원할수도 있죠. 그런 속성을 정의하는것이 `background-attachment`입니다. 배경을 어디다가 붙일건지(attach)를 결정하는 속성이죠.

키워드 값 세 종류를 이용해서 이 속성을 사용할 수 있습니다.

- `scroll` : 배경을 **요소 테두리**에 부착합니다. 요소에 스크롤을 해도, 배경은 함께 스크롤 되지 않습니다.
- `fixed` : 배경을 **뷰포트**에 대해서 부착합니다. 요소에 스크롤을 해도, 배경은 함께 스크롤 되지 않습니다.
- `local` : 배경을 **요소 컨텐츠**에 부착합니다. 요소에 스크롤을 하면, 배경이 함께 내려갑니다.

### scroll vs fixed : 둘의 차이점은?

<https://meyerweb.com/eric/css/edge/complexspiral/demo.html>을 봅시다. 페이지를 위아래로 스크롤해도 암모나이트 그림은 움직이지 않아서, body에 배경을 적용해서 고정해두고, content 부분에는 배경을 좀 투명하게 해서 둔것처럼 보입니다. 하지만, 웹 페이지의 코드를 개발자 도구를 사용해서 해체해 보면, content 부분의 배경 색에는 alpha 값이 적용되어 있지 않아 불투명 하여, 배경이 비쳐 보이지 않습니다. 그럼 어떻게 된걸까요?

content의 배경에는 body에 그려진 그림과 동일한 그림에, 이미지 자체적으로 색을 덧씌운 그림이 적용되어 있습니다. 그리고 `background-attachment` 속성으로 `fixed`가 적용되어서, 이미지의 기준점이 뷰포트에 맞춰져 있어서, 연속되게 이어져 보이는 것입니다.

![body](https://meyerweb.com/eric/css/edge/complexspiral/shell-bg.jpg)

<div style="display:flex; justify-content:center;">body에 적용된 배경 이미지 파일</div>
![content](https://meyerweb.com/eric/css/edge/complexspiral/shell-blue.jpg)
<div style="display:flex; justify-content:center">content에 적용된 배경 이미지. 이미지의 배경색 빼고 같은 이미지다.</div>
```css
body {background: black url(shell-bg.jpg) 0 0 no-repeat fixed;}
div#content {background: #468 url(shell-blue.jpg) 0 0 no-repeat fixed;}
div#links a {background: transparent url(shell-fade.jpg) 0 0 no-repeat fixed;}
div#links a:hover {background: transparent url(shell-wash.jpg) 0 0 no-repeat fixed;}
```
<div style="display:flex;justify-content:center;">해당 페이지의 배경을 결정하는 css</div>

## 배경이 그려질 위치

`background-clip`은 배경이 적용될 범위를 명시하는 속성입니다. 적용 가능한 값은 다음과 같습니다.

- `border-box` : 배경이 테두리의 바깥 경계까지 차지합니다. (z축 순서상 배경보다 아래에 있음)
- `padding-box` : 배경이 padding 안쪽까지 차지합니다.
- `content-box` : 컨텐츠 공간에 맞춰서 배경을 그립니다.
- `text` <i class="fas fa-flask"></i>: 텍스트에만 배경을 그립니다. CSS level 4에 명시된 속성이라 <i class="fab fa-safari"></i> 에서만 현재 지원하고 있습니다.

![text](/images/frontend/background-clip-text.png)

<div style="display:flex; justify-content:center">`background-clip`에 `text`가 적용되면 나오는 결과</div>

# 마치며

css에서 배경을 제어할 수 있는 속성이 이정도로 꽤 방대했다는 사실은 이번 글을 보면서 알게된것 같습니다. 글을 쓰게 된 계기는 분명 그라데이션을 적용한 배경 떄문이었는데 정작 정리한 글은 그라데이션을 제외한 배경에 관한 모든 속성이 된것 같아서 기분이 약간 이상하기는 합니다. 다음 글은 아마 css 그라데이션 속성들의 총 정리일것 같습니다. 제 글을 찾아서 읽어주셔서 감사합니다.

# 참고한 글들

background-attachment:scroll vs fixed : [stack overflow](https://stackoverflow.com/questions/23050326/css-background-attachment-scroll-fixed-and-background-size-cover)
