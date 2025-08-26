---
layout: post
title: 알쏭달쏭 nextJS 서버컴포넌트
subtitle: app router next.js에서 몰라서는 안되는 지식을 짚어봅시다.
category: react
image: /images/thumbnails/nextJS.png
---

# 들어가며

최근 제 글을 보셨다면 아시겠지만, 저는 커리어 관리를 위해 nextJS 경험을 쌓고 있습니다. 이전에도 여러 기업 과제에서 잠깐씩 nextJS를 다뤘지만, App router 시대의 핵심인 서버 컴포넌트(server component)는 꼭 짚고 넘어가야 할 ‘지식 부채’였습니다.

이번 글에서는 이 지식 부채를 조금이라도 줄이기 위해, ‘헷갈리기 쉬운 서버 컴포넌트와 제약사항’을 명확히 정리해보겠습니다.

# 서버 컴포넌트란?

## 레거시 pages router와 app router 간단 비교

레거시 nextJS(pages router)에서는 `getServerSideProps`나 `getStaticProps` 같은 함수를 써서 서버에서 데이터를 먼저 가져왔습니다.

NextJS 13.4부터 stable 기능으로 `App router`가 도입되었고, React 18부터 정식 도입된 서버 컴포넌트를 적극 활용합니다. 이를 통해 클라이언트 번들 사이즈를 줄이고, 로딩 속도도 개선할 수 있죠.

app router의 큰 차이점은 기본이 서버 컴포넌트로 동작하며, 필요할 때만 `'use client'` 지시어를 최상단에 써서 클라이언트 컴포넌트로 전환할 수 있다는 점입니다.

## 서버 컴포넌트란 무엇인가

