---
title: CSS gradient 정리
layout: post
subtitle: 분량문제 때문에 다루지 못한 gradient에 대해서 정리를 해봅니다.
image: /images/thumbnails/CSS.png
category: frontend
---

# 들어가며

[직전 포스팅](/about-css-background/)에서 원래 다뤄야 했지만, 글이 너무 길어질 것 같아서 쓰지 못한 내용을 이 글에 담아 보려고 합니다. 그라데이션 요소를 적용하는 `<gradient>`에 관한것을 다루어볼까 합니다.

# CSS에서 사용하는 그라데이션 `<gradient>` 자료형

앞 포스팅에서도 다뤘듯, `<gradient>`자료형, 즉 css에서 쓰이는 그라데이션은, 이미지로 취급됩니다. 그리고, 크기에 대한 정보가 거의 없다는 특징 또한 가지고 있지요. 따라서, 배경으로 쓰일 때는 배경을 취하는 박스의 크기에 딱 맞게 설정이 됩니다. 크기 설정이 필요 없다는 뜻이지요.

`<gradient>`자료형을 만들기 위해서는 아래에서 소개할 함수 중 하나를 사용하는 것입니다.

# 선형 그라데이션 : `linear-gradient`

설명을 시작하기 전에, 선형 그라데이션이 어떻게 구성되는지 부터 소개할 필요가 있을 듯 합니다.

선형 그라데이션은 **그라데이션 선**이라는 축과 두개 이상의 색 기준점 으로 이루어져 있습니다. 원문에서는 color stops라고 되어 있었는데, 색 중단점으로 번역 하기에는, 적절하지 않다고 판단했습니다. 실제로 사용되는 예를 보았을 때, 기준점이 되는 역할을 보여줬기에, 저는 _색 기준점_ 이라는 번역어를 사용하고자 합니다.

## 그라데이션 선(gradient line)에 대한 설명

`linear-gradient()`함수는 그라데이션 선에 수직으로 색 선을 그리면서, 색 기준점까지 그림을 그린다고 생각하면 됩니다.

