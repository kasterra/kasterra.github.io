---
title: React의 useEffect에 대한 간단한 설명과 무한루프 예방하기
layout: post
subtitle: React.js의 강력하지만 조심히 다루어야 하는 도구인 useEffect를 다루는 법을 알아봅시다.
image: /images/thumbnails/react-infinity.png
category: frontend
---

# 들어가며

## 잡설

몇주동안 회사일을 하느라, 블로그에 먼지만 수북히 쌓인것 같습니다. 이번주에는 회사 일을 기적같이 다 끝내놓고, 주말을 즐기면서, 오랫만에 여유를 즐기기도 한 만큼, 그동안 먼지만 쌓여왔던 블로그에 다시 관심을 줘볼까 싶어서 다시 한번 글을 써봅니다.

## 진짜 개요

지금 회사에서 하는 일은 React.js로 프론트엔드 개발을 하는 일입니다. React.js에서 함수형 컴포넌트로 화면들을 작성하다가 보면, `useState`나, `useEffect`같은 Hook들을 쓰게 되는데, `useEffect`를 사용하면서 실수하면 생기는 무한루프 때문에 이런저런 삽질을 많이 했습니다. 이번 글에서는 useEffect가 뭔지 간단하게 짚어보고, 아까 말했던 그러한 무한루프의 원인과, 해당 무한루프를 예방하는 방법에 대해서 써볼까 합니다.

# useEffect에 대해

useEffect는 컴포넌트가 처음 렌더링 되었을 때(`componentDidMount`), 업데이트 되기 전에(`componentDidUpdate`), DOM 상에서 제거될 때(`componentWillUnmount`) 실행될 코드를 정의하는 Hook 입니다. useEffect는 `function`그리고 `deps`라는 두 인자를 받는데, `function`은 아까 말했던 실행될 코드를 정의하는 부분이고, `deps`는 그 코드가 실행될 조건에 대해서 입니다. 이렇게 말로만 풀어놓으면 쉽게 알 수 없으니, 간단한 예시를 들어서 설명하겠습니다.

1. 렌더링 될 때 실행될 코드 작성

```js
useEffect(() => {
  console.log("렌더링 될 때마다 실행됨");
});
```

기본적으로 useEffect에 `deps`가 없다면, useEffect가 들어있는 컴포넌트가 렌더링 될 때마다 실행이 됩니다. 만약에 처음 렌더링 되었을떄만 코드를 실행시키게 하려면 아래와 같이 하면 됩니다.

```js
useEffect(() => {
  console.log("처음 렌더링 될 때만 실행됨");
}, []);
```

`deps`는, 해당 배열에 있는 원소들이 변경 되었을 때, useEffect내의 코드를 실행해라는 뜻입니다. 그런데 그 `deps`를 비워두면, 다시 실행될 조건이 아예 없는 것이니, 처음 렌더링 될 때만 해당 코드가 실행이 되는 것이죠. 특정한 변수가 변경되었을떄만, 해당 코드가 실행되게 하고 싶으면 아래와 같이 코드를 작성하면 될 것임을 예측할 수 있을 것입니다.

```js
useEffect(() => {
  console.log(name);
  console.log("name changed!");
}, [name]);
```

2. Unmount나 업데이트 되기 직전에 실행될 코드 작성

`useEffect`안에 들어가는 첫번째 파라미터 이름이 `function`임에 한번 유의해서 생각해 봅시다. 프로그래밍 언어에서 함수라는 것은, 반환하는 값이 있을 수 있다는 뜻입니다. `useEffect`안에 들어가는 이 `function`의 return은 어떻게 사용될까요? 바로 cleanup 함수로 쓰이게 됩니다. 그러니까, Unmount 되거나 업데이트 되기 직전에 실행될 청소 함수로의 기능을 수행합니다. 해당 기능을 가장 잘 이해하기 좋은 코드라고 생각되는, useWindowSize라는 커스텀 훅 코드를 가져오겠습니다.

