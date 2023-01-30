---
layout: post
title: React Suspense와 비동기 통신
subtitle: DX와 UX를 둘다 잡을 수 있는 Data Fetching은 어떻게 해야 할까요
category: react
image: /images/thumbnails/reactAsync.png
---

# 🚪 들어가며

웹 프론트엔드 개발을 하면서, 서버와 통신을 하는 일은 정말 많습니다. 로그인을 한다거나, 서버에 저장된 여러 정보들을 불러온다거나 하는 이유 등으로 말이죠. 그리고 그런 서버와 통신하는 방식은 우리가 일반적으로 사용하는 동기적으로 돌아가는 코드가 아닌, **비동기**코드를 사용한다는 것을 잘 알고 계실겁니다.

비동기 통신을 사용하는 특성상, 컴포넌트가 렌더링 될 때, 렌더링에 필요한 모든 데이터가 처음 컴포넌트가 렌더링 될 시점에 없는 경우가 많습니다. 앱 내부의 상태만을 이용하는 다른 컴포넌트와는 다르게, 외부 상태를 이용하는 컴포넌트에서는 그런 점을 감안해서, 컴포넌트 작성 시 달라지는 부분이 있습니다.

이번 글에서는, 그런 비동기 통신을 사용할 때, 우리가 만든 웹을 사용할 사용자 경험(UX)와 코드를 적는 개발자의 편의(DX)의 관점에서 비동기 통신을 React.js에서 작성하는 방법론들에 대해서, 그리고 React의 최신 버전인 React18에서 비동기 통신까지 지원할 수 있도록 더욱 강력해진 `<Suspense/>`의 작동원리에 대해서 한번 다루어 보고자 합니다.

# ⌛ React Suspense란?

