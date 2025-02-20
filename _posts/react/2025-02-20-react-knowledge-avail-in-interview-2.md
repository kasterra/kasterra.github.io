---
layout: post
title: 면접에서 대답할 수 있는 React 지식 ②
subtitle: React의 근간 Fiber를 약간 더 알아보기
category: react
image: /images/thumbnails/React.png
---

# 들어가며

[저번 글](https://kasterra.github.io/react-knowledge-avail-in-interview-1/) 에서는 JSX, component,element,component instance, reconciliation 과 같은 React의 본질적인 요소를 대략적으로 살펴보았습니다.

이번 글에서는 약간만 깊이를 더해서, Fiber에 대해서 알아보는 글이 되겠습니다. 저번 글에서 여러 개념들을 설명하면서, React가 작업들을 어떻게 효율화 하는지에 대해서 reconciliation 섹션에서 살펴보았지만, 실제로 어떻게 ‘작업’ 들이 진행되는지는 이번 글에서 피상적으로나마 다루고자 합니다. 생애주기 메서드, hook이 실행되는 시점 역시 알아볼 예정입니다.

이번 글의 주제는 단순히 react의 작동원리를 다루는것 뿐만 아니라, react로 앱을 작성하면서 한번정도는 궁금해 해 봤을 react 컴포넌트에서 ‘단 하나의 컴포넌트만 return 해야 하는 이유도 알아 볼 것 입니다. 그리고 에러를 선언적으로 처리하는 `ErrorBoundary`의 작동원리도 간단히 알아보겠습니다.

# Fiber란 무엇인가?

React element가 사실은 단순한 javascript object였던 것 처럼, React Fiber도 단순한 javascript object 입니다. 이 Fiber에 기반하여 현대 React(react 16 이상)에서는 Fiber reconciler가 reconciler의 기본값이 되었습니다.

Fiber는 애니메이션과 반응성에 주요 초점을 둡니다. 그리고 아래와 같은 특징들이 있습니다.

- 작업을 단위로 분할
- 특정 작업들을 우선하게 할 수 있음
- 작업을 중단하고, 다시 재개할 수 있음
- 기존 작업을 재사용 하고, 작업이 필요하지 않다면 취소할 수 있음
- 동기적으로 작동하지 않고, **비동기**로 작동함

## 레거시 스택 기반 reconciler

Fiber가 무엇인지 단순히 역할을 암기하는 것 보다, 왜 이것이 필요했는가 등에 대해서, 약간은 더 본질적인 탐구를 해봅시다. 아까 Fiber의 특징들을 목록들로 나열했던 것과 같이, 레거시 스택 기반 reconciler의 특징 역시 정리해 보겠습니다.

- 스택은 **동기**로 작동함
- 스택이 빌 때 까지 계속 작업을 진행
- 중간 인터럽트 불가

## 기존 reconciler의 문제점

이제 해당 스택 기반 reconciler에서 생길 수 있는 문제들을 예시를 통해서 알아봅시다.

우리의 웹 앱에 텍스트 입력 필드인 `<input>`이 하나 있다고 생각해 봅시다. 입력 필드에다 포커스를 하고 키보드 입력을 한다면, 입력을 한 값이 즉각적으로 해당 필드에 보이는 것이 일반적으로 기대되는 것이지요. 만약 작업들이 중간 인터럽트 등이 불가능 하다면, 이 일반적인 기대를 충족시킬 수 없는 상황이 발생할 수 있습니다.

백그라운드에서 네트워크 요청과, 그 요청에 따른 새로운 요소를 렌더링 해야한다면 어떻게 될까요? 그러한 작업이 진행되고 있는 중에 타이핑이 이루어진다면, 방금 입력한 값이 바로 텍스트 입력 필드에 보이지 않고, 약간의 딜레이와 함께 입력한 값이 한번에 보이게 되는 일이 생길 수 있겠네요

이것이 기존 스택 기반 reconciler의 가장 큰 문제라고 할 수 있겠습니다.

# React 작동의 기반인 Fiber

Fiber는 단순히 React 성능 개선을 위한 하나의 요소로 끝내기에는 상당히 많은 부분을 차지하고 있습니다. React 코어 개발을 더 쉽게 해주는 등의 장점들도 있다고 하나, 우선 react의 핵심 개념들을 제대로 이해하는 것부터 해보지요.

React의 작업의 ‘단위’로 사용되며, React는 fiber를 처리하여, ‘완료된 작업’들로 변환 됩니다. 그리고 이것은 commit이 되어, DOM에 반영이 되어서, 실제 사용자가 볼 수 있게 됩니다.

이러한 작업들은 크게 두 단계로 나눌 수 있겠습니다.

1. render phase(processing)
2. commit phase(commiting)

자세히 알아봅시다.

## 1. Render Phase

- 사용자들에게 **보이지 않는** 작업들을 **비동기**적으로 처리합니다.
- 그리고 이 단계에서는 작업들이 **우선순위**를 조정 할 수 있고, 작업을 일시중지 하거나, 취소 할 수도 있습니다.
- 이 단계 동안에 react 내부적으로 쓰이는 함수인 `beginWork()`가 작업을 시작할 때, 작업을 마무리 할 때`completeWork()`가 호출되게 된다.

## 2. Commit Phase

- `commitWork()`가 호출됩니다
- 이 phase는 **동기**적으로 동작하며, 인터럽트 될 수 없습니다

# Fiber의 몇몇 속성

Fiber는 React작동의 기반이라고 하였고, react의 작업의 단위라는 것도 대략적으로 이제 이해를 했습니다. 그리고 두개의 phase에서 어떠한 식으로 처리가 되는구나 하는 것 까지 대략적으로 느낌을 파악했네요

Fiber에 대해서 조금 더 이야기 해보자면, fiber는 “무언가”와 **반드시 1:1관계**를 가지고 있습니다. 컴포넌트 인스턴스일 수도 있고, DOM 노드일수도 있는 것이지요.

## 무엇에 대응되는지 알려주는 `tag`

그 1:1로 대응되는 “무언가”는 `tag` 속성 안에 저장됩니다. 0부터 24까지의 수로 정의가 되는데 [실제 코드](https://github.com/facebook/react/blob/a4d122f2d192fe0b6480e669cca43c8f953aaf85/packages/react-reconciler/src/ReactWorkTags.js)에서 긁어온것은 다음과 같습니다. 변수들의 이름을 보면, react로 개발을 하면서 봤던 여러 용어들을 볼 수 있습니다. `FunctionComponent`부터 `ForwardRef`, 이번 글에서 더 자세히 알아볼 `SuspenseComponent` 같은 것들이 보이네요

```js
export const FunctionComponent = 0;
export const ClassComponent = 1;
export const HostRoot = 3; // Root of a host tree. Could be nested inside another node.
export const HostPortal = 4; // A subtree. Could be an entry point to a different renderer.
export const HostComponent = 5;
export const HostText = 6;
export const Fragment = 7;
export const Mode = 8;
export const ContextConsumer = 9;
export const ContextProvider = 10;
export const ForwardRef = 11;
export const Profiler = 12;
export const SuspenseComponent = 13;
export const MemoComponent = 14;
export const SimpleMemoComponent = 15;
export const LazyComponent = 16;
export const IncompleteClassComponent = 17;
export const DehydratedFragment = 18;
export const SuspenseListComponent = 19;
export const ScopeComponent = 21;
export const OffscreenComponent = 22;
export const LegacyHiddenComponent = 23;
export const CacheComponent = 24;
export const TracingMarkerComponent = 25;
export const HostHoistable = 26;
export const HostSingleton = 27;
export const IncompleteFunctionComponent = 28;
export const Throw = 29;
export const ViewTransitionComponent = 30;
```

## 실제 레퍼런스를 저장하는 `stateNode`

tag는 어떠한 타입의 데이터와 관련이 있는지 알려준다면, stateNode에서는 실제 해당 객체에 대한 참조를 가지고 있다고 할 수 있습니다.

### Fiber vs React element

React element도 javascript 객체였고, 기본 동작의 한 요소입니다. 또 Fiber의 속성 중에는 `key`, `type`와 같은 react element와 비슷해보이는 속성들도 여럿 있습니다.

Fiber는 React element로부터 생성되는 경우가 많고, `type`와 `key`속성을 공유하는 것 또한 사실입니다. Fiber와 React element가 다른 주요한 점은, **React element는 매번 새로 생성되지만, Fiber는 최대한 많이 재활용된다**는 점입니다. react element는 단순히 외형만을 묘사하는 껍데기일 뿐이고, 상태나 생애주기 메서드, 혹은 hook 같은 것들 까지 관리되는 부분이 Fiber라고 이해하면 대략적으로는 맞습니다.

`createFiberFromElement()`나 `createFiberFromFragment()` `createFiberFromText()`같은 함수가 있는 것으로 대략적으로 느낌을 이해할 수 있었으면 좋을 것 같습니다.

## Fiber 관계 : `child` ,`sibling` ,`return`

이 글을 보시는 여러분들은 트리나 그래프등의 탐색에서 쓰이는 DFS, BFS 라는 탐색 방법을 까먹고 있지 않기를 바라겠습니다. Fiber도 react element처럼 트리를 형성하고, 그 트리들을 처리하기 위한 방법으로 위 방법론들을 사용하거든요.

react element와 약간 다른 부분들도 있습니다. `child` 속성이 **첫번째 자식**을 향한 참조 라는 것입니다.

```html
<div>
  <h1>heading 1</h1>
  <h2>heading 2</h2>
  <h3>heading 3</h3>
</div>
```

이러한 구조에서 `div`에서 나온 fiber의 `child`는 `h1`에서 나온 fiber가 되는 것입니다. `h2`나 `h3`에 접근하기 위해서는 h1의 `sibling` 속성을 활용해서 접근하는 식으로 구성이 되는 것이지요

`return` 속성은 부모를 참조하는 속성입니다. `h1` ,`h2` , `h3` 모두 `return` 속성은 `div` 에 대한 참조가 되는 그러한 식이 됩니다.

# **작업**이란 무엇인가

Fiber에 대해서는 가볍게 훑어 볼 수 있었지만, ‘작업’의 단위라는 말로 뭉뚱그려서 넘어갔습니다. react에서 이야기 하는 ‘작업’ 이란 무엇인지 살펴볼 필요가 있을 듯 하군요

- 상태 변화
- 생애주기 함수
- DOM의 변경

이러한 작업들은 바로 실행될 수 있고, 미래를 위해 스케줄링 될 수도 있습니다.

타임 슬라이싱을 이용해서 이 작업들이 chunk라는 보조 단위로 쪼개질 수도 있는데, 이는 통상 60fps나 120fps로 보이는 현대 브라우징 환경에서, 렌더링 작업을 한번에 처리하지 않고, 주 스레드를 오랫동안 차단하지 않게 하여, 사용자 입력을 원활하게 받을 수 있게 합니다.

또한 애니메이션 같이 높은 우선순위가 필요한 작업들은 `requestAnimationFrame()`을 이용해서 높은 우선순위의 작업들을 스케줄링 하고, `requestIdleCallback()`을 사용해서, 상대적으로 우선순위가 낮은 작업들을 처리할 수 있게 해주지요

# Fiber Tree

React 내부에서는 fiber tree가 **두개** 생성됩니다. 현재 실제 DOM으로 렌더링 되어 우리가 화면에 보고 있는 `current` 트리와, 현재 우리 눈에는 보이지 않는 `workInProgress`라는 또 다른 한 종류의 트리가 있습니다.

`current` 트리는 DOM과 동기화가 되어 있기 때문에 불변성을 유지해야 하고, 실제 비동기 적인 변경 작업 들이 render phase 동안 `workInProgress` 트리에 반영된 다음에, 해당 작업들이 모두 완료가 되면 `current`와 `workInProgress`의 포인터만 바꿔 주는 것으로, 또 다시 변경 작업을 이어나가는 구조입니다.

하지만 단지 트리를 스왑 하는 것으로는 React가 모든 문제를 해결할 수는 없습니다. 비동기적으로 동작하는 render phase는 별 문제가 없지만, 동기적으로 작업이 처리되는 commit phase에서도 작업이 이루어져야 합니다. 가장 대표적인 것이, DOM 작업이나, 생애주기 함수/메서드의 처리죠. render phase에서는 주어진 prop에 따른 UI 계산이라는 ‘순수한 작업’이 React에 의해서 일시정지/중단/재시작 될 수 있기 때문에, DOM 작업이나 생애주기 함수 처럼 ‘순수하지 않은’ 것들은 실행될 수 없고, 어떤 effect가 실행될지 리스트만 뽑아내는 것이 최선이라 볼 수 있겠습니다.

## Effect란?

간단하게 Effect라는 것은, DOM을 수정하거나, 특정 생애주기 함수를 호출하는 것을 말합니다. 이 effect 라는 친구들은 Fiber에 강하게 의존합니다. 주의해야 할 점은 렌더링 되어야 하는 것들을 순수하게 알려만 주는 `render()` 함수나, 기존 렌더링 된 요소를 그대로 써도 되는지 판단하는 `shouldComponentUpdate()`와 같은 함수들은 render phase에서 처리된다는 점을 알아두면 좋을 듯 합니다. `render()`는 사실상 정적인 템플릿이라고 볼 수 있고, `shouldComponentUpdate`는 prop에만 의존하는 순수 함수기 때문이죠.

## Effect가 처리되는 방법

React는 commit phase 동안 effect 목록들을 확인하며, 컴포넌트 인스턴스에 반영합니다. 이 변화들은 사용자에게 보여야 하기 때문에 (즉시 반영, 일관된 UX, 상태와 동기화된 UI)동기적으로 이루어 져야 합니다. 이런 변화는 한번의 연속적인 변경사항(single pass)안에 이루어 지게 됩니다. 어떤 노드 들이 추가/삭제/갱신 되거나, 생애주기 함수를 호출해야 하는지가 이 effect list에 의해서 결정된다고 볼 수 있습니다.

## React가 Fiber tree를 처리하는 방법

Fiber는 `beginWork()`나 `completeWork()`등의 함수로 처리되는 작업의 단위입니다. React는 이것들을 처리하기 위해서, 자식이 없는 Fiber에 도달할 때 까지, `beginWork()`로 깊이 들어갑니다. DFS와 비슷한 접근인 것 이지요. 그리고 그러한 Fiber에 도달하고 작업을 끝내면 `completeWork()`를 호출하는 것으로 작업의 마무리를 선언합니다.

일반적인 트리 탐색과는 다른 점이라면, **재귀로 이루어지지 않는다** 라는 것 입니다. Work loop라는 while 반복문으로 구성된 루프 안에서 이 작업들이 이루어지고, 이러한 방법은 트리 구조가 **하나의 child**와 **하나의 sibling** 그리고 **하나의 return**으로 구성되어 있기에 가능합니다.

## Fiber의 alternate 속성

이 섹션에서 Fiber 트리는 ‘두개’로 구성된다고 하였습니다. 이 `alternate` 속성은 Fiber 트리의 ‘반대편’ Fiber 요소와 연결되어 있어, Fiber 재사용을 극대화 하려는 React의 전략에 도움을 줍니다.

# Fiber의 활용

React를 활용한 개발에서 Fiber의 도움을 받는 부분은 `ErrorBoundary`, `Suspense` 그리고 **Concurrent Mode**가 있겠습니다. React 18의 시대가 되어서 이들이 정규 API가 될 만큼, 정말 실험적인 기술이지만, '선언형'으로 뷰를 작성하는데에 참 도움이 되는 친구들입니다.

## ErrorBoundary

본디 React로 작성된 뷰에서 에러가 발생하게 되면, 아무것도 없는 흰 화면만 덜렁 보이게 되는 것이 기본값 입니다. 이것을 방지하기 위해서 에러가 발생할 수 있는 함수 호출 등에 `catch`문을 부착하면 되지만, 컴포넌트 단위의 `catch`를 하면 좋지 않을까 하는 생각을 충분히 할 법 하다고 생각합니다.

이러한 문제를 해결하는데 도움을 주는 `ErrorBoundary`는 생명주기 메서드 중 하나인 `static getDerivedStateFromError(error)` 그리고 `componentDidCatch(error, info)`가 핵심이 되어 돌아가는 컴포넌트의 명칭이라 볼 수 있습니다.

정식 `React.Component`와 같이 React에서 조립해줘서 나오는 물건은 아니지만, 많은 리액트 개발자들이 `static getDerivedStateFromError(error)`를 통해서 에러 플래그를 활성화 시켜, 폴백 UI를 대신 렌더링하고, `componentDidCatch(error, info)`를 통해서 별도의 에러 로깅 등을 하는 그러한 컴포넌트를 지칭합니다.

해당 생명주기 메서드의 hook 버전은 현재 존재하지 않기에(언젠가는 개발한다고 합니다. [문서](https://react.dev/reference/react/Component#static-getderivedstatefromerror)에 'yet'이라는 표현이 있으니...), 직접 class형 컴포넌트를 작성하거나, [react-error-boundary](https://github.com/bvaughn/react-error-boundary)와 같은 라이브러리 등을 가져와서 사용해야 합니다.

## Suspense

[이전에 Suspense와 react 앱에서의 fetch에 대해 적은 글](https://kasterra.github.io/data-fetching-and-react-suspense/)이 있습니다.

그래도 간단하게 설명하자면, state를 통한 조건부 렌더링으로 명령형으로 작성하기 보다는, 선언형으로 로딩중 일때 렌더링할 컴포넌트와 렌더링이 완료되었을 때의 컴포넌트를 각각 선언함으로서 코드의 가독성을 높이는 방법이라고 볼 수 있겠습니다.

향후에 맨 바닥에서 Suspense를 구현해보는 글을 써볼 예정이고, 완성이 되면 링크하겠습니다. [React Suspense from scratch!](https://www.youtube.com/watch?v=cdOyOgwt9Zc&t=4s)라는 제목의 유튜브 영상을 보고 제작하지 않을까 싶습니다.

## Concurrent Mode

긴급하지 않은 상태 업데이트인 `Transition`을 이라는 개념을 통해서 급하지는 않지만 길게 걸리는 작업 때문에 작업들이 Block 되어 UX를 해치는 것을 해결하고자 나온 방법입니다. 해당 내용에 관해서 정리할 것은 많지만, 글의 분량 조절상 현재로서는 잘 정리되어 있는 [외부 링크](https://blue-tang.tistory.com/95)를 남기고 마무리 하고자 합니다.

# 마치며

취준이라는 것을 하면서, 기존의 지식들을 '면접에서 말할 수 있는 지식'으로 다시 정제하는 일은 쉬운 일만은 아닌것 같습니다. 이번 Fiber에 관해서 정리하면서, 어디까지 깊게 다루어야 하는가에 관한

## 참고 자료

- [What Is React Fiber? React.js Deep Dive #2](https://www.youtube.com/watch?v=0ympFIwQFJw&list=PLxRVWC-K96b0ktvhd16l3xA6gncuGP7gJ&index=3)
- [Error Boundaries - 리액트 공식 레거시 docs](https://legacy.reactjs.org/docs/error-boundaries.html)
- [React-Fiber-아키텍처-딥다이브 - @alsgud8311 velog](https://velog.io/@alsgud8311/React-Fiber-아키텍처-딥다이브)