```js
function useWindowSize() {
  // Initialize state with undefined width/height so server and client renders match
  // Learn more here: https://joshwcomeau.com/react/the-perils-of-rehydration/
  const [windowSize, setWindowSize] = useState({
    width: undefined,
    height: undefined,
  });
  useEffect(() => {
    // Handler to call on window resize
    function handleResize() {
      // Set window width/height to state
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }
    // Add event listener
    window.addEventListener("resize", handleResize);
    // Call handler right away so state gets updated with initial window size
    handleResize();
    // Remove event listener on cleanup
    return () => window.removeEventListener("resize", handleResize);
  }, []); // Empty array ensures that effect is only run on mount
  return windowSize;
}
```

해당 코드를 보면, `window`의 `resize`이벤트를 listen하다가, 해당 hook을 사용하는 컴포넌트등이 unmount 될 때, event listener가 계속 메모리에 남아있으면 낭비이기 떄문에, `window.removeEventListener`로 정리해주는 모습을 볼 수 있습니다.

`useEffect`에 대한 성명은 이정도로 충분할 것 같습니다. 이제 이 글을 쓴 원인이라고 볼 수 있는 `useEffect`에서의 무한루프에 대해서 다루어 보겠습니다.

# useEffect에서의 무한루프

`useEffect`에서 무한루프가 일어나는 대부분의 원인은, `useState`와 연관이 높을 가능성이 큽니다. 일단 저는 그랬거든요. `useState`는 다들 아시겠지만, state로 쓰이는 변수와, 그 변수를 수정하는 setter를 반환해주는 hook 입니다. 그리고, setter를 호출해서 state를 수정하면 **컴포넌트가 다시 렌더링 됩니다.** 그렇기에 아래의 코드는 무한루프가 나는 코드가 됩니다.

```js
useEffect(() => {
  setCount(count + 1);
});
```

잘 생각해 봅시다. 처음 컴포넌트가 렌더링 될 때, count의 값을, 1증가 시킵니다. 그런데 `setCount`를 통해서 증가시켰기 떄문에 해당 컴포넌트는 **다시 렌더링 되고**, 다시 렌더링 되었기 떄문에, `useEffect`안에 있는 코드가 한번 더 돌고... 훌륭한 무한 루프 코드입니다. 이러한 문제를 해결하는 방법은 여러가지가 있습니다.

1. 특정 값이 변화되었을 때만 해당 코드가 실행되게 하기(`deps` 이용)

```js
import { useEffect, useState } from "react";

function CountInputChanges() {
  const [value, setValue] = useState("");
  const [count, setCount] = useState(-1);

  useEffect(() => setCount(count + 1), [value]);

  const onChange = ({ target }) => setValue(target.value);

  return (
    <div>
      <input type="text" value={value} onChange={onChange} />
      <div>Number of changes: {count}</div>
    </div>
  );
}
```

참 간단한 코드입니다. value가 몇번 변경되었는지를 확인해서, value가 변경될 때 마다, count를 갱신하는 식인 것이지요.

2. useRef를 사용해서 해결하기

```js
import { useState, useRef } from "react";

function CountInputChanges() {
  const [value, setValue] = useState("");
  const countRef = useRef(0);

  const onChange = ({ target }) => {
    setValue(target.value);
    countRef.current++;
  };
  return (
    <div>
      <input type="text" value={value} onChange={onChange} />
      <div>Number of changes: {countRef.current}</div>
    </div>
  );
}
```

useRef가 반환하는 객체의 `current` 속성은 state처럼 변경될 때마다 컴포넌트가 다시 렌더링 되지 않는다는 점을 이용해서 아래와 같이 구현할 수 있겠네요.

# useEffect에서의 무한루프 : Object를 다룰 때

위와 같은 상황에서 useState로 원시 타입 변수가 아닌 Object들을 다룰 때에도 같은 문제가 생기는데, Object자체의 특성 때문에 생기는 몇몇 문제들이 더 있는지라 해당 내용도 다루어 보고자 합니다. 하단의 코드는 특정 입력에서 secret라는 값이 몇번이나 입력되었는지 세는 간단한 카운터 예제입니다.

