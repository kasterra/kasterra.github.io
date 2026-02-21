---
layout: post
title: Node API - AsyncLocalStorage 소개
subtitle: Node 기반 SSR의 임시저장소로 쓰일 수 있는 API를 알아봅시다
category: node
image: /images/thumbnails/nodejs.png
---

## 들어가며

이 블로그는 분명 프론트엔드 개발자의 블로그면서 SSR을 깊게 다룬 글은 많지 않았던 것 같습니다. 이번 글에서는 사실상 업계 표준에 가까운 Next.js와, 그 런타임 기반이 되는 Node.js의 이야기를 해보려 합니다.

웹 서비스에서 사용자 인증은 보통 세션 기반 인증이나 토큰 기반 인증으로 이루어집니다. 클라이언트는 API 요청에 자신의 인증 정보를 함께 실어 보내고, 서버는 이를 검증하여 "이 사용자가 누구인지"를 식별합니다.

Next.js와 같은 SSR 프레임워크를 사용하는 이유 중 하나는, 비어 있는 HTML이 아니라 미리 데이터가 채워진 마크업을 반환함으로써 SEO와 초기 UX를 개선하기 위함입니다. 그렇다면, 그 "미리 불러와야 하는 데이터"가 인증을 필요로 하는 데이터라면 어떻게 처리해야 할까요?

SSR 구조에서는 서버가 하나이고, 동시에 여러 사용자의 요청을 처리합니다. 각 요청은 서로 다른 쿠키, 세션, 토큰을 가지고 있습니다. 이때 서버 내부 어딘가에 "현재 로그인한 사용자" 정보를 저장해야 한다면, 그 정보는 반드시 요청 단위(request-scoped)로 격리되어야 합니다.

만약 이를 전역 변수에 저장한다면, 서로 다른 사용자의 요청이 동시에 처리되는 과정에서 인증 정보가 섞일 위험이 있습니다. 그렇다고 모든 함수 호출마다 user 정보를 인자로 전달하는 방식은 코드 전반에 인증 컨텍스트를 흩뿌리게 되어 구조를 복잡하게 만듭니다.

이 지점에서, 비동기 흐름 속에서도 요청 단위의 컨텍스트를 안전하게 유지할 수 있는 방법이 필요해집니다. 이제 AsyncLocalStorage의 이야기를 시작해보겠습니다.

## 약간 재미없는 -그러나 중요한- 기술적 배경 이야기