[링크](https://miro.medium.com/max/972/1*6QjmQlmwLqgSnWdAm4UuLw.gif)를 클릭해서 gif 애니메이션을 확인해 보시면 더욱 확실히 알 수 있습니다. gif의 해상도가 낮아서 그라데이션이 줄처럼 보이는 효과가 있는데, 실제 그라데이션은 이렇게 구성됩니다.

아까 애니메이션을 봤다면, `linear-gradient()`함수 내의 특정 값을 바꿈으로써 그라데이션의 방향을 다르게 할 수 있음을 알 수 있었습니다. 그 점에 대해서 간단히 알아봅시다.

두가지 방법을 통해서 그라데이션 선을 조작할 수 있습니다.

- 키워드(`to top`, `to bottom`, `to left`, `to right`, `to top left` `to top right`, `to bottom left`, `to bottom right`)
- `<angle>`자료형을 통한 그라데이션 선의 기울기

기본값은 `to bottom`그리고 `180deg`(또는 그에 준하는 다른 `<angle>`값)

`linear-gradient()` 함수의 **첫번째** 매개 변수에 이를 넣어서 그라데이션 선의 기울기를 조정할 수 있습니다. 키워드 사용법은 직관적으로 눈에 들어오는 것 그자체 입니다. `to top`은 위에서 아래로, `to left`는 오른쪽에서 왼쪽으로...

`<angle>`자료형을 사용하면, 그라데이션 선의 기울기를 각도를 명시해서 조정할 수 있습니다. 기본값이 180도라는 것에 착안해서 회전하시면 됩니다. 각도가 정의되는 방법은 간단합니다. 그라데이션이 들어갈 박스의 중점을 지나는 수직선과, 그라데이션 선의 기울기 입니다. 45도 기울인 아래의 예시를 보면서 확인해 봅시다.

![45deg](/images/frontend/45deg_gradient.png)
그림 가운데의 각 표시 부분을 보면 아까 설명한 대로 45도를 이루고 있음을 볼 수 있습니다. 아래에는 여러 각을 명시했을 때의 예시를 보이겠습니다.

![180deg](/images/frontend/180deg_gradient.png)

<div style="display:flex;justify-content:center;">180deg만큼 회전. 기본값과 같다</div>

![-45deg](/images/frontend/-45deg_gradient.png)

<div style="display:flex;justify-content:center;">음수 각도 또한 사용할 수 있다</div>

## 그라데이션 시작점과 종점에 관해

다음 파트에서 색 기준점에 관해서 설명을 할 것인데, 명확한 이해를 위해서 필요하다고 생각해서 배치해 보았습니다. 위에 올린 이미지 중에, 45도와 -45도의 이미지를 자세히 보았다면, 시작점과 끝점이 박스 내부에 있는것이 아니라, 박스 외부에 있는것을 보셨을 것입니다. 이는 오류가 아니라, 마땅한 논리적인 이유가 있습니다.

그라데이션을 이용해서 배경을 만들려면, 일반적으로 그라데이션이 꽉 차게 표시가 되기 원할 것입니다. 그라데이션 축이 45도 기울어 졌을때를 기준으로 한번 설명해 보겠습니다.
![45deg](/images/frontend/45deg_gradient.png)

그라데이션은 그라데이션 선의 수직 방향으로 색 선들이 촤르륵 펼쳐진다고 설명을 했습니다. (기억이 안나시면, 위에 올렸던 [링크](https://miro.medium.com/max/972/1*6QjmQlmwLqgSnWdAm4UuLw.gif)의 이미지를 한번 더 참고하시길...)

오른쪽 위 꼭지점부터, 왼쪽 아래 꼭지점까지 그라데이션이 꽉 차게 되기 위해서는 당연히 그 꼭지점에도 색 선이 지나가야 할 것입니다. 따라서, 그라데이션의 시작점과 끝점은 아래와 같이 정의를 할 수 있겠죠?

그라데이션 선에서 제일 가까운 꼭지점에 수선을 내렸을 때, 수선을 내린 지점이 그라데이션의 **시작점/끝점** 이다.
{:.success}

강조표시까지 하는데에는 다 이유가 있습니다. 하술할 색 기준점의 위치를 실제로 적용시켜 볼 떄, 생각한대로 줄이 잘 그어지지 않을 떄도 있을 것 입니다. 그럴떈 이 부분을 다시 정독하시면서, "아, 웹 브라우저가 잘못한것이 아니고, 내가 생각을 잠깐 잘못한 것이구나!" 하는것을 깨닫고 가시길 바라는 마음에서 이 부분을 강조했습니다. 더 나아가, 그런 꺠달음을 느낄 필요가 없도록 머리속에 저장할려는 목적 또한 있지요.

## 색 기준점(color stop)에 관한 설명

이론적인 설명은 이쯤 하고, 이제 어떻게 그라데이션을 구성할 수 있는지 알아보도록 하겠습니다. `linear-gradient`에서는 색 기준점을 추가로 더 넣으면서 상당히 많은 부분을 커스텀 할 수 있습니다. 색 기준점의 위치는 `<length>`나 `<percent>`를 이용해 직접적으로 명시가 가능합니다. 따로 직접적으로 명시를 하지 않는다면, 해당 색의 직전 색과 직후 색의 중단점의 중간 위치에 배치가 됩니다. 따라서 후술할 두 속성은 동일하다고 할 수 있습니다.

```css
linear-gradient(red, orange, yellow, green, blue);
linear-gradient(red 0%, orange 25%, yellow 50%, green 75%, blue 100%);
```

기본 값으로는, 색 전환은 서로 다른 색 기준점의 중간 지점부터 부드럽게 이루어 집니다. 이 값을 변경하려면, 색상 정보로 라벨되어 있지 않은 퍼센트 값을 사용해서, 다른 지점에서 색 전환이 시작될 수 있도록 할 수 있습니다. 이것을 변경하는 값을 color transition hint라고 합니다.

```css
linear-gradient(red 10%, 30%, blue 90%);
```

해당 코드에서는 두번째 인자에 색으로 태깅되지 않은 값인 `30%`라는 값을 넣어서, 색 전환 지점을 바꾸고 있습니다. color transition hint가 적용되지 않았을 떄의 결과와 비교해 봅시다.

![without transition](/images/frontend/without-transition-hint.png)

<div style="display:flex;justify-content:center;"><strong>transition hint를 적용 시키지 않았을 때. 색 기준점의 중간부분(녹색 선으로 표시)에서 전환이 일어나고 있음을 볼 수 있다</strong></div>

![with transition](/images/frontend/with-transition-hint.png)

<div style="display:flex;justify-content:center;"><strong>transition hint를 적용 시켰을 때. 색 기준점의 중간부분이 아닌 30%(녹색 선으로 표시)에서 전환이 일어나고 있음을 볼 수 있다</strong></div>

색 기준점들을 나열할때는, 오름차순으로 나타내져야 합니다. 나중에 나오는 낮은 값의 색 기준점이 이전 값을 오버라이드 해서, 의도치 않게 부드러운 전환이 아니라, 급격한 전환이 일어나서 색 전환이 일어나지 않게 됩니다.
![override](/images/frontend/hard-transition.png)

한 색에 여러개의 색 기준점을 설정 할 수도 있습니다. 다음의 세 css 코드는 동일한 의미를 가집니다.

```css
linear-gradient(red 0%, orange 10%, orange 30%, yellow 50%, yellow 70%, green 90%, green 100%);
linear-gradient(red, orange 10% 30%, yellow 50% 70%, green 90%);
linear-gradient(red 0%, orange 10% 30%, yellow 50% 70%, green 90% 100%);
```

## <i class="fab fa-internet-explorer"></i>(IE) 에서의 유의사항

IE는 이제 거의 쓰이지 않지만, 혹시나 IE에 대응 해야하는 환경에서 `linear-gradient()`를 사용할 때, 주의해야 할 점이 있습니다. 아래의 사항은 IE에서 작동하지 않습니다.

- 같은 색 기준점 여러곳에 설정하기
- 색 전환점 명시하기
- 각도에서 0을 표시할 때, 단위 생략하기

이런 점만 유의해 주시면, IE10 이상에서 사용이 가능합니다.

## 이대로 끝내기는 아쉬워 적어보는 `linear-gradient()` 문법

맨 처음에 글을 구상할때는 이걸 처음에 적어볼랬는데, 다른 CSS문법과는 다르게, 설명할것이 꽤나 길어서, 맨 끝으로 보냈습니다.

```
linear-gradient([<angle> | to <side-or-corner>]? , <color-stop-list>)
```

- `<side-or-corner>` : `to left`와 같은 키워드
- `<angle>` : 그라데이션 선의 기울기
- `<color-stop-list>` : 색 기준점들과 색 변환점(나타낸다면)을 나열한 리스트들

# 원형 그라데이션 : `radial-gradient()`

선형 그라데이션과 다르게, 원형 그라데이션은 이해하기가 쉽습니다. 원형 그라데이션은, 그라데이션의 **중심부**와 **크기**, 그리고 그라데이션의 **모양**으로 구성됩니다.

사용 가능한 문법은 아래와 같습니다.
<code class="language-plaintext highlighter-rouge">
radial-gradient([<span style="font-style:italic">shape size</span> [at <span style="font-style:italic">position</span>]?]? , <span style="font-style:italic">start-color</span>, ..., <span style="font-style:italic">last-color</span>)
</code>
이탤릭체로 표시한 부분이 우리가 사용할 문법 요소입니다. `at`은 일종의 키워드라서 이탤릭 처리를 하지 않은 것 입니다.

## 그라데이션의 모양 : shape

말 그대로 그라데이션의 모양을 결정합니다. 키워드 값을 사용해서 나타내며, 두가지의 값이 가능합니다.

- `ellipse` : 타원형 **기본값**
- `circle` : 원형

## 그라데이션의 크기 : size

그라데이션의 크기를 결정한다 하지만, 사실 그라데이션의 끝나는 모양을 결정한다고 말하는 쪽이 시각적으로 더 와닿지 싶습니다. elipse와 circle 모두 아래의 키워드를 사용할 수 있습니다.

- `closest-side`
  - shape가 circle일 경우에는, 그라데이션의 중심부에 가장 가까운 박스의 모서리에 그라데이션의 끝이 만나게 됩니다.
  - shape가 elipse일 경우에는, 타원의 수평축과 수직축이 모두 모서리에 부딛치게끔 됩니다.
- `closest-corner`
  - 그라데이션의 모양의 중심부에서 가장 가까운 모서리에 닿이도록 크기를 설정합니다.
- `farthest-side`
  - `closest-side`와 비슷한데, 가장 가까운 모서리가 아닌, 가장 먼 모서리를 기준으로 사이즈를 책정합니다.
- `farthest-corner`: **기본값 입니다**
  - 그라데이션의 모양이 중심부에서 가장 먼 꼭지점과 만나도록 세팅합니다.

이렇게만 설명하면 와닿지가 않으니, codepen 예시를 첨부하면서 한번 더 확인해 보겠습니다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="RwggQoW" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/RwggQoW">
  radial-gradient size</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

## 그라데이션의 중심의 위치 : position

css의 `<position>`자료형을 이용해서 그라데이션 도형의 중심부를 설정해 줄 수 있습니다. `<position>`자료형에 대한 설명은 [이전 포스트의 `background-image`에 대한 설명](/about-css-background/#배경의-시작점-지정-background-origin)을 참고하시면 됩니다.

참고로 <i class="fab fa-safari"></i>(safari 브라우저) 에서는 `at`키워드가 적용되지 않아서, 이게 적용이 되지 않습니다. 애플 기기를 상대로는 별로 좋지 않은 소식이네요.

## 색 기준점과 전환점 지정

이건 `linear-gradient()`와 동일한 문법을 사용합니다. 설명은 패스~

## 그라데이션을 반복하고 싶을때 : `repeating-linear-gradient()` & `repeating-radial-gradient()`

박스보다 크기가 작은 이미지를 배경으로 했을 때, 이미지가 반복되어서 적용이 되는 경우를 우리는 전 포스팅에서 학습하였습니다. 그라데이션은 크기가 정해져 있지 않아서, 그런 경우가 자연스럽게 생기지는 않지만, 그러한 기능을 사용하고 싶을 수도 있지요. 이럴때 사용하는것이 제목에도 적은 `repeating-linear-gradient()`와 `repeating-radial-gradient()` 입니다.

`linear-gradient`로 전체 배경보다 작게 그라데이션을 그리면, 마지막 색으로 배경을 마저 칠해버립니다.
![linear-graidient-small](/images/frontend/linear20%25.png)

하지만 `repeating-linear-gradient`를 사용한다면, 배경보다 작은 그라데이션이 주어졌을 때, 배경을 가득 채우기 위해서, 마지막 색으로 칠해버리는것이 아닌, 해당 패턴을 반복해서 배경을 칠해버립니다.
![repeating-linear-gradient](/images/frontend/repeating-linear20%25.png)

`radial-gradient`에서도 비슷한 상황이 생깁니다. 백문이 불여일견 이므로, 차이를 시각적으로 바로 볼 수 있는 codepen을 첨부하도록 하겠습니다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="bGRRKbB" data-user="kasterra-the-bashful" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/kasterra-the-bashful/pen/bGRRKbB">
  radial-gradient vs repeating-radial-gradient</a> by Huichan Lee (<a href="https://codepen.io/kasterra-the-bashful">@kasterra-the-bashful</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

같은 식으로 크기가 작은 그라데이션을 생성했을 때, 반복하는 모습을 볼 수 있습니다.

# 참고한 자료

- 언제나 그렇듯, 웹 개발자의 영원한 친구 mdn : [링크](https://developer.mozilla.org/ko/docs/Web/CSS/gradient)와 해당 웹 문서에 있는 링크들
- `linear-gradient`를 이해하는데 정말로 많은 도움을 주었던 자료 : [링크](https://patrickbrosset.medium.com/do-you-really-understand-css-linear-gradients-631d9a895caf)
  - 과장 안보태고 mdn만 봤을떄는 알쏭달쏭함 그자체 였는데, 이걸 보고 다시 읽으니, 이해가 되었습니다.
- [위의 블로그 작성자가 작성한 도구](https://codepen.io/captainbrosset/full/ByqRMB) : 몇몇 버그가 있어서 큰 기대는 하지 말라고 적혀져 있었지만, `linear-gradient`의 개념을 익히는데에는 훌륭한 도구였던것 같습니다. 해당 프로그램의 실행결과 스크린샷을 이 블로그에서도 열심히 써먹었습니다.
