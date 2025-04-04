---
layout: post
title: 맨바닥부터 react query 따라해보기
subtitle: 라이브러리 단순 사용을 넘어서, 블랙 박스를 재현해보기
category: react
image: /images/thumbnails/React.png
---

# 들어가며

프론트엔드 채용 공고에 많이 나오기도 하고, 관련 글들도 참 많이 나오는 ‘상태 관리 라이브러리’에 대해서 밀도있게 다루는 글을 써보고자 합니다. 3년 넘는 시간동안 블로그를 써보면서 프론트엔드를 공부하고 싶다라는 생각으로 야심차게 시작한 공부에서 부터, 더 나은 개발자로 도약하겠다라는 희망을 품고 살고 있는 2025년 봄이 오기까지, ‘서버 상태 관리 라이브러리’에 대해서 다룬 적이 없다라는 것 또한 인지하게 되어, 이번글의 주제는 ‘맨바닥부터 만들어보는 React query’로 할까 합니다.

이번 글에서는 tanstack query의 `useQuery` 의 코어 기능을 구현해보겠습니다. 상태 저장 매커니즘과 데이터 페칭, stale time 등의 개념을 다루어 보고자 합니다. 저번 시리즈인 ‘면접에서 이야기 할 수 있는 React 지식’ 시리즈에 이어서 [Philip Fabinek님의 영상 튜토리얼](https://www.youtube.com/watch?v=YmrnPOgOvy4)을 참조하여 구성 하였습니다.

이 글은 기초적인 React query 라이브러리 사용 경험이 있다는 전제와, React Context가 무엇인지 알고 있다는 전제 하에 쓰였습니다. 혹시 이해가 잘 안되는 부분이 있다면 공식 docs 및 튜토리얼들을 참고해 주시면 좋을 듯 합니다.

## 들어가기 전에 고지 사항

이 글은 React query를 100% 재현하는 것을 목표로 하지 않습니다. 코어 컨셉트의 빠른 설명을 위해서 의도적으로 누락한 부분이 있으며, 해당 내용들은 글을 마무리 하면서 정리하겠습니다.

라이브러리를 만들어 보는 글인 만큼 TS 대응 또한 중요한 사항이기 때문에, 타입을 붙이면서 진행하겠습니다.

# 시작 - 프로젝트 설정하기

해당 섹션은 프로젝트 셋업이 주요한 일이고, 기존의 라이브러리의 동작을 체험해보는 것에 가깝기에 복사-붙여넣기의 반복입니다.

2025년 4월 현재, 간단히 React 프로젝트를 슥삭 하고 셋업하는 방법인 vite로 프로젝트를 셋업합시다.

```bash
npm create vite@latest react-query-from-scratch -- --template react-ts
```

기본적인 템플릿에서 제공하는 예제들과 스타일을 일부 삭제하고 진행하겠습니다.

```tsx
function App() {
  return <></>;
}

export default App;
```

만 남기고, `App.css` 도 삭제하고 말이죠.

데이터를 가져오는 상황을 전제하므로, 가상의 data fetching 역할을 할 함수를 `App.tsx`에 작성해 보겠습니다. json fake api 등을 사용해도 큰 문제는 없을 듯 합니다.

```tsx
const fetchData = () => {
  console.log("fetching data...");
  return new Promise((resolve) =>
    setTimeout(() => {
      resolve([
        {
          name: "Title 1",
          description: "This is post 1",
        },
        {
          name: "Title 2",
          description: "This is post 2",
        },
      ]);
    }, 1000)
  );
};
```

## 따라 만들기 전에 원본을 써보기

react query를 직접 만들어보기 이전에, 실제 react query의 동작을 직접 확인하고 실습을 시작해 봅시다.

```bash
npm i @tanstack/react-query
```

그리고, react-query라이브러리를 사용해봤다면 처음에 진행하게 될, queryClient설정도 `index.tsx` 에 해줍시다.

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App.tsx";
import "./index.css";

import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient();

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
    </QueryClientProvider>
  </StrictMode>
);
```

react query로 데이터를 페칭해서 렌더링 하는 컴포넌트까지 해서 `App.tsx`를 완성시킵시다.

```tsx
import { useQuery } from "@tanstack/react-query";

