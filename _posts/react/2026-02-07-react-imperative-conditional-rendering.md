---
layout: post
title: JSX의 본질로 보는 선언적 조건 렌더링 – v-if, v-for를 React에서
subtitle: React에서도 vue.js의 DX를 가져갈 수 있다구요
category: react
image: /images/thumbnails/React.png
---

# 들어가며

오랜만에 글을 쓰는군요. '돈을 받으면서 하는 일'을 시작하다 보니, 그간 바쁘다는 이유로 글을 제대로 쓰지 못한 것 같습니다. 오랜만에 온 글인 만큼, 생산성 그리고 JSX의 본질 활용이라는 두 마리 토끼를 한 번에 잡아보는 토픽으로 이야기를 나눠보려 합니다.

# vue를 써보셨습니까?

사실 저는 안 써 봤습니다. 그치만, 제가 vue에 대해서 아는 것은 `v-if`, `v-for` 등의 키워드를 통해서 선언적으로 렌더하는 프레임워크라는 사실입니다. vue의 이런 ‘확실히 정해져 있는 문법’이라는 성격은 코드가 일정하게 나오게 만들고, react가 사실상 판을 잡은 현재에도 이 깔끔함은 꽤 매력적입니다.

react는 반면 같은 조건부 렌더링을 하더라도 `? :`의 삼항 연산자로도, `&&`로도 할 수 있지요. JSX는 그저 JS로 DOM(혹은 유사한 것)을 선언하는 도구일 뿐이니까요.

하지만, 하나만 짚고 넘어갑시다.

