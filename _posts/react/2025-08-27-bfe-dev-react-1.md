---
layout: post
title: BFE.dev React Quiz 풀이 - 1
subtitle: 리액트 렌더링, 디버깅의 근원을 조금 더 실전적으로 알려주는 리액트 퀴즈 풀이
category: react
image: /images/thumbnails/React.png
---

# 들어가며

## 동기

'딥 다이브'라는 말은 참 좋은 말이긴 합니다만, 사실 실전성이 없으면, 흥미도 잘 안 느껴지고, 실전에서 어떻게 쓰이는지도 힘듭니다. 즉, 실전성이 있다는 것을 알게 되면, 지식의 깊이를 가져가면서, 흥미와 실용성 또한 챙겨 갈 수 있다는 것이지요.

'리액트 렌더링 딥 다이브'가 실용성을 가질 수 있기 위해서는, 리액트의 렌더 스텝을 활용한 디버깅이나 최적화 기술을 배우는 것일 것입니다. 아마 React 기반 프론트엔드 개발을 열심히 하다 보면, '어디까지 리렌더 되는 것일까?' 하는 것을 얼추 감으로 찍고 넘어가는 일들이 꽤나 비일비재합니다만, 조금 더 자신감 있게 알 수 있다면, 분명 경쟁력 있고, 최적화 잘 되어 있는 웹 앱을 만드는 개발자로서의 역량을 키우는 데 분명 도움이 될 것입니다.

## BFE.dev 소개

