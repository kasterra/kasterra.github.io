---
layout: post
title: 면접에서 대답할 수 있는 React 지식 ①
subtitle: JSX, react component, reconciliation에 대해
category: react
image: /images/thumbnails/React.png
---

# 들어가며

기술 면접 등을 준비하다 보면, 너무나도 당연한 이야기이지만, 기술적인 지식의 필요성을 느끼게 됩니다. 그리고, 몇몇 지식들은 피상적으로 알고 있다가, 질문에 답을 못하고, 참 마음이 불편해지기도 하죠. ‘면접에서 대답할 수 있는 React 지식’ 이라는 이름으로 이 글을 쓰면서, 기술 면접에서 React 관련 질문이 나왔을 때, 자신있게 답을 할 수 있는 확률을 높여보려고 합니다.

React를 사용해서 무언가들을 만들면서, 정작 이러한 것들이 어떻게 돌아가는지 실제 소스 코드 레벨은 아니더라도, 이러한 것은 이러이러하게 돌아간다 하는 이야기 정도는 할 수 있어야 한다 생각합니다. 단순히 질-답을 외운다면, 리스트에 없는 질문을 마주했을 때, 본인 머리 위에 로딩 인디케이터가 보이는 아주 재밌는 경험을 할 수 있다 생각합니다. ‘제대로 된 공부’를 해서, 기초가 탄탄한 주니어 개발자로 도약해 봅시다.

이 시리즈의 글들은 정말 괜찮은 구성의 3년전 업로드 된 유튜브 강의를 기반으로 하되, 현재 React 버전의 실정에 맞지 않은 부분이나, 개인적으로 아쉬웠던 부분 등을 보충해서 작성을 해보았습니다.

이 글의 대상 독자는, react를 통해서 웹 앱 프로젝트를 진행해 본 사람들을 대상으로 합니다. 프로젝트를 진행하면서 당연히 해봤을 경험 같은것은 가정 한마디로 넘어가는 형태로 글의 분량을 조절하겠습니다.

# JSX로 작성한 내 코드. 어떻게 돌아가는가?

```jsx
const App = () => {
  return <div>App component</div>;
};
```

우리에게 참으로 친숙한 JSX를 사용한, react element를 리턴하는 react component 입니다. 이 App 이라는 컴포넌트를 `console.log` 찍어보면 어떤 결과물이 나올까요?

![component를 console.log 한 결과물](/images/react/react-knowledge-avail-in-interview/1/component-console-log.png)

거두절미 하고 말하면, JS object가 하나 나오네요. react에서 사용하는 하나의 형식이라 생각하고 넘어가면, 앞으로 설명 할 것들이 조금이라도 더 유기적으로 연결되지 않을까 하면서 계속 설명을 이어 가 보겠습니다.

## 위의 코드가 어떻게 일반적인 JS 코드가 되는가

JSX 라는것은 어디까지나, React 컴포넌트를 조금 더 HTML 처럼 작성하기 위한 문법적 설탕으로, 결론적으로는 브라우저에서 돌아가야 하는 코드이기 때문에, 일반적인 JS 코드로 변환이 된다는 사실은 많이 알고 계실 것 입니다.

JSX로 작성이 된 컴포넌트는 `React.createElement` 함수 호출로 변경되어서, 위의 `App` 컴포넌트 처럼, object를 하나 리턴하게 되었습니다.

