---
title: CSS Flex 정리글
layout: post
subtitle: 웹 페이지 레이아웃을 편리하게 해주는 flexbox를 정리해 봅시다.
image: /images/thumbnails/CSSHTML.png
category: frontend
---

# 들어가며

css와 HTML로 웹 페이지를 만드는 상황을 한번 생각해 봅시다. 제 블로그를 포함한 웹 페이지는 대부분 위에서 아래로 스크롤을 하면서 내용물을 보도록 구성되어 있습니다. 이런 수직적 레이아웃은 구성하기 어렵지 않습니다. css 에서의 `display`속성 중, `display:block`을 사용하면, 수직적 레이아웃을 간단히 만들 수 있습니다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="bGRbVNd" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/bGRbVNd">
  display:block</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

문제 상황은 수평적인 구성에서 발생합니다. [display:float](https://developer.mozilla.org/ko/docs/Web/CSS/float)나 `display:inline-block`을 사용해서 구성할 수는 있지만 사용이 불편하다는 단점이 있습니다. 그리고, `display:inline-block`에는 골때리는 **특징**이 있습니다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="gORYaXp" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/gORYaXp">
  display:block</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

따로 CSS나 HTML에서 여유 공간을 두라고 설정한 적이 없는데도, 자기 멋대로 여유 공간이 생깁니다. 이 공간을 지우는 방법에 대한 [글](https://css-tricks.com/fighting-the-space-between-inline-block-elements/)([번역본](https://webclub.tistory.com/533))또한 있는데, 결론이, flexbox를 사용하라 라는 결론이 나오더라고요. 그런 의미에서도, 이제부터 flexbox에 대한것을 정리해보도록 하겠습니다.

# Flexbox 사용 시작하기

Flexbox에서는 레이아웃에 배치될 각각의 컨텐츠들(이하 flexbox item)에게가 아닌, 그 컨텐츠들을 감싸는 하나의 틀(이하 flexbox container)에, 자식 요소로 있는 컨텐츠들을 어떤 방식으로 정렬할지 **부모**에게 지정합니다. 몇몇 예외가 있긴 합니다만, 대부분은 flexbox container에 해당합니다. 예외에 해당하는 것들은 이후에 후술하도록 하겠습니다.

Flexbox를 사용하는 방법은, flexbox container가 될 div 등에게 `display:flex`나 `display:inline-flex`의 css 속성을 지정해 주면 됩니다.

`display:flex`와 `display:inline-flex`의 차이는, 컨테이너 자체가, inline요소로 취급되는지, block으로 취급되는지를 결정하기만 합니다. 아래의 codepen 예시를 보면 간단히 알 수 있을 것입니다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="oNwvpBX" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/oNwvpBX">
  flex vs inline flex</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

상단은 `display:flex`가 적용 되어서, 컨테이너가 한줄을 가득 차지하는 block처럼 행동함을 볼 수 있지만, 하단의 `display:inline-flex`로 적용된 컨테이너는 inline-block 처럼 행동함을 확인할 수 있습니다.

# 메인 축(Main Axis) 그리고 교차 축(Cross Axis)

Flexbox를 이해하려면 꼭 제대로 알아야 하는 내용입니다. 메인 축은, flexbox item이 배치될 방향을 나타내는 축이고, 교차 축은 메인 축에 수직 방향인 축입니다. 메인 축이 가로라면, 교차 축은 세로이고, 메인 축이 세로 방향이라면 교차 축은 가로 방향이 됩니다.

처음 flexbox를 배울 때, 메인 축이 가로이고, 교차 축이 세로다 라는 오개념을 가질 수 있지만, 반드시 이 점을 기억해야 합니다. 혹시 이 글을 읽다가, 이해가 되지 않는다면 이해가 될 때까지 몇번이라도 확인해 보면서 머리속에 넣어두길 바랍니다.

메인 축의 방향은 **flexbox container**에서 `flex-direction`이라는 속성으로 설정할 수 있습니다. 기본값은 row(가로방향, 행)이며, column(세로방향, 열) 입니다. 이것 말고도 적용할 수 있는 속성에는 row-reverse, column-reverse가 있는데, 여기에 관한 것은 직접 codepen 예제를 눈으로 보면서 설명하겠습니다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="zYzOpEV" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/zYzOpEV">
  flex-direction</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

아무것도 적용하지 않은 기본 flexbox는 왼쪽부터 오른쪽으로 가로 방향으로 나열되는 모습을 볼 수 있습니다. `flex-direction:column`을 적용하면 위에서 아래로 세로 방향으로 나열됩니다.
흥미로운 부분은 row-reverse와 column-reverse 입니다. **HTML의 수정이 없었음에도** 반대로 정렬이 되는 모습을 볼 수 있습니다.

## 메인축을 활용해서 컨텐츠 정렬 : justify-content

이때까지 flex를 이용한 예제들은 공간이 서로 떨어져있지 않고, 다닥다닥 붙어 있습니다. 하지만 레이아웃을 짤 때, 컨텐츠들 사이에 서로 공간이 있기를 바라는 경우가 있을 수 있죠. flexbox 에서는 이 공간을 우리가 직접 계산할 필요 없이, flexbox container가 알아서 정렬하도록 부탁할 수 있습니다.

justify-content는 flexbox container에서 아이템들이 메인축 방향으로 늘어설 때의 배치를 어떻게 할지 명시하는 부분입니다. 주로 사용하는 속성을 알아보겠습니다.

-   **flex-start**
    -   기본값 입니다. flex-direction의 시작부분부터 차례 차례 배열됩니다.
-   flex-end
    -   flex-start의 반대 개념으로 flex-direction의 끝부분부터 차례 차례 배열됩니다.
-   center
    -   가운데 정렬입니다. 아이템들이 메인축을 기준으로 가운데에 옹기종기 모일 것입니다.
-   space-between
    -   아이템들의 사이(between)에 균일한 간격을 넣어서 정렬합니다.
-   space-around
    -   아이템들의 둘레(around)에 균일한 간격을 넣어서 정렬합니다.
-   space-evenly
    -   아이템들의 양 끝에 완전히 동일한(evenly)간격으로 정렬합니다. <i class="fab fa-internet-explorer"></i> <i class="fab fa-edge-legacy"></i>👎(<i class="fab fa-edge"></i>는 크로미움 엔진이라 가능 👍)

이 속성은 여러분이 보시는 이 블로그에서 참 많이 쓰이고 있습니다. 가운데 정렬을 위한 HTML 태그 `<center>`는 작동은 하지만, 비표준이라서, 사용을 자제해야 합니다. 대체 할 수 있는 코드로 저는 `<div style="display:flex; justify-content:center;"></div>`를 절찬리에 사용하고 있습니다.

space-between, space-around, space-evenly 이 셋은 언뜻 보기에는 서로 비슷해 보이지만, 차이가 있습니다.
![정리](https://www.w3.org/TR/css3-flexbox/images/flex-pack.svg)

space-between은 아이템 사이의 공간을 동일하게 설정합니다. 반면, space-around는 아이템 양쪽 끝에 동일한 공간을 주어서, 아이템들을 정렬합니다. 그래서 박스의 경계에서 아이템 사이의 간격은, 아이템과 아이템 사이의 공간의 절반이 되지요. space-evenly는 박스의 경계에서부터 아이템 까지의 거리와, 아이템과 아이템 사이의 거리까지도 같게 합니다.

이래도 나는 잘 모르겠다 하시는 분은 [1분 코딩의 flex에 대해 설명한 게시물](https://studiomeal.com/archives/197)을 읽어보시면 이해가 빠르시지 싶습니다.

위에서 말한 속성 외에도, start,end,safe,unsafe 등의 여러 요소들 또한 있지만, 해당 속성들은 파이어폭스나 사파리 등의 일부에서만 지원이 되고, 크롬에서 지원이 되지 않아서, 작성자가 써본적이 없는지라, 자세히 설명할게 없없습니다.

## 교차축을 활용해서 컨텐츠 정렬 : align-items

교차축을 기준으로도 정렬을 시킬 수 있습니다. align-content가 **아니라** align-item**s**(s가 있음)에 유의하세요. align-content는 나중 섹션에서 다룰 속성입니다. flexbox container에서 교차축을 기준으로 정렬하는 CSS 속성은 align-items임에 유의하세요. 강조하는 이유는 별건 아니고, 처음에 HTML과 CSS를 배웠을 때, 이런 어이없는 오타를 해놓고, 왜 CSS가 적용이 안되지 하고, 30분정도 어리둥절 했던 경험이 있는지라... 이 글을 읽으시는 분들은 그런 일 없길 바라는 마음에 한번 적어봤습니다.

이 역시 주로 사용하는 속성에 대해서 알아보겠습니다.

-   **stretch**
    -   기본값 입니다. 아이템들이 컨테이너의 수직방향으로 쭉 늘어납니다.
-   flex-start
    -   아이템들을 교차축의 시작점으로 정렬합니다.
-   flex-end
    -   flex-start와 반대로, 아이템들을 교차축의 끝점으로 정렬합니다.
-   center
    -   교차축을 기준으로 가운데 정렬입니다. 메인축이 row 라면, 세로 가운데 정렬인 식이죠.
-   baseline
    -   아이템들을 텍스트 베이스라인을 기준으로 정렬합니다.

![베이스라인](https://upload.wikimedia.org/wikipedia/commons/thumb/3/39/Typography_Line_Terms.svg/1920px-Typography_Line_Terms.svg.png)

<div style="display:flex;justify-content:center;"><strong>baseline이란 무엇인가에 대한 그림</strong></div>

justify-content와 align-items를 적절히 사용하면, 다양한 레이아웃들을 간단히 꾸밀 수 있기에, 잘 알아두는것이 중요하다고 생각 합니다.

# 공간이 부족할 때 생기는 일 wrap

flexbox 콘테이너에 아이템들을 넣을 때, 공간이 부족하면 어떻게 될까요? 부족한 공간에 어거지로 끼울 수 있고(nowrap), 부족한 공간을 넓혀서, 내부의 아이템들을 제 크기대로 넣을 수도 있습니다(wrap)

해당 현상을 제어하려면, flex 컨테이너 내에서, `flex-wrap`이라는 속성을 지정해주면 됩니다.

-   nowrap
    -   기본값 입니다. 아이템을 어떻게든 한줄로 정렬시킵니다. 줄일수 있으면 줄이고, 안되면 뭐 삐져나오는거죠. 여기에 대한 자세한 얘기는 flex-shrink에서 다루겠습니다.
-   wrap
    -   개행이 필요할 것 같으면 개행을 실시합니다. 마지막에 있는 아이템이 밑 행으로 내려갑니다.
-   wrap-reverse
    -   개행을 하되, wrap과는 반대(reverse)입니다. 들어가지 못했던 마지막에 있던 아이템이 윗 행으로 올라갑니다.

## 여러 줄이 생겼을때, 그 줄들을 정렬하는 방법 : align-content

wrap이 발생하면, 여러 줄이 생깁니다. 그 줄들을 어떻게 정렬할것인가에 대한 전략을 여기에서 결정합니다. 사용 가능한 속성은 아래와 같습니다.

-   stretch
-   flex-start
-   flex-end
-   center
-   space-between
-   space-around
-   space-evenly <i class="fab fa-internet-explorer"></i> <i class="fab fa-edge-legacy"></i>👎(<i class="fab fa-edge"></i>는 크로미움 엔진이라 가능 👍)

# Flexbox 자식들에게 직접 지정하는 요소들

이떄까지 알아본 요소들은 모두, 부모인 flexbox 컨테이너에 적용하는 속성이었습니다. 지금부터 소개할 속성은, flexbox에 들어있는 아이템들에 직접 지정할 요소들 입니다.

## 나만 특별히 align-items 시켜줘 : align-self

`align-items`는 교차축을 기준으로 아이템들을 정렬하는 속성임을 우리는 위에서 확인했습니다. 만약 특정한 아이템을 다른 방식으로 정렬하고 싶다면, 기존에 알고 있는 지식만으로는 flexbox 만으로는 해결이 힘들것 같아 보이지요. 이럴때 쓰는 것이 align-self 입니다. align-items의 속성값을 그대로 쓸 수 있습니다. 기본값은 `auto`로 align-items에서 지정된 값을 그대로 사용하는 것입니다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="GRERJKM" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/GRERJKM">
  </a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

해당 코드를 보면 특정 원소만 다르게 align 되어 있는 모습을 볼 수 있습니다.

## 정렬 순서를 다르게 : order

HTML을 변경하지 **않고**, flexbox에서 배치되는 순서를 변경할 수 있습니다. HTML 구조가 변경되지 않으므로, 웹 접근성 측면에서 유의해야 합니다. 스크린 리더기는 이 시각적 차이를 읽지 못하거든요.

`order:정수`의 형태로 사용합니다. 작은 수일수록 먼저 배치됩니다. 기본값은 0 입니다.

## 나에게 공간을 더 다오 : flex-grow

컨테이너에 메인 축을 따라 아이템들을 배치하고 공간이 남아서, 여백이 생길 수 있습니다. 이때, 남는 공간을 특정한 아이템이 사용했으면 좋겠다 라는 생각을 했을 때, 사용할 수 있는 옵션입니다. 기본값은 0이기 떄문에, 아무것도 설정하지 않으면 아이템이 늘어나지 않습니다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="qBjBdrR" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/qBjBdrR">
  flex-grow</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

이렇게 flex-grow가 적용되었을 때, 여백의 공간을 더 차지함을 알 수 있습니다.

이 속성을 사용해서, 여러 아이템들을 특정한 비율만큼 공간을 더 주는것도 가능합니다. 위의 코드는 모든 아이템들에게 같은 공간을 배분해서 1:1:1:1:1 비율이었지만, 특정한 아이템에 `flex-grow:2`를 설정한 3번째 경우를 보면, 해당 아이템에 2배의 **여백**이 할당됨을 볼 수 있습니다.

## 내가 이만큼 공간을 양보할게 : flex-shrink

flex-grow의 반대 개념같은 느낌입니다. 공간이 모자랄 때, 얼마나 쭈그러들건지를 결정합니다. 기본값이 flex-grow와는 다르게 1이기 때문에, 공간이 모자라면, 아이템들이 크기가 줄어드는 이유가 이것입니다. flex-shrink에는 정수 값이 들어가는데, 해당 값은, 공간이 모자라서 줄어들어야 할 때, 다른 아이템에 비해서 얼만큼 줄어들건지를 명시하는 용도입니다.

## 나의 처음 사이즈는 이만큼이야 : flex-basis

**메인 축 방향으로** 배치될 아이템의 크기를 명시하는 속성입니다. 메인축이 row라면, 너비가 되겠고, column이라면 높이가 되겠네요. 기본적으로 차지하는 크기라고 한 이유는 앞에서 설명한 `flex-grow`나 `flex-shrink` 등의 상황에 의해서 바뀔 수 있기 떄문입니다. width나 height등도 똑같아요.