네. 솔직히 재미없는 이야기인거 알고 있습니다. 하지만 이 이야기를 하지 않을수도 없습니다. [혼새미로님의 async_hooks글](https://remocon33.tistory.com/707)의 내용이 훌륭한 배경설명을 하고 있어, 이 글의 내용을 재해석 하면서 이 섹션을 시작해볼까 합니다. 자세한 내용은 위의 글에서 하고 있기 때문에, 최대한 짧고 굵게 끝내보고자 합니다.

### 도입부에서도 했던 식별자 이야기

웹 서버에 요청을 전송할 때, 헤더에 `x-request-id`라는 고유값을 함께 기록하여 문제상황 등을 확인할 때, 로그를 간편하게 찾을 수 있게 하는 경우가 많습니다. 그리고, 이 고유값이 아니더라도, 도입부에서 이야기 하였듯, 서로 다른 ID로 로그인한 클라이언트들의 인증키값은 다를 수 밖에 없지요.

여기에서 가져가야 할 핵심은, '요청마다 다른 값을 가질 수 있는 웹 서버' 라는 명제에 동의하는 것 입니다.

### 싱글 스레드인 Node와, 함수별 store 확보

C++이나 Java 같은 언어로 웹서버를 작성한다고 하면, 스레드 풀에서 스레드를 꺼내서 각 요청에 할당을 할 수 있습니다. 소켓 프로그래밍 수업시간때, 스레드 기반 예제 작성 같은것들을 해본 기억을 떠올려 보면 좋을 것 같네요.

하지만, 우리가 사용하는 Node는 요청마다 스레드를 할당하는 모델이 아니라, 이벤트 루프 기반으로 비동기 작업이 인터리빙(interleaving)되는 구조를 가지고 있습니다. 즉, 자바나 C++처럼 “스레드 로컬 저장소(Thread Local Storage)”를 요청 단위로 자연스럽게 부여하는 방식이 그대로 적용되지는 않습니다. 자바스크립트 실행 자체는 단일 스레드에서 이루어지지만, I/O는 libuv의 스레드 풀과 이벤트 루프를 통해 처리되므로, 하나의 요청 흐름이 여러 비동기 경계를 넘나들며 실행됩니다. 자세한 이야기는 위에서 언급한 블로그에서 확인하시고, node의 `async_hooks` 안에 있는 `AsyncLocalStorage`라는 것을 이용하면, 비동기 함수에서 각 함수별로 독립적인 store를 확보할 수 있게 됩니다. 처음 문제제기에서 말했던, 서로 다른 인증 토큰을 관리해야 하는 `getServerSideProps`같은 상황에서 쓰일 수 있겠다 라는 것을 유추할 수 있습니다.

## AsyncLocalStorage를 활용한 예시들

이제 역사적 배경설명이 끝났으니, 본격 코드 설명에 들어갑시다.

참고로, 글의 도입부에서는 Next.js의 SSR 맥락을 예시로 들었지만, 아래의 코드는 특정 프레임워크에 종속되지 않도록 Node의 표준 http 명세(IncomingMessage, ServerResponse)를 기준으로 작성했습니다. 즉, Next.js의 getServerSideProps는 물론이고, Express, Fastify, 순수 Node 서버 환경에서도 동일한 개념으로 적용할 수 있습니다. 이 글의 초점은 프레임워크가 아니라 “요청 단위 컨텍스트 설계” 그 자체에 있습니다.

### 기초적 사용법

```ts
import { AsyncLocalStorage } from "node:async_hooks";
import type { IncomingMessage, ServerResponse } from "http";

type RequestContext = {
  requestId: string;
  accessToken?: string;
  cookies: Record<string, string>;
};

const asyncLocalStorage = new AsyncLocalStorage<RequestContext>();

function parseCookies(cookieHeader?: string): Record<string, string> {
  if (!cookieHeader) return {};
  return Object.fromEntries(
    cookieHeader.split(";").map((cookie) => {
      const [key, ...rest] = cookie.trim().split("=");
      return [key, rest.join("=")];
    }),
  );
}

export function withRequestContext(
  req: IncomingMessage,
  _res: ServerResponse,
  handler: () => Promise<void>,
) {
  const cookies = parseCookies(req.headers.cookie);

  const context: RequestContext = {
    requestId: crypto.randomUUID(),
    accessToken: cookies["access_token"],
    cookies,
  };

  return asyncLocalStorage.run(context, handler);
}

export function getRequestContext() {
  const store = asyncLocalStorage.getStore();
  if (!store) {
    throw new Error("RequestContext가 초기화되지 않았습니다.");
  }
  return store;
}
```

`withRequestContext`로 요청 단위 컨텍스트를 초기화하고, 이후 어떤 깊은 비동기 함수 안에서도 `getRequestContext()`를 통해 현재 요청의 accessToken, requestId 등을 안전하게 꺼내 쓸 수 있습니다.

해당 코드베이스가 아래에서 쓰일 구체적 용례의 기초가 될 것이니, 아래를 읽다가 기억이 안나면 다시 올라오는 식으로 사용해 주세요.

### 특정 데이터 prefetch

```ts
// userService.ts
export async function fetchMe() {
  const { accessToken } = getRequestContext();

  if (!accessToken) {
    throw new Error("인증 토큰이 없습니다.");
  }

  const response = await fetch("https://api.example.com/me", {
    headers: {
      Authorization: `Bearer ${accessToken}`,
    },
  });

  if (!response.ok) {
    throw new Error("사용자 정보 조회 실패");
  }

  return response.json();
}

// getServerSideProps 예시
export const getServerSideProps = async ({ req, res }) => {
  return withRequestContext(req, res, async () => {
    const me = await fetchMe();

    return {
      props: {
        me,
      },
    };
  });
};
```

이 구조의 장점은 `fetchMe`가 더 이상 accessToken을 인자로 받지 않는다는 점입니다. 인증 컨텍스트는 함수 시그니처가 아니라 런타임 컨텍스트에 존재합니다. 따라서 서비스 레이어는 보다 깔끔해지고, 요청 단위 격리도 유지됩니다.

#### 로깅 및 추적(Observability) 확장 예시

AsyncLocalStorage의 가치는 인증 정보 전달에만 국한되지 않습니다. 실제 서비스 환경에서는 요청 추적을 위한 `requestId`를 로깅 시스템과 연동하는 경우가 많습니다.

예를 들어, 아래와 같이 repository 레이어에서도 요청 단위 식별자를 자연스럽게 활용할 수 있습니다.

```ts
// userRepository.ts
export async function findUserById(id: string) {
  const { requestId } = getRequestContext();

  logger.info({ requestId, id }, "user lookup");

  // 실제 DB 조회 로직
  return db.user.findUnique({ where: { id } });
}
```

이 방식의 핵심은 함수 시그니처에 `requestId`를 추가하지 않으면서도, 깊은 레이어까지 요청 단위 맥락을 전달할 수 있다는 점입니다. 이는 인증을 넘어, 로깅·트레이싱·모니터링 체계와 자연스럽게 결합될 수 있는 설계입니다.

### accessToken 만료시 refresh하는 예제

```ts
async function refreshAccessToken(refreshToken: string) {
  const response = await fetch("https://api.example.com/refresh", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ refreshToken }),
  });

  if (!response.ok) {
    throw new Error("토큰 재발급 실패");
  }

  return response.json() as Promise<{ accessToken: string }>;
}

export async function fetchWithAutoRefresh(
  input: RequestInfo,
  init?: RequestInit,
) {
  const ctx = getRequestContext();

  const response = await fetch(input, {
    ...init,
    headers: {
      ...init?.headers,
      Authorization: `Bearer ${ctx.accessToken}`,
    },
  });

  if (response.status !== 401) {
    return response;
  }

  // accessToken 만료 → refresh 시도
  const refreshToken = ctx.cookies["refresh_token"];
  if (!refreshToken) {
    throw new Error("refresh_token이 없습니다.");
  }

  const { accessToken: newAccessToken } =
    await refreshAccessToken(refreshToken);

  // 현재 요청 컨텍스트 업데이트
  ctx.accessToken = newAccessToken;

  return fetch(input, {
    ...init,
    headers: {
      ...init?.headers,
      Authorization: `Bearer ${newAccessToken}`,
    },
  });
}
```

이 예제에서 중요한 포인트는 refresh 이후에도 같은 요청 컨텍스트 안에서 accessToken이 갱신된다는 점입니다. 즉, 동일 요청 흐름 내에서는 항상 최신 토큰이 사용됩니다. 전역 변수를 건드리지 않으면서도, 요청 단위 상태를 안전하게 변경할 수 있다는 것이 AsyncLocalStorage의 핵심 가치입니다.

### 추가 - 구조 확장에 대한 짧은 고찰

지금 예제에서는 `withRequestContext`가 하나의 `handler` 콜백만을 받아 실행하도록 되어 있습니다. 하지만 실제 서비스 코드에서는 이 구조를 더 확장할 여지가 충분히 있습니다.

예를 들어, 여러 개의 비동기 작업을 순차적으로 실행해야 한다면 단일 `handler` 내부에서 모두 처리할 수도 있고, 아래처럼 유틸 레벨에서 여러 콜백을 조합하는 방식으로 확장할 수도 있습니다.

```ts
export async function runWithContext(
  req: IncomingMessage,
  res: ServerResponse,
  ...tasks: Array<() => Promise<void>>
) {
  return withRequestContext(req, res, async () => {
    for (const task of tasks) {
      await task();
    }
  });
}
```

또는, 요청 컨텍스트를 단순 함수가 아니라 클래스로 감싸는 방식도 고려할 수 있습니다. 예를 들면 `RequestScope`라는 클래스를 만들어 내부에서 `AsyncLocalStorage`를 관리하고, `run`, `get`, `set` 등의 메서드를 노출하는 식입니다. 이렇게 하면 인증 정보 외에도 로깅 메타데이터, 트랜잭션 정보, 추적 ID 등을 점진적으로 확장해 나갈 수 있습니다.

핵심은 `AsyncLocalStorage` 자체가 목적이 아니라, “요청 단위 스코프”라는 개념을 명시적으로 도입하는 것입니다. 함수형으로 감싸든, 클래스로 추상화하든, 이 스코프를 명확히 설계하는 것이 SSR 환경에서의 안정성과 가독성을 좌우합니다.

이제 AsyncLocalStorage를 단순한 API 소개 수준이 아니라, 아키텍처 레벨의 도구로 바라볼 준비가 되셨기를 바랍니다.

## AsyncLocalStorage 사용 시 주의할 점

AsyncLocalStorage는 강력한 도구이지만, 몇 가지 현실적인 제약과 주의사항이 존재합니다.

### 1. 성능 오버헤드

AsyncLocalStorage는 내부적으로 `async_hooks`를 기반으로 동작합니다. 이는 모든 비동기 리소스의 생성과 해제 과정에 훅이 개입함을 의미합니다. 일반적인 서비스에서는 큰 문제가 되지 않지만, 초고성능이 요구되는 환경에서는 미세한 오버헤드가 누적될 수 있습니다. 도입 전에는 실제 트래픽 환경에서의 벤치마킹이 권장됩니다.

### 2. 컨텍스트 전파가 깨질 수 있는 경우

대부분의 표준 비동기 흐름에서는 컨텍스트가 유지되지만, 다음과 같은 경우에는 주의가 필요합니다.

- `worker_threads`를 사용하는 경우
- 일부 네이티브 모듈이 내부적으로 비동기 경계를 재구성하는 경우
- 별도의 런타임 경계를 넘는 경우

이러한 상황에서는 컨텍스트가 자동으로 전파되지 않을 수 있으므로, 구조 설계 시 명확한 런타임 경계를 인지해야 합니다.

### 3. Edge Runtime에서는 사용 불가

AsyncLocalStorage는 Node.js 런타임 API입니다. 따라서 Next.js의 Edge Runtime과 같은 Web Standard 기반 환경에서는 사용할 수 없습니다. Edge 환경에서는 별도의 요청 컨텍스트 설계 전략이 필요합니다.

## 마무리하며

결국 AsyncLocalStorage는 “편리한 전역 변수”가 아니라, 런타임 차원에서 요청 스코프를 명시적으로 도입하는 도구입니다. 이 점을 이해하고 사용한다면, SSR 환경에서의 인증·로깅·추적 설계를 보다 안정적으로 구축할 수 있습니다.

끝까지 읽어주셔서 감사드립니다.