const fetchData = () => {
  console.log("fetching data...");
  return new Promise((resolve) =>
    setTimeout(() => {
      resolve([
        {
          name: "Title 1",
          description: "This is post 1",
        },
        {
          name: "Title 2",
          description: "This is post 2",
        },
      ]);
    }, 1000)
  );
};

export const usePostsQuery = () => {
  return useQuery({
    queryKey: ["postsData"],
    queryFn: fetchData,
  });
};

const Posts = ({ title }: { title: string }) => {
  const { status, isFetching, error, data } = usePostsQuery();

  if (status === "pending") return "Loading...";

  if (error) {
    return `An Error has occured: ${error.message}`;
  }

  return (
    <div style={{ padding: 20 }}>
      <h1>{title}</h1>
      {isFetching && <p>Refetching...</p>}
      {data.map((post) => (
        <div key={post.name}>
          <h2>{post.name}</h2>
          <p>{post.description}</p>
        </div>
      ))}
    </div>
  );
};

function App() {
  return (
    <div style={{ display: "flex" }}>
      <Posts title="Posts 1" />
      <Posts title="Posts 2" />
    </div>
  );
}

export default App;
```

이렇게 구성한 다음 `npm run dev` 로 실행을 해보면

![image.png](/images/react/something-from-scratch/react-query/1-react-query-behaviour.png)

이렇게 console.log가 한번만 뜨는것을 볼 수 있고, react query의 상태관리 스토어 덕분임을 간단히 알 수 있습니다.

# 라이브러리 보일러 플레이트 만들기

지금까지 간단한 복사-붙여넣기를 통해서 react-query가 동작하는 모습을 간단히 알아보았습니다. 지금부터는, 기성품같은 라이브러리로 경험했던 해당 결과물을, 우리의 손으로 재현 해 봅시다.

지금부터의 작업물은 `src/react-query`라는 폴더를 만들어서 그 안에서 진행됩니다.

설계상 다음과 같은 구조로 진행이 됩니다.

```
src/
├─ react-query/
│  ├─ QueryClient.ts
│  ├─ QueryClientProvider.tsx
│  ├─ createQuery.ts
│  ├─ useQuery.ts
│  ├─ types.ts
│  └─ index.ts
```

## 그리고 다른 보일러플레이트…

해당 내용들을 구현해야 하지만, 전체적인 아웃라인을 위해서 사실상 빈 파일들을 만들어 놓읍시다.

`QueryClient.ts`

```tsx
export class QueryClient {}
```

`useQuery.ts`

```tsx
export const useQuery = ({ queryKey, queryFn }) => {
  return {};
};
```

`index.ts`

```tsx
import { QueryClient } from "./QueryClient";
import { QueryClientContext, QueryClientProvider } from "./QueryClientProvider";
import { useQuery } from "./useQuery";

export { QueryClient, QueryClientContext, QueryClientProvider, useQuery };
```

`createQuery.ts`

```tsx
export const createQuery = ({ queryKey, queryFn }) => {};
```

`types.ts` 는 구현할 목표의 청사진을 설명한 후에 완성할 수 있도록 틀만 만들어 두겠습니다.

```tsx
export type QueryKey = string[];
export type QueryFn<T> = () => Promise<T>;

export interface QueryState<T> {
  queryKey: QueryKey;
  queryHash: string;
  status: "pending" | "success" | "error";
  isFetching: boolean;
  data?: T;
  error?: Error;
}

