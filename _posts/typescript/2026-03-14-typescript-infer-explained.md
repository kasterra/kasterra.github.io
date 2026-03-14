---
title: TypeScript infer 키워드 제대로 이해하기 (extends와 함께)
subtitle: 제네릭, extends, infer를 통해 TypeScript가 타입을 '계산'하는 방식을 살펴봅니다
layout: post
image: /images/thumbnails/TS.png
category: typescript
---

## 들어가며

요즈음 저는 구현우선주의라는 안좋은 습관을 끊어내기 위해서 여러 노력을 하고 있습니다. 최근 글들도, 아키텍처에 직간접적으로 관련이 있는 글들을 적으면서, 보완하려고 노력을 하고 있죠.

이번 글은 '타입적으로 단단한 코드'를 적기 위한 문법적 도구에 대해 다루어 보고자 합니다. 개발을 하다 보면 '타입을 정한다' 라는 것이, 그리 간단한 일이 아닐 때가 종종 있는데, 그러한 '개발 서사'의 일부와, 그런 서사속의 문제해결을 다루어 보고자 합니다.

## 간단한 상황 - 계산할 것이 없는 타입

저의 과거 이야기로 시작하고자 합니다. JS는 타입이 없기 때문에, 그러한 문제를 해결하기 위해서 TS가 등장했다 라는 것을 FE 개발을 처음 시작할 때 배웠습니다. 당시 학교에서 C언어나 Java같은 강타입 언어들을 배웠기 때문에, 뭐 그닥 어려울 것 없다라고 생각하면서 코드를 작성하였습니다. `number`, `string`, `object` 와 같은 문법들은 C언어의 비슷한 것들과 다를것 없어 보였죠.

```ts
let a: number;
a = "abc"; // typeError!
```

C언어를 사용해서 배운 기초프로그래밍 수업때와 크게 다를 것 없이, 단순한 것들이구나 하고 넘어가고 즐거운 구현 생활을 이어갔습니다.

## 약간의 복잡성을 더해보자 - 타입 템플릿

본격적으로 FE에 흥미를 가지기 이전에 PS 공부 용으로 C++을 얕게 접한 적이 있습니다.

```cpp
int main(){
  std::vector<int> v{1,2,3,4};
}
```

와 같은 코드들에서, `vector` 자료구조에 `int`가 들어간다고 넣기도 했죠. TS에서도 제네릭 타입을 사용해서 비슷한 것을 할 수 있습니다.

```ts
const fruits: Array<string> = ["apple", "banana"];
const scores: Array<number> = [100, 95, 88];
```

타입의 종류를 받을 수 있기 때문에, 타입 변수 T로부터 여러 타입을 파생시켜 조금 더 복잡한 객체 등을 정의할 수 도 있지요.

```ts
type ApiResponse<T> = {
  data: T;
  success: boolean;
  error: string | null;
};

type User = {
  id: number;
  name: string;
};

const userResponse: ApiResponse<User> = {
  data: {
    id: 1,
    name: "kasterra",
  },
  success: true,
  error: null,
};
```

FE 개발을 하면서 상당히 자주 마주할 API 응답 타입을 이렇게 만들 수 있습니다.

## 아무 타입이나 들어가지 않게 하기 - extends 키워드

이렇게 타입 템플릿을 활용해서 특정 타입에서 다른 타입을 만들어 낼 수 있지요. 그치만, 이 타입에 제한을 두고 싶다면 어떨까요? 이를테면, 배열 타입만 받게 하고 싶은 상황이 충분히 생길 수 있을겁니다.

TypeScript에서는 제네릭 타입 변수에 `extends`를 사용하여 **받을 수 있는 타입의 범위를 제한**할 수 있습니다.

예를 들어, 배열 타입만 허용하고 싶다면 다음과 같이 작성할 수 있습니다.

```ts
type FirstItem<T extends unknown[]> = T[0];
```

이 타입은 "배열 타입만 받는다"라는 제약을 가지게 됩니다.

