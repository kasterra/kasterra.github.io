---
layout: post
title: BFE.dev React Quiz 풀이 - 3
subtitle: React의 신규 문법 Suspense, 제대로 알고 있나요?
category: react
image: /images/thumbnails/React.png
---

# 들어가며

이 글은 [bfe 풀이 2편](/bfe-dev-react-2)에 이어 계속됩니다. 이어서 React 딥다이브 내용을 현실의 예제로 옮겨봅시다!

## 오늘 풀 문제 설명

[2022년에 쓴 data fetching과 Suspense 글](/data-fetching-and-react-suspense)에서 다뤘던, 비교적 새로운 기능인 `Suspense`의 동작을 정확히 이해하고 있는지 확인하는 문제입니다. 해당 API에 대한 배경지식이 부족하다면 먼저 위 글을 읽고 오길 권합니다.

# 문제 1 - Suspense 1

[원본 링크](https://bigfrontend.dev/react-quiz/Suspense-1)

```tsx
// This is a React Quiz from BFE.dev

import * as React from "react";
import { Suspense } from "react";
import { createRoot } from "react-dom/client";

const resource = (() => {
  let data = null;
  let status = "pending";
  let fetcher = null;
  return {
    get() {
      if (status === "ready") {
        return data;
      }
      if (status === "pending") {
        fetcher = new Promise((resolve, reject) => {
          setTimeout(() => {
            data = 1;
            status = "ready";
            resolve();
          }, 100);
        });
        status = "fetching";
      }

      throw fetcher;
    },
  };
})();

function A() {
  console.log("A1");
  const data = resource.get();
  console.log("A2");
  return <p>{data}</p>;
}

function Fallback() {
  console.log("fallback");
  return null;
}

function App() {
  console.log("App");
  return (
    <div>
      <Suspense fallback={<Fallback />}>
        <A />
      </Suspense>
    </div>
  );
}

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

## 풀이

```text
App
A1
fallback
A1
A2
```

`Suspense`의 동작만 이해하면 어렵지 않은 문제입니다. React의 `Suspense`는 대기 중일 때 `Promise`를 *throw*하고, 데이터가 준비되면 값을 *return*하여 데이터 페치를 선언적으로 다룰 수 있게 합니다.

흐름은 다음과 같습니다. _Suspense로 감싼 영역 렌더링 시도 → 렌더링 중 데이터 페치 발생 → 완료 전이면 Suspense의 fallback 렌더링 → 완료되면 감싼 영역을 다시 처음부터 렌더링 시도_. 이때 렌더링은 중단 지점에서 이어지지 않고 처음부터 다시 실행됩니다.

# 문제 2 - Suspense 2

[원본 링크](https://bigfrontend.dev/react-quiz/Suspense-2)

```tsx
// This is a React Quiz from BFE.dev

import * as React from "react";
import { Suspense } from "react";
import { createRoot } from "react-dom/client";

const resource = (() => {
  let data = null;
  let status = "pending";
  let fetcher = null;
  return {
    get() {
      if (status === "ready") {
        return data;
      }
      if (status === "pending") {
        fetcher = new Promise((resolve, reject) => {
          setTimeout(() => {
            data = 1;
            status = "ready";
            resolve();
          }, 100);
        });
        status = "fetching";
      }

      throw fetcher;
    },
  };
})();

function A() {
  console.log("A1");
  const data = resource.get();
  console.log("A2");
  return <p>{data}</p>;
}

function B() {
  console.log("B");
  return null;
}

function Fallback() {
  console.log("fallback");
  return null;
}

function App() {
  console.log("App");
  return (
    <div>
      <Suspense fallback={<Fallback />}>
        <A />
        <B />
      </Suspense>
    </div>
  );
}

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

## 풀이 (React 18.3 기준)

```text
App
A1
B
fallback
A1
A2
B
```

현실적인 상황 가정입니다. `Suspense` 안에는 데이터 페치와 무관한 컴포넌트도 들어올 수 있습니다. 그렇다면 _무관한 컴포넌트도 다시 렌더링되는가?_ 또한 로딩을 유발하는 컴포넌트를 만났을 때 다른 자식들의 렌더링은 어떻게 되는가?를 묻는 문제입니다.

핵심 아이디어는 다음과 같습니다. (출처 : [bfe.dev discussion](https://bigfrontend.dev/react-quiz/Suspense-2/discuss))

> Suspense 'checks' all components. That is, the body code of each component will be executed. Then it waits for each loading it finds.
>
> 요약: Suspense는 먼저 자식 컴포넌트들의 본문을 실행해보며 체크하고, 그 과정에서 로딩을 발견하면 해당 트리를 커밋하지 않고 fallback으로 전환합니다.

즉, 이 케이스에서는 B처럼 동기 렌더링만 하는 컴포넌트도 다시 실행됩니다. 다만 구현 세부는 React 버전에 따라 달라질 수 있으니 *행동의 개요*로 이해하면 좋습니다. 렌더링 비용이 큰 컴포넌트라면 적절한 메모이제이션(예: `React.memo`, `useMemo`)을 고려하는 것이 좋을 것 같군요

# 마무리하며

정식으로 편입된지 얼마 되지 않은 Suspense이다보니, 알면 그렇게 어렵지 않지만, 알기 전까지는 헷갈리는 부분들을 정리해 보았습니다. 면접 준비들이 겹치다 보니, 많은 것들을 하지 못했지만, 꾸준히 진행하면서, 저의 자산이자, 다른 누군가에게 도움이 될 수 있는 이 블로그를 열심히 가꾸어 나가도록 하겠습니다.
