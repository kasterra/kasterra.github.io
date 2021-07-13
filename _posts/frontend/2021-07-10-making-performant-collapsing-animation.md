---
layout: post
title: 60프레임 collapsible 만들기
excerpt: 프론트 엔드 최적화를 위해 transition 사용하기
category: FRONTEND
image: /images/thumbnails/CSSJS.png
---
# 시작하며
지금 블로그에 있는 한 시리즈들을 묶어서 보여주는 시리즈 기능은, 기능 자체는 잘 작동하나, 심미적 관점에서 별로인건 사실인것 같습니다. 그래서, [Bulma](https://bulma.io/)에서 제공하는 여러 컴포넌트 중에서 [Card](https://bulma.io/documentation/components/card/)기능을 활용해서, 헤더 부분을 클릭하면 목록이 펼쳐지고, 다시 클릭하면 접혀지는, 흔히 `collapsible`이라고 불리는걸 만들어서, 애니메이션을 적용할까 했습니다.

이왕이면, 좋은 사용자 경험을 위해서는 60fps의 애니메이션이 좋지 않을까 하여, 구글링을 해서 [구글에서 게시한 글](https://developers.google.com/web/updates/2017/03/performant-expand-and-collapse)을 읽었고, 다른 문서와는 다르게 얘는 한국어로 번역이 안되있기도 했고, 이걸 읽고 바로 이해를 하지 못했어서, 제가 이해한 과정을 이 블로그 글로써 남겨 보려고 합니다. 이 포스트에 적힌 JS 코드는 해당 글에서 가져왔음을 밝힙니다.

# 직관적이지만 최적화되지 않은 방법
일반적으로 접히는 애니메이션을 만든다면, 이렇게 구현할 생각을 많이 합니다.
```css
.content {
  padding: 0 18px;
  background-color: white;
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.2s ease-out;
}
```
이런식으로 `max-height`에 `transition`을 주고, javascript로 클릭했을 때, `max-height`를 변화시키면, 그 사이에 `transition`이 적절하게 처리를 해줄거라고 믿고 맡겨버리는 거지요. 하지만, 이 방법은 버벅거림을 유발하기 딱 좋은 방법입니다.

이유를 설명해 보겠습니다. `developers.google.com`에 올라온 [렌더링 성능에 대한 글](https://developers.google.com/web/fundamentals/performance/rendering)을 보면, 알 수 있는데, 이를 간단히 요약하자면 이렇습니다.

사이트를 렌더링 하는데는 크게 다섯가지 핵심 파이프라인을 거칩니다.
1. JS/CSS로 인한 스타일 수정
    - 바닐라 자바스크립트나 CSS 애니메이션에 의해서 스타일이 수정됩니다. 자바스크립트의 이벤트 리스너라던지, CSS의 pseudo selector 라던지 알잖아요.
2. 스타일 계산
    -  CSS 스타일시트를 훑으면서, 해당 요소에 들어갈 스타일 요소를 검사하고, 최종적으로 어떤 스타일 요소를 반영할 것인지 판별하는 단계입니다.
3. 레이아웃
    - 브라우저가 해당 요소가 화면에 얼만큼의 공간을 차지할지 등의 레이아웃을 계산합니다. 한 요소의 크기가 다른 요소의 배치에 영향을 끼칠 수 있을 경우가 있기 때문에, 이러한 단계를 거친다고 생각하면 됩니다.
4. 페인트
    - 레이아웃이 완료되었으니, 실제로 화면에 그리는 작업입니다. 이 단계에서, 그림자 표현 등 **시각적인 요소**들이 이떄 처리되며, 여러 레이어에서 진행합니다.
5. 합성
    -  여러 레이어들을 순서에 맞게 배치해서 최종적인 화면을 그리는 작업입니다.

요소의 스타일 값이 바뀔 때, 이 모든 과정을 다 실행하지 않습니다. 각 스타일 요소에 따라서, 어떤 파이프라인을 실행할 수도 있고, 하지 않을수도 있습니다. 당연히 이러한 차이들은, 렌더링 속도의 차이를 만들게 되어서,  요소의 스타일을 반복적으로 수정하는 애니메이션의 성능을 결정하는데에 중대한 영향을 미친다고 생각할 수 있습니다. 각 요소가 렌더링 될 때, 어떤 파이프라인을 거치는지 확인하려면 [CSS 트리거](https://csstriggers.com/)를 참고해서 보면 됩니다. `height`나 `max-height`등의 기하학적 요소들은 레이아웃, 페인트, 합성의 모든 과정을 거치게 되어서, 60fps의 애니메이션을 만들기 어렵고, 또 컴퓨터의 더 많은 연산을 필요로 합니다.
# 최적화 된 방법
그렇다면 컴퓨터에 부담을 덜 주고, 60fps의 꿈을 실현시킬 좋은 방법은 무엇일까요?  [CSS 트리거](https://csstriggers.com/)를 확인해 보면, Webkit(애플 계열)과 EdgeHTML(구버전 엣지) 엔진을 빼고 `transform`에 관해서는 합성 단계만 거치면 된다고 하군요. 

애니메이션을 웹 프론트에서 구현하는 방법은 CSS 애니메이션도 있지만, javascript로도 구현할 수 있습니다. 둘다 각각의 장단점이 있습니다. 이것도 [google developers에서 쓴 글](https://developers.google.com/web/fundamentals/design-and-ux/animations/css-vs-javascript)이 있습니다.

간단한 오브젝트에는 CSS 애니메이션을 쓰면 된다고 하는군요.
우리는 그저 간단하게 펴지고 접히는것 정도를 보기 원하지 되게 고급 기능은 사용하지 않을것이기 때문에 CSS 애니메이션으로 작성할 것입니다. 

우리가 애니메이션을 적용할 박스의 모습은 아래와 같습니다.
<style>
     .panel {
        padding: 15px;
        margin: 20px 0px;
        background-color: #fff;
        border: 1px solid #ddd;
        border-radius: 4px;
        -webkit-box-shadow: 0 1px 1px rgba(0,0,0,0.05);
        box-shadow: 0 1px 1px rgba(0,0,0,0.05)
    }
    .panel h4, .panel h5, .panel h6, .panel i{
        margin: 0.4rem 0rem;
    }
    .panel h4, .panel h5{
        display: inline;
    }
    .panel-heading {
        padding: 10px 15px;
        margin: -15px -15px 15px;
        background-color: #f5f5f5;
        border-bottom: 1px solid #ddd;
        border-top-right-radius: 3px;
        border-top-left-radius: 3px
    }
    </style>
<div class="panel seriesNote">
    <div class="panel-heading" style="position:relative;">
        <h4>이 포스트는 <strong>"시리즈 제목"</strong> 시리즈의 <strong>n</strong>번째 포스트 입니다.</h4>
        <i class="fas fa-chevron-up" style="position:absolute; right:5%"></i>
    </div>
    <ul>
        <li>
            <h5>제목1</h5>
            <h6>부제목1</h6>
        </li>
        <li>
            <h5>제목2</h5>
            <h6>부제목2</h6>
        </li>
        <li>
            <h5>제목3</h5>
            <h6>부제목3</h6>
        </li>
    </ul>
</div>    
이건 펼친 모습이고 접은 모습은
<div class="panel seriesNote">
    <div class="panel-heading" style="position:relative;">
        <h4>이 포스트는 <strong>"시리즈 제목"</strong> 시리즈의 <strong>n</strong>번째 포스트 입니다.</h4>
        <i class="fas fa-chevron-down" style="position:absolute; right:5%"></i>
    </div>
</div>
정도가 될 것 같습니다.

## transform으로 애니메이션을 만들자!
메뉴를 접고 펴는 애니메이션을 만들 때, `transform`을 통해서 `scale`을 다르게 해줄 것이라고 아까 언급한 바 있습니다. 접을때는 전체 패널의 사이즈를, 제목의 사이즈 만큼 줄이고, 펼때는 다시 원래의 사이즈대로 펼치는 식이겠죠.

JS 코드로 크기를 계산한다면 아래와 같겠죠
```js
function calculateCollapsedScale () {
  // 메뉴 타이틀 부분만 접혀졌을때 보이도록
  const collapsed = menuTitle.getBoundingClientRect();

  // 확장되었을때는 메뉴 전체가 보이도록 해야하기 때문
  const expanded = menu.getBoundingClientRect();
  return {
    x: collapsed.width / expanded.width,
    y: collapsed.height / expanded.height
  };
}
```
전체 패널의 사이즈를 줄여버린다니, 뭔가 이상한 느낌이 들지 않나요? 이전에 언급한 [구글에서 올린 글](https://developers.google.com/web/updates/2017/03/performant-expand-and-collapse)에 첨부된 동영상을 확인해 봅시다.

<div style="display:flex; justify-content:center">
<video loop autoplay muted controls>
<source src="https://developers.google.com/web/updates/images/2017/03/performant-expand-and-collapse/squashed.webm" type="video/webm">
</video>
</div>

패널 전체의 크기를 줄여버리면 자식 요소인 내용도 함께 줄어들어서 위의 모습처럼 내용도 같이 찌그러져서 보이게 되기 때문에, 우리가 원하는 모습이 되지 않음을 확인할 수 있습니다. 우리가 원하는 결과를 내기 위해선, 당연하게도 전체 패널이 줄어든만큼 내용물의 크기는 늘려서, 시각적으로 봤을 때, 자연스럽게 줄어들도록 해야 합니다. 내용물 부분의 스타일에 `overflow : hidden` 등을 적용한다면, 우리 눈에는 마치 보이는 영역이 점점 줄어드는 것처럼 보일테니까요.

## 즉석(on-the-fly)으로 CSS 애니메이션 만들기
위에서 잠깐 지나가는 식으로 패널의 사이즈가 줄어드는 만큼 사이즈를 늘린다고 하였습니다. 패널의 사이즈가 1/5가 되었다면, 내용물의 크기는 5배가 되어야 한다는 것이죠. 일반적으로 애니메이션을 적용할 때, `ease-function`들을 CSS에서 `ease-in` 등등으로 적용을 하는데, 우리가 마주친 상황에서는 이러한 방법을 쓸 수 없습니다. `cubic-bezier(0, 0, 0.3, 1)`의 역함수를 쉽게 계산할 수는 없으니까 말이죠.

따라서, 애니메이션을 바로 CSS 파일에 적는것이 아닌, JS에서 애니메이션을 만들어서 스타일시트에 별개로 반영시키는 방법을 사용할 것입니다. 매 프레임마다 얼마나 움직여야 하는지를 JS로 계산해서 스타일시트에 넣어줍시다. 

이것도 JS코드를 나타내면 아래와 같습니다.
```js
function createKeyframeAnimation () {
  // 접혀졌을때의 요소의 크기를 알아낸다.
  let {x, y} = calculateCollapsedScale();
  let animation = '';
  let inverseAnimation = '';

  for (let step = 0; step <= 100; step++) {
    //각 step을 easing 시켜서 적용한다. 
    let easedStep = ease(step / 100);

    // 해당 step에서 요소의 scale을 계산한다.
    const xScale = x + (1 - x) * easedStep;
    const yScale = y + (1 - y) * easedStep;

    animation += `${step}% {
      transform: scale(${xScale}, ${yScale});
    }`;

    // 내용물 부분은 해당 비율의 역수만큼의 배율을 적용
    const invXScale = 1 / xScale;
    const invYScale = 1 / yScale;
    inverseAnimation += `${step}% {
      transform: scale(${invXScale}, ${invYScale});
    }`;

  }

  return `
  @keyframes menuAnimation {
    ${animation}
  }

  @keyframes menuContentsAnimation {
    ${inverseAnimation}
  }`;
}
```

`ease()`라는 함수는 왜, 어디서 나온걸까요? CSS easeing을 쓰지 못하기 때문에 JS에서 처리를 해준 것이라고 생각하면 됩니다. [easeing.net](https://easings.net/ko)이라는 서비스에서 다양한 easing function들을 `bezier-curve`나 JS에서 쓸 수 있는 함수의 형태로 제공하고 있으니, 본인의 입맛대로 골라서 쓰면 됩니다. 

## CSS 애니메이션 적용하기
위의 JS 코드에서 CSS 애니메이션을 만들었습니다. 이제 이걸 실제 웹페이지에서 사용할 수 있도록 CSS파일 같은 스타일시트에 명시를 해주면 됩니다. 
```css
.menu--expanded {
  animation-name: menuAnimation;
  animation-duration: 0.2s;
  animation-timing-function: linear;
}

.menu__contents--expanded {
  animation-name: menuContentsAnimation;
  animation-duration: 0.2s;
  animation-timing-function: linear;
}
```
여기서 `animation-timing-function`에 `linear`를 쓴 점을 주목하길 바랍니다. 이미 우리는 애니메이션을 작성할때 `easing`을 거쳤기 때문에 별도의 easing 을 더 거친다면 의도한 easing이 안되고 부자연스럽게 나올 수 있기 때문이라는 점을 알아두세요.

# 최종적으로 적용한 것
위의 사항을 적용해서 최종적으로 만든 기능입니다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="XWRjdNv" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/XWRjdNv">
  performant flexible</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>


# 마치며
위의 결과물을 보면 알 수 있듯, 문자열이 확장이 되었을때 즉 원래 사이즈를 기준으로 레이아웃 됩니다. `transfrom`자체가 레이아웃에 영향을 끼치지 않는 시각적인 부분이라서 당연한 것이지만, 이 결과물 때문에 블로그에 넣기 약간 애매한 부분도 없잖아 있습니다. `position:absolute`를 적용하고, 제목 부분의 높이만 적당히 가지는 div를 넣어서 해결해볼까 했는데,  [이 글](https://baeharam.github.io/posts/css/css-line-height/)을 읽어보면, 텍스트만의 높이를 구하는것은 결코 만만한것은 아니라고 하여, 위치를 잡아줄 div를 만들기는 쉽지 않은것 같군요. 원래 블로그의 시리즈 기능에 반영하려고 찾아본 것이지만, 실제 블로그 시리즈 기능에는, 애니메이션 빼고, 높이를 변화하는 식으로 구현해야 할 것 같습니다.

이해가 안되는게 하나 있었는데,

**menu.js**의 108 번째줄의
```js
window.getComputedStyle(this._menu).transform;
```
는 없애니까 더 성능이 좋아지던데... <https://developers.google.com/web/fundamentals/performance/rendering/avoid-large-complex-layouts-and-layout-thrashing?hl=ko%5D> 이걸 참고해 보면, 어느 레이아웃에 관하여 쿼리를 던질 떄, 예전에 캐싱 해놓은 값이 유효하지 않을 가능성이 있다면 (이를테면 다른 스타일 변경이 있다거나) 하면 다시 스타일의 값을 계산해야 해서 계산한다고 병목현상의 원인이 될 수 있다고 합니다. 주석의 내용으로는 클래스를 `take-hold` 하기위해 (영어사전에 찾아보니, 뿌리를 내리다, 보존하다 정도의 뜻이 있던데)라고 되어 있던데, 이걸 왜 넣은지는 모르겠습니다. 우선 다른것은 잘 모르겠지만, 렌더링 속도의 측면에서는 이 코드를 빼는게 맞는것 같아서, 저는 제거를 해서 위에 적용하였습니다.