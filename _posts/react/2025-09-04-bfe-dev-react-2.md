---
layout: post
title: BFE.dev React Quiz 풀이 - 2
subtitle: React의 re-render와 batch 처리에 대해 다루는 문제들
category: react
image: /images/thumbnails/React.png
---

# 들어가며

이 포스트는, [bfe 풀이 1편](/bfe-dev-react-1)에 이어서, 계속해서 이어나가는 포스트 입니다.

## 오늘 풀 문제 설명

오늘 풀어볼 문제들을 소제목에서도 언급했듯, re-render에 대한 문제들을 주로 풀어볼 것 입니다. 이번 포스트에서 다룰 문제는 1편에서 다뤘던 문제들에서 다뤘던 원리에다가 몇몇 개념을 더 이해시키기 위한 문제들이기 때문에, 같은 분량 기준으로 조금 더 많은 문제를 다룰 수 있지 않을까 싶습니다.

# 문제 1 - React re-render 3

[원본 링크](https://bigfrontend.dev/react-quiz/React-re-render-3)

```tsx
// This is a React Quiz from BFE.dev

import * as React from "react";
import { useState, useEffect } from "react";
import { createRoot } from "react-dom/client";

function A({ children }) {
  console.log("A");
  return children;
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
      <A>
        <B />
      </A>
      <D />
    </div>
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
"App"
"A"
"B"
"C"
"D"
```

풀이는, [React re-render 1의 풀이](/bfe-dev-react-1#시도했지만-틀린-오답과-교정하는-풀이)에 있는 내용 대로입니다. react fiber가 dfs와 같은 방식으로 렌더링 순서가 결정되기 때문에, `"App"` `"A"` `"B"` `"C"` `"D"`의 순서로 로그가 찍히고, `useEffect`에 의해서, `App` 컴포넌트가 다시 렌더링 되면서, JSX가 다시 객체를 만들어 내게 되기 때문에, 모든 것이 다시 렌더링 되는 간단한 문제입니다.

# 문제 2 - React re-render 4

[원본 링크](https://bigfrontend.dev/react-quiz/React-re-render-4)

```tsx
// This is a React Quiz from BFE.dev

import * as React from "react";
import { useState, useEffect } from "react";
import { createRoot } from "react-dom/client";

function A({ children }) {
  console.log("A");
  const [state, setState] = useState(0);
  useEffect(() => {
    setState((state) => state + 1);
  }, []);
  return children;
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
  console.log("App");
  return (
    <div>
      <A>
        <B />
      </A>
      <D />
    </div>
  );
}

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

## 시도했지만 실패한 답안

```text
"App"
"A"
"B"
"C"
"D"
"A"
"B"
```

처음 이렇게 생각했던 이유는 다음과 같습니다.

처음 마운트 되었을 때에는 `"App"` `"A"` `"B"` `"C"` `"D"` 순으로 로그가 찍힐 것 입니다. 그 다음 `A`의 `useEffect`가 실행되서, `"A"`가 로그에 찍히고, `return`문까지 실행되서, `B`의 로직까지는 실행이 될 것으로 생각했습니다. 그 이상으로는 `App`에서 만들어진(JSX가 만든) object는 변하지 않았으니 더 실행이 되지 않는다고 생각했습니다.

## 실제 정답과 풀이

꽤나 정답에 근접했었지만, 실제 정답은 다음과 같았습니다.

```text
"App"
"A"
"B"
"C"
"D"
"A"
```

결론부터 말하자면, `B`는 렌더링 되지 않는다는 것 이었습니다.  
[`children`](https://react.dev/reference/react/createElement#caveats)—React 엘리먼트는 **불변**이기 때문에 A가 재렌더되어도 새 엘리먼트를 만들지 않으면 [`참조`](https://react.dev/reference/react/createElement#caveats)가 **변하지 않습니다**.

React는 **리렌더 시 최소한의 변경만 DOM에 적용**하므로 ([업데이트 최소화](https://react.dev/learn/render-and-commit))  
같은 `type`이면 **서브트리를 재사용**해 전체를 다시 렌더하지 않습니다([reconciliation: 같은 타입 재사용](https://react.dev/learn/preserving-and-resetting-state#state-is-tied-to-a-position-in-the-render-tree)).

따라서 **로그에는 ‘A’만 추가**되게 되는 것이지요. 리렌더 되면서 새롭게 객체가 생성된 re-render 3번 문제와는 다르게, 단순히 `children` prop으로 넘어간 이 상황에서는 `B`를 렌더하려는 `return`문을 react에서 bailout 하는 것 이었습니다.

# 문제 3 - Automatic Batching 1

[원본 링크](https://bigfrontend.dev/react-quiz/Automatic-batching)

```tsx
// This is a React Quiz from BFE.dev

import * as React from "react";
import { useState } from "react";
import { createRoot } from "react-dom/client";
import { screen, fireEvent } from "@testing-library/dom";

function App() {
  const [state, setState] = useState(0);
  console.log("App " + state);
  return (
    <div>
      <button
        onClick={() => {
          setState((count) => count + 1);
          setState((count) => count * 2);
        }}
      >
        click me
      </button>
    </div>
  );
}

(async () => {
  const root = createRoot(document.getElementById("root"));
  root.render(<App />);

  fireEvent.click(await screen.findByText("click me"));
})();
```

## 정답과 풀이

```text
"App 0"
"App 2"
```

문제가 간단한 만큼, 정답도 간단합니다. 이 문제를 풀이하기 위한 간단한 배경상식 몇 가지를 짚고 넘어가 보도록 하겠습니다.

React의 [`setState`](https://react.dev/reference/react/useState#updating-state-based-on-the-previous-state)에서는 직접 변경할 값을 넣을 수도 있지만, 기존 값에 기반해서 새로운 값을 반환하는 **업데이트 함수**를 넣을 수도 있습니다.

React는 상태 변경을 즉시 동기적으로 처리하지 않고, 특정 페이즈에 **[배치(batching)](https://react.dev/reference/react/useState#batching-of-state-updates)**하여 한 번에 처리합니다.  
이때의 상태값은 호출 즉시 갱신되는 것이 아니므로, 이전 상태값이 필요하다면 앞서 언급한 것처럼 **업데이트 함수를 전달**할 필요가 있는 것이지요.

따라서 첫 번째 `setState`로 `count + 1`이 예약되고, 이어지는 두 번째 `setState`는 그 **업데이트된 값**을 받아 `* 2`를 적용합니다.  
결국 `"App 2"`가 출력되는 이유가 됩니다.

# 문제 4 - React re-render 5 - Context

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

## 정답과 풀이

```text
"App"
"A"
"B"
"C"
"App"
"B"
"C"
```

react context가 무엇인지 알고 있다면, 직관적으로 답안을 작성 할 수 있는 문제입니다.

`useContext(MyContext)`를 사용하는 **소비자 컴포넌트(`B`)는**, `Provider`의 `value`가 바뀌면 자동으로 **재렌더링**됩니다. React 공식 문서에도 “React automatically re-renders components that read some context if it changes.”라고 나와 있습니다.([useContext Caveats](https://react.dev/reference/react/useContext#caveats))

반면, `A`는 **`React.memo`로 감싸졌고**, props도 변경되지 않기 때문에 부모에서 상태가 바뀌어도 **render을 건너뛸 수 있음**을 의미합니다. 공식 문서에서는 “Even when a component is memoized, it will still re-render when a context that it’s using changes. Memoization only has to do with props...”라고 명시하고 있죠.([React.memo and context](https://react.dev/reference/react/memo#updating-a-memoized-component-using-a-context))

렌더 순서는 항상 **깊이 우선(DFS), 왼쪽 자식 → 형제 순서**로 진행되는데, 이는 React reconciliation 처리 방식이기도 해요. 따라서 `A`의 렌더가 스킵되더라도, 같은 트리 위치에 있는 **`B`가 먼저 실행되고**, 그 다음에 **형제 컴포넌트 `C`가 실행**됩니다.

결과적으로 두 번째 렌더에서는 `"App" → "B" → "C"` 순으로 로그가 찍히게 되는 것 입니다.

# 마무리하며

최근 과제 전형들이나, 새롭게 구직용 서류들을 정리하느라, 기존 진행되었던 프로젝트를 손을 잘 대지 못했습니다. 다시 원래 트랙으로 돌아오기 위해서 bfe 풀이로 다시 블로그의 먼지를 털어내는 일을 시작해봅니다. 꾸준히 다시 원래의 루틴으로 돌아왔으면 좋겠네요.

끝까지 읽어주셔서 감사합니다.