```js
import { useEffect, useState } from "react";

function CountSecrets() {
  const [secret, setSecret] = useState({ value: "", countSecrets: 0 });

  useEffect(() => {
    if (secret.value === "secret") {
      setSecret((s) => ({ ...s, countSecrets: s.countSecrets + 1 }));
    }
  }, [secret]);

  const onChange = ({ target }) => {
    setSecret((s) => ({ ...s, value: target.value }));
  };

  return (
    <div>
      <input type="text" value={secret.value} onChange={onChange} />
      <div>Number of secrets: {secret.countSecrets}</div>
    </div>
  );
}
```

사실 이 코드도 무한루프 문제를 안고 있는 코드입니다. `secret` 변수가 변경 되는 순간마다 , useEffect내의 코드가 한번 실행이 되고, `secret.value`가 `"secret"`이 되면 `s.countSecrets`가 증가하는데, 문제는 `s.countSecrets`가 변경되는것이 secret를 변경시킨다는 사실입니다. 물론 이 문제에 대해서도 해결방법은 있습니다.

1. `deps` 배열에 오브젝트 통째로 넣지 말기

정말 당연한 말입니다. 저기 `useEffect`의 `deps`에 `secret`을 넣는게 아니라, `secret.value`를 넣었다면, `secret.countSecrets`가 변경된다 해도 딱히 문제가 없을것인것은 자명한 이치입니다.

```js
import { useEffect, useState } from "react";

function CountSecrets() {
  const [secret, setSecret] = useState({ value: "", countSecrets: 0 });

  useEffect(() => {
    if (secret.value === "secret") {
      setSecret((s) => ({ ...s, countSecrets: s.countSecrets + 1 }));
    }
  }, [secret.value]);

  const onChange = ({ target }) => {
    setSecret((s) => ({ ...s, value: target.value }));
  };

  return (
    <div>
      <input type="text" value={secret.value} onChange={onChange} />
      <div>Number of secrets: {secret.countSecrets}</div>
    </div>
  );
}
```

# 번외 : prop으로 넘어온 useState의 setter

이 내용은 제가 회사 일을 하면서 겪은 문제였는데, 해당 문제를 제 블로그 글을 읽는 분들은 겪지 말았으면 해서 한줄 적어보는 내용입니다. useState의 결과물로 나온 setter를 prop으로 넘기는 일이 꽤나 자주 있습니다. 이렇게 prop으로 setter를 넘기는 이유는 자식 컴포넌트에서 부모 컴포넌트로 값을 넘기기 위함입니다. 이 패턴은 꽤나 편리하고, 그렇게 구조상으로 문제가 없어서 많이 쓰이는데, 이게 문제가 되는 경우가 있습니다.

일단 결론부터 이야기 하자면, setter로 값을 변경하면, **setter자체도 변경됩니다.** 그렇기 때문에, useEffect에서 state의 값을 변경할 일이 있다면, 해당 state의 근원이 되는 useState는 해당 컴포넌트에 두는 것이 좋습니다. useEffect의 `deps` 배열에 해당 state의 setter를 넣지 않는 방법도 있긴 있으나, 그러면 ESLint에서 exhaustive-deps-warning 라는 경고를 띄우면서 여러분의 멋진 리액트 코드 컴파일 결과에 멋지지 않은 노란색 경고들이 나올 것입니다.

# 마무리

이번 글에서는 제가 react.js로 회사 일을 하면서 겪은 일들 중 하나인, useEffect와 무한루프에 대한 고찰에 대한 글을 적어 봤습니다. 사실 몇주동안 일하면서 주니어 개발자로 겪은 개발 이슈들이 상당히 많지만, 그간 일이 바빠서 잊기도 하고 해서, 100% 기억이 나지 않는것이 아쉽네요. 앞으로도 여유가 날 때마다, 주말에 이렇게 글을 적어서 올리면서 개인 역량 강화에도 힘써야 하겠다라는 생각이 듭니다. 끝까지 읽어주셔서 감사합니다.

# 참고 자료

- <https://ko.reactjs.org/docs/react-component.html>
- <https://usehooks.com/useWindowSize/>
- <https://krpeppermint100.medium.com/js-useeffect%EB%A5%BC-%ED%86%B5%ED%95%9C-react-hooks%EC%9D%98-lifecycle-%EA%B4%80%EB%A6%AC-3a65844bcaf8>
- <https://dmitripavlutin.com/react-useeffect-infinite-loop/>