export interface Query<T> {
  queryKey: QueryKey;
  queryHash: string;
  state: QueryState<T>;
  fetch: () => Promise<void>;
  subscribe: (cb: () => void) => () => void;
}
```

그리고 `main.tsx` `App.tsx`의 import 들을 기성품 라이브러리가 아닌, 우리의 react-query로 import로 바꿔 놓읍시다.

# 구현물의 개념적 설명 + 구조

도입부에서도 잠깐 이야기 했지만, react query에는 여러 기능들이 있지만, 이번 글에서는 ‘상태 관리 라이브러리’로서의 react query에 대해서 집중하고자 합니다. 정확히 말하자면 react query의 queryClient의 캐싱, 컴포넌트간 데이터 공유에 집중하고자 함을 염두에 두시고 글을 읽어 나가주셨으면 좋겠습니다.

## 1. Context로 공유되는 query들

![image.png](/images/react/something-from-scratch/react-query/react-query-concept-1.png)

도식으로 나타내자면 다음과 같습니다. 우리가 복사-붙여넣기를 통해서 구성한 프로젝트에서는 query client가 React context로 공유되어, 컴포넌트에서 `useQuery`훅을 통해서 사용할 수 있게 되지요

![image.png](/images/react/something-from-scratch/react-query/react-query-concept-2.png)

그리고 다른 컴포넌트더라도, 같은 query를 `useQuery`훅을 통해서 사용하려고 하면, 새로운 query를 만드는 것이 아니고, 기존의 query에 연결됩니다. stale time 이내에 같은 조건으로 요청을 굳이 여러번 보낼 필요는 없으니까요

## 2. 상태와 생명주기를 가진 query들

Query Client안에 보관되는 query들은, 상태들을 가지고 있을 것 입니다. 데이터를 가져오기 있는 상황(진행중, 완료), 데이터, 에러 등등이죠. 상태값을 가진 object로 구현을 하면 될 듯 합니다. 그리고 그러한 상태가 변하면, 해당 query를 사용하고 있는 오브젝트들에게 알려줘서 다시 최신 상태에 맞게 리렌더링 되게 해야 할 것입니다. 이는 어떻게 구현할 수 있을까요?

## 3. query 구독을 위한 observer pattern

디자인 패턴 교과서에 나올법한 상황 이군요. observer pattern을 사용하면 가장 적합한 상황입니다. pub-sub 패턴이라고 해도 큰 문제는 없을것 같네요. query object안에 subscriber 배열도 넣어서, 상태값이 변할때 마다 자신을 구독하는 컴포넌트들에게 알려주는 식으로 구현을 해야겠군요.

컴포넌트가 더이상 렌더링 될 필요가 없어서 detatch될 때 구독을 취소하는 그러한 필수적인 기능도 함께 구현을 하면 될 듯 합니다.

# 라이브러리 만들어보기

보일러 플레이트를 만들고 무엇을 만들지에 대한 청사진도 정리를 하였으니, 이제는 코드를 써내려가는 일만 남았습니다.

## type.ts

이전 섹션에서의 청사진을 토대로, 흐름을 잡기 위해서 타입 정의부터 해봅시다.

```tsx
export type QueryKey = string[];
export type QueryFn<T> = () => Promise<T>;

export interface QueryState<T> {
  queryKey: QueryKey;
  queryHash: string;
  status: "pending" | "success" | "error";
  isFetching: boolean;
  data?: T;
  error?: Error;
}

export interface Query<T> {
  queryKey: QueryKey;
  queryHash: string;
  state: QueryState<T>;
  fetch: () => Promise<void>;
}

export interface UseQueryOptions<T> {
  queryKey: QueryKey;
  queryFn: QueryFn<T>;
}
```

## QueryClientProvider.tsx

react context로 QueryClient를 내려줘서 props drilling을 방지하는 딱히 첨언할 것 없는 간단한 코드입니다.

```tsx
import { createContext, ReactNode } from "react";
import type { QueryClient } from "./QueryClient";

export const QueryClientContext = createContext<QueryClient | null>(null);

interface Props {
  client: QueryClient;
  children: ReactNode;
}

export const QueryClientProvider = ({ client, children }: Props) => {
  return (
    <QueryClientContext.Provider value={client}>
      {children}
    </QueryClientContext.Provider>
  );
};
```

## QueryClient.ts

이전 섹션에서 언급한 바와 같이, query object들을 저장하고, 관리하는 역할을 담당할 부분입니다. `useQuery`등으로 query들에 접근하는 것을 관리해 주어야죠.

`types.ts`에서 선언한 타입들을 불러오고

```tsx
import type { Query, QueryKey, QueryFn } from "./types";
```

**query 여러개들을 담을 프로퍼티를 만들어주고**

```tsx
  private queries: Query<unknown>[] = [];
