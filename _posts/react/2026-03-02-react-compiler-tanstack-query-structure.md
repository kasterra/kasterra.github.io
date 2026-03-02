---
layout: post
title: React Compiler 시대의 TanStack Query 설계 전략
subtitle: Hook은 얇게, option은 응집되게 — 현대 React에서의 구조적 선택
category: react
image: /images/thumbnails/React.png
---

## 들어가며 - 비슷한 성격의 Query를 한데 묶어보기

일반론적인 이야기를 해봅시다. 구직시장의 JD를 살펴보면 Tanstack query를 사용해서 Data fetching을 관리해본 사람이 필수 혹은 우대사항으로 있는 것을 찾아보기 어렵지 않을 정도로, 사실상의 업계 표준으로 자리 잡았습니다.

그리고 어떤 기술을 사용하던 간에, 소프트웨어 개발에 있어서, 여러 모듈들의 독립성을 높혀서 유지보수를 원활히 하는 것이 본질적으로 중요한 것이지요. 관련 개념이 낯설거나, 배운지 오래되서 가물거리시는 분들은 [inpa님의 글](https://inpa.tistory.com/entry/OOP-💠-객체의-결합도-응집도-의미와-단계-이해하기-쉽게-정리)을 보고 오시면 좋을 것 같습니다.

이 두가지 이야기를 듣고 예측하셨을 수 있겠지만, 이번 글에서는 react 프로젝트에서 tanstack query를 사용할때에, 소프트웨어 모듈 관점에서 어떻게 작성하면 좋을까에 대한 내용을 다루고자 합니다.

## 비슷한 역할을 묶는 고전적 도구 - Class

Javascript는 다양한 문법적 도구를 지원합니다. 이번 섹션에서 주목할 문법적 도구는, 복잡한 현실을 Object Oriented모델로 추상하는데에 핵심 도구인 `class` 문법 입니다.

글의 앞부분에서 언급했던, react 기반으로 구성되어 있고, tanstack query를 사용해서 외부 요청을 관리하는 프로젝트를 조금 더 구체적으로 상상해 봅시다. 해당 프로젝트에서 사용자 인증을 수행한다고 가정하죠. 다음과 같은 mutation을 class에 묶어서 관리할 수 있을 것이라고 예측할 수 있습니다.

이를테면 export class AuthQueries와 같이 정의하고, 그 안에

- `loginMutation`
- `logoutMutation`
- `changePasswordMutation`

와 같은 메서드로 관리한다 라는 아이디어 인 것이죠.

아래는 "클래스에 Mutation을 묶어두면 깔끔하지 않을까?"라는 발상을 그대로 코드로 옮긴 예시입니다.

```ts
import { useMutation } from "@tanstack/react-query";

type LoginInput = { email: string; password: string };
type LoginResult = { accessToken: string };

// 예시용 API 함수
async function postLogin(input: LoginInput): Promise<LoginResult> {
  const res = await fetch("/api/login", {
    method: "POST",
    headers: { "content-type": "application/json" },
    body: JSON.stringify(input),
  });

  if (!res.ok) throw new Error("login failed");
  return (await res.json()) as LoginResult;
}

export class AuthQueries {
  /** 로그인 Mutation */
  loginMutation() {
    return useMutation({
      mutationFn: (input: LoginInput) => postLogin(input),
    });
  }

  /** 로그아웃 Mutation (예시) */
  logoutMutation() {
    return useMutation({
      mutationFn: async () => {
        const res = await fetch("/api/logout", { method: "POST" });
        if (!res.ok) throw new Error("logout failed");
        return true;
      },
    });
  }

  /** 비밀번호 변경 Mutation (예시) */
  changePasswordMutation() {
    return useMutation({
      mutationFn: async (nextPassword: string) => {
        const res = await fetch("/api/password", {
          method: "POST",
          headers: { "content-type": "application/json" },
          body: JSON.stringify({ nextPassword }),
        });
        if (!res.ok) throw new Error("change password failed");
        return true;
      },
    });
  }
}
```

겉으로는 `AuthQueries`라는 단일 진입점 아래에 인증 관련 Mutation들이 모여 있어서 "응집도가 높아 보이는" 느낌을 줍니다.

하지만 위 코드는 **React Hook 규칙(Rules of Hooks)**을 정면으로 위반합니다. `useMutation`은 Hook이므로 **리액트 함수 컴포넌트 혹은 커스텀 훅의 최상위에서만** 호출되어야 하는데, 위처럼 **임의의 클래스 메서드 내부에서 호출**되면 ESLint의 `react-hooks/rules-of-hooks`에 걸리고(또는 런타임에서 예상치 못한 동작을 유발할 수 있고), 무엇보다 "렌더 순서/호출 순서가 React가 추적 가능한 형태"로 보장되지 않습니다.

즉, "Mutation을 클래스에 묶어 관리"라는 아이디어는 **OOP 관점의 묶기(응집도)**는 얻을 수 있어 보이지만, React의 실행 모델(함수형 렌더링 + Hook 호출 규칙)과 충돌하기 때문에 실전에서는 그대로 쓰기 어렵습니다.

또한 다음 섹션에서 다룰 문제에 대해서도 꽤나 큰 문제가 생기게 됩니다.

## 현대 React 환경중 하나가 되가는 React Compiler

React Compiler는 React 팀이 주도하여 발전시키고 있는 정적 분석 기반 컴파일러로, 기존의 "개발자가 직접 memoization을 설계"하던 방식을 점진적으로 대체하는 것을 목표로 합니다. 과거에는 `useMemo`, `useCallback`, `memo` 등을 통해 "이 값은 안정적이다", "이 함수는 동일하다"를 사람이 직접 표현해야 했다면, React Compiler는 **컴파일 타임에 컴포넌트의 순수성(purity)과 의존성 관계를 분석**하여 자동으로 최적화를 수행하려 합니다.

핵심 전제는 명확합니다.

- 컴포넌트는 순수 함수여야 한다.
- Hook 호출은 정적으로 추론 가능해야 한다.
- 렌더링 결과는 입력(props, state)에만 의존해야 한다.

이 말은 곧, "React의 실행 모델을 교란하는 구조"는 점점 더 강하게 제약될 가능성이 높다는 뜻이기도 합니다.

앞서 살펴본 `class AuthQueries` 예시는 단순히 ESLint 규칙 위반에서 끝나는 문제가 아닙니다. React Compiler의 관점에서 보면 다음과 같은 문제가 발생합니다.

1. **Hook 호출 위치가 정적으로 고정되어 있지 않다.**  
   클래스 인스턴스를 어떻게 생성하고, 언제 메서드를 호출하는지에 따라 Hook 호출 순서가 달라질 수 있습니다.

2. **컴파일러가 의존성 그래프를 추론하기 어렵다.**  
   함수형 컴포넌트/커스텀 훅이 아닌 임의의 객체 메서드 내부에 Hook이 숨겨져 있으면, 정적 분석 단계에서 해당 호출을 안정적으로 추적하기 힘듭니다.

3. **향후 최적화 패스에서 예외 케이스로 분류될 가능성**  
   React Compiler는 "표준적인 함수형 패턴"을 전제로 최적화를 설계하고 있기 때문에, 클래스 기반 우회 패턴은 최적화 대상에서 제외되거나 경고/에러로 처리될 여지가 있습니다.

즉, 지금 당장은 "어떻게든 동작하게 만들 수 있는 코드"라 하더라도, React가 점점 더 **컴파일 타임 최적화 중심의 프레임워크**로 이동하는 흐름 속에서는 기술 부채로 전환될 확률이 높습니다.

결론적으로, TanStack Query 구조를 설계할 때에도 단순히 "응집도가 높아 보이는가?"가 아니라,

- 이 구조가 **Hook 규칙을 정적으로 보장하는가?**
- React Compiler가 분석하기에 **예측 가능한 함수형 구조인가?**

를 함께 고려해야 합니다.

다음 섹션에서는 이러한 제약을 만족시키면서도 "비슷한 Query를 논리적으로 묶는" 보다 React 친화적인 전략을 살펴보겠습니다.

## 해결책 - option만 class 등으로 분리하기

앞선 문제의 핵심은 이것이었습니다.

> Hook을 "묶으려" 했기 때문에 문제가 생겼다.

그렇다면 방향을 바꿔봅시다. Hook 자체를 묶는 것이 아니라, **Hook에 전달되는 option(설정 객체)만을 묶는 것**입니다.

TanStack Query의 `useQuery`, `useMutation`은 결국 아래와 같은 형태를 가집니다.

```ts
useMutation({
  mutationFn,
  mutationKey,
  onSuccess,
  onError,
  ...
})
```

여기서 React의 실행 모델과 직접적으로 연결되는 부분은 `useMutation`이라는 Hook 호출입니다.
하지만 `mutationFn`, `mutationKey`, 각종 콜백 등은 **단순한 데이터(설정 객체)**에 불과합니다.

즉, 우리가 묶어야 할 것은 Hook이 아니라 "설정"입니다.

### 1 option을 정적 객체/팩토리로 분리하기

예를 들어 다음과 같이 구성할 수 있습니다.

```ts
// src/queries/auth.mutationOptions.ts

import { MutationOptions } from "@tanstack/react-query";

type LoginInput = { email: string; password: string };
type LoginResult = { accessToken: string };

async function postLogin(input: LoginInput): Promise<LoginResult> {
  const res = await fetch("/api/login", {
    method: "POST",
    headers: { "content-type": "application/json" },
    body: JSON.stringify(input),
  });

  if (!res.ok) throw new Error("login failed");
  return res.json();
}

export const authMutationOptions = {
  login(): MutationOptions<LoginResult, Error, LoginInput> {
    return {
      mutationKey: ["auth", "login"],
      mutationFn: postLogin,
    };
  },

  logout(): MutationOptions<boolean, Error, void> {
    return {
      mutationKey: ["auth", "logout"],
      mutationFn: async () => {
        const res = await fetch("/api/logout", { method: "POST" });
        if (!res.ok) throw new Error("logout failed");
        return true;
      },
    };
  },
};
```

이 파일에는 **Hook이 단 한 줄도 등장하지 않습니다.** 단지 "인증 도메인에 속한 mutation 설정"만을 논리적으로 묶고 있을 뿐입니다.

### 2 실제 Hook은 컴포넌트/커스텀 훅에서 호출

이제 Hook은 다음처럼 사용합니다.

```ts
// src/features/auth/useLoginMutation.ts

import { useMutation } from "@tanstack/react-query";
import { authMutationOptions } from "@/queries/auth.mutationOptions";

export function useLoginMutation() {
  return useMutation(authMutationOptions.login());
}
```

여기서는

- Hook은 최상위에서 호출되고
- 호출 순서는 정적으로 고정되어 있으며
- React Compiler가 분석하기에도 예측 가능한 구조입니다.

우리는 여전히 "auth 관련 mutation이 한 곳에 모여 있는" 응집도를 얻으면서도, React의 함수형 실행 모델을 전혀 침범하지 않습니다.

### 3 왜 이 구조가 더 현대적인가

이 구조는 다음 조건을 만족합니다.

- ✅ Hook 규칙을 완전히 준수한다.
- ✅ React Compiler가 정적 분석하기에 안전하다.
- ✅ 도메인 단위로 query/mutation을 논리적으로 묶을 수 있다.
- ✅ 테스트 시 option 단위로 mocking이 가능하다.

특히 중요한 점은, 이제 우리는 OOP의 "클래스로 묶기" 대신 **함수형 환경에 맞는 방식으로 응집도를 설계하고 있다는 점**입니다.

React는 점점 더 "컴파일 타임 최적화 + 함수형 추론" 중심으로 이동하고 있습니다.
그 흐름 위에서 TanStack Query 구조를 설계한다면, Hook은 최대한 얇게 두고,
**설정과 도메인 로직을 분리하는 전략**이 장기적으로 더 안전합니다.

결국 우리가 지켜야 할 것은 이것입니다.

- Hook은 React의 영역이다.
- option은 우리의 설계 영역이다.

이 경계를 명확히 나누는 순간, 구조는 훨씬 단단해집니다.

## 마무리하며

지금까지의 블로그 글과는 다르게, 구현 위주의 설명이 아닌, 약간의 소프트웨어 공학적 내용을 담은 글을 써보았습니다.

물론 구현을 하는 것은 여느때나 중요하지만, 생성형 인공지능의 발달로, 구현 자체의 난이도가 낮아진 현 시점에서 인간 개발자가 AI보다 더 잘하는 소프트웨어 공학 같은 설계적인 부분에 대해서도 관심을 가지는것이 중요하다 생각합니다.

앞으로도, SW 개발을 하면서 느끼는 점들을 정리해서 더욱 풍성하고 의미있는 블로그를 만들어가기 위해 노력해보고자 합니다. 끝까지 읽어주셔서 감사합니다.
