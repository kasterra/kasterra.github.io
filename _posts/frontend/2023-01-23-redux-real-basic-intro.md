---
layout: post
title: 진짜 군더더기 없는 redux 생기초
subtitle: 진짜 다른 첨가물 없이 vanila JS와 redux 본체만 사용한 튜토리얼
category: frontend
image: /images/thumbnails/redux.png
---

# 개요

## 전역 상태 관리의 필요성

**상태** 라는 개념은 프론트엔드에서 정말 빠질 수 없는 매우 중요한 개념입니다. 사용자와 상호작용하는 웹 페이지에서 상태 관리는 빠질 수 없는 개념이지요. 상태는 한 컴포넌트 에서만 쓰일수도 있지만, 여러 컴포넌트에서 공유하는 전역 상태가 있을 수 있지요. 쇼핑몰 서비스를 만든다고 생각해 봅시다. 장바구니에 담긴 제품들의 목록들을 클라이언트에서 관리한다고 할 때, 전역 상태 관리를 따로 하지 않는다면, 부모 컴포넌트 등에서 상태를 내려줘야 하고, 부모-자식 관계가 아닌 꽤나 먼 친척 관계라면, 자신이 쓰지도 않을 상태를 전달하고 하느라 불필요한 코드가 많이 생기겠죠. 이를 'props drilling'이라고 하며, 이것이 Flux 구조가 등장한 이유다 하는 이야기는 참 많이 들어봤을 것입니다. 여기에 대한 더 자세한 이야기를 더 읽고 싶다면 [링크](https://medium.com/@jsh901220/react%EC%97%90-redux-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0-a8e6efd745c9)에 들어가서 확인해 보세요.

## 이 글을 쓰게 된 이유

여러 리덕스 튜토리얼 글에서 Redux는 React에만 묶여 있지 않고, 다른 프레임워크 등에서도 사용할 수 있음을 언급하지만, 여러 글에서는 react환경에서 redux를 사용하는 방법에 대해서만 언급합니다. redux 공식 홈페이지의 튜토리얼도 react 환경에서 사용하는 예시만 올려두었고요.

React로 개발을 하긴 하지만, React에만 매몰되어 있기 보단, '변하지 않는 기술'인 Vanila JS에 집중을 하기 시작하면서 범용적으로 쓰일 수 있는 상태 관리 솔루션인 Redux를 Vanila JS 환경에서 사용하려고 하니, 제 지식의 부족을 느꼈고, 제가 검색 했을 때, 한국어로 쓰인 밑바닥부터 시작하는 Vanila JS 환경에서 사용하는 Redux 튜토리얼 같은것은 보이질 않아서, 관련 자료를 학습하고 제 지식을 정리하기 위해서 이 글을 쓰기 시작해 보기로 했습니다.

## 이 글을 읽기 전에

이 글은 기초적인 Flux 구조에 대한 이해와, Redux가 아니더라도, 다른 상태 관리 라이브러리를 사용해봤다는 전제 하에 쓸 예정입니다. 만약에 그런 경험이 없다면, 아래에 남겨둔 읽을거리를 읽고 오시는게 어떨까요?

- [flux 구조란?](https://velog.io/@alskt0419/FLUX-%EC%95%84%ED%82%A4%ED%85%8D%EC%B3%90%EB%9E%80)
- [상태 관리 라이브러리 zustand docs](https://github.com/pmndrs/zustand)
  - 그렇게 막 길지 않아서 readme 가볍게 읽고 오실 수 있습니다.

# 본격적인 Redux 기초 설명

이 설명은 [Youtube 영상 하나](https://www.youtube.com/watch?v=hMSTO4cpPaQ)에 나온 개념설명을 많이 참조했습니다.

## 정보를 저장하는 Store

Redux에서는 상태를 Store에 넣어서 관리합니다. Store는 `Redux.createStore()`를 이용해서 만들 수 있고, `createStore()` 내부에는 **Reducer** 라는 함수를 안에 넣습니다.

## Reducer

아래는 에러를 일으키지 않는(복잡한 동작도 없는) 매우 간단한 Redux를 사용한 코드 예제입니다.

```js
const reducer = (state = []) => {
  return state;
};
const store = Redux.createStore(reducer);
```

`Redux.createStore`안의 매개변수로 reducer를 넣는다고 했습니다. 왜 reducer라고 부를까요? 여기 함수 `reducer`의 매개변수 이름에서 볼 수 있듯, state(상태)를 reducer에 통과 시켜서, 무언가를 반환하게 하고 있습니다. JS쪽에 익숙한 분이라면, `Array.prototype.reduce`라던가를 통해서 복잡한 값을 하나의 단순한 값으로 만들어내는 `reducer`의 개념에 익숙할 것인데, Redux에서의 reducer도 이와 같은 개념입니다. 다양한 값이 들어있는 state에서 일부만 뽑아내서, 덜 복잡한 값으로 만들어 낸다거나 하는 것이지요.

## Store.getState()

현재 store의 `state`를 반환하게 하는 함수입니다. store에 접근할 수 있다면, state또한 접근 할 수 있다라는 점을 기억하고 넘어갈 수 있으면 좋겠습니다.

# Redux의 핵심! 구독! (`store.subscribe`)

Redux는 전역 상태 관리 솔루션 입니다. 상태가 바뀔때마다 컴포넌트에서 특정한 동작을 실행할 수 있으면 참 여러모로 쓸모가 있지요. 그러한 역할을 `store.subscribe`가 담당합니다.

```js
const reducer = (state = []) => {
  return state;
};

const store = Redux.createStore(reducer);

store.subscribe(() => {
  console.log(store.getState());
});
```

이 코드는 매우 간단하게, `store`의 `state`가 바뀔 때 마다, `getState`로 `state`를 받아와서 log를 찍어주는 매우 간단한 함수가 되겠습니다. 하지만 이 코드들만 가지고서는 아무것도 log에 찍히지 않습니다. 이유는 매우 간단하지요. **아무런 변화가 없으니까요!!**

# 변화를 담당하는 dispatch

앱에서 특정한 동작이 일어나서, 전역 상태를 수정해야 할 일이 생겼다고 합시다. 그러면, 구독자들에게 상태가 변경되었다는 것을 알리기 위해서 발행(dispatch)를 해야겠죠? dispatch를 할 때는, `action`을 넘깁니다. 어떤 dispatch를 할 것이냐는 것이죠. 이 방법 외에는 전역 상태 객체를 수정할 수 있는 방법이 없습니다. 이를 single flow of data라고 말 하지요. Redux의 강력한 장점입니다.

dispatch를 할 때에는 `action`을 넘기는데, 이 형태는 Object입니다. 이유는 사실 간단합니다. 어떤 `action`을 보낼것이냐 하나로도 충분할 때도 있겠지만, 추가적인 데이터가 더 필요할 때도 있으니까요. 이를테면, 유저를 추가하는 액션의 경우에는, 추가할 유저의 정보를 넘긴다거나 하는 것이 있겠지요.

어떤 `action`을 보낼것이냐는 Object의 `type`이라는 필드로 정의됩니다. **필수** 이며, **문자열** 이어야 합니다.

한 모듈에서는 구독을, 또다른 모듈에서는 발행을 하는 그런 구조를 간단히 그려낼 것입니다. 최대한 Redux의 기본에 충실하기 위해서 코드는 아래와 같이 작성하겠습니다.

```js
const reducer = (state = []) => {
  return state;
};

const store = Redux.createStore(reducer);

// module1
store.subscribe(() => {
  console.log("module 1", store.getState());
});

// module2
store.dispatch({ type: "ADD_USER" });
```

이 코드를 실행해보면, `module 1 []` 라는 결과가 나옵니다. dispatch라는 액션이 전달이 된것이죠. 하지만, 아무런 일도 일어나지 않았습니다.

![nothing happened](https://w.namu.la/s/334c65889b1513350b8d79447d56911435ed8b12873c30100ddd1900538d81c5187b1d9958c49026b136e26aab69bbab536f283d369844e6cffc7e5d8471e325b3c815c167653ef97e27682cadb0d1fa1f7265e163ab99683caa159c9f0443a3)

왜냐면 뭘 해야할지 **모르니까요!**

## Reducer에게 무엇을 해야할지 알려주기

그러니까 알려줍시다. reducer의 두번째 매개변수로 action을 지정해주고, log를 찍게 하면 아래와 같은 코드가 될 것입니다.

```js
const reducer = (state = [], action) => {
  console.log("reducer", state, action);
  return state;
};

const store = Redux.createStore(reducer);

// module1
store.subscribe(() => {
  console.log("module 1", store.getState());
});

// module2
store.dispatch({ type: "ADD_USER" });
```

해당 코드를 실행하면 결과는 아래와 같이 나옵니다.

```
reducer [] {type : "@@redux/INITc.e.e.e.o.a"}
reducer [] {type : 'ADD_USER'}
module 1 []
```

처음에 나오는것은 redux의 초기화를 위한 action이고, 두번째는 우리가 직접 요청한 action입니다. 그리고 action이 실행된 이후에 subscribe의 callback이 실행된것을 볼 수 있지요.

dispatch를 실행할 때마다, reducer에 action이 전달되므로, 이를 이용해서 실제 state에 변화를 줄 수 있겠지요.

reducer 부분만 간단하게 적어보면 아래와 같을 것입니다.

```js
const reducer = (state = [], action) => {
  if (action.type === "ADD_USER") {
    return [...state, "foo"];
  }
};
```

이렇게 코드를 바꿔서 돌려보면 잘 작동합니다만, 일반적으로 유저를 추가하는 액션에서는, 추가할 유저에 대한 정보를 입력받고 싶어 하지 이런식으로 매번 똑같은 유저를 추가하고 싶지는 않을 것입니다.

## Action에 추가로 전하는 payload들

`store.dispatch`를 할 때, 객체를 넘겼다는 사실을 잘 기억하실 겁니다. 객체에는 자유롭게 여러 속성을 추가할 수 있지요. payload(적재 화물이라는 뜻)이라는 속성을 추가하여 실제로 반영시켜 봅시다.

```js
store.dispatch({ type: "ADD_USER", payload: "jack" });
```

reducer부분에도 payload에 해당하는 코드를 작업해 줍시다.

```js
const reducer = (state = [], action) => {
  if (action.type === "ADD_USER") {
    return [...state, action.payload];
  }
  return state;
};
```

이렇게 해서 코드를 실행해보면, 전역 state가 올바르게 변경되는것을 확인할 수 있지요.

# 이게 끝?

네 이게 끝입니다. Redux 라이브러리 자체는 50줄 가량 되고, Redux에서 기억할것은 아까 소개드린것이 전부입니다.

- 전역 Reducer
- Store
- 구독 (Subscribe)
- 발행 (Dispatches)

다음 글에서는 Redux의 기초중 기초에서 조금 벗어나서, 점점 앱이 복잡해 지면서 생기는 일들이라던지, 여러 파일로 쪼개진 바닐라 JS로 이루어진 웹앱의 예시를 보면서 실제 Redux를 적용시켜 본다던지 하는 그런 것들을 알아보고자 합니다. 끝까지 읽어주셔서 감사합니다!!