![jsx meme](https://www.mattstobbs.com/_astro/what-if-i-told-you.aec0805d_Z11JB1h.webp)

JSX는 이름 그대로 JS 코드의 단순 alias들이기 때문에 우리가 일반적인 프로그래밍 언어에서 쓸 수 있는 것을 다 할 수 있다는 사실을요. 이번 시간에는 단순 템플릿 언어가 아니라 JS라는 관점에서 이 JSX를 접근해 보고자 합니다.

이 밈이 농담처럼 보이지만, JSX를 단순한 템플릿 언어로 오해할 때 조건 렌더링은 가장 먼저 복잡해 지거든요.

# 오늘 소개할 라이브러리 - ilokesto/utilinent

오늘 소개할 라이브러리는 [@ilokesto/utilinent](https://github.com/ilokesto/utilinent)라는 라이브러리 입니다. `<Show>` `<For>` `<Switch>` `<Match>` 등의 선언형 의미를 가진 태그를 이용해서 `v-for`와 같이 '한가지 방법' 으로만 UI 코드를 작성할 수 있도록 도와주죠.

## 기존 JSX 코드의 문제점 아닌 문제점

개인 취향이 갈릴 수 있는 부분이지만, `.map`으로 UI 반복을 표현하거나, `? :`의 삼항 연산자로도 혹은 `&&` 라는 두가지 도구로 조건부 렌더링을 할 수 있다는 점은, 약간의 인지 비용을 만든다고 생각합니다. 위 라이브러리의 Readme에 있는 예제를 그대로 가져와보죠

```jsx
import React, { useState, useEffect } from 'react';

const UserList = () => {
  const { data: users } = useQuery( ... )

  return (
    <div>
      <h2>User List</h2>
      {isLoading ? (
        <p>Loading users...</p>
      ) : (
        users && users.length > 0 ? (
          <ul>
            {users.map(user => (
              <li key={user.id}>{user.name}</li>
            ))}
          </ul>
        ) : (
          <p>No users found.</p>
        )
      )}
    </div>
  );
};

export default UserList;
```

이것을 아래와 같이 바꿀 수 있다면?

```jsx
import React, { useState, useEffect } from 'react';
import { Show, For } from '@ilokesto/utilinent';

const UserListAfter = () => {
  const { data: users } = useQuery( ... )

  return (
    <div>
      <h2>User List</h2>
      <Show when={!isLoading} fallback={<p>Loading users...</p>}>
        <For.ul each={users ?? []} fallback={<p>No users found.</p>}>
          {(user) => (
            <li key={user.id}>{user.name}</li>
          )}
        </For.ul>
      </Show>
    </div>
  );
};

export default UserListAfter;
```

훨씬 간단하고, 무엇을 하려는지도 확실하게 보입니다.

다만 여기에서 '와 정말 신기하죠? 여러분도 써보세요' 로 끝날 거였으면 이 글을 쓴다고 밈 이미지까지 찾아오면서 여러분들의 주위를 환기할 필요도 없겠죠. 리누스 토르발즈 선생님의 명언인, '말을 됐고, 코드 가져와'를 실천해봅시다.

> ⚠️ 이 아래부터는 TypeScript 제네릭/오버로드에 익숙한 독자를 기준으로 설명합니다.
>
> TS 타입이 이해가 안되면 런타임 코드만 이해하시고 넘어가셔도 무방합니다.

# 단순 버전으로서의 구현

다만, 원본 그대로를 가져오면 설명할 것이 많아지므로, 해당 역할을 비슷하게 수행하지만, 최적화 등이 덜 들어가 이해가 조금 더 쉬운 버전으로 말이죠.

## `<For>`의 구현

우선 흔히 말하는 `forEach`문에 해당하는 `<For>` 컴포넌트를 구현해 보도록 해봅시다. JSX에서는 `someList.map()` 같은 패턴으로 처리하는 그것이죠.

우선 For라는 것의 기본적 정의부터 내려봅시다.

```tsx
type ForProps<T> = {
  each: readonly T[] | null | undefined;
  children: (item: T, index: number) => React.ReactNode;
  fallback?: React.ReactNode;
};

export function For<T>({ each, children, fallback = null }: ForProps<T>) {
  if (!each || each.length === 0) {
    return <>{fallback}</>;
  }

  return <>{each.map(children)}</>;
}
```

간단하지요. 더이상 헷갈릴 수 있는것이 없는, 대단한 지식이 필요하지 않은 코드입니다.

이 구현에서 중요한 점은, `<For>`가 특별한 마법을 부리는 컴포넌트가 아니라 단순히 `Array.prototype.map`을 JSX 문법으로 감싼 것에 불과하다는 점입니다.

JSX가 JS의 alias라는 말은, 이런 수준의 추상화가 가능하다는 의미이기도 합니다.

## `<Show>`의 구현

이번에는 조건부 렌더링을 구현해 보겠습니다. 아래의 것을 목표로 말이죠.

```tsx
<Show when={user}>
  {(user) => <Profile name={user.name} />}
</Show>

<Show when={[user, permissions]}>
  {([user, permissions]) => (
    <Dashboard user={user} perms={permissions} />
  )}
</Show>
```

실제 로직을 TS로 구현한다면 아래와 같은 구현체가 가능할 겁니다.

> 여기서부터 타입 정의가 길어집니다. 구현 아이디어만 궁금하면 resolveWhen과 Show의 런타임 구현만 보고 넘어가셔도 됩니다.

```tsx
// when 해석 로직:
// - 배열이면 모든 원소가 truthy일 때만 true
// - 단일 값이면 truthy면 true
/*
개념 전달을 위해 truthy/falsy 기반으로 단순화했습니다.
실무에서는 predicate/guard 기반으로 더 엄격히 설계하는 편이 안전합니다.
*/
function resolveWhen(when: unknown): boolean {
  if (Array.isArray(when)) return when.every(Boolean);
  return Boolean(when);
}

interface ShowProps<T = unknown> extends Fallback {
  when: T;
  children: React.ReactNode | ((item: NonNullable<T>) => React.ReactNode);
}

interface ShowPropsArray<T extends unknown[]> extends Fallback {
  when: T;
  children:
    | React.ReactNode
    | ((item: NonNullableElements<T>) => React.ReactNode);
}

// 오버로드 1) when이 배열/튜플인 경우
export function Show<T extends readonly unknown[]>(
  props: ShowPropsArray<T>,
): React.ReactNode;
// 오버로드 2) when이 단일 값인 경우
export function Show<T>(props: ShowProps<T>): React.ReactNode;

export function Show(props: any): React.ReactNode {
  const { when, children, fallback = null } = props;

  const shouldRender = resolveWhen(when);
  if (!shouldRender) return fallback;

  if (typeof children === "function") {
    // resolveWhen이 true일 때:
    // - 단일 when: NonNullable<T>
    // - 배열 when: NonNullableElements<T>
    return children(when);
  }

  return children;
}
```

`when`이 truthy하거나, `when`이 배열로 들어왔을 때, 전부 truthy 하다면 children을 렌더링 하는 패턴입니다. children으로 함수를 넣을 수 있는 것은, when이 resolve되었을 때 렌더링되는 자식에 타입 내로잉을 제공하기 위함입니다. 일반적인 TS 로직에서 조건문을 통과한 블록 안에서는 상위 조건문에서 추론할 수 있는 타입만큼 내로잉을 제공 하니까요.

resolveWhen을 통과한 시점에서는

- 단일 when은 truthy → NonNullable
- 배열 when은 every(Boolean) → 각 요소가 NonNullable

라는 개념적 타입 가드가 성립한다고 가정합니다. 하지만 TS 타입 가드에서 잡아주지 못할 수 있기 때문에, as를 쓸 수도 있습니다.

### TS 참고 문서

참고: 타입스크립트 용어가 익숙지 않다면 [inpa dev 블로그 글](https://inpa.tistory.com/entry/TS-📘-타입-추론-타입-호환-타입-단언-타입-가드-💯-총정리)을 먼저 읽고 오는 것을 추천합니다.

그리고 아래의 TS공식 핸드북 링크도 참조하면 좋을 것 같습니다.

- [핸드북(메인)](https://www.typescriptlang.org/docs/handbook/intro.html)
- [타입 내로잉 (Narrowing)](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)
- [함수 오버로드 (More on Functions 섹션 내 Overload Signatures)](https://www.typescriptlang.org/docs/handbook/2/functions.html)
- [제네릭 (Generics)](https://www.typescriptlang.org/docs/handbook/2/generics.html)
- [타입 호환/타입 시스템 기초 Everyday-types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)

## `<Switch>`와 `<Match>`의 구현

`<Show>`가 "조건이 참이면 렌더"라면, `<Switch>`/`<Match>`는 흔히 말하는 if / else if / else 체인을 JSX로 옮긴 형태입니다. 즉, "여러 조건 중 처음으로 매칭되는 분기 하나만 렌더" 하는 패턴이죠.

왜 필요하냐면... 아래의 코드를 읽어보면 아마 react 사용자라면 공감할 것 같습니다.

```tsx
return (
  <div>
    {isLoading ? (
      <Spinner />
    ) : error ? (
      <ErrorView message={error.message} />
    ) : users && users.length > 0 ? (
      isAdmin ? (
        <AdminUserList users={users} />
      ) : (
        <UserList users={users} />
      )
    ) : (
      <Empty />
    )}
  </div>
);
```

분명 말이 안되는 코드는 아닌데... 읽으려면 약간의 숨이 차죠.

목표 사용례는 아래와 같습니다.

```tsx
<Switch>
  <Match when={isLoading}>
    <Spinner />
  </Match>

  <Match when={[!isLoading, items.length === 0]}>
    <Empty />
  </Match>

  <Match when={!isLoading}>
    <List items={items} />
  </Match>

  <Switch.Else>
    <UnknownError />
  </Switch.Else>
</Switch>
```

가능한 구현체는 아래와 같습니다.

```tsx
import * as React from "react";

// Show에서 썼던 로직 재사용
function resolveWhen(when: unknown): boolean {
  if (Array.isArray(when)) return when.every(Boolean);
  return Boolean(when);
}

/**
 * Match: "case" 역할
 * - Switch가 자식들을 훑을 때, Match면 when을 평가해서 통과 여부를 본다.
 * - Match 자체는 단독 사용보다는 Switch 아래에서 쓰는 것을 전제로 한다.
 */
type MatchProps<T = unknown> = {
  when: T | readonly unknown[];
  children: React.ReactNode;
};

export function Match<T = unknown>({ children }: MatchProps<T>) {
  // Match는 Switch가 고를 대상일 뿐, 스스로 렌더링 책임은 Switch가 가진다.
  return <>{children}</>;
}

/**
 * Switch.Else: default 케이스 역할
 */
type ElseProps = {
  children: React.ReactNode;
};

function Else({ children }: ElseProps) {
  return <>{children}</>;
}

/**
 * Switch:
 * - children에서 Match/Else를 찾아서 분기한다.
 * - 첫 매칭 Match를 렌더, 없으면 Else 렌더, Else도 없으면 null
 */
type SwitchProps = {
  children: React.ReactNode;
};

export function Switch({ children }: SwitchProps) {
  const arr = React.Children.toArray(children);

  // Else는 하나만 허용한다고 가정(단순화)
  let elseNode: React.ReactNode = null;

  for (const child of arr) {
    if (!React.isValidElement(child)) continue;

    // Switch.Else 탐지
    if (child.type === Else) {
      elseNode = child.props.children;
      continue;
    }

    // Match 탐지
    if (child.type === Match) {
      const when = (child.props as MatchProps).when;
      if (resolveWhen(when)) {
        return child.props.children;
      }
    }
  }

  return elseNode;
}

// 네임스페이스처럼 사용하기 위한 정적 프로퍼티. 가능함을 알려주기 위해 삽입
Switch.Else = Else;
```

# 조금 더 본격적인 구현 - `<For.ul>`등의 확장형

앞서 구현한 `<For>`는 충분히 유용하지만, 실제 UI 코드에서는 이런 요구가 자연스럽게 따라옵니다.

```tsx
<For each={users}>{(user) => <li key={user.id}>{user.name}</li>}</For>
```

이 코드가 나쁘다는 건 아닙니다. 다만 HTML 구조를 생각해보면, 우리는 보통 이렇게 쓰고 싶어집니다.

```tsx
<ul>
  {users.map((user) => (
    <li key={user.id}>{user.name}</li>
  ))}
</ul>
```

즉, 반복 로직과 래퍼 엘리먼트를 한 덩어리로 묶고 싶은 욕구가 생기죠. 그래서 다음과 같은 API를 목표로 해봅시다.

```tsx
<For.ul each={users} fallback={<p>유저 없음</p>}>
  {(user) => <li key={user.id}>{user.name}</li>}
</For.ul>
```

이건 JSX 문법적으로 전혀 이상하지 않습니다. For가 객체라면, 그 안에 ul이라는 프로퍼티가 있는 것뿐이니까요.

## 핵심 아이디어 - For은 컴포넌트이면서 객체다.

이제 JSX의 본질로 들어가봅시다.

JSX에서 아래 두 표현은 본질적으로 같습니다.

```tsx
<For />
<For.ul />
```

이는 결국 다음과 같이 해석 되기 때문이지요.

```tsx
React.createElement(For);
React.createElement(For.ul);
```

즉, For.ul이 함수(컴포넌트)라면 JSX는 아무 불만도 내지 않고 코드를 실행시킵니다. 렌더 가능한 컴포넌트를 만들 수 있다는 것이지요.

문제는 이것이죠.

- For.div
- For.ul
- For.ol
- For.section
- …

를 다 만들어야 할까요? 여기에서 우리의 구원자 `Proxy`가 등장합니다. JS Proxy에 대한 개념이 낯선 분들은 [이전에 쓴 proxy글](/middleware-of-js-proxy)과 [후속 proxy 심화 글](/proxy-advance-use)를 보면 이 유즈케이스에서 사용할 수 있으리라 짐작하실 수 있으니, 읽어보셨으면 합니다.

## Proxy를 이용한 `<For.tag>` 자동생성

우선 기본 For의 구현을 재사용합시다.

```tsx
type ForProps<T> = {
  each: readonly T[] | null | undefined;
  children: (item: T, index: number) => React.ReactNode;
  fallback?: React.ReactNode;
};

function BaseFor<T>({ each, children, fallback = null }: ForProps<T>) {
  if (!each || each.length === 0) {
    return <>{fallback}</>;
  }

  return <>{each.map(children)}</>;
}
```

그리고 아까 말했던 래퍼까지 포함된 확장형은

```tsx
function createForWithWrapper(tag: keyof JSX.IntrinsicElements) {
  return function ForWithWrapper<T>(props: ForProps<T>) {
    const { each, children, fallback } = props;

    if (!each || each.length === 0) {
      return <>{fallback}</>;
    }

    const Wrapper = tag as any;

    return <Wrapper>{each.map(children)}</Wrapper>;
  };
}
```

로 구현할 수 있겠죠. 여기서 `as any`가 싫다면 모든 HTML 태그를 선언해놓은 `htmlTag.ts` 같은 것을 선언해서 가져오면 깔끔할 것 같습니다.

그리고 이것을 `For`라는 컴포넌트 이름으로 쓸 수 있도록 연결 해주면...!

```tsx
export const For = new Proxy(BaseFor, {
  get(target, prop) {
    if (typeof prop !== "string") return undefined;

    return createForWithWrapper(prop as keyof JSX.IntrinsicElements);
  },
}) as typeof BaseFor & {
  [K in keyof JSX.IntrinsicElements]: typeof BaseFor;
};
```

아래의 로직이 동작하게 됩니다.

```tsx
<For.ul each={users}>
  {(user) => <li key={user.id}>{user.name}</li>}
</For.ul>

<For.div each={items}>
  {(item) => <span>{item}</span>}
</For.div>

<For.section each={sections}>
  {(section) => <Article section={section} />}
</For.section>
```

## 재강조 - JSX는 JS 그 자체다

여기서 오해하면 안 되는 점이 하나 있습니다. 이건 React의 특별한 기능도, JSX의 확장 문법도 아닙니다. For은 그저 함수고, For.ul은 Proxy가 반환한 함수이며, JSX는 이것을 그대로 호출했을 뿐이지요.

즉, 이 패턴에서 우리는 다음 진리를 재발견할 수 있습니다.

> JSX는 템플릿이 아니라 JavaScript 표현식일 뿐이다.

# 결론

이 글의 목적은 utilinent 라이브러리가 대단하니 쓰자! 라기보다는, JSX가 JS임을 알고 있을 때 이런 선언적이고 읽기 쉬운 코드가 ‘설계’될 수 있다는 점을 보여주는 데 있습니다. 물론 utilinent에는 여기서 소개한 것보다 더 다양한 기능이 들어 있고, 그 모든 것을 한 글에서 다루긴 어렵습니다.

다만 이 글이 JSX를 단순히 마술처럼 소비하는 것이 아니라, JS 그 자체로 바라보는 시선을 주고, “왜 고급 TS 문법이 동기부여가 되는가”까지 연결된다면 충분히 제 역할을 했다고 생각합니다.