그래서 react 17 이전의 시대에서는 `import React from 'react'`를 반드시 위에 달았어야 했고, [react snippet 같은 vscode 확장](https://marketplace.visualstudio.com/items?itemName=dsznajder.es7-react-js-snippets) 에서도 `imr`이라는 단축키로 위를 빠르게 입력하게 해주는 것들도 있었습니다.

하지만 2025년을 살고 있는 우리는 맨 위에 직접 프로그래머가 사용하지도 않는 import를 딱히 작성하지 않고도 react component를 작성하고 있습니다. babel과 같은 트랜스파일러와 react 팀이 합의를 하여서 `jsx`라는 함수를 `react/jsx-runtime`에서 가져오는 것으로 가져오는 것으로 합의를 하여서 react 17이 보급된 이후에 이렇게 될 수 있었습니다.

react-native에서는 react 18버전을 기반으로 하는 0.77 에서도 여전히 `import React from 'react'`가 필요한것 보니, 정확히는 'react-dom'을 사용하는 프로젝트를 위한 babel 설정에 의존한다고 볼 수 있겠습니다. 이 부분은 너무 깊어질 수 있기 때문에, 참조 링크로 최대한 빠르게 설명을 갈음하도록 하겠습니다.

- [컴파일러는 JSX를 React.createElement() 로 바꿀까?](https://devmill.tistory.com/3)
- [그 많던 import React from ‘react’는 어디로 갔을까](https://so-so.dev/react/import-react-from-react/)

## 실제로 우리가 JSX로 컴포넌트를 사용할때는...

`<App/>` 과 같은 이런 형식으로 사용합니다. div 같은 intrinsic 한것도 아닌데, 명시적으로 호출 같은건 안하는데…? 어떻게 돌아가는걸까 하는 생각을 해본 적이 있으신가요?

React 프로젝트 에서는 이런 JSX문법으로 선언된 컴포넌트를 우리가 직접 보지 않는 babel 과 같은 곳에서 바꾸어 줍니다. 함수 문법으로 작성된 컴포넌트라면 할당된 prop을 통해서 호출되고, class 문법으로 작성된 컴포넌트라면, 새로운 인스턴스를 만들고 `render()`메서드를 호출 하는 것으로 말이죠.

이 형태 역시 `console.log`로 직접 확인해 봅시다.

![함수형 컴포넌트 호출 시 console.log 모습](/images/react/react-knowledge-avail-in-interview/1/func-component-console-log.png)

이전에 확인했던 element의 console.log 결과와 유사하지만, `type`에 함수가 들어 있음을 확인할 수 있습니다. 우리가 선언한 함수형 컴포넌트죠.

그렇다면 덤으로 class형 컴포넌트도 찍어보면 어떨까요?

![class 컴포넌트 호출 시 console.log 모습](/images/react/react-knowledge-avail-in-interview/1/class-component-console-log.png)

`type`에 클래스가 들어있군요. 이 역시 우리가 선언한 클래스형 컴포넌트 입니다.

이렇게 react element는 DOM node도 나타낼 수 있지만, 우리가 만들어낸 component instance 또한 나타낼 수 있다는 것 또한 확인할 수 있었습니다.

# react element, react component, react component instance

이 세가지의 정의를 설명해라고 누군가가 물어본다면, 정확하게 대답할 자신이 있으신가요? 일단 저는 가물가물 할 것 같습니다. 그래서 이번 기회에 정리를 해보려고 합니다.

## react element

![component를 console.log 한 결과물](/images/react/react-knowledge-avail-in-interview/1/component-console-log.png)

아까 이 jsx를 console.log한 object를 기억하시나요? 이렇게 render 될 예정인 결과물을 묘사하는 객체를 react element 라고 합니다.

## react component

> element tree를 내놓는 class 또는 함수

라는 간단한 정의입니다. class형 컴포넌트, 함수형 컴포넌트 라는 처음 react를 배웠을 때 배웠던 말들이 기억나네요.

## react component instance

OOP 언어를 배웠다면 클래스와 인스턴스에 대해서 들어본 기억이 있을 것입니다. 네, 그것과 같은 맥락에서 이해하면 됩니다. 클래스형 컴포넌트나 함수형 컴포넌트를 이용해서 실제로 렌더링 될 친구들이고, 단순히 렌더링 될 결과물 이상으로, 생명주기 메서드(class형 컴포넌트)나 react hook(함수형 컴포넌트)를 사용해서 렌더링 이상의 것들 또한 할 수 있지요

클래스형 컴포넌트에서는 `this`를 이용해서 인스턴스에 접근할 수 있고, 함수형 컴포넌트에서는 클로저와 JS 함수의 성질 + 인덱스 등등을 활용한 hook을 통해서 인스턴스에 접근하는 듯한 효과를 낼 수 있습니다. hook에 관해서는 향후 이 시리즈가 진행되어, 관련 글이 작성이 완료되면 링크하겠습니다.

# Reconciliation (feat VDOM)

React가 하는 일의 궁극적인 목적은, react element 들의 tree를 만들어서, 실제 DOM에 반영하는 것 입니다. react element들은 아까 확인한 바와 같이, 그저 javascript object 이기 때문에, 빠르게 이루어집니다. `render()` 페이즈에 이루어 지는 작업이지요.

이러한 react element들의 tree를 virtual DOM 이라고들 합니다. 메모리에 저장이 되지요. 이것 자체는 단순 object이기 때문에, 실제 DOM 으로 옮기는 작업이 필요하고, 처음에는 해당 트리를 생으로 전부 DOM으로 렌더링 합니다.

렌더링 된 이후부터 끝까지 계속 똑같은 상태만 보여준다면 별 일이 없겠지만, 많은 경우에 화면에 표시할 요소들은 변경 됩니다. 이 때 일어나는 것이 reconciliation 입니다.

Reconciliation에 대한 지식을 제쳐두고, 이러한 상황에서 무엇을 할 수 있을까요?

1. virtual DOM이니 뭐니 제쳐두고, 생으로 다시 렌더링 한다
2. 이전 virtual DOM과 새로운 virtual DOM의 차이를 알아내서, 필요한 부분만 다시 렌더링 한다.

DOM을 실제로 그리는 비용은 JS object를 쌓아 올리는 것에 비하면 결코 싸지 않습니다. DOM 변경→리페인트→리플로우의 과정을 거치게 되는데, 실제 DOM을 건드리는 작업을 줄이고, 비교적 가벼운 작업인 object들의 트리를 수정하는 쪽이 효율성 측면에서 유리할 것이 너무나도 자명하지요.

다시 렌더링 해야할 부분을 찾기 위해서 React는 효율적인 diff를 위해 두가지 가정을 합니다.

1. 트리의 element의 타입이 달라지면 해당 element 부터의 서브 tree의 결과도 달라진다. 해당 노드와 서브트리를 다시 렌더링(DOM 연산 수행 필요) 해야한다.
2. key라는 prop은 컴포넌트별로 unique하며, 이 값이 달라지면 1번의 경우처럼 해당 노드와 서브트리를 다시 렌더링(DOM 연산 수행 필요) 해야 한다.

UI 라이브러리인 React의 관점에서는 div에서 span으로 타입이 바뀌면, 하위 타입을 재활용 하기 위해서 추가적인 비교 등을 하지 않고, 해당 노드로부터의 서브트리를 싹 다 렌더링을 해버린다는 전략입니다. 일반적인 트리 비교에서는 유효하지 않을지도 모르나, UI 라이브러리라는 관점에서는, 완전히 새로운 트리를 생성한다 생각하고 비교를 위해 시간을 더 소모하지 않는다는 선택을 하는 것입니다. 기존 컴포넌트를 Unmount하고 새로운 컴포넌트를 Mount 하는 것이지요.

key값에 대해서도, 더 복잡한 트리 내부를 비교하지 않아 비교 시간을 줄이면서, ‘unique’함을 가정해서 더욱 효율적으로 트리 비교 과정을 수행한다고 볼 수 있을 듯 합니다.

## 해당 Diff 과정에서 실제로 일어나는 일

### 1. element의 타입이 달라졌을 때

intrinsic element인 `<h1>`이 `<h2>`가 된다거나, 우리가 만든 element인 `<Component1/>`이 `<Component2/>` 가 되는 등 타입이 바뀌어 버리면, 아까 설명했던 diff의 첫번째 가정으로 인해서, 기존의 트리를 완전히 갈아엎고, 기존 컴포넌트는 Unmount되고, 새로운 컴포넌트가 mount 됩니다.

### 2. key prop의 변경이 발생하였을 때 (unique한 key의 필요성)

특히 동적으로 렌더링 되는 리스트에서 이 상황이 발생합니다. element의 타입이 같으면서, 이전과의 동등성을 비교하는 경우 중에서 가장 흔하게 볼 수 있는 경우겠죠.

원활한 이해를 위해 간단한 상황을 하나 가정해 봅시다. 기존 react element의 트리에서 4개의 서로 다른 컴포넌트가 형제 관계로 있었다고 생각해봅시다. 표현의 복잡성의 함축을 위해서, a,b,c,d 라는 노드가 있었다고 합시다. 여기에서 새로운 노드가 맨 뒤에 추가되기만 해서, a,b,c,d,e 형태가 되었다면 너무나도 당연하게도 a,b,c,d는 그대로 두고, e만 새롭게 렌더링 하게 하면 될 것입니다.

새로운 노드가 맨 앞에 추가되는 상황도 마찬가지 일 것 입니다. a,b,c,d에서 e,a,b,c,d로 바뀌어도, a,b,c,d가 굳이 비싼 DOM 리렌더링을 거칠 당위성은 보이지 않습니다.

그렇다면 기존의 노드와 같은 노드이기 때문에, 비싼 DOM연산을 거치지 않아도 된다 라는것을 어떻게 알 수 있을까요? 위에서도 이야기 했듯, key값의 비교로 최대한 비교를 효율적으로 합니다. React에서는 기본적으로 key 값을 컴포넌트의 순서. 즉 index를 기준으로 하는데, 이렇게 아무것도 해놓지 않았을 때는, 실제 논리적으로 노드들의 변경사항이 생겨도, 정확히 추적할 수 없습니다. 분명히 논리적으로는 노드들의 순서를 바꿨는데 사용자가 보는 DOM이 최신화가 되지 않는 경우가 있을 수 있는 것이지요. 이러한 문제점들이 있기 때문에, index를 key 값으로 사용하는것도 옳지 않다고 할 수 있습니다.

### 3. element의 key 이외의 attribute(혹은 prop)이 달라졌을 때

`div` 의 `className`이나 우리가 만든 컴포넌트의 특정 prop이 바뀌었을때는 어떨까요?

이러한 상황에서는 vdom 트리에 별다른 변경사항이 발생하지 않고, 속성값만 갱신되고, 컴포넌트 인스턴스도 유지되며, 컴포넌트의 상태 또한 유지됩니다.

# Rendering과 setState

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";

const rootElement = document.getElementById("root")!;
const root = ReactDOM.createRoot(rootElement);

root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

```

위 코드는 codesandbox 에서 react app template로 만들어진 파일중에 `src/index.tsx` 입니다. 실제로 render가 되는 부분은 ReactDOM에서 렌더링을 하고 있음을 확인할 수 있지요.

이처럼 흔히 React로 웹앱을 작성한다고 하지만, React는 직접 렌더링을 하지 않습니다. Web 환경에서는 react-dom이 렌더링을 담당하고, 모바일 환경이나 windows, macOS 환경에서 네이티브로 실행될 수 있는 앱이 `react-native`로 구성되는 것을 보면 상당히 직관적이지요. 그렇다면, setState 등으로 컴포넌트 내부 상태값이 변경되었을 때에 해당 컴포넌트와 하위 컴포넌트들이 재렌더링 되는것은 너무나도 익숙합니다. React는 직접 렌더링 하지 않는데 이것이 어찌 가능한걸까요?

그 이유는 react-dom의 코드 한 부분에서 찾을 수 있습니다. 우선 class component의 예시부터 확인해 보면 이러한 **react-dom** 코드가 컴포넌트가 mount될 때 실행이 됩니다.

```jsx
const instance = new Component();
instance.props = props;
instance.updater = ReactDOMUpdater;
```

updater라는 프로퍼티에, `ReactDOMUpdater`가 호출됨을 주목합시다.

그리고 setState가 호출되면 아래와 같은 **React** 코드가 실행됩니다.

```jsx
setState(partialState, callback) {
  this.updater.enqueSetState(this, partialState, callback);
}
```

React에 대해서 조금 더 깊은 곳까지 흥미를 가지고 학습을 하셨던 분이라면, 리액트는 상태 변화가 있을 때, 즉각 렌더링으로 반영하는 것이 아니라, 여러개의 주문을 모아서 비동기적으로 동작한다고 하는 것을 들으셨을 것인데, 그 부분입니다. 여기에서 `this.updater`는 `ReactDOMUpdater`로 할당이 되었기에, DOM으로 반영이 될 수 있는 것입니다.

hook을 사용하는 함수형 컴포넌트의 경우도 살펴봅시다. `useState`와 클래스형 컴포넌트의 updater에 해당하는 부분만 남겨둔 코드는 아래와 같습니다.

```jsx
const React = {
  // ....
  __currentDispatcher: null,

  useState(initialState) {
    return React.__currentDispatcher.useState(initialState);
  },
};
```

여기에서도 `__currentDispatcher` 라는 것이 등장 하고… 이 `__currentDispatcher` 또한 class component와 유사하게 `react-dom` 에서 설정합니다.

```jsx
const previousDispatcher = React.__currentDispatcher;
React.__currentDispatcher = ReactDOMDispatcher;
let result;
try {
  result = Component(props);
} finally {
  React.__currentDispatcher = previousDispatcher;
}
```

`__currentDispatcher`가 설정되고, hook을 사용하는 함수형 컴포넌트이기 때문에 try 블록 내에서 함수 형태로 호출이 되는 것을 확인 할 수 있습니다.

# 마치며

'React, Vue, Angular' 등의 SPA 프레임워크에 대한 지식이 있으신 분이 되기 위한 첫 포스팅 이었습니다. 앞으로 기획되어 있는 글들이 이 시리즈만 해도 5개 정도가 넘게 남았는데, 생각보다 제대로 된 주니어 개발자가 되는 일이 만만치 않은 것 같습니다.

이 글을 적기 위해서 여러 자료들을 다시 읽어보고, 골똘히 생각해보면서, 많은 일을 하는 **javascript 라이브러리** 로서의 react를 다시 생각해 볼 수 있었던 것 같습니다.

![Wait it's all javascript?](https://preview.redd.it/imsorryitjusthadtobesaid-v0-nk1atfvcd28d1.png?width=640&crop=smart&auto=webp&s=90d8213c68e3df5b379ff3d9dabf97066993be21)
r/ProgrammerHumor 에서

끝까지 글 읽어주셔서 감사드리며, 혹시 이해가 안되시거나 문제가 있는 부분 등이 있다면 댓글 등으로 알려주시면 감사하겠습니다!

## 참고 자료

- [How Does React Actually Work? React.js Deep Dive #1](https://www.youtube.com/watch?v=7YhdqIR2Yzo&list=PLxRVWC-K96b0ktvhd16l3xA6gncuGP7gJ&index=1)
- [Rendering Lists - React](https://react.dev/learn/rendering-lists)
- [컴파일러는 JSX를 React.createElement() 로 바꿀까?](https://devmill.tistory.com/3)
- [그 많던 import React from ‘react’는 어디로 갔을까](https://so-so.dev/react/import-react-from-react/)
