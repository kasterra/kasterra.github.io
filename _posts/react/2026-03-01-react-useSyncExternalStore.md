---
layout: post
title: 외부의 데이터를 React의 생명주기 속으로 - useSyncExternalStore
subtitle: React 18+의 concurrent mode에서도 안전하게 외부 상태를 사용하기
category: react
image: /images/thumbnails/React.png
---

## 들어가며 - React 생명주기와 외부 상태의 충돌

React의 생명주기라는 제약사항이자 규칙은, 프론트엔드 코드를 작성할 때, 예측가능성을 준다는 점에서 인지적 부담을 줄여주는 꽤 괜찮은 도구라고 저는 생각합니다.

다만 React를 통한 개발에서 사용하는 상태는 모두 개발자의 로직에서 오지는 않습니다. 웹 FE 환경을 놓고 보자면, 사용자가 인터넷에 연결되어 있는지(`navigator.onLine`)도 있겠고, 화면의 크기 등을 활용하는 media query(`window.matchMedia(query)`)도 있을 수 있겠습니다. React의 입장에서는 '외부'의 상태이지만, React를 통해 웹 FE 개발을 한다고 하여, UI에 대해 관련성이 높을 수 있는 해당 요소들을 사용하지 못하는것은 상당히 불편하죠.

이러한 문제를 해결하기 위해서 React에서는 `useSyncExternalStore`라는 외부의 상태를 구독할 수 있는 hook을 제공합니다.

### `useSyncExternalStore` API 사용법

아래와 같은 형태로 사용할 수 있습니다.

```ts
const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)
```

subscribe는 외부 상태의 변경을 구독하는 api (addEventListener와 같은), getSnapShot은 현재의 상태를 가져오는 api, getServerSnapShot은 SSR 환경 등에서 사용할, 초기값 fallback을 가져오는 함수입니다.

여기서 중요한 부분은 **getSnapshot은 반드시 동기적으로 현재 상태를 반환해야 한다** 라는 점 입니다.

## 왜 useEffect는 부족한가 (tearing 문제와 SSR)

초입 부분을 읽고 아래와 같은 궁금증이 분명 드셨을 것이라고 예상합니다.

> useEffect 같은 기존의 hook으로도 해결할 수 있는 문제 아닌가?

맞습니다. React 18 시대의 이전 React에서는 query에 따라서 eventListener하나 달아주면 react 생명주기에 맞게 사용할 수 있었습니다.

```ts
useEffect(() => {
  const mql = window.matchMedia(query);
  setMatches(mql.matches);

  const handler = () => setMatches(mql.matches);
  mql.addEventListener("change", handler);

  return () => mql.removeEventListener("change", handler);
}, [query]);
```

그러면 기존에 잘 작동하는 도구가 있는데 왜 새로운 hook을 react 팀에서 발표한 것일까요? 가장 핵심적인 이유는, React 18 부터 도입된 non-blocking transition 업데이트에서([이전에 jank 줄이기를 주제로 쓴 글](/react-reduce-jank-using-transition)) 같은 상태임에도 다른 UI가 렌더되는 tearing 현상이 발생할 수 있고, SSR과 초기 hydration에서 mismatch가 발생할 수 있는 문제가 있기 때문입니다.

### tearing에 대한 간단한 설명

우선 [잘 정리된 글](https://sungjihyun.vercel.app/blog/tearing)이 있어, 해당 글을 링크하고, 이 섹션에서는 최대한 간단하게 다루어 보겠습니다.

Concurrent 기능이라고도 불리는 non-blocking transition update는, 즉각 업데이트 되어야 하는 상태와 그렇지 않은것을 구분할 수 있기에, 무거운 렌더링 동작을 non-block 형태로 미룰 수 있습니다. 무거운 렌더가 발생하는 도중에 외부의 상태 변경이 일어난다면, 누군가는 과거의 상태에 기반해 렌더할 것이고, 누군가는 현재의 상태를 보고 렌더하는 불일치가 일어날 것이죠. 그것이 tearing 입니다.

즉, 하나의 화면 안에서 일부 컴포넌트는 이전 상태를, 일부는 새로운 상태를 보고 그려지는 현상입니다. UI가 말 그대로 '찢어지는'거죠

그래서 [react 공식 useSyncExternalStore 한국어 문서](https://ko.react.dev/reference/react/useSyncExternalStore)에서도 다음과 같이 설명해주고 있습니다. non-block transition을 취소하고, 다시 최신 값 기반으로 렌더해서 tearing을 막아주는 것 이지요

> non-blocking transition 업데이트 중에 스토어가 변경되면, React는 해당 업데이트를 blocking으로 수행하도록 되돌아갑니다. 구체적으로, 모든 Transition 업데이트에 대해 React는 DOM에 변경 사항을 적용하기 직전에 getSnapshot을 한 번 더 호출합니다. 처음 호출했을 때와 다른 값을 반환하면, React는 처음부터 다시 업데이트를 시작하고, 이번에는 blocking 업데이트를 적용하여 화면의 모든 컴포넌트가 같은 스토어 버전을 반영하도록 합니다.

### SSR과 hydration

Gatsby나 Next와 같은 SSR 기능을 사용한 리액트 기반 프레임워크를 쓰다보면 'hydration failed' 와 같은 에러 메세지를 볼 경우가 꽤나 있습니다. 외부 상태의 값이 클라이언트에서만 읽을 수 있다면, SSR에서 내려준 초기값과 맞지 않아, SSR의 장점을 상실하게 되기 때문에 나오는 오류이죠.

`useSyncExternalStore`는 해당 문제를 해결하는 도구이기도 합니다. `useSyncExternalStore`의 마지막 인수인 `getServerSnapshot`의 값을 넣어서, 해당 오류를 발생하지 않도록 수정할 수 있겠지요.

당연한 이야기지만 getServerSnapshot은 서버와 클라이언트 초기 렌더에서 동일한 값을 반환할 수 있게 해야겠지요

## 실제 사용 예시

간단히 media query를 구독하는 예제는 다음과 같습니다.

```ts
import { useSyncExternalStore } from "react";

export function useMediaQuery(query: string): boolean {
  return useSyncExternalStore(
    (callback) => {
      const mql = window.matchMedia(query);
      /*
      옛날 브라우저를 지원하기 위해서는 addListener로 부착해야 하나,
      모던 브라우저 타겟으로는 아래로도 충분합니다.
      */
      mql.addEventListener("change", callback);
      return () => mql.removeEventListener("change", callback);
    },
    // getSnapshot은 반드시 동기적으로 현재 값을 반환해야 한다
    () => window.matchMedia(query).matches,
    () => false, // SSR 폴백
  );
}
```

또한, 외부 전역 상태 관리 라이브러리인 `redux`도, redux 8 부터 내부 구현이 `useSyncExternalStore`기반으로 바뀌었을만큼, react 18 이상 버전에서 '안전하게' 동작하기 위해서는 필요하다고 볼 수 있겠네요.

## 마무리 - When to use or not

사실 기존의 도구인 useEffect + useState가 나쁜 것은 아닙니다. 단순 외부 상태 기반 side-effect 처리에는 큰 문제가 없었으니까요. 따라서, 굳이 쓸 필요가 없는 상황을 정리하면 다음과 같습니다.

- 단순 side-effect 처리에는 필요 없다
- 외부 store가 아닌 내부 state에는 쓰지 않는다
- async 데이터 fetch용 훅이 아니다 (Sync니까)

위 사항만 파악해서 적재적소에 사용할 지식이 된다면, UI 개발에 있어서 큰 도움이 되리라 생각합니다. 끝까지 읽어주셔서 감사합니다.