서버 컴포넌트는 nextJS 고유 기능이라기보다 [React 자체 기능](https://ko.react.dev/reference/rsc/server-components)입니다. nextJS가 튼튼하게 지원해주지만, vite 같은 환경에서는 [별도의 플러그인 세팅](https://github.com/nicobrinkkemper/vite-plugin-react-server)이 필요해 쉽게 접하기 어려웠습니다.

서버 컴포넌트는 미리 렌더링된 정적 서버와, 요청마다 동적으로 실행되는 서버 두 가지 방식이 있습니다. 클라이언트에서 실행되는 CSR과는 다릅니다.

하지만 서버 컴포넌트는 브라우저에서 실행되지 않아 `window`, `document` 접근이나 런타임 React hook 사용이 불가능합니다. 이런 부분은 `'use client'`를 붙인 클라이언트 컴포넌트로 분리해야 합니다.

# 서버 컴포넌트와 클라이언트 컴포넌트 함께 사용하기

서버 컴포넌트는 번들 사이즈 축소와 FCP(First Contentful Paint) 개선에 유리하지만, 대부분의 웹 페이지는 사용자 입력과 브라우저 API 활용이 필요합니다.

따라서 두 컴포넌트가 함께 쓰이는 경우가 많고, 어떻게 공존시키는지 이해해야 합니다.

## 서버 컴포넌트와 클라이언트 컴포넌트를 섞기 어려운 이유

이 둘은 실행 환경이 달라 단순히 섞어 쓸 수 없습니다. 서버 컴포넌트는 서버 환경에서 실행되므로, 렌더링 경계가 명확해야 합니다.

아래 예제를 봅시다.

```tsx
/* app/[id]/page.tsx – 서버 컴포넌트 */
import LikeButton from "@/app/ui/like-button";
import { getPost } from "@/lib/data";

export default async function Page({ params }: { params: { id: string } }) {
  const post = await getPost(params.id); // 서버에서 데이터 패치
  return (
    <main>
      <h1>{post.title}</h1>
      {/* ...게시글 내용... */}
      <LikeButton likes={post.likes} /> {/* 클라이언트 컴포넌트 사용 */}
    </main>
  );
}

/* app/ui/like-button.tsx – 클라이언트 컴포넌트 */
("use client");
import { useState } from "react";

export default function LikeButton({ likes }: { likes: number }) {
  const [count, setCount] = useState(likes);
  return <button onClick={() => setCount(count + 1)}>좋아요 {count}개</button>;
}
```

간단한 블로그 예제입니다. `LikeButton`을 제외한 페이지는 서버에서 처리하지만, `LikeButton`은 클라이언트에서 실행됩니다. 그래서 `'use client'`가 클라이언트 컴포넌트 경계를 알려줍니다.

## 서버 → 클라이언트 전달 (가능)

서버 컴포넌트에서 클라이언트 컴포넌트를 import하고 자식으로 넣는 것은 자연스럽고 흔한 패턴입니다. app router 구조에서 최상단이 서버 컴포넌트이므로 클라이언트 컴포넌트를 쓸 수 없는 경우는 거의 없습니다.

## 클라이언트 → 서버 전달

React에 익숙하다면 단순한 일이겠지만, 이 부분이 헷갈릴 수 있어 정리합니다.

### 클라이언트에서 서버 import (불가능)

클라이언트 컴포넌트에서 서버 컴포넌트를 import하는 것은 불가능합니다. 서버와 클라이언트 컴포넌트는 환경이 달라 NextJS가 분리된 모듈 그래프로 관리하기 때문입니다. `'use client'`가 이 경계 역할을 하며, 클라이언트에서 서버 전용 API 호출이나 비동기 컴포넌트를 사용하면 빌드 타임에 에러가 납니다.

### 서버 → 클라이언트 prop 전달 (가능)

서버 컴포넌트에서 비동기 데이터(fetch 등)를 받아 클라이언트 컴포넌트에 prop으로 넘기는 것은 가능합니다.

```tsx
function ClientComponent({ name }: { name: string }) {
  return <div>Hello, {name}!</div>;
}

async function ServerComponent() {
  const user = await getUser();
  return <ClientComponent name={user.name} />;
}
```

prop은 직렬화 가능한 값이면 되고, React 컴포넌트도 가능합니다.

```tsx
async function BirthdayCake() {
  const userBirthdayIsToday = await checkIfBirthdayIsToday();
  return userBirthdayIsToday ? <span>🎂</span> : null;
}

async function ServerComponent() {
  const user = await getUser();
  return <ClientComponent name={user.name} birthdayCake={<BirthdayCake />} />;
}
```

클라이언트 컴포넌트 구현 예:

```tsx
"use client";

function ClientComponent({
  name,
  birthdayCake,
}: {
  name: string;
  birthdayCake: React.ReactNode;
}) {
  return (
    <div>
      Hello, {name}! {birthdayCake}
    </div>
  );
}
```

`birthdayCake`는 서버에서 렌더링되어 클라이언트로 전달되므로, 클라이언트 컴포넌트는 단순한 정적 HTML을 받는 셈입니다.

### 서버 → 클라이언트 children 전달 (가능)

children은 JSX 문법적 설탕으로, `children`이라는 prop에 전달하는 것입니다. 아래 예제처럼 사용할 수 있습니다.

```tsx
async function BirthdayCake() {
  const userBirthdayIsToday = await checkIfBirthdayIsToday();
  return userBirthdayIsToday ? <span>🎂</span> : null;
}

async function ServerComponent() {
  const user = await getUser();
  return <ClientComponent name={user.name} children={<BirthdayCake />} />;
}
```

관련해서 제가 이전에 [바텀시트 구현](https://kasterra.github.io/user-friendly-bottomsheet-implement/)한 것에서도 children 타입을 별개로 지정했던 섹션에서도 다루고 있으니, 관심 있으시다면 확인을 해보시면 좋을 것 같습니다.

# 번외: 서버 컴포넌트에서 클라이언트 전용 라이브러리 사용하기

서버 컴포넌트는 브라우저 환경이 아니므로 DOM 조작이나 이벤트 핸들링 같은 클라이언트 전용 라이브러리는 직접 사용할 수 없습니다. 예를 들어 `framer-motion`, `react-hook-form`, `zustand` 등이 있습니다.

이런 라이브러리는 `'use client'` 지시어를 최상단에 선언한 클라이언트 전용 래퍼 컴포넌트를 만들어서 감싸야 합니다.

예시:

```tsx
"use client";
import { motion } from "framer-motion";

export default function AnimatedBox() {
  return <motion.div animate={{ opacity: 1 }}>Hello</motion.div>;
}
```

서버 컴포넌트에서 이 `AnimatedBox`를 import해 사용하면, 서버에서 데이터는 패치하고 애니메이션은 클라이언트에서 안전하게 동작합니다. 즉, 클라이언트 전용 로직은 반드시 `'use client'`로 경계를 지정해 격리해야 합니다.

# 마무리하며

이번 글이 Next.js App Router의 서버 컴포넌트와 클라이언트 컴포넌트 기본기를 이해하고 활용하는 데 도움이 되길 바랍니다. 앞으로 nextJS를 다루실 때 이 지식이 큰 도움이 될 것입니다. 감사합니다.
