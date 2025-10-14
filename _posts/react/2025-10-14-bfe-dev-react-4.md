---
layout: post
title: BFE.dev React Quiz 풀이 - 4
subtitle: BFE 풀이는 계속 되어야 한다. 이것저것들
category: react
image: /images/thumbnails/React.png
---

# 들어가며

한동안 블로그에 글을 적지 못했습니다. 9월 내내 구직 관련 활동들이 상당히 많았고, 거기에서 쌓인 피로들을 회복한다는 명분으로 글을 쓰지 못했던것 같네요.

## 이번에 풀 문제들

이번 글에서 다룰 문제들은 이전까지의 글들과는 다르게 하나의 주제가 있다라기 보다는, 말 그대로 문제 풀이에 조금 더 초점을 두는 글이 될 것 같습니다.

# 문제 1 - React re-render 5 - context

React에서 props driling이라는 불편함을 없애주기 위한 도구인 `context`에 대해서 알고 있다면 해결 할 수 있는 문제입니다.

[원본 링크](https://bigfrontend.dev/react-quiz/React-re-render-5)

```tsx
// This is a React Quiz from BFE.dev

import * as React from "react";
import { useState, memo, createContext, useEffect, useContext } from "react";
import { createRoot } from "react-dom/client";

const MyContext = createContext(0);

function B() {
  const count = useContext(MyContext);
  console.log("B");
  return null;
}

const A = memo(() => {
  console.log("A");
  return <B />;
});

function C() {
  console.log("C");
  return null;
}
function App() {
  const [state, setState] = useState(0);
  useEffect(() => {
    setState((state) => state + 1);
  }, []);
  console.log("App");
  return (
    <MyContext.Provider value={state}>
      <A />
      <C />
    </MyContext.Provider>
  );
}

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

## 풀이

```text
"App"
"A"
"B"
"C"
"App"
"B"
"C"
```

B의 부모 컴포넌트인 A가 `memo`로 감싸져 있지만, B는 `useContext`로 `App`의 state를 직접 구독하고 있기 때문에, 다시 렌더링 된다라는 사실을 알고 있다면 어렵지 않게 풀 수 있는 문제였습니다.

# 문제 2 - React re-render 6 - Context

앞선 문제에서는 memo와 context의 관계를 다뤘습니다.이번에는 동일한 개념을 한 단계 더 확장해, Provider의 value 변경이 트리 전체에 어떤 영향을 주는지 살펴보겠습니다.

앞서 다룬 re-render 5 문제와 비슷하지만, 이전에 배운 지식을 재점검 하는 문제입니다.

[원본 링크](https://bigfrontend.dev/react-quiz/react-rerender-6-context)

```tsx
// This is a React Quiz from BFE.dev

import * as React from "react";
import { useState, createContext, useEffect, useContext } from "react";
import { createRoot } from "react-dom/client";

const MyContext = createContext(0);

function B({ children }) {
  const count = useContext(MyContext);
  console.log("B");
  return children;
}

const A = ({ children }) => {
  const [state, setState] = useState(0);
  console.log("A");
  useEffect(() => {
    setState((state) => state + 1);
  }, []);
  return <MyContext.Provider value={state}>{children}</MyContext.Provider>;
};

function C() {
  console.log("C");
  return null;
}

function D() {
  console.log("D");
  return null;
}
function App() {
  console.log("App");
  return (
    <A>
      <B>
        <C />
      </B>
      <D />
    </A>
  );
}

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

## 풀이

```text
"App"
"A"
"B"
"C"
"D"
"A"
"B"
```

[이전 글의 react-re-render-4](/bfe-dev-react-2#문제-2---react-re-render-4)풀이에서도 언급한 헷갈릴 수 있는 내용을 기억한다면 딱히 어려울 내용이 없는 문제입니다.

매우 깔끔하게 요약한 bfe.dev의 discuss 내용이 있어서 발췌하자면

> There are 3 insights needed to solve this:

> Upon re-render, if the component returns new instances of child elements, then of course they will all be rendered
> But if the component's parent provided it with the list of children elements to render, then of course there is no need to render them again
> Unless, they are consumers of the context themself.

당초부터 렌더링 될 목록들을 제시한 `App` 컴포넌트에는 아무런 변화가 일어날 여지가 없기에, `A`에서 아무리 변화가 일어난다고 해도 `A`말고는 딱히 렌더링 될 것이 없습니다. 다만, `B` 컴포넌트는 context로 `A`의 상태 일부를 구독하고 있기에 다시 렌더링 된 것이지요.

조금 더 자세한 설명이 필요하다면, 위 문단에서 언급한 [이전 글의 react-re-render-4](/bfe-dev-react-2#문제-2---react-re-render-4) 풀이를 참고하면 좋을 듯 합니다.

# 문제 3 - useLayoutEffect()

앞선 두 문제에서는 re-render와 context를 중심으로 React의 렌더링 메커니즘을 살펴봤습니다. 이번에는 렌더링 이후 단계에서 실행되는 effect 훅 중 하나인 useLayoutEffect를 다뤄보겠습니다.

`useEffect`말고도, effect를 활용하는 `useLayoutEffect`가 있습니다. `useEffect`와 어떤점이 다른지, 실제 동작 예시를 통해서 살펴보는 풀이까지 함께 다루어 볼까 합니다.

[원본 링크](https://bigfrontend.dev/react-quiz/useLayoutEffect)

```tsx
// This is a React Quiz from BFE.dev

import * as React from "react";
import { useState, useEffect, useLayoutEffect } from "react";
import { createRoot } from "react-dom/client";

function App() {
  console.log("App");
  const [state, setState] = useState(0);
  useEffect(() => {
    setState((state) => state + 1);
  }, []);

  useEffect(() => {
    console.log("useEffect 1");
    return () => {
      console.log("useEffect 1 cleanup");
    };
  }, [state]);

  useEffect(() => {
    console.log("useEffect 2");
    return () => {
      console.log("useEffect 2 cleanup");
    };
  }, [state]);

  useLayoutEffect(() => {
    console.log("useLayoutEffect");
    return () => {
      console.log("useLayoutEffect cleanup");
    };
  }, [state]);

  return null;
}

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

## 풀이

```text

"App"
"useLayoutEffect"
"useEffect 1"
"useEffect 2"
"App"
"useLayoutEffect cleanup"
"useLayoutEffect"
"useEffect 1 cleanup"
"useEffect 2 cleanup"
"useEffect 1"
"useEffect 2"
```

우선 이 문제를 풀기 위해서는 크게 두가지의 사전지식이 필요하다고 생각합니다.

1. effect와 cleanup에 대한 개념적 이해
2. `useEffect`와 `useLayoutEffect`의 차이

cleanup이란, 기존의 컴포넌트가 마운팅 해제 될 때 실행되는 일종의 '정리 함수' 입니다. 렌더링 해제가 될 수 있겠고, 그리고 새롭게 다시 렌더링 될 때도 실행이 되지요.

두번째 토픽에 대해서는 사실 한마디로 요약할 수 있습니다.

> `useEffect`는 렌더링을 블록하지 않지만, `useLayoutEffect`는 렌더링 될 때, 나머지 작업들을 블록하는 최우선순위 작업이다.

[공식 react docs](https://react.dev/reference/react/useLayoutEffect)에서도 이 차이를 다루고 있으며, `useLayoutEffect`는 성능 하락을 야기할 수 있다고, 맨 처음 써놓기도 했습니다. 나머지 작업들을 블록하는 API는 당연히 조심스럽게 쓸 필요가 있으니까요.

여튼 이러한 특징들을 파악하였으므로, 초기 렌더링 이후에, 리렌더 될 때 `useLayoutEffect`의 cleanup과 effect가 먼저 실행되고 `useEffect`의 cleanup들이 실행되고, effect가 실행됨을 볼 수 있습니다. useEffect가 처리되는 시점은 '렌더링 중'이고, `useLayoutEffect`의 모든 과정은 **브라우저의 리페인트 전**에 이루어 져야 하니까요

간단하게 useLayoutEffect의 사용처를 설명하면 다음과 같을 것 입니다.

> useLayoutEffect는 DOM이 업데이트된 직후, 브라우저가 화면을 그리기 전에 실행됩니다.
> 반면 useEffect는 렌더링이 완료된 뒤 비동기로 실행되죠.

> 따라서 레이아웃 측정을 하거나 화면 깜빡임(flicker)을 방지해야 하는 경우라면 useLayoutEffect를 사용하는 편이 안전합니다.

# 마무리하며

정말 빡빡한 구직 일정 이후, 휴식기를 거쳐서 오랜만에 써본 글입니다. 다시 쓰기까지 꽤나 많은 휴식과 용기가 필요했는데, 성공적으로 쓸 수 있어서 참 다행이라고 생각합니다.

앞으로도 bfe 풀이, 그리고 이전까지 제가 글을 쓰지 못했던 면접 관련한 소소한 개인적 에세이도 풀어볼 수 있지 않을까 하는 생각 역시 듭니다.

끝까지 읽어주셔서 감사합니다.
