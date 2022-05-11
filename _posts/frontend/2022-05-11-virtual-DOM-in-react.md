---
layout: post
title: 리액트에 쓰이는 가상 DOM이 뭔가요?
subtitle: DOM, document fragment, Virtual DOM, React fragment, shadow DOM 까지
category: frontend
image: /images/thumbnails/React.png
---

# 개요

프론트엔드 개발에서 리액트를 사용하는 이유를 말해보라고 하면 여러가지가 있을 수 있겠습니다만, 이번 글에서는 그 이유중 하나인 "가상 DOM"에 대해서 다뤄 보겠습니다.
변경이 잦은 웹 페이지에서 필요한 부분만 빠르게 다시 렌더링 하여서, 속도적인 측면에서 이득을 볼 수 있다고 간략하게만 알고 있었는데, 한번 시간을 내서, 정리해보자 합니다.

# 가상 DOM

## 가상 DOM의 동작 구조

> Virtual DOM은 UI의 "가상"적인 표현을 메모리에 저장하고, ReactDOM과 같은 라이브러리에 의해 "실제" DOM과 동기화 하는 개념입니다. -[React 공식 문서](https://ko.reactjs.org/docs/faq-internals.html#what-is-the-virtual-dom)-

React를 조금이라도 끄적거려 봤다면, 한번쯤은 들어봤을 말입니다. 하지만, 이게 어떤 식으로 돌아가는지에 대해서는, 사실 잘 모르고 쓰는 경우가 많았던것 같습니다.

React 버전 16부터 React는 [Fiber Architecture](https://github.com/acdlite/react-fiber-architecture)라는 구조를 통해서 작업 스케줄링을 한다고 합니다.

> UI에서는 모든 업데이트가 즉시 실행될 필요가 없습니다. 사실, 그렇게 하는것은 상당히 낭비스러운 일이며, 프레임 저하를 일으켜, 유저 경험을 저하시킵니다.
>
> 각기 다른 타입의 업데이트들은 다른 우선순위를 가지고 있습니다. 이를테면 애니메이션 업데이트는 데이터 저장소에서 업데이트 하는 작업보다 더 빨라야 하죠
>
> Push 기반의 접근은 어플리케이션(을 만드는 당신)이 어떻게 스케줄링 할것인지 결정해야 합니다. 반면, pull 기반의 접근은 프레임워크(React)가 당신을 위해서 그러한 것들을 대신 똑똑하게 해주죠. -[Fiber Architecture 문서](https://github.com/acdlite/react-fiber-architecture)에서 발췌-

리액트는 재조정(reconciliation)과 렌더링 과정을 따로 따로 실행하도록 디자인 되었습니다.

- 재조정 : 리액트에서 두 가상 DOM을 비교해서 어떤 부분이 변경되야 하는지 검사합니다. 다른 컴포넌트 타입은 상당히 다른 트리를 생성할 것으로 유추할 수 있으므로, 리액트는 트리 두개를 비교하기 보다, 기존의 트리를 완전히 대체해 버립니다. 리스트들의 비교는 `key`를 통해서 이루어 집니다. `key`들은 안정적이고, 예측가능 해야하며, 유일해야 합니다.
  - 더 자세한 사항은 [한글 공식 문서](https://ko.reactjs.org/docs/reconciliation.html)를 참조하면 좋을 것 같습니다.
- 렌더링 : 이 과정에서 두 트리에 대한 차이에 대한 정보(diff)를 활용해서, 렌더링 되는 앱을 업데이트 합니다. 여기에서 렌더링 작업을 덩어리로 나누어서, 여러 프레임에 걸쳐서 전파를 할 수 있습니다. 가상 스택 프레임은 이 과정에서, 잠시 하던 일을 멈추고 다시 돌아오는 역할(함수를 호출하는 것 처럼), 다른 작업에 우선순위를 할당하거나, 이전에 사용했던 작업의 결과물을 사용하거나(메모이제이션), 이제는 필요 없는 작업을 취소합니다.

이렇게 두 과정으로 나누었기 때문에, React.js와, React Native는 서로 다른 렌더러를 사용하면서도, 같은 재조정 과정을 공유할 수 있습니다.

## 가상 DOM의 이점

값비싼 렌더링 과정을 여러번 수행 할 필요를 최대한 줄여준다 라고 하였지만, 사실 말로만 들어서는 체감하기가 어렵습니다. `f12`키를 눌러서 개발자 콘솔에 들어가서 HTML코드들이 번쩍거리는걸 볼 수는 있지만, 확실히 시각적으로 보는쪽이 더 도움이 될것 같기에 예제를 하나 들고왔습니다.

https://codesandbox.io/s/real-dom-vs-virtual-dom-4jwi92

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1, shrink-to-fit=no"
    />
    <title>React App</title>
  </head>

  <body>
    <noscript> You need to enable JavaScript to run this app. </noscript>
    <span>Virtual DOM</span>
    <div id="root"></div>
    <span>Real DOM</span>
    <div id="pureDom"></div>
  </body>
</html>
```

<div style="display:flex;justify-content:center;">index.html</div>

```jsx
function App() {
  return (
    <select>
      <option value="apple" selected>
        Apple
      </option>
      <option value="pear">Pear</option>
    </select>
  );
}
export default App;
```

<div style="display:flex;justify-content:center;">App.js</div>

```js
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";

const updateRender = () => {
  document.getElementById("pureDom").innerHTML = `
  <select>
    <option value="apple">Apple</option>
    <option value="pear">Pear</option>
  </select>
  `;

  ReactDOM.render(
    <React.StrictMode>
      <App />
    </React.StrictMode>,
    document.getElementById("root")
  );
};

setInterval(updateRender, 1000);
```

<div style="display:flex;justify-content:center;">index.js</div>

이 코드들을 살펴보면, DOM API와, React 모두 동일한 형태의 컴포넌트를 렌더링 하고 있음을 알 수 있습니다. 그리고 렌더링을 하는 함수인 `updateRender`는 1초마다 실행되도록 `setInterval`되어 있네요. 위의 링크를 클릭해서 이 예제를 실험해 보면 흥미로운 사실을 하나 알아낼 수 있습니다.

DOM API로 만든 컴포넌트는 말 그대로 1초마다 새로 렌더링 되어서 `<option>`에서 뭔가를 선택하려면 다시 렌더링이 되어서 원래 상태로 돌아가버립니다. 하지만 가상 DOM을 사용하는 React 쪽은 사용자가 입력을 해도 DOM구조는 변함이 없으니 불필요한 재 렌더링 작업을 하지 않는 것을 명확히 확인할 수 있습니다.