```ts
type A = FirstItem<[string, number]>; // string
```

하지만 배열이 아닌 타입을 넣으려고 하면 에러가 발생합니다.

```ts
type B = FirstItem<string>; // ❌ Type 'string' does not satisfy the constraint 'unknown[]'
```

이렇게 `extends`는 단순한 "상속" 개념이라기보다, **"이 타입은 최소한 이것의 형태를 만족해야 한다"** 라는 의미로 사용하는 경우가 많습니다.

그래서 실제 코드에서는 다음과 같은 형태로 자주 등장합니다.

```ts
function getLength<T extends { length: number }>(value: T) {
  return value.length;
}

getLength("hello"); // OK
getLength([1, 2, 3]); // OK
getLength(123); // ❌ number에는 length가 없음
```

즉 `extends { length: number }` 라는 제약 덕분에, 이 함수는 **length 속성을 가진 값들만 받을 수 있는 함수**가 됩니다.

이 패턴은 라이브러리 타입을 읽다 보면 정말 자주 등장합니다. 예를 들어:

- 배열만 허용
- 객체 타입만 허용
- 특정 속성을 가진 객체만 허용

같은 제약을 표현할 때 `extends`가 핵심 도구로 사용됩니다.

그런데 여기서 한 단계 더 나아가면, TypeScript는 단순히 "제약"만 거는 것이 아니라 **타입을 계산하는 기능**도 제공합니다. 그때 등장하는 것이 바로 `infer` 입니다.

## 템플릿 변수 타입에서 파생 타입을 이끌어내기 - infer 키워드

앞에서 `extends`를 이용해서 **제네릭 타입에 제약을 걸 수 있다**는 것을 보았습니다. 그런데 TypeScript는 여기서 한 발 더 나아가, **타입 내부에서 일부를 꺼내어 새로운 타입으로 재사용**할 수 있는 기능을 제공합니다.

이때 사용하는 키워드가 바로 `infer` 입니다.

처음 보면 다소 추상적으로 느껴질 수 있습니다. 그래서 이번에는 먼저 **가장 단순한 형태의 `infer` 사용 예제**를 살펴보고, 이후에 React 예제로 확장해 보겠습니다.

### 가장 단순한 infer 예제 - 함수 반환 타입 꺼내기

`infer`는 조건부 타입 안에서 **타입의 일부를 추출하기 위한 변수 선언**이라고 생각하면 이해하기 쉽습니다.

예를 들어 어떤 함수 타입이 있을 때, 그 함수의 반환 타입만 꺼내고 싶다고 해봅시다.

```ts
type ReturnTypeOf<T> = T extends (...args: any[]) => infer R ? R : never;
```

이 타입의 의미는 다음과 같습니다.

- `T`가 함수 타입이면
- 그 함수의 반환 타입을 `R`이라는 이름으로 추론(`infer`)해서
- 그 타입을 결과로 사용한다.

실제로 사용해보면 다음과 같이 동작합니다.

```ts
type A = ReturnTypeOf<() => number>;
// number

type B = ReturnTypeOf<(name: string) => string[]>;
// string[]
```

이 예제에서 `infer R`은

> "이 위치에 있는 타입을 `R`이라는 이름으로 추론해서 꺼내라"

라는 의미입니다.

즉 `infer`는 **타입 패턴 매칭 과정에서 새로운 타입 변수를 만들어내는 문법**이라고 볼 수 있습니다.

이제 이 개념을 조금 더 실전적인 예제로 확장해 보겠습니다.

### 자식의 타입에 따라 렌더 전략이 달라지는 경우

이를테면 어떤 UI 컴포넌트는 다음과 같은 정책을 가진다고 해봅시다.

- `children`이 문자열이면 `<p>`로 감싼다.
- `children`이 React Element이면 그대로 렌더한다.
- `children`이 렌더 함수이면 실행해서 결과를 렌더한다.

이런 설계는 실제로 headless UI를 만들거나, 선언형 slot 패턴을 설계할 때 꽤 자주 등장합니다.

