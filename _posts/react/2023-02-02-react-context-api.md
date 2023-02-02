---
layout: post
title: 나를 위한 context API 사용법
subtitle: React context API의 사용법과 기타등등 이야기를 압축해서!
category: react
image: /images/thumbnails/React.png
---

{%raw%}

# 들어가며

요즘 통 React 글을 쓰지 않은것 같네요. 부캠 끝나고 이런저런 공부를 하다가, [context를 통한 modal 관리 글](https://nakta.dev/how-to-manage-modals-1)을 보면서, 이때까지 미루고 미뤄왔던 useContext 공부를 해야함을 느꼈고, 블로그 글을 쓸 수 있을 정도로 글이 쌓였다고 판단하여 이렇게 글을 써볼까 합니다.

## 이 글의 대상 독자

글을 최대한 필요한 부분만 들어있도록 가볍게 유지하기 위해서, 이 글을 읽으시는 분들은 아래의 지식들을 알고 있다고 가정하겠습니다.

- 적당한 JS 지식
- 클래스형 컴포넌트와 함수형 컴포넌트로 간단한 Todo앱을 개발할 수 있는 React 지식
- Context API가 왜 필요한지에 대한 필요성 인식을 할 수 있는 분
  - props drilling을 해소하기 위한 React 자체적인 도구라는 말을 이해할 수 있는 분
  - 하지만 Context API에 대해서 말만 들어봤지 써본적이 없는 분 ~~(그게 나야 둠빠둠빠)~~
- TypeScript에 대한 기초적인 이해
  - 이 글에서는 클래스형, 함수형 모두 TypeScript로 컴포넌트를 작성할 거에요

# 간단한 개념 설명

Context API의 개념과 필요성을 대강 아는 분들을 위해서 쓴 글이기 때문에, 개념 부분은 간단하게 말만하고 짚고 넘어가겠습니다. 아시다시피, React Context API는 Props drilling을 피하기 위해서 사용되는 도구이고, Context를 발행하는 Provider가 있고, 해당 context를 사용하는(소비하는) Consumer가 있는 구조입니다. 그리고 너무나도 당연한 이야기겠지만, 특정 Context를 사용하기 위해서는 그 Context의 Provider의 children으로 nesting되어 있어야 합니다.

당연하면 당연한 것이지만, 특정 context의 내용이 바뀌면, 해당 context를 소비하는 모든 컴포넌트가 리렌더링 됩니다. `React.memo`나 `useMemo`를 통한 메모이제이션을 통해서 Context에 내리는 value가 변하지 않게 하거나, 관심사를 생각해서 Context를 분리(state와 setState의 Context 분리)하여 불필요한 리렌더링을 줄이는것이 Context API를 사용하는데 정말 중요합니다!!

# 우리가 만들 간단한 앱

Redux를 설명 할 떄, 대부분 Todo 앱을 하나 만드는 것으로 데이터의 흐름 등을 설명하곤 합니다. 이번 Context API를 설명하는 예제에서는 적당히 만듦새있는 독서 리스트 앱을 만드는 것으로 설명을 진행해볼까 합니다.

우선 우리의 앱이 동작하는 간단한 모습부터 살펴봅시다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/wciw5CebKaA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

간단한 한 페이지로 구성된 앱입니다. 이 앱을 단 하나의 컴포넌트로 통으로 만들 수 있지만, 관심사 분리를 위해 아래와 같이 컴포넌트를 분리해 봅시다.

[![Untitled.png](https://i.postimg.cc/m2fLB55B/Untitled.png)](https://postimg.cc/3W15Znhc)

1. 책 리스트의 원소의 개수를 표시해주는 영역
2. 책 리스트의 내용을 표시하고, 리스트의 원소를 삭제할 수 있는 영역
3. 책 리스트에 새로운 내용을 넣을 수 있는 영역

1~3 컴포넌트 모두 서로 형제지간인 컴포넌트라서 props drilling까지가 발생하지 않아 props로 넘겨서 작업을 할 수도 있지만, context API를 사용해서 한번 작업해 봅시다.

## 템플릿 레포지토리

우리가 이 포스트에서 집중하고자 하는것은, HTML, CSS와 같은 마크업이 아닌, 데이터를 넘기는 Context API 입니다. 해당 영역에만 집중하면서, 화면에 시각적으로 보이는 앱을 만들기 위해서, 미리 템플릿을 만들어 놨으니, 하단 링크에서 받아서 같이 진행하면 될 것 같습니다.

[템플릿 github 레포 링크](https://github.com/kasterra/context-app-template)

# Context 만들고 제공하기

우선 클래스형 컴포넌트부터 살펴봅시다. hooks를 활용하는 함수형 컴포넌트가 대세지만, [클래스형 컴포넌트를 삭제할 계획이 없다](https://reactjs.org/docs/hooks-faq.html#do-i-need-to-rewrite-all-my-class-components)고 React 측에서 한 발언이 있고, 여전히 유효합니다. 완전 저세상 레거시도 아니니 알아둬서 나쁠건 없지요.

Context Provider는 함수형 컴포넌트와 크게 다른 점이 없기 때문에(`this.setState`를 `useState`로 바꾸고 하는 정도) 클래스형 컴포넌트로만 예시를 들어보겠습니다.

## createContext

context를 만들때에는 `createContext`라는 React API를 활용합니다. 우선 코드를 먼저 제시하고, 코드를 보면서 설명을 해볼까 합니다.

```tsx
import { createContext, Component, PropsWithChildren } from "react";
import { bookType } from "../types/bookType";
import { v4 as uuid } from "uuid";

interface BookContextType {
  books: bookType[];
  addBook: (title: string, author: string) => void;
  removeBook: (id: string) => void;
}

export const BookContext = createContext<BookContextType | null>(null);

class BookContextProvider extends Component<PropsWithChildren, bookType[]> {
  state: bookType[] = [];
  addBook = (title: string, author: string) =>
    this.setState([...this.state, { title, author, id: uuid() }]);
  removeBook = (id: string) =>
    this.setState(this.state.filter((book) => book.id !== id));

  render() {
    return (
      <BookContext.Provider
        value={{
          books: this.state,
          addBook: this.addBook,
          removeBook: this.removeBook,
        }}
      >
        {this.props.children}
      </BookContext.Provider>
    );
  }
}

export default BookContextProvider;
```

코드가 약간 긴것 같지만, 그렇게까지 어렵지 않으니 가봅시다.

```typescript
export const BookContext = createContext<BookContextType | null>(null);
```

Context를 만드는것은 실제 이 코드 한줄입니다. JS로 이루어진 코드에서는 간혹 `createContext()`같은 것이 보이기도 하는데, `createContext`의 콜 시그니쳐가 매개변수 하나를 Context의 초기값으로 무조건 받도록 되어 있기 때문에, 딱히 값을 넘기고 싶지 않다면 명시적으로 `undefined`나 `null`을 넘겨주어야 합니다. 위 코드를 보면 알 수 있듯, 제네릭 매개변수로, 해당 Context에 들어갈 수 있는 값의 타입을 적어줍니다. 초기값으로 `null`이나 `undefined`를 반드시 적어주어야 하는것은 아니지만, 해당Context의 범위가 아닌 곳에서 Context가 호출될 때, 에러를 내기 위해서 많이 사용하는 방법이라고 합니다.

이제 아래쪽 코드를 봅시다.

```tsx
class BookContextProvider extends Component<PropsWithChildren, bookType[]> {
  state: bookType[] = [];
  addBook = (title: string, author: string) =>
    this.setState([...this.state, { title, author, id: uuid() }]);
  removeBook = (id: string) =>
    this.setState(this.state.filter((book) => book.id !== id));

  render() {
    return (
      <BookContext.Provider
        value={{
          books: this.state,
          addBook: this.addBook,
          removeBook: this.removeBook,
        }}
      >
        {this.props.children}
      </BookContext.Provider>
    );
  }
}
```

prop으로 `propsWithChildren`타입(React에서 기본적으로 제공해주는 children이 들어가 있는 prop type)을 받는 간단한 클래스형 컴포넌트 입니다. `addBook`과 `removeBook`은 이름 그대로 `this.state`에 아이템을 추가/제거하는 간단한 함수조각들 입니다.

본격적으로 Context를 사용하기 위해서, `children`을 받아서 그대로 렌더링 해주되, `BookContext.Provider`로 감싸서, Context에 접근할 수 있게 해주는 간단한 컴포넌트를 작성하였습니다. `value`에는 object의 형태로 읽고 쓸 수 있는 수단을 넘겨줍시다.

# Context 사용하기

## 클래스형 컴포넌트 에서

### 1. `contextType` 정적 프로퍼티 사용

첫번째로 사용할 수 있는 방법은 `contextType`라는 정적 프로퍼티를 활용하는 것입니다. 우선 코드부터 봅시다.

```tsx
import { Component } from "react";
import { BookContext } from "../contexts/BookContext";

class Navbar extends Component {
  static contextType = BookContext;
  context!: React.ContextType<typeof BookContext>;
  render() {
    return (
      <div className="navbar">
        <h1>Ninja Reading List</h1>
        <p>
          Currently you have {this.context?.books.length} books to get
          through...
        </p>
      </div>
    );
  }
}

export default Navbar;
```

`static contextType = BookContext`이 문 하나를 통해서 우리의 `Navbar` 컴포넌트에서 사용할 Context를 받아옵니다. 그리고 그 밑의 context의 타입을 정의해 줌으로서, TypeScript의 자동완성의 은혜를 누릴 수 있습니다. 그리고 사용할 때는
`this.context`를 이용해서 접근할 수 있습니다.

이 방법은 클래스형 컴포넌트에서만 사용할 수 있으며, 하나의 Context만 사용할 수 있다는 특징을 가지고 있기 때문에 사용 시 유의가 필요합니다. 아래에서 설명할 Consumer 컴포넌트 사용보다 클래스형 컴포넌트에 더 잘 녹아들어가기 때문에, 클래스형 컴포넌트를 사용할 때에는 이 방법을 알아두면 좋겠습니다 :)

### 2. `Context.Consumer` 컴포넌트 사용

`Provider`가 있다면, `Consumer`가 있는것이 자연스럽겠죠. 그리고, TypeScript를 통해서 이 글의 내용을 따라오신 분이라면 분명 위의 Context 문법을 볼 때 `Consumer`가 무엇인지에 대한 궁금증 역시 가졌을 것입니다. 이번 섹션에서는 함수형 컴포넌트에서도 사용할 수 있고, **여러 Context를 사용할 수 있는** `Context.Consumer`에 대해서 다루어 봅시다.

아래의 코드는 위에서 적었던 코드와 동일한 역할을 하는 코드입니다.

```tsx
import { Component } from "react";
import { BookContext } from "../contexts/BookContext";

class Navbar extends Component {
  render() {
    return (
      <BookContext.Consumer>
        {(context) => (
          <div className="navbar">
            <h1>Ninja Reading List</h1>
            <p>
              Currently you have {context!.books.length} books to get through...
            </p>
          </div>
        )}
      </BookContext.Consumer>
    );
  }
}

export default Navbar;
```

가장 확실한 차이는 `BookContext.Consumer`라는 컴포넌트가 들어왔다는 것이고, children이 일반 JSX의 형태가 아니고, **JSX를 리턴하는 함수**가 되었다는 것입니다.

## 함수형 컴포넌트에서

함수형 컴포넌트에서는 위에서 말한 `Context.Consumer`역시 사용할 수 있지만, 더욱 더 함수형 hook스러운 방법을 제공합니다. 그리고 코드 양도 훨 간결하고, JSX를 반환하는 함수가 아닌 JSX그 자체를 통해서 렌더링 할 수 있도록 더욱 편안합니다.

### `useContext` hook 사용

```tsx
import { useContext } from "react";
import { BookContext } from "../contexts/BookContext";

const Navbar = () => {
  const { books } = useContext(BookContext)!;
  return (
    <div className="navbar">
      <h1>Ninja Reading List</h1>
      <p>Currently you have {books.length} books to get through...</p>
    </div>
  );
};

export default Navbar;
```

훨 코드가 단순해진것을 볼 수 있습니다. `useContext` 이후에 `!` 하나를 붙여서 not null에 대한 보장을 컴파일러에게 해주면 null일수도 있다는 경고도 내지 않기 때문에 더욱 간단히 할 수 있지요.

# 마치며

이번 글에서는 간단하게, react context API의 사용법에 대해서 알아보았습니다. 이후의 글에서는 react context API를 사용한다고 하면 지겹도록 듣는, 리렌더링에 대한 이슈와, 그리고 hook을 사용할 때, 커스텀 훅으로 조금 더 편리하게 꺼내는 법 등에 대해서 다루어 볼까 합니다.

끝까지 읽어주셔서 감사합니다 :)

{%endraw%}
