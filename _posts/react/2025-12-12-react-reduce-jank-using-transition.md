---
layout: post
title: React의 Transition(동시성 렌더링)으로 대시보드 입력 지연 줄이기
subtitle: 면접 질문에서 출발한 naive vs transition/useDeferredValue 비교 실험
category: react
image: /images/thumbnails/React.png
---

# 들어가며

최근 글들을 돌아보면 기술 인터뷰/커리어 회고가 꽤 많았습니다. 정리해둔 경험을 잘 “재사용”하는 것도 중요하지만, 면접에서 반복적으로 등장하는 질문을 계기로 제가 아직 손에 익지 않은 영역을 보강하는 글도 필요하겠다고 느꼈습니다.

이번 글은 그 연장선입니다. 과거 면접에서 “React의 동시성 렌더링(Transition, useDeferredValue)을 대시보드 성능 개선에 어떻게 적용하느냐” 같은 질문을 받았는데, 당시에는 개념 설명은 가능해도 **직접 실험해보고 자신 있게 말할 만큼의 근거**가 부족했습니다. (참고 : [이전 면접 복기 자료](several-interview-review-3#1차%20온라인%20면접%20복기))

그래서 가상의 데이터를 활용한 미니 대시보드를 만들어, **naive한 구현(즉시 재계산/즉시 렌더)** 과 **Transition/지연 값으로 우선순위를 분리한 구현**을 비교해 보려 합니다.

## TL;DR

동일한 대시보드 로직에서

- naive 구현은 입력이 연산에 막혀 타이핑이 끊기고
- Transition / useDeferredValue를 적용하면 입력 반응성은 유지된 채 결과만 지연됩니다.

이 글은 그 차이를 **실제 코드와 실험 영상**으로 확인합니다.

# 실험 소개

이 글에서는 React 내부의 lane 스케줄링을 직접 다루는 것이 아니라, Transition / Deferred API를 통해 **업데이트 우선순위를 분리한 구현**을 “우선순위 분리 구현(Transition/Deferred)”이라고 부릅니다.

어떤 도구가 체감 성능에 얼마나 영향을 주는지는 결국 실험이 가장 확실합니다. 이번 글에서는 faker.js로 가상의 트래픽/매출 데이터를 생성하고, 이를 바탕으로 작은 대시보드 프로젝트를 만들어 비교 실험을 진행했습니다.

사용한 프로젝트 코드는 [GitHub](https://github.com/kasterra/naive-vs-nonlane-dashboard)에 올려두었고, 결과물은 빠르게 확인할 수 있도록 [Netlify](https://naive-vs-nonlane-dashboard.netlify.app)에도 배포해 두었습니다.

## 미니 프로젝트 소개와 배경

프론트엔드 개발자의 수요처는 다양하지만, 제 커리어와 관심사(그리고 실제 면접 경험)와 가장 자주 맞닿는 문제는 “대시보드”였습니다. 대시보드는 사용자 입력(검색어/필터/정렬)에 따라 데이터를 다시 가공하고, 차트·테이블을 재렌더링하는 일이 잦습니다.

이때 데이터 규모가 커지면(혹은 연산이 무거워지면) 입력처럼 **즉각 반영되어야 하는 업데이트**가 다른 계산에 밀려 늦게 반영되는 상황이 생깁니다. 흔히 말하는 _jank_ 또는 “타이핑이 버벅이는” 경험이죠.

그래서 이번 실험의 목표는 단순합니다. **‘당장 반응해야 하는 업데이트’와 ‘조금 늦어도 되는 업데이트’를 분리**해서, 사용자가 느끼는 입력 지연을 줄여보는 것입니다.

## 기술적 배경

React 내부에는 업데이트 우선순위를 관리하기 위한 _lane_ 기반 스케줄링 개념이 존재합니다. 다만 애플리케이션 개발자가 이 “lane”을 직접 조작하는 API를 쓰는 것은 아니고, 대신 **Transition / Deferred** 같은 고수준 API로 “이 업데이트는 급하지 않다”라는 힌트를 전달합니다.

이 글에서 사용하는 핵심 도구는 다음 두 가지입니다.

- `useDeferredValue`: 값 자체는 즉시 갱신하되, **무거운 파생 계산(필터링/차트 데이터 생성)** 에는 지연된 값을 사용해 타이핑/클릭 같은 입력 반응성을 지키는 데 도움을 줍니다.
- `useTransition`: 무거운 상태 업데이트를 **낮은 우선순위 전환(transition)** 으로 감싸, 입력 반영이 먼저 일어나고(urgent) 결과 렌더링이 뒤늦게 따라오도록(non-blocking) 만들 수 있습니다.

정리하면, “렌더링이 빨라진다”기보다는 **업데이트의 우선순위를 조절해 체감 성능을 개선하는 접근**입니다. 이번 미니 프로젝트에서는 두 가지를 함께 사용해, “입력은 즉시 반영 + 무거운 계산은 뒤로 밀기”를 달성하였고, 이 차이가 대시보드 UX에서 어떻게 드러나는지 확인해 보겠습니다.

## 프로젝트 개요

- 프레임워크: React 19 + Vite + TypeScript
- 라우팅: React Router 7 (`/`, `/naive`, `/deferred-transition`)
- 스타일/차트: Tailwind CSS v4, Recharts
- 더미 데이터: `@faker-js/faker`로 시드 기반 클라이언트 생성
- 비교 축: 동기 계산(Naive) vs 우선순위 분리 구현(Transition/Deferred) + 코드 스플리팅(Suspense)

# 구현

## 데이터 모델과 생성

- 타입은 `src/api/types.ts`, 데이터 생성은 `src/api/data.ts`에서 담당합니다.
- 전역 싱글턴 데이터셋을 유지하고, UI에서 데이터 볼륨을 바꿔 실험할 수 있도록 `regenerateDataset`를 제공합니다.

코드 스니펫

```ts
// src/api/types.ts
export type Channel = "Direct" | "Search" | "Ads" | "Social" | "Referral";
export type Segment = "All" | "New" | "Returning" | "VIP";

export interface LogRow {
  id: string;
  timestamp: number;
  date: string; // YYYY-MM-DD
  orderId: string;
  userId: string;
  revenue: number;
  channel: Channel;
  segment: Exclude<Segment, "All">;
  queryText: string;
}

export interface Filters {
  startDate: string;
  endDate: string;
  query: string;
  segment: Segment;
}
// src/api/data.ts (요약)
faker.seed(20251209);

export function generateDataset(count = 15000): LogRow[] {
  // 최근 180일 범위에서 무작위 로그 생성
  // ... channel/segment/revenue/product/user 조합
  // 날짜 문자열 "YYYY-MM-DD" 추가
  // 시간순 정렬
}

let DATASET: LogRow[] | null = null;

export function getDataset(): LogRow[] {
  if (!DATASET) DATASET = generateDataset(20000);
  return DATASET;
}

export function regenerateDataset(count?: number): LogRow[] {
  DATASET = generateDataset(count ?? (DATASET ? DATASET.length : 20000));
  return DATASET;
}
```

## 공통 집계 로직

- 두 화면 모두 같은 집계 함수(computeDashboard)를 사용합니다.
- 차이는 “언제/어떻게 실행하느냐”입니다.

```ts
// src/lib/compute.ts (요약)
export interface Derived {
  filtered: LogRow[];
  dailyRevenue: { day: string; value: number }[];
  dailyVisitors: { day: string; value: number }[];
  channelShares: { channel: Channel; value: number }[];
}

export function computeDashboard(data: LogRow[], filters: Filters): Derived {
  // 기간/세그먼트/검색어로 필터
  // 일별 매출/방문 집계, 채널별 매출 합산
  // 정렬 후 배열로 변환
  return { filtered, dailyRevenue, dailyVisitors, channelShares };
}
```

## Naive 구현: 입력마다 동기 계산

- 입력(검색/기간/세그먼트)이 바뀔 때마다 `useMemo`에서 무거운 `computeDashboard`를 **즉시** 실행합니다.
- 데이터가 커질수록 타이핑/클릭 같은 입력 반응성이 떨어질 수 있습니다.

```tsx
// src/screens/Naive.tsx (발췌)
const [data, setData] = useState(() => getDataset());
const [filters, setFilters] = useState<Filters>({
  /* 초기값 */
});

// 무거운 계산을 동기적으로 즉시 실행
const { filtered, dailyRevenue, dailyVisitors, channelShares } = useMemo(
  () => computeDashboard(data, filters),
  [data, filters]
);

// UI 조작: 기간 빠른 버튼, 데이터 볼륨 변경, 페이지네이션 등...
```

체감 포인트

- 검색 인풋에 빠르게 타이핑하면 키 입력이 밀리거나 끊깁니다.
- 데이터 볼륨을 5만으로 올리면 차이가 선명해집니다.

## 우선순위 분리 구현: Transition/Deferred + 코드 스플리팅

핵심 전략은 다음과 같습니다.

- `useDeferredValue`로 “입력값은 즉시 반영하되, 무거운 계산에는 지연된 값”을 사용합니다.
- `useTransition`으로 무거운 파생 계산을 **낮은 우선순위 전환**으로 감싸 입력 반응을 우선합니다.
- `lazy` + `Suspense`로 차트/테이블을 코드 스플리팅하고 로딩 피드백을 제공합니다.

> **주의:** `startTransition`은 무거운 계산을 **비동기 작업으로 바꾸지 않습니다.**
> 이 예제에서 `computeDashboard`는 여전히 **메인 스레드에서 동기 실행**되며, 특히 아래처럼 `startTransition(() => { ... })` 콜백 안에서 계산을 수행하면 그 계산 자체는 그대로 **긴 동기 작업(long task)** 이 될 수 있습니다.
>
> 그럼에도 체감이 좋아지는 이유는 주로 다음 때문입니다.
>
> 1. `useDeferredValue`로 **계산이 트리거되는 빈도**를 줄여(타이핑 중에는 지연된 query가 즉시 따라오지 않음) 입력이 계산에 덜 자주 막히게 만들고,
> 2. `startTransition`으로 **결과 상태 업데이트/커밋을 낮은 우선순위**로 처리해, 입력(urgent)과 결과 갱신(non-urgent)을 분리해 UX가 무너지지 않게 하기 때문입니다.
>
> 계산 자체를 메인 스레드 밖으로 보내거나(진짜 CPU 병목 해결), 더 쪼개서 처리하려면 Web Worker/청크 처리 같은 **별도 접근**이 필요합니다.

```tsx
// src/screens/NonLane.tsx (핵심 발췌)
const [data, setData] = useState(() => getDataset());
const [filters, setFilters] = useState<Filters>({
  /* 초기값 */
});

// 1) 급한 업데이트(입력)와 덜 급한 업데이트(무거운 계산)를 분리
const deferredQuery = useDeferredValue(filters.query);
const effectiveFilters = useMemo(
  () => ({
    startDate: filters.startDate,
    endDate: filters.endDate,
    segment: filters.segment,
    query: deferredQuery,
  }),
  [filters.startDate, filters.endDate, filters.segment, deferredQuery]
);

// 2) 전환으로 래핑하여 입력 반응성 보장
const [derived, setDerived] = useState(() =>
  computeDashboard(data, effectiveFilters)
);
const [isPending, startTransition] = useTransition();

useEffect(() => {
  startTransition(() => {
    const next = computeDashboard(data, effectiveFilters);
    setDerived(next);
  });
}, [data, effectiveFilters]);

// 3) 코드 스플리팅 + 로딩 스켈레톤
const Charts = lazy(() => import("./parts/Charts"));
const TablePane = lazy(() => import("./parts/Table"));

return (
  <>
    <Suspense fallback={<div className="... animate-pulse" />}>
      <Charts
        dailyRevenue={derived.dailyRevenue}
        dailyVisitors={derived.dailyVisitors}
        channelShares={derived.channelShares}
        totalRevenue={totalRevenue}
      />
    </Suspense>

    <Suspense fallback={<div className="... animate-pulse" />}>
      <TablePane /* 페이지네이션 props */ />
    </Suspense>

    {/* 4) 미세한 UX: 검색 인풋 아래 상태 표시 */}
    <div className="mt-1 h-4 text-xs text-muted-foreground">
      {isPending ? "Updating results…" : "\u00A0"}
    </div>
  </>
);
```

체감 포인트

- 타이핑은 즉각적(urgent), 결과는 살짝 늦어도 자연스러운 전환과 로딩 피드백으로
  불편이 적습니다.
- 데이터 볼륨이 커질수록 우선순위 분리 구현의 이점이 커집니다.

## 라우팅과 인덱스

- 인덱스 페이지(/)에서 두 실험 화면(naive / 우선순위 분리 구현)으로 이동합니다.
- 버튼만 있는 심플한 진입점입니다.

```tsx
// src/App.tsx (요약)
<BrowserRouter>
  <Routes>
    <Route path="/" element={<IndexScreen />} />
    <Route path="/naive" element={<NaiveScreen />} />
    <Route path="/deferred-transition" element={<DefferedTransitionScreen />} />
  </Routes>
</BrowserRouter>
```

# 실험 방법과 결과 요약

- 데이터 볼륨을 20,000 → 50,000으로 올린 뒤 비교하면 차이가 더 뚜렷합니다.
- 검색 인풋에 빠르게 타이핑해 보세요.
  - Naive: 입력 지연·끊김이 체감됩니다.
  - 우선순위 분리 구현: 입력은 부드럽고, 결과가 지연되어도 “Updating results…” 안내와 Suspense 로딩으로 UX가 안정적입니다.
- 기간/세그먼트 필터도 함께 비교해 보세요.
  - 우선순위 분리 구현은 상호작용 중 UI 전체가 “멈춘 느낌”이 줄어듭니다.

## 실제 실험 자료

데이터 볼륨을 최대로 하여 chrome에서 실험한 결과를 참고용으로 첨부합니다.

크롬 개발자 도구에서 cpu 4배 스로틀링을 적용하고, 최대한 많은 데이터를 사용하게 하고(그래프 데이터 볼륨 5만, 테이블 데이터 1천), 동일한 타이핑 속도로 handmade라는 단어를 입력한 결과물 입니다.

우선 naive 버전입니다

<iframe width="560" height="315" src="https://www.youtube.com/embed/m_91SKBISiU?si=iRg942I5ECKN13xD" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

키보드 입력이 렌더링에 막혀서 즉각적으로 나오지 않음을 알 수 있습니다.

같은 조건에서 우선순위 분리 구현 버전입니다

<iframe width="560" height="315" src="https://www.youtube.com/embed/6U-tXti2m2M?si=f8wflIE_bpGLJB6t" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

우선순위 분리 구현 버전에서는 데이터 계산 중일 때, 처리중이라는 별도의 표시와 함께, 키보드 입력을 덜 방해함을 볼 수 있습니다.

# 마치며

이 예제의 핵심은 “렌더링이 마법처럼 빨라진다”가 아니라, **업데이트의 우선순위를 조절해 입력 반응성을 지키는 것**입니다.

같은 집계 로직을 사용하더라도,

- Naive 구현은 입력 → 계산 → 렌더가 한 덩어리로 묶여 입력이 밀리기 쉽고
- Transition / useDeferredValue를 적용한 우선순위 분리 구현은 입력(urgent)과 결과 갱신(non-urgent)을 분리해 체감 지연을 크게 줄일 수 있습니다.

정리하면, 대시보드·검색·필터처럼 **입력이 잦고 파생 계산이 무거운 화면**에서 Transition / Deferred는 “성능 최적화 기법”이라기보다 **UX를 안정화하기 위한 도구**에 가깝습니다.

면접에서 이 주제가 나왔을 때는 이렇게 설명할 수 있습니다.

- “전체 계산을 줄이기보다는, 입력이 끊기지 않도록 우선순위를 분리했습니다.”
- “결과는 늦어질 수 있지만, 로딩 피드백으로 사용자가 화면이 멈췄다고 느끼지 않게 했습니다.”

## 언제 이 접근이 유효하지 않은가

Transition / Deferred가 만능 해법은 아닙니다.

- 계산 자체가 가볍거나
- 데이터 규모가 작거나
- 입력 빈도가 낮은 화면이라면

우선순위 분리로 얻는 이점은 거의 없습니다. 오히려 코드 복잡도만 증가할 수 있습니다.

반대로 다음 조건에 가까울수록 효과는 분명해집니다.

- 사용자가 빠르게 연속 입력을 한다
- 입력마다 무거운 파생 계산이 발생한다
- 결과 영역이 크고 렌더링 비용이 높다(테이블, 차트 등)

이번 실험은 그 차이를 작은 예제로 직접 체감해보는 데 목적이 있었고, 같은 패턴을 실무에서도 **필요한 곳에만 선별적으로** 적용할 수 있을 준비가 되지 않을까 합니다.