먼저 자식으로 받을 수 있는 타입을 다음과 같이 정의할 수 있습니다.

```ts
import type { ReactElement, ReactNode } from "react";

type SmartChildren = string | ReactElement | (() => ReactNode);
```

그런데 여기서 끝내지 않고, **자식 타입에 따라 내부에서 어떤 렌더 결과를 기대해야 하는지** 타입 차원에서 계산하고 싶을 수 있습니다. 바로 이런 지점에서 `infer`가 등장합니다.

```ts
type ResolveChildResult<T> = T extends () => infer R ? R : T;
```

이 타입의 의미는 다음과 같습니다.

- `T`가 함수라면, 그 함수의 반환 타입 `R`을 꺼낸다.
- 함수가 아니라면 `T` 자체를 그대로 사용한다.

즉 "렌더 함수면 실행 결과 타입으로 바꾸고, 아니면 원래 타입을 유지한다" 라는 규칙을 타입으로 적은 것입니다.

이 패턴은 render prop 패턴이나 headless UI 라이브러리에서 자주 등장하는 타입 설계 방식입니다.

이제 이 타입을 활용해서 실제 렌더 함수를 설계해볼 수 있습니다.

```ts
import { isValidElement } from "react";
import type { ReactElement, ReactNode } from "react";

type SmartChildren =
  | string
  | ReactElement
  | (() => ReactNode);

type ResolveChildResult<T> =
  T extends () => infer R ? R : T;

function renderChild<T extends SmartChildren>(child: T): ResolveChildResult<T> {
  if (typeof child === "function") {
    return child() as ResolveChildResult<T>;
  }

  if (typeof child === "string") {
    return <p>{child}</p> as ResolveChildResult<T>;
  }

  if (isValidElement(child)) {
    return child as ResolveChildResult<T>;
  }

  throw new Error("지원하지 않는 children 타입입니다.");
}
```

이 예제에서 핵심은 `ResolveChildResult<T>` 입니다.

호출하는 쪽에서는 넘긴 자식의 형태에 따라 반환 타입이 달라집니다.

```ts
const a = renderChild("hello");
// JSX.Element

const b = renderChild(() => <strong>world</strong>);
// ReactNode

const c = renderChild(<button>click</button>);
// ReactElement
```

즉 `infer` 덕분에, 단순히 "함수도 받을 수 있다" 수준을 넘어서 **함수인 경우에는 그 반환 타입을 뽑아서 이후 타입 계산에 반영**할 수 있게 됩니다.

### 왜 이런 것이 실전에서 중요한가

React 코드를 작성하다 보면, 단순히 props의 모양만 선언하는 것으로는 부족한 경우가 있습니다.

예를 들어:

- `children`으로 element를 받을 수도 있고 render function을 받을 수도 있는 컴포넌트
- `as` prop에 따라 허용되는 props가 달라지는 polymorphic component
- 특정 컴포넌트를 받아서 그 props 타입을 추출해야 하는 wrapper component

이런 경우에는 타입의 일부를 꺼내서 다시 조립하는 작업이 필요해집니다. `infer`는 바로 그때 쓰입니다.

이를테면 컴포넌트 타입에서 props를 꺼내오는 것도 비슷한 원리입니다.

```ts
type PropsOf<T> = T extends (props: infer P) => any ? P : never;
```

이렇게 해두면 어떤 함수형 컴포넌트의 props 타입을 추출해서, wrapper나 HOC를 만들 때 재사용할 수 있습니다.

```ts
function Button(props: { disabled?: boolean; children: ReactNode }) {
  return <button {...props} />;
}

type ButtonProps = PropsOf<typeof Button>;
// { disabled?: boolean; children: ReactNode }
```

즉 `infer`는 단순한 타입 장난감이 아니라, **컴포넌트의 입력과 출력 사이의 관계를 타입 시스템에 반영하기 위한 도구**라고 보는 편이 더 정확합니다.

처음에는 문법이 낯설어 보이지만, 실제로는 다음과 같은 질문에 답하기 위해 사용됩니다.