```

**요청한 쿼리가 있으면 가져오고 없으면 만드는 메서드를 만들어줍시다**

```tsx
  getQuery<T>({
    queryKey,
    queryFn,
  }: {
    queryKey: QueryKey;
    queryFn: QueryFn<T>;
  }): Query<T> {
    const queryHash = JSON.stringify(queryKey);

    let query = this.queries.filter((q) => q.queryHash === queryHash);

    if (!query) {
      // TODO: query 만들기
      query = {};
      this.queries.push(query);
    }

    return query;
  }
```

query object는 이전 섹션에서 다루었듯이 간단한 인라인 오브젝트라기 보다는 여러 기능들을 담고 있기에, query 만드는 부분을 분리하는 것이 좋을 듯 합니다.

내부적으로는 `unknown` 타입으로 관리하지만, 실제로 사용자가 보는 부분은 `getQuery`로 반환되는 쿼리이고, 이는 인자인 queryFn의 타입으로 어느정도 추론할 수 있기 때문에, 타입에 관한 DX를 조금 더 챙길 수 있게 됩니다.

## createQuery.ts

이전 섹션에서 query object가 어떤 역할을 해야하는지에 대한 대략적인 명세를 해보았으니, 이를 코드로 옮겨봅시다.

```tsx
import { Query, QueryFn, QueryKey, QueryState } from "./types";

export const createQuery = <T,>({
  queryKey,
  queryFn,
}: {
  queryKey: QueryKey;
  queryFn: QueryFn<T>;
}): Query<T> => {
  const query = {
    queryKey,
    queryHash: JSON.stringify(queryKey),
    state: {
      status: "pending",
      isFetching: false,
    } as QueryState<T>,
    fetch: async (): Promise<void> => {
      query.state = {
        ...query.state,
        isFetching: true,
        error: undefined,
      };

      try {
        const data = await queryFn();
        query.state = {
          ...query.state,
          data,
          status: "success",
          isFetching: false,
        };
      } catch (error) {
        query.state = {
          ...query.state,
          error: error as Error,
          status: "error",
          isFetching: false,
        };
      }
    },
  };

  return query;
};
```

데이터를 가져오고, 상태를 저장한다는 기능에는 분명히 충실하지만, 각기 다른 컴포넌트에서 동시에 이 query를 사용해서 데이터를 가져오게 하면, 요청을 여러번 할것이 자명하니, 약간의 수정을 해보겠습니다. 접근하는 시점에 fetch가 실행중이면, 굳이 한번 더 호출 하지 않게 막아줍시다.

간단한 조건문을 `fetch` 메소드의 맨 윗줄에 넣어서 fetching이 진행중인 상황에서는 추가로 fetch가 진행되지 않게 해봅시다.

```tsx
if (query.state.isFetching) return;
```

### observer pattern 관련 기능 구현

현재의 구조를 조금 더 확장시켜서, pub-sub 패턴이라고 불리는 observer pattern과 관련된 요소 역시 추가해서 상태가 변경되었을 때, 최신 정보를 담아서 렌더링 될 수 있도록 해봅시다.

```tsx
export type QueryKey = string[];
export type QueryFn<T> = () => Promise<T>;

export interface QueryState<T> {
  queryKey: QueryKey;
  queryHash: string;
  status: "pending" | "success" | "error";
  isFetching: boolean;
  data?: T;
  error?: Error;
}

// 구독을 관리하기 위한 타입
export interface QuerySubscriber {
  notify: () => void;
}

export interface Query<T> {
  queryKey: QueryKey;
  queryHash: string;
  state: QueryState<T>;
  fetch: () => Promise<void>;
  // 아래의 세가지가 추가됨
  subscribe: (subscriber: QuerySubscriber) => () => void;
  setState: (updater: (prev: QueryState<T>) => QueryState<T>) => void;
  subscribers: QuerySubscriber[];
}
```

어려운것은 없으니 기능을 붙이는 느낌으로 빠르게 갑시다. 불변성을 지키면서 상태관리를 하는 react 개발 경험이 있고, pub-sub 패턴에 대한 이해가 있다면 어려울 내용이 없으니까요. subscribe 함수의 콜백으로 구독자에게 알림을 주는… 어렵지 않죠?

```tsx
// createQuery.ts