[BFE.dev](https://bigfrontend.dev)는 프론트엔드 기술 면접에서 나올 법한, 프론트엔드 관련 기반 지식들을 쌓을 수 있는 좋은 플랫폼입니다. JS, react, typescript, css에 대한 질문, 심지어 과제 테스트까지 준비해놓은, 프론트엔드 커리어를 쌓아가는 데 꽤나 많은 도움을 주는 공간이라 할 수 있겠습니다.

저는 여기에서 **React Quiz** 부분을 풀어볼까 합니다. React 코드를 제공한 다음, 이 코드의 실행 결과 (주로 `console.log` 실행 결과)를 예측하는 식으로 구성이 됩니다. 후반 문제들로 갈수록, React 내부 동작인 커밋 페이즈, 렌더 페이즈라던가, 작업들을 관리하는 lane이라던가를 알아야 해서, 문제가 점점 재밌어지고, 공식 답변도 제공되지 않기 때문에, 이전에 풀어보고 언젠가 블로그에 정리를 해야겠다고 생각하였고, 지금 이렇게 시간을 내어 풀이 시리즈 포스트를 작성하고 있습니다.

# 문제 1 - React re-render 1

[원본 링크](https://bigfrontend.dev/react-quiz/React-re-render-1)

```tsx
// This is a React Quiz from BFE.dev

import * as React from "react";
import { useState, useEffect } from "react";
import { createRoot } from "react-dom/client";

function A() {
  console.log("A");
  return <B />;
}

function B() {
  console.log("B");
  return <C />;
}

function C() {
  console.log("C");
  return null;
}

function D() {
  console.log("D");
  return null;
}

function App() {
  const [state, setState] = useState(0);
  useEffect(() => {
    setState((state) => state + 1);
  }, []);
  console.log("App");
  return (
    <div>
      <A state={state} />
      <D />
    </div>
  );
}

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

## 시도했지만 틀린 오답과 교정하는 풀이

```text
"App"
"A"
"B"
"C"
"D"
"App"
"A"
"B"
"C"
```

처음 이렇게 생각했던 이유는 다음과 같습니다.

처음 마운트가 되면, 당연히 `App`이 렌더링되어 'App'이 로깅될 것이고, 렌더링 순서는 fiber 트리를 dfs처럼 탐색한다고 알고 있으므로([참고 링크 : fiber 딥다이브 velog 글](https://velog.io/@alsgud8311/React-Fiber-아키텍처-딥다이브) [참고 링크2 : jser.dev 영문 블로그. Fiber 트리의 DFS적 접근을 풀이해준다](https://jser.dev/react/2022/01/16/fiber-traversal-in-react/)) "A", "B", "C", "D"의 순서는 보장된다고 생각했습니다.

처음 마운트가 끝나면, 리액트는 `useEffect`를 실행하고, `App`의 state가 갱신되므로, `App`이 리렌더 되어 다시 'App'이 콘솔에 찍히는 것까지는 맞았습니다. 그 다음이 문제였습니다.

바로, state에 관련된 `A`만 리렌더 될 것이라고 생각했는데, 사실 JSX가 내부적으로 어떻게 동작하는지 가장 겉면만 생각해도, 그 생각에는 어폐가 있음을 단번에 알 수 있습니다.

결국 JSX는 javascript 객체입니다. `App`이 리렌더 될 때마다 return을

```ts
{
  type: "div",
  children: [
    {
      type: A,
      props: {
        state: state
      }
    },
    {
      type: D,
      props: {}
    }
  ]
}
```

이렇게 한다는 것인데, `A`, `D` 모두, prop으로 **주소값이 다른** 무언가를 받게 됩니다.
`{} === {}`가 false인 JS이기에, 당연히, prop이 달라지면 다시 렌더링 되는 React이므로 정답은,

```text
"App"
"A"
"B"
"C"
"D"
"App"
"A"
"B"
"C"
"D"
```

가 되는 것입니다.

## 소감

"부모가 흔들리면 자식도 함께 흔들린다"라는 React의 대원칙을 다시 한번 확인시켜주는 문제였습니다. 첫 문제이기에, 난해한 문법보다는 React의 기본 원칙을 일깨워주는 좋은 유형의 문제라고 생각합니다.

# 문제 2 - React

[원본 링크](https://bigfrontend.dev/react-quiz/React-re-render-2)

```tsx
// This is a React Quiz from BFE.dev

import * as React from "react";
import { useState, useEffect, memo } from "react";
import { createRoot } from "react-dom/client";

function A() {
  console.log("A");
  return <B />;
}

const B = memo(() => {
  console.log("B");
  return <C />;
});

function C() {
  console.log("C");
  return null;
}

function D() {
  console.log("D");
  return null;
}

function App() {
  const [state, setState] = useState(0);
  useEffect(() => {
    setState((state) => state + 1);
  }, []);
  console.log("App");
  return (
    <div>
      <A state={state} />
      <D />
    </div>
  );
}

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

## 풀이 - 단번에 정답

```text
"App"
"A"
"B"
"C"
"D"
"App"
"A"
"D"
```

2번 문제는 1번 문제와 단 하나의 차이점이 있습니다. `B` 컴포넌트에 `React.memo` 처리가 되어 있다는 점입니다. [React memo는 prop이 변경될 때만, 감싸진 컴포넌트를 리렌더](https://ko.react.dev/reference/react/memo)하고, 기본적으로 `Object.is`를 이용해 비교하므로, 이 상황에서는 리렌더 될 때 `A`까지는 리렌더 되지만, `B`부터는 리렌더 되지 않아, `B`의 자식인 `C`도 리렌더 되지 않습니다. 결국 다음 렌더에서 콘솔에 찍히는 것은 `D`가 됩니다.

## 소감

`React.memo`가 prop이 변경되지 않는다면 리렌더하지 않는다는 사실은 알고 있었지만, 공식 React docs를 보면서 기본값이 `Object.is`이고, 이 또한 커스텀할 수 있다는 사실을 알게 되었습니다.

# 마무리하며

예전에 미뤄두었던 숙제를 꺼냈습니다. 지금 이 문제는 초반의 문제이지만, 앞으로 갈수록 기존의 '면접에서 이야기할 수 있는 React 지식' 내용을 되짚어보는 좋은 문제들이 나오기에, 기본부터 차곡차곡 정리하고자 이렇게 글을 작성했습니다. 문제 분량이 길고 풀이도 분량이 길어지므로, 스크롤의 압박이 꽤 되기에 이번 글은 문제 2개만 이렇게 정리합니다.

끝까지 읽어주셔서 감사합니다.