- 이 함수가 돌려주는 타입만 꺼낼 수 없을까?
- 이 컴포넌트의 props만 재사용할 수 없을까?
- 이 render prop의 결과 타입을 다음 계산에 활용할 수 없을까?

이런 질문에 타입 차원에서 답하는 것이 바로 `infer`의 역할입니다.

## 덤 - TS는 타입 유추를 해줘서 편해요

React를 통한 개발을 하면서, 보지 않기 힘든 `useState`도 관심이 있으신 분들은 알고 계시겠지만 타입 템플릿 기반입니다. 하지만 다음 코드처럼 타입을 사용량에서 유추할 수 있으면 굳이 제네릭 변수에 타입을 적지 않아도 되죠.

```ts
const [count, setCount] = useState(0); // count는 number로 추론됨
```

이처럼 TypeScript는 **값을 기반으로 타입을 역으로 유추(inference)** 할 수 있습니다.

`useState(0)`을 호출했을 때 TypeScript는 다음과 같이 판단합니다.

- `0`은 `number` 타입이다.
- 따라서 `useState<number>`를 호출한 것과 동일하게 취급한다.

즉 우리가 다음과 같이 작성한 것과 같은 의미가 됩니다.

```ts
const [count, setCount] = useState<number>(0);
```

이러한 타입 유추 덕분에 대부분의 코드에서는 **제네릭 타입을 굳이 명시하지 않아도** 충분히 타입 안전성을 유지할 수 있습니다.

하지만 항상 타입을 유추할 수 있는 것은 아닙니다. 예를 들어 초기값이 `null`이거나, 여러 타입이 섞일 수 있는 경우에는 TypeScript가 의도를 정확히 파악하지 못할 수도 있습니다.

```ts
const [user, setUser] = useState(null);
// user는 null 타입으로만 추론됨
```

이런 경우에는 개발자가 직접 제네릭 타입을 명시해 주는 것이 좋습니다.

```ts
const [user, setUser] = useState<User | null>(null);
```

즉 TypeScript의 타입 시스템을 사용할 때는

- **가능하면 타입 유추에 맡기되**
- **유추가 모호해지는 지점에서는 명시적으로 타입을 제공하는 것**

이 가장 자연스러운 사용 방식이라고 볼 수 있습니다.

앞에서 살펴본 `extends`와 `infer` 같은 키워드들도 결국 이 흐름 속에 있습니다.

TypeScript는 가능한 많은 타입을 자동으로 계산하려고 하고, 개발자는 필요한 지점에서만 **타입 계산 규칙을 조금 더 명확하게 알려주는 역할**을 하게 되는 것입니다.

## 마무리

2020년대 초반, 제가 처음으로 TS를 입문할 때, 단순히 JS에 '타입을 붙이는 도구'로만 생각했습니다. 틀린 설명은 아니지만, 이 글에서 소개한 `extends`와 `infer`그리고, TS의 적극적인 타입추론과, 타입가드로 상당히 많은 것을 할 수 있다는 것을 보았지요

```ts
type PreferGenerated<Generated, Fallback> = {
  [K in keyof Generated | keyof Fallback]: K extends keyof Generated
    ? Generated[K]
    : K extends keyof Fallback
      ? Fallback[K]
      : never;
};
```

바로 이러한 복잡한 타입을 술 술 쓸 수 있는 타입의 마법사가 되기까지는 많은 숙련이 필요하겠지만, 이번 포스트에서 다룬 많은 부분이 이러한 복잡한 타입을 이해하는데에 도움이 되리라 생각하여 이렇게 정리해 보았습니다.

본 포스트에는, '원하는 특정한 타입을 이끌어내기' 였다면, 다음 포스트에는 즉정한 변수의 타입이 어떠한 타입으로 안전하게 쓸 수 있게 하는 단순 as 타입단언을 넘어, as const, satisfies 키워드의 개념과 실전적 사용예시로 글을 써볼까 합니다.

끝까지 읽어주셔서 감사합니다.
