---
layout: post
title: 면접에서 대답할 수 있는 React 지식 ③
subtitle: React의 hook을 직접 만들어보며 조금 더 알아보기
category: react
image: /images/thumbnails/React.png
---

# 들어가며

[저번 글](https://kasterra.github.io/react-knowledge-avail-in-interview-2/)에서는 fiber를 살짝 찍먹 해보는 느낌으로 다루어 봤습니다.

이번 글에서는 react hook의 동작 원리와, react hook이 지켜야 하는 원칙을 알아보고자 합니다.

# React hook를 사용하면서

React hook은 클래스 컴포넌트를 사용하지 않고도, 리액트의 기능을 사용할 수 있도록 해 줍니다. 상태 관리(`useState`, `useReducer`)를 사용하거나, 생명주기(`useEffect`), 기타등등(`useRef`, `useContext` 등)의 기능을 사용할 수 있지요.

React hook을 쓰면서, 조건문이나 반복문 등에 hook을 사용하지 말라는 경고를 본 기억이 있을 수 있을 것이고, react docs에도 그러한 내용이 있습니다.

![rule-of-react](/images/react/react-knowledge-avail-in-interview/3/react-hook-rule.png)

여기에 적혀 있는 내용들을 조합하면, 조건부로 호출되거나, 여러번 호출 될 수 있는 상황에서 hook을 사용하지 말라는 내용이라는 결론을 얻어낼 수 있습니다. 이 이유를 가장 잊지 않으면서 확실하게 알 수 있는 방법은, 직접 만들어 보고 깨닫는 것 입니다. 생각보다 어렵지 않으니 진행해 봅시다.

# 직접 hook을 만들어 보기

여러가지 hook이 있겠지만, react hook을 사용해서 개발 한다면, 안 쓰기 힘든 `useState`와 `useEffect`를 만들어 보고자 합니다.

클로저의 개념과 배열이라는 것이 무엇인지만 알면 '너도 할 수 있다' 정도로 간단하니, 진행해 보도록 하겠습니다.

## useState 만들어보기

우선 useState부터, 대략적인 형태를 잡아봅시다.

```jsx
const ReactX = (() => {
  const useState = (initialValue) => {
    let state = initialValue;

    const setterFunction = (newValue) => {
      state = newValue;
    };

    return [state, setterFunction];
  };
  return {
    useState,
  };
})();

const { useState } = ReactX;

const Component = () => {
  const [counterValue, setCounterValue] = useState(1);

  console.log(counterValue);

  if (counterValue !== 2) {
    setCounterValue(2);
  }
};

Component();
Component();
```

물론 이것은 우리가 알고있는 useState처럼 동작하지 않습니다. 원본 useState였다면, 1이 로깅 되고 2가 로깅 되어야 하는데 그렇지 않고, 1만 로깅 되니 말이지요.

어떤 점에서 그러하고 왜 그런지에 대해서 생각해 봅시다.

`setCounterValue(2)`를 통해 state가 2로 갱신이 되도, 그것은 첫번째 component에서 호출 된 useState속에서나 그렇지, 두번째 호출에서는 그렇지 않습니다. 다른 함수이기 때문에 상태가 보존이 되지 않기 때문이지요.

이것을 어떻게 고칠 수 있을까요? 상태가 있는 함수라고 하면, ‘클로저’라는 개념이 떠올랐다면, 잘 생각 하셨습니다. react에서도 hook을 이용한 상태를 보존하기 위해서 클로저를 사용합니다. 클로저를 사용하는 형태로 수정해 `state`라는 변수를 외부 스코프에 넣어서, 이 상태가 보존되게 해 봅시다.

```jsx
const ReactX = (() => {
  let state;
  const useState = (initialValue) => {
    if (state === undefined) {
      // 초기화가 되어 있지 않을때만, 초기값으로 초기화
      state = initialValue;
    }

    const setterFunction = (newValue) => {
      state = newValue;
    };

    return [state, setterFunction];
  };
  return {
    useState,
  };
})();

const { useState } = ReactX;

const Component = () => {
  const [counterValue, setCounterValue] = useState(1);

  console.log(counterValue);

  if (counterValue !== 2) {
    setCounterValue(2);
  }
};

Component();
Component();
```

이제 우리가 처음 의도한 대로 1과 2가 로그에 찍힌다만, 이것 만으로는 React의 useState를 온전히 구현했다 할 수 없습니다. 사실 꽤나 큰 문제가 남아 있습니다.

상태를 `state`라는 변수 하나에만 담아서 단 하나의 상태만 담을 수 있는 것입니다. 이제 이를 해결하기 위해서 `state`를 배열로 관리하게 수정 해봅시다.

배열에 여러 데이터를 저장하고, 필요한 데이터를 접근하려면, 너무나도 당연하게도 인덱스 또한 관리해야 합니다.

이 명세들을 포함해서 상태 저장 및 인덱스 관리를 추가한 코드는 다음과 같습니다.

```jsx
const ReactX = (() => {
  let state = [];

  let index = 0;

  const useState = (initialValue) => {
    const localIndex = index;
    index++;
    if (state[localIndex] === undefined) {
      // 초기화가 되어 있지 않을때만, 초기값으로 초기화
      state[localIndex] = initialValue;
    }

    const setterFunction = (newValue) => {
      state[localIndex] = newValue;
    };

    return [state[localIndex], setterFunction];
  };

  const resetIndex = () => {
    index = 0;
  };
  return {
    useState,
    resetIndex,
  };
})();

const { useState, resetIndex } = ReactX;

const Component = () => {
  const [counterValue, setCounterValue] = useState(1);

  console.log(counterValue);

  if (counterValue !== 2) {
    setCounterValue(2);
  }
};

Component();
resetIndex(); // 다시 렌더링 될 때는 인덱스 초기화가 필요함
Component();
```

`resetIndex`라는 보조 메서드를 하나 더 추가 하였습니다. 이게 없으면 또 1,1 이 출력되는데, 인덱스가 계속 증가 되기만 하면 이전 인덱스를 찾아갈 수 없을 것이기 때문이라는 매우 당연한 이유 입니다. 컴포넌트 렌더링을 여러번 한다고 해서 상태가 유실되면 상태 보존이라는 의미가 없지요.

이렇게 만든 useState는 실제 리액트의 useState를 대체할 수는 없습니다. 리렌더링 시 인덱스 관리나, 실제 리렌더링을 하는 그러한 로직이 없지만, 적어도 상태관리 측면에서는 이 로직을 기반으로 돌아간다고 할 수 있습니다.

이제 왜 hook을 조건문이나 반복문 내부에서 부르면 안되는지가 명확해 졌습니다. 조건문이나 반복문이나 기타 조건적으로 호출 되는 블록 안에서는, 해당 코드가 실행되는 순서가 런타임이 되어야만 알 수 있기 때문에, 올바른 인덱스를 찾아가는 것이 불가능 해지기 때문이 되는 것이지요.

## useEffect 만들어보기

다음은 useEffect도 구현해 보겠습니다. useState만큼 많이 쓰는 hook일 뿐더러, useState처럼 배열과 JS의 클로저와 관련된 지식만 있으면 이해할 수 있는 구현이니, 큰 난이도 문제도 없을 것 같습니다.

`useEffect`는 callback 함수와 dependencyArray를 받습니다. dependencyArray 안에 있는 것이 이전 렌더링 때의 값과 다르다면, callback을 실행하는 hook 임을 알고 있을 것 입니다.

useState도 렌더링이 될 때 값을 보존하기 위해서 외부 스코프의 배열을 사용했는데, useEffect의 의존성 배열을 관리하기 위해서도 같은 방법을 쓰면 될 것이라고 어렵지 않게 생각할 수 있을 것 입니다.

useEffect에 필요한 정보들도 담아야 하니, state라는 배열 명 대신에 hooks 라는 조금 더 일반적인 변수 이름을 사용해서 간단히 작성해 봅시다.

```jsx
const ReactX = (() => {
  let hooks = [];

  let index = 0;

  const useState = (initialValue) => {
    const localIndex = index;
    index++;
    if (hooks[localIndex] === undefined) {
      // 초기화가 되어 있지 않을때만, 초기값으로 초기화
      hooks[localIndex] = initialValue;
    }

    const setterFunction = (newValue) => {
      hooks[localIndex] = newValue;
    };

    return [hooks[localIndex], setterFunction];
  };

  const useEffect = (callback, dependencyArray) => {
    let hasChanged = true;

    const oldDependencies = hooks[index];

    if (oldDependencies) {
      hasChanged = false;

      dependencyArray.forEach((dependency, index) => {
        const oldDependency = oldDependencies[index];
        const areTheSame = Object.is(dependency, oldDependency); // NaN등의 edge case를 처리하기 위함
        if (!areTheDame) {
          hasChanged = true;
        }
      });
    }

    if (hasChanged) {
      callback();
    }

    hooks[index] = dependencyArray;
  };

  const resetIndex = () => {
    index = 0;
  };
  return {
    useState,
    resetIndex,
  };
})();

const { useState, resetIndex } = ReactX;

const Component = () => {
  const [counterValue, setCounterValue] = useState(1);

  console.log(counterValue);

  if (counterValue !== 2) {
    setCounterValue(2);
  }
};

Component();
resetIndex(); // 다시 렌더링 될 때는 인덱스 초기화가 필요함
Component();
```

해당 useEffect 를 테스팅 하기 위해서

```jsx
useEffect(() => {
  console.log("useEffect");
}, []);
```

또는

```jsx
useEffect(() => {
  console.log("useEffect");
});
```

를 컴포넌트에 넣어보면 우리가 일반적으로 알고 있던 useEffect와 동일하게 위 케이스는 단 한번만 로그가 찍히고, 아래 케이스는 매 렌더링마다 찍힘을 확인할 수 있습니다.

# Hook의 원칙. 왜 이러한 역할을 하는 hook은 없는가?

글 작성에 참고가 된 자료가 react hook이 태동하여 이제 막 정착을 하던 시기의 내용이라, 이 부분에 대해서는 그러려니 하고 넘기는 사람들이 많지만, 꽤나 의미가 있을 수 있는 질문이라 생각합니다.

<https://overreacted.io/why-isnt-x-a-hook/> 라는 유명한 글이 있습니다. 글을 읽기 귀찮으신 분을 위해서 간단히 설명하자면, `PureComponent` 역할을 직접적으로 하는 hook은 왜 없지? 하는 의문을 지금부터 품어보시면 됩니다. `useMemo`라는 물건이 있지만, 아예 통으로 렌더링을 스킵하는 통큰 hook이 왜 존재하지 않는지, 정확하게는 왜 존재할 수 없는지 이야기를 해보도록 하겠습니다.

hook이 지켜야할 두가지 성질이 있습니다.

- 조합(composition)을 해도 견고해야 함 : 다른 hook과 충돌을 일으키지 말아야 합니다.
- 디버깅(debugging)이 용이해야 함 : 버그를 쉽게 찾을 수 있어야 합니다. 어떤 함수나 라이브러리이건 마찬가지지요.

`PureComponent` 역할을 직접적으로 하는 가상의 hook `useSkipRender` 라는 hook이 왜 현실의 react에 추가될 수 없는지에 대해서 이 두가지 조건을 통해 알아봅시다.

이 hook은 두가지 조건을 다 어깁니다.

```jsx
const DisplayDoubleCounter = ({ counter1, counter2 }) => {
  useSkipRender((oldCounter1) => oldCounter1 !== counter1, counter1);
  useSkipRender((oldCounter2) => oldCounter2 !== counter2, counter2);

  return (
    <div>
      <p>Counter1 : {counter1}</p>
      <p>Counter2 : {counter2}</p>
    </div>
  );
};
```

만약 counter1이 변경되었고, counter2가 변경이 되지 않았다면, 두번째 `useSkipRender` 에 걸려서 리렌더가 되지 않을 것이고, vice versa 역시 같은 방식으로 생각할 수 있습니다. `useSkipRender`가 다른 하나를 방해하고, 다른 `useSkipRender`를 부숴먹는다는 사실을 알 수 있지요. 조합이 가능해야 한다는 원칙을 어긴 셈입니다.

다른 하나인 디버깅에 대해서 생각해봐도, 왜 이것이 렌더링이 안되는지 알아보자 하면서 참 많은 시간이 걸릴것으로 예측할 수 있지요

# 마치며

React hook은 클로저 + 배열이다. 이 한마디로 이번 포스트의 핵심을 정리할 수 있다고 생각합니다. 면접에서 대답할 수 있는 React 지식 시리즈 중에서 제가 감히, ‘가장 쉽게 이해할 수 있는’ 섹션이라고 감히 생각합니다.

혹시나 이 글이 한번에 이해가 되지 않는다면, hook 구현을 스탠드업 코미디를 하는것처럼 직접 구현해보는 [youtube 영상](https://www.youtube.com/watch?v=KJP1E-Y-xyo)을 보는것도 좋은 도움이 될 것이라 생각합니다.