import { Query, QueryFn, QueryKey, QueryState, QuerySubscriber } from "./types";

export const createQuery = <T,>({
  queryKey,
  queryFn,
}: {
  queryKey: QueryKey;
  queryFn: QueryFn<T>;
}): Query<T> => {
  const queryHash = JSON.stringify(queryKey);

  let state: QueryState<T> = {
    queryKey,
    queryHash,
    status: "pending",
    isFetching: false,
  };

  const subscribers: QuerySubscriber[] = [];

  const setState = (updater: (prev: QueryState<T>) => QueryState<T>) => {
    state = updater(state);
    subscribers.forEach((s) => s.notify());
  };

  const subscribe = (subscriber: QuerySubscriber): (() => void) => {
    subscribers.push(subscriber);
    return () => {
      const index = subscribers.indexOf(subscriber);
      if (index !== -1) subscribers.splice(index, 1);
    };
  };

  const fetch = async (): Promise<void> => {
    setState((prev) => ({
      ...prev,
      isFetching: true,
      error: undefined,
    }));

    try {
      const data = await queryFn();
      setState((prev) => ({
        ...prev,
        data,
        status: "success",
        isFetching: false,
      }));
    } catch (error) {
      setState((prev) => ({
        ...prev,
        error: error as Error,
        status: "error",
        isFetching: false,
      }));
    }
  };

  return {
    queryKey,
    queryHash,
    state,
    fetch,
    setState,
    subscribe,
    subscribers,
  };
};
```

이제, `createQuery`를 모두 만들었으니, `QueryClient.ts`에 import 해서 사용하도록 합시다.

```tsx
import { createQuery } from "./createQuery";
import type { Query, QueryKey, QueryFn } from "./types";

export class QueryClient {
  private queries: Query<unknown>[] = [];
  getQuery<T>({
    queryKey,
    queryFn,
  }: {
    queryKey: QueryKey;
    queryFn: QueryFn<T>;
  }): Query<T> {
    const queryHash = JSON.stringify(queryKey);

    let query = this.queries.filter((q) => q.queryHash === queryHash)[0];

    if (!query) {
      query = createQuery({
        queryKey,
        queryFn,
      });
      this.queries.push(query);
    }

    return query;
  }
}
```

## useQuery.ts

여기까지만 구현을 하면 처음 react-query 원본을 사용했다가, import를 돌려놓은 우리의 예제를 실행시킬 수 있습니다.

핵심적인 query 상태관리에 관한 사항들은 구현을 다 하였으니, 사실상 남은 부분은 pub-sub 패턴에서 구독자의 역할을 수행할 observer 구현입니다. 데이터에 접근하고, 변경사항이 있으면 리렌더링을 발생시키고, cleanup 함수를 통해서 구독 해제를 하는 작업이 필요합니다.

여기서 유의할 점은 observer는 단순한 값이 아니라 내부에 메서드를 갖는 state 객체라는 것입니다. 이 객체는 컴포넌트의 라이프사이클 동안 변하지 않고 **항상 동일한 인스턴스를 유지**해야 하며, 그렇기 때문에 useRef를 이용한것에 유의한다면, 어려울 포인트는 없을 것 같습니다.

```tsx
import { useContext, useEffect, useRef, useState } from "react";
import { QueryClientContext } from "./QueryClientProvider";
import type { QueryClient } from "./QueryClient";
import type { QueryState, UseQueryOptions } from "./types";

interface QueryObserver<T> {
  notify: () => void;
  subscribe: (rerender: () => void) => () => void;
  getQueryState: () => QueryState<T>;
}

const createQueryObserver = <T,>(
  queryClient: QueryClient,
  { queryKey, queryFn }: UseQueryOptions<T>
): QueryObserver<T> => {
  const query = queryClient.getQuery({
    queryKey,
    queryFn,
  });

  const observer: QueryObserver<T> = {
    notify: () => {},
    subscribe: (rerender) => {
      const unsubscribe = query.subscribe(observer);
      observer.notify = rerender;
      query.fetch();
      return unsubscribe;
    },
    getQueryState: () => query.state,
  };

  return observer;
};