Suspense를 영어사전에서 찾아보면, 연기하다를 뜻하는 영단어 suspend의 명사형이라고 나와 있습니다. 그렇다면 React에서 `<Suspense>`는 어떤 역할을 할까요? [React18에서 Suspense를 다루는 문서](https://github.com/reactjs/rfcs/blob/main/text/0213-suspense-in-react-18.md)를 찾아보면 Suspense의 정의에 대해서

> Suspense lets you declaratively specify the loading state for a part of the component tree if it's not yet ready to be displayed

라고 합니다. Suspense는 아직 컴포넌트가 표시 될 준비가 되지 않았을 때, 렌더링 되어야 할 부분을 선언적으로 정의 할 수 있게 해주는 컴포넌트라고 되어 있네요.

Suspense는 그 자체로는 `fetch`나 `axios`와 같이 데이터를 가져오는 라이브러리도 아니고, `redux`나 `recoil`처럼 상태를 관리하는 라이브러리도 아닙니다. 비동기 상태 처리를 도와준다는 점에서 JavaScript의 `Promise`와 유사하다고 생각하는 쪽이 좋을것 같네요.

그럼 Suspense를 사용해서 얻을 수 있는 이점은 무엇이냐 하는 질문이 나올 것 같습니다. 간략하게 이야기 하자면, 위에서 이야기 한것처럼 로딩에 관한 처리를 간략하게 해줄 수 있으며, 기존의 우리가 썼던 코드를 갈아엎지 않고, 쉽게 migration을 할 수 있다는것이 장점이 아닐까 합니다.

# 🤔 Suspense를 사용하는 방법

Suspense는 아직 렌더링할 준비가 안된 컴포넌트를 위해서 사용한다고 했습니다. 기존의 우리가 비동기 통신을 이용해서 컴포넌트를 작성해야 할때는 아래와 같이 했을 것입니다.

```jsx
const [todos, isLoading] = fetchData("/todos");

if (isLoading) {
  return <Spinner />;
}

return <Todos data={todos} />;
```

저를 포함한 많은 프론트엔드 개발자 분들에게 익숙한 코드일 것입니다. `isLoading`이라는 변수를 사용해서 요청의 상태를 확인합니다. 만약 `true`라면 아직 내용물이 준비가 되지 않았으니, `<Spinner>`를 렌더링 하는 로직이지요. 이 코드는 전혀 **틀리지 않았습니다.** 이제 같은 역할을 하는 Suspense를 사용한 코드를 확인해 봅시다.

```jsx
const todos = fetchData("/todos");

return (
  <Suspense fallback={<Spinner />}>
    <Todos data={todos} />
  </Suspense>
);
```

아까의 코드와 다른점을 확인해 봅시다. 이전의 코드는 `isLoading`이라는 변수를 통해서 로딩 상태를 관리했다면, 이 코드는 React Suspense에 의해서 로딩 상태와 Spinner를 렌더링하는 로직을 `Suspense`의 `fallback`이라는 prop으로 **선언적**으로 관리하고 있습니다. React는 `fallback`으로 제공된 것을 네트워크 요청이 종료될 때 까지 렌더링 합니다.

그럼 자연스럽게 이런 질문이 들 것 입니다. '그럼 React는 어떻게 통신이 종료되었는지 아닌지 알 수 있을까?' 이 부분을 이제 데이터를 fetching하는 라이브러리가 담당합니다. 우리가 `axios`나 `fetch`등을 이용해서 통신을 할 때, `async await`나 `Promise`를 사용해서 데이터 통신이 완료되었는지 확인할 수 있는것 처럼요. SWR이나 react-query같은 라이브러리들이 Suspense 대응을 하고 있고, 더욱 많은 라이브러리들이 지원해줄것으로 예상됩니다.

# 🛰️ Data Fetching 방법론

Suspense를 사용해서 어떻게 Data Fetching을 하는지 다루기 전에, 기존의 Data Fetching을 하기 위한 방법론에는 어떤것이 있는지, 또 그것들의 문제점이 무엇인지부터 한번 알아봅시다.

## Fetch-on-render

해당 접근법에서는, **컴포넌트가 마운팅 된 이후**에 네트워크 요청이 발생합니다. 이러한 특징 때문에 "waterfall"이라고 불리는 문제가 생길 수 있습니다. 아래 예제 코드를 살펴봅시다.

```jsx
const App = () => {
  const [userDetails, setUserDetails] = useState({});

  useEffect(() => {
    fetchUserDetails().then(setUserDetails);
  }, []);

  if (!userDetails.id) return <p>Fetching user details...</p>;

  return (
    <div className="app">
      <h2>Simple Todo</h2>

      <UserWelcome user={userDetails} />
      <Todos />
    </div>
  );
};
```

API에서 데이터를 받아오는 일반적인 컴포넌트입니다만, 여기에는 잠재적인 문제점이 있습니다. 만약에 이 컴포넌트 내에 nested된 `<Todos/>` 컴포넌트 또한 API에서 데이터를 fetch해와야 한다면, `<Todos/>`컴포넌트는 `fetchUserDetails()`가 resolve될 때 까지, 대기해야 하고, resolve된 이후에야 fetching을 진행할 수 있습니다. **fetching이 병렬적으로 진행되지 않는다** 라고 간단히 요약할 수 있겠네요.

네트워크 탭을 살펴보면 이것을 더욱 직관적으로 확인할 수 있습니다.
![fetching waterfall](https://blog.logrocket.com/wp-content/uploads/2019/11/waterfall-networks-tab.png)

<div style="display:flex;justify-content:center">개발자 도구에서 살펴본 waterfall (출처 : <a href="https://blog.logrocket.com/react-suspense-data-fetching">logrocket 블로그</a>)</div>

각자 자신의 비동기 통신을 하는 자식 컴포넌트가 많은 컴포넌트에서 이러한 특징은 느리고 불편한 UX를 초래할 것 입니다.

물론 해당 문제를 해결하기 위해서, `UserWelcome` 컴포넌트가 자체적으로 데이터 fetching을 처리하도록 할 수 있지만, 이 글에서 중요하게 다루어 질 것은 네트워크 요청을 조정하는 아이디어이며, 아래에서 볼 수 있듯이 Suspense는 이 문제를 깔끔하게 해결합니다.

## Fetch-then-render

해당 접근법에서는 **컴포넌트가 렌더링 되기 이전에** 네트워크 요청이 발생합니다. 이전에 살펴본 예제와 비교하면서 무엇이 변경되었는지 확인해 보세요.

```jsx
const fetchDataPromise = fetchUserDetailsAndTodos(); // We start fetching here

const App = () => {
  const [userDetails, setUserDetails] = useState({});
  const [todos, setTodos] = useState([]);

  useEffect(() => {
    fetchDataPromise.then((data) => {
      setUserDetails(data.userDetails);
      setTodos(data.todos);
    });
  }, []);

  return (
    <div className="app">
      <h2>Simple Todo</h2>

      <UserWelcome user={userDetails} />
      <Todos todos={todos} />
    </div>
  );
};
```

`App`컴포넌트 밖에 fetching 로직을 옮김으로서, 컴포넌트가 마운트 되기 이전에 네트워크 요청이 시작될 수 있도록 하였습니다. 또 다른 변경사항은 `<Todos/>`가 이제 더이상 자신의 비동기 요청을 발생시키지 않고, 부모 컴포넌트인 `App`에서 처리합니다.

이 역시 네트워크 탭을 살펴보면 두 요청이 동시에 시작되는것을 확인할 수 있지만, 한눈에는 알아보기 힘든 미묘한 문제점이 있습니다.

![waterfall image](https://blog.logrocket.com/wp-content/uploads/2019/11/fetch-then-render-networks-tab.png)

<div style="display:flex;justify-content:center">개발자 도구에서 살펴본 waterfall (출처 : <a href="https://blog.logrocket.com/react-suspense-data-fetching">logrocket 블로그</a>)</div>

`fetchUserDetailAndTodos`의 구현이 아래와 같다고 한번 가정해 봅시다.

```js
function fetchUserDetailsAndTodos() {
  return Promise.all([fetchUserDetails(), fetchTodos()]).then(
    ([userDetails, todos]) => ({ userDetails, todos })
  );
}
```

`fetchUserDetails()`와 `fetchTodos()`가 병렬적으로 시작했지만, 이 상황에서 우리는 두 요청중에 더 느린 요청이 완료될 때 까지 기다려야지 데이터를 렌더링 할 수 있게 됩니다. 예를 들어서 `fetchTodos()`가 200ms 걸리고, `fetchUserDetails()`가 900ms가 걸린다면, 우리는 700ms동안 `fetchTodos()`가 완료되었음에도 불구하고, 기다려야 한다는 것이죠.

이는 `Promise.all`이 모든 promise가 resolve될 때까지 resolve를 시켜주지 않기 때문입니다. 물론 우리는 여기서 `Promise.all`을 제거하고 두 요청을 따로 기다릴 수 있지만, 이것은 우리의 어플리케이션이 성장하면서 금세 골칫거리가 됩니다.

또한 부모 컴포넌트가 자식 컴포넌트의 상태를 관리한다는것은 UX와 DX의 관점에서 봤을 때에 그렇게 바람직하지도 않습니다.

## 🌟Render-as-you-fetch🌟

의심할 여지 없이 Suspense가 React에게 가져다 주는 가장 큰 이점이라고 할 수 있습니다. 이 방법론을 통해서 다른 방법론에서 마주한 문제점들을 간단하게 해결할 수 있습니다. 이 방법론 에서는 **네트워크 요청을 발생시킨 직후에 컴포넌트를 렌더링** 합니다.

다시 말하자면 fetch-then-render 처럼, 렌더링 이전에 fetching을 실행합니다만, 렌더링 시작전에 응답을 기다릴 필요가 없습니다. 코드로 확인해 봅시다.

```jsx
const data = fetchData(); // Promise가 아닙니다. 아래에서 더욱 자세히 살펴보겠습니다.

const App = () => (
  <>
    <Suspense fallback={<p>Fetching user details...</p>}>
      <UserWelcome />
    </Suspense>

    <Suspense fallback={<p>Loading todos...</p>}>
      <Todos />
    </Suspense>
  </>
);

const UserWelcome = () => {
  const userDetails = data.userDetails.read();
  // code to render welcome message
};

const Todos = () => {
  const todos = data.todos.read();
  // code to map and render todos
};
```

이 코드는 약간 낯설어 보이지만, 그렇게 복잡하지 않습니다. 대부분의 일은 `fetchData()`내에서 일어나고, 해당 함수의 구현에 대해서는 아래에서 살펴보겠습니다. 지금 여기에서는 다른 부분에 더 주목해 봅시다.

컴포넌트를 렌더링 하기 이전에 네트워크 요청을 트리거 합니다. 그리고 main App component에서 `UserWelcome`, `Todos` 컴포넌트를 각각 Suspense 컴포넌트로 wrap한 다음, 각각 fallback을 달아 주고 있습니다.

처음 `App`이 마운트 되었을 때, `UserWelcome`을 렌더링 하려고 시도합니다. 그리고, 이는 `data.userDetails.read()`를 실행하게 합니다. 이때 데이터가 아직 준비가 되어 있지 않다면(다시 말하자면, 요청이 resolve되지 않았다면), Suspense로 돌아가고, `<p>Fetching user details...</p>`를 렌더링 하게 합니다. 같은 일이 `Todos`에서도 일어납니다.

이 코드에서 fallback은 데이터가 준비될때까지 렌더링 되고, 데이터가 준비가 되면 컴포넌트가 렌더링 됩니다. 해당 접근법의 좋은 점은 어떤 컴포넌트도 다른 컴포넌트를 기다릴 필요가 없다는 것입니다. 어느 컴포넌트가 온전한 데이터를 받는대로 다른 컴포넌트의 요청이 resolve가 되는지 상관 없이 렌더링 됩니다.

멋진 병렬 네트워크 요청을 유지하면서도, 렌더링 코드에서도 데이터가 있는지 확인하기 위한 if 검사를 제거했기 때문에 더욱 간결하게 보입니다.

![waterfall image](https://blog.logrocket.com/wp-content/uploads/2019/11/render-as-you-fetch-networks-tab.png)

<div style="display:flex;justify-content:center">개발자 도구에서 살펴본 waterfall (출처 : <a href="https://blog.logrocket.com/react-suspense-data-fetching">logrocket 블로그</a>)</div>

이제 이 방법론을 이용한 간단한 앱을 만들면서 위에서 나중에 다루겠다고 넘어간 `fetchData()`함수를 어떻게 구현하는지 알아봅시다.

# ⚒️ React Suspense를 사용하는 예제 앱 만들기

API로부터 데이터를 받아와서 DOM에 렌더링 하는 간단한 앱을 만들어 봅시다. 해당 예제를 만들 때 사용할 Data Fetching 접근법은 당연히 render-as-you-fetch를 사용할 것 입니다. React hook에 대해서 기초적인 지식만 있다면 쉽게 따라올 수 있으니 걱정 안하셔도 됩니다~

![sample app result](https://blog.logrocket.com/wp-content/uploads/2019/11/Finished-to-do-app.png)

## 셋업

빠르게 create-react-app으로 예제를 만들고 vscode로 열어줍시다.

```bash
npx create-react-app suspense-data-fetching
cd suspense-data-fetching
code .
```

그리고 파일 구조가 다음과 같이 되게끔 파일을 만들어 주세요

```text
├── README.md
├── package-lock.json
├── package.json
├── public
│   ├── favicon.ico
│   ├── index.html
│   ├── logo192.png
│   ├── logo512.png
│   ├── manifest.json
│   └── robots.txt
└── src
    ├── api
    │   ├── fetchData.js
    │   └── wrapPromise.js
    ├── components
    │   ├── App.jsx
    │   ├── Todos.jsx
    │   └── UserWelcome.jsx
    ├── index.css
    └── index.jsx
```

지금부터 차곡 차곡 파일들을 완성해 봅시다.

## API

우선 `api` 폴더부터 시작해 봅시다.

### `wrapPromise.js`

이 글에서 가장 중요한 부분이 되겠습니다. 이 부분을 통해서 Suspense와 통신하고, 라이브러리 작성자들이 Suspense API를 위한 추상화를 작성하는데 정말 많은 시간을 공들이기 때문이지요.

`wrapPromise.js`는 Promise를 감싸고 Promise에서 반환되는 데이터가 준비되었는지 확인하는 메서드를 제공하는 wrapper 입니다. Promise가 resolve되면, resolve된 데이터를 반환하고, reject된다면 에러를 throw하고, 여전히 prepending 중이라면, Promise를 throw 합니다.

해당 Promise argument는 보통 API로부터 데이터를 받아내는 네트워크 요청이지만, 기술적으로는 어떠한 Promise object가 와도 상관이 없습니다. 실제 구현은 해당 부분을 구현하는 사람에게 달린 것이므로, 이 방식 외에도 다른 방식으로 구현할 수 있습니다.(근데 굳이 다른 Promise를 넣을 이유가...?)

`wrapPromise` 함수는 아래의 요구사항을 가지고 있습니다.

- 인수로 Promise를 받습니다.
- Promise가 resolve되면, resolve된 값을 반환합니다.
- Promise가 reject되면, reject된 값을 반환합니다.
- Promise가 여전히 pending 상태이면, Promise를 반환합니다.
- Promise의 상태를 읽을 수 있는 메서드를 노출합니다.

요구사항들이 정의되었으니, 실제 코드를 적어봅시다. `api/wrapPromise.js`를 여셔서 코드를 작성해 봅시다.

```js
function wrapPromise(promise) {
  let status = "pending";
  let response;

  const suspender = promise.then(
    (res) => {
      status = "success";
      response = res;
    },
    (err) => {
      status = "error";
      response = err;
    }
  );

  const read = () => {
    switch (status) {
      case "pending":
        throw suspender;
      case "error":
        throw response;
      default:
        return response;
    }
  };
  return { read };
}

export default wrapPromise;
```

이제 코드를 풀이해 봅시다. `wrapPromise` 함수 내에서 우리는 변수 2개를 선언하고 있습니다.

1. `status`: promise 인수의 상태를 추적함
2. `response`: Promise의 결과를 저장함(resolve되던 rejected 되던)

`status`는 Promise의 기본 상태인 "pending"으로 초기화 됩니다. 그리고 `suspender`라는 새로운 변수를 초기화 하고, 그 값을 Promise로 할당한 다음, `then` 메서드를 부착합니다.

해당 `then` 메서드 내에서, 우리는 콜백 함수 2개를 선언합니다. 첫번째는 resolve되었을 때의 값을 처리하고, 두번째는 reject되었을 때의 값을 처리하는 함수죠(JavaScript Promise가 원래 동작하는 방식). Promise가 성공적으로 resolve 된다면, `status`는 "success"가 될 것이고, `response`에는 resolve된 값이 들어갈 것이고, Promise가 reject 된다면, `status`가 "error"가 될 것이고, response에는 rejected된 값이 들어가면 되겠네요.

그리고 `read`라는 새로운 함수를 만들어서, 안에 `switch`문을 작성해 줍니다. Promise의 `status`가 "pending"이라면, 우리는 방금 정의한 `suspender`변수를 throw하고, "error"라면 `response`변수를 throw, 마지막으로 그 두개가 아니라면(이를테면 "success") `response` 변수를 반환 합니다.

`suspender` 변수나 에러 `response` 변수를 throw하는 이유는, Suspense에게 Promise가 아직 resolve되지 않았다고 소통하기 위함입니다. 우리가 throw한 값은 Suspense 컴포넌트에 의해서 catch 되어서 실제 에러인지 Promise인지 확인하게 됩니다. 만약 Promise라면, Suspense 컴포넌트는 컴포넌트가 여전히 데이터를 기다리고 있음을 알아채고, fallback을 렌더링 합니다. 만약 에러라면, 에러를 버블링 해서 가장 가까운 Error Boundary로 넘기거나, 앱이 crash 되게 됩니다.

`wrapPromise`함수의 마지막 부분에 우리는 `read`함수를 메서드로 가지는 객체를 반환하고, 이것을 통해 우리의 React 컴포넌트들이 Promise의 값을 읽어내기 위해서 상호작용하게 됩니다. 이제 이 `wrapPromise`를 export 하여 다른 파일에서도 사용할 수 있게 합니다. 이제 `fetchData.js` 파일로 넘어가 봅시다.

### `fetchData.js`

이 파일 내에서 우리는 컴포넌트가 요청하는 데이터를 fetch 해올 것입니다. 해당 함수에서는 아까 만든 `wrapPromise`로 래핑 된 Promise를 반환할 것입니다.

```js
import wrapPromise from "./wrapPromise";

function fetchData(url) {
  const promise = fetch(url)
    .then((res) => res.json())
    .then((res) => res.data);

  return wrapPromise(promise);
}

export default fetchData;
```

더 설명할 것이 없네요...

## Component

### `App.jsx`

처음 설명했던 Suspense의 적용법 대로 적용이 된것을 확인할 수 있을 것입니다. 컴포넌트들이 데이터 fetching을 기다리고, 아직 완료되지 않았을 때, fallback을 렌더링 하는 간단한 구조지요.

```jsx
import React, { Suspense } from "react";

import UserWelcome from "./UserWelcome";
import Todos from "./Todos";

const App = () => {
  return (
    <div className="app">
      <h2>Simple Todo</h2>

      <Suspense fallback={<p>Loading user details...</p>}>
        <UserWelcome />
      </Suspense>
      <Suspense fallback={<p>Loading Todos...</p>}>
        <Todos />
      </Suspense>
    </div>
  );
};

export default App;
```

### `userWelcome.jsx`

이 컴포넌트는 유저를 위한 welcome message를 렌더링 합니다.

```jsx
import React from "react";
import fetchData from "../api/fetchData";

const resource = fetchData(
  "https://run.mocky.io/v3/d6ac91ac-6dab-4ff0-a08e-9348d7deed51"
);

const UserWelcome = () => {
  const userDetails = resource.read();

  return (
    <div>
      <p>
        Welcome <span className="user-name">{userDetails.name}</span>, here are
        your Todos for today
      </p>
      <small>Completed todos have a line through them</small>
    </div>
  );
};

export default UserWelcome;
```

여기서 `resource` 변수를 통해 `.read()` 메서드를 호출하여 request Promise를 쿼리할 수 있습니다. 만일 request가 resolve되지 않았다면, `resource.read()`를 호출하는 것은 상위 `Suspense` 컴포넌트로 Promise를 throw하게 되고, fallback이 렌더링 되게 됩니다. Promise가 resolve 되었다면, `resource.read()`메서드 에서는 Promise에서 resolve된 데이터를 반환할 것이고, 우리는 해당 값을 통해서 렌더링을 진행하게 됩니다.

### `Todos.jsx`

이 컴포넌트는 to-do 아이템들을 렌더링 합니다.

```jsx
import React from "react";
import fetchData from "../api/fetchData";

const resource = fetchData(
  "https://run.mocky.io/v3/8a33e687-bc2f-41ea-b23d-3bc2fb452ead"
);

const Todos = () => {
  const todos = resource.read();

  const renderTodos = todos.map((todo) => {
    const className = todo.status === "Completed" ? "todo-completed" : "todo";
    return (
      <li className={`todo ${className}`} key={todo.id}>
        {todo.title}
      </li>
    );
  });

  return (
    <div>
      <h3>Todos</h3>
      <ol className="todos">{renderTodos}</ol>
    </div>
  );
};

export default Todos;
```

`UserWelcome` 컴포넌트와 유사하기 때문에 설명을 생략하도록 하겠습니다.

# ⏰ Suspense에서 렌더링 되는 순서 관리하기

지금 상태로도 충분히 나쁘지는 않습니다만, `Todos`가 먼저 렌더링 되고, 그 다음에 `UserWelcome`이 렌더링 된다면 아래와 같은 상황이 생길 것입니다.

![waterfall image](https://blog.logrocket.com/wp-content/uploads/2019/11/Janky-Loading-In-Our-Demo-App.gif)

<div style="display:flex;justify-content:center">순서가 없는 Suspense (출처 : <a href="https://blog.logrocket.com/react-suspense-data-fetching">logrocket 블로그</a>)</div>

`Todos`컴포넌트가 `UserWelcome` 컴포넌트가 렌더링 완료되었을 때만 렌더링 되기 원한다면, `Todos`의 `Suspense`를 `UserWelcome`의 `Suspense`안에 감쌈으로써 해결할 수 있습니다.

```jsx
<Suspense fallback={<p>Loading user details...</p>}>
  <UserWelcome />

  <Suspense fallback={<p>Loading Todos...</p>}>
    <Todos />
  </Suspense>
</Suspense>
```

아니면 **expermental version에서만 사용 가능한** [`SuspenseList`라는 컴포넌트](https://17.reactjs.org/docs/concurrent-mode-patterns.html#suspenselist)를 이용해서 두 `Suspense`를 감쌀 수 있습니다. (`npm i react@experimental`을 사용해서 설치 가능)

```jsx
<SuspenseList revealOrder="forwards">
  <Suspense fallback={<p>Loading user details...</p>}>
    <UserWelcome />
  </Suspense>

  <Suspense fallback={<p>Loading Todos...</p>}>
    <Todos />
  </Suspense>
</SuspenseList>
```

# 마치며

React Suspense를 사용하면 비동기 처리를 더욱 선언적으로 할 수 있었습니다. JavaScript에서 비동기 통신을 처리하는데 `Promise`를 사용하듯, React에서는 `Promise`를 통해서 데이터를 받아오는 컴포넌트에 대한 렌더링을 더욱 효율적이고 선언적으로 하기 위해서 `Suspense`를 사용한다고 생각하면 좋을 것 같습니다. 위에서 만든 코드의 레포지토리 또한 링크로 남겨두겠습니다. 긴 글 끝까지 읽어주시느라 정말 고생 많으셨습니다. 감사합니다. 🙏🙏

## 참고한 글

- <https://blog.logrocket.com/react-suspense-data-fetching>

## 예제 앱 Repo

- <https://github.com/kasterra/suspense-data-fetching>