export const useQuery = <T,>({
  queryKey,
  queryFn,
}: UseQueryOptions<T>): QueryState<T> => {
  const queryClient = useContext(QueryClientContext);

  if (!queryClient) {
    throw new Error("useQuery must be used within a QueryClientProvider");
  }

  const observer = useRef(
    createQueryObserver(queryClient, {
      queryKey,
      queryFn,
    })
  );

  const [, setCount] = useState(0);
  const rerender = () => setCount((count) => count + 1);

  useEffect(() => {
    return observer.current.subscribe(rerender);
  }, []);

  return observer.current.getQueryState();
};
```

## staleTime 구현

도입부에서 말한 마지막 내용 입니다. 이 내용은 그렇게 어렵지 않기 때문에 딥하게 다루지는 않겠습니다.

`useQuery.ts`에 있는 useQuery 훅과, observer 명세에 staleTime을 옵셔널 하게 받을 수 있도록 합시다. 즉 다음과 같은 형태가 되게 됩니다.

```tsx
import { useContext, useEffect, useRef, useState } from "react";
import { QueryClientContext } from "./QueryClientProvider";
import type { QueryClient } from "./QueryClient";
import type { QueryState, UseQueryOptions } from "./types";

interface QueryObserver<T> {
  notify: () => void;
  subscribe: (rerender: () => void) => () => void;
  getQueryState: () => QueryState<T>;
}

const createQueryObserver = <T,>(
  queryClient: QueryClient,
  { queryKey, queryFn, staleTime = 0 }: UseQueryOptions<T>
): QueryObserver<T> => {
  const query = queryClient.getQuery({
    queryKey,
    queryFn,
  });

  const observer: QueryObserver<T> = {
    notify: () => {},
    subscribe: (rerender) => {
      const unsubscribe = query.subscribe(observer);
      observer.notify = rerender;
      if (
        query.state.lastUpdated &&
        Date.now() - query.state.lastUpdated > staleTime
      )
        query.fetch();
      return unsubscribe;
    },
    getQueryState: () => query.state,
  };

  return observer;
};

export const useQuery = <T,>({
  queryKey,
  queryFn,
  staleTime = 0,
}: UseQueryOptions<T>): QueryState<T> => {
  const queryClient = useContext(QueryClientContext);

  if (!queryClient) {
    throw new Error("useQuery must be used within a QueryClientProvider");
  }

  const observer = useRef(
    createQueryObserver(queryClient, {
      queryKey,
      queryFn,
      staleTime,
    })
  );

  const [, setCount] = useState(0);
  const rerender = () => setCount((count) => count + 1);

  useEffect(() => {
    return observer.current.subscribe(rerender);
  }, []);

  return observer.current.getQueryState();
};
```

그리고, query의 상태값에, 마지막으로 갱신된 시점이 언제인지 저장해야 하므로, type에도 이를 반영해주고

```ts
export interface QueryState<T> {
  queryKey: QueryKey;
  queryHash: string;
  status: "pending" | "success" | "error";
  isFetching: boolean;
  data?: T;
  error?: Error;
  lastUpdated?: number; // 이 부분이 추가됨
}
```

query.ts의 fetch 메서드에도 이를 반영해 줍시다.

```ts
fetch: async (): Promise<void> => {
      if (query.state.isFetching) return;
      query.setState((prev) => ({
        ...prev,
        isFetching: true,
        error: undefined,
      }));

      try {
        const data = await queryFn();
        query.setState((prev) => ({
          ...prev,
          data,
          status: "success",
          isFetching: false,
          lastUpdated: Date.now(), // 이 부분이 추가됨!
        }));
        // ... 하략 ...
```

# 마치며

글의 분량을 조절하기 위해서 코어 컨셉트 위주로만 하였는데도, 상당히 많은 분량이 들었던 것 같습니다... 세상 모든 react 라이브러리를 다 만들어 보면서 이해할 수는 없겠지만, 몇몇 대표적인 라이브러리들을 직접 만들어 봄으로서, 단순히 남의 것 받아서 가져다 쓰기만 하는 개발자가 아닌, 도구를 이해하고, 어떤 도구인지 이해하는 개발자가 되기를 소망하면서 이번 글을 마무리 합니다...
