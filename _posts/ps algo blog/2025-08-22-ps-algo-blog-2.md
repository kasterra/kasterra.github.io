---
layout: post
title: 커리어 점프를 위한 PS 풀이 블로그 분관 개설 - 08.22
subtitle: codex와의 첫만남과 GIGO 원칙
category: short essay
image: /images/thumbnails/shortEssay.jpg
---

# 들어가며

두괄식으로 말하면, `codex`와의 첫 만남은 긍정적이었지만, GIGO(Garbage In Garbage Out)의 원리도 체감했습니다.

[지난 포스트](/ps-algo-blog-1)에서 공유한 `v0` 프로토타입에 codex와 함께 기능을 하나씩 추가하며 8월 22일 하루 동안의 기록을 정리합니다.

# 목표

v0 단계에서 마크업과 퍼블리싱은 거의 완료되었으니, 다음은 기능을 차근차근 추가하는 일입니다.

[지난 포스트](/ps-algo-blog-1)에서 언급한 블로그 목표 중 일부를 구현하려 했습니다. 블로그의 핵심은 **mdx 콘텐츠를 리스트에 보여주고 의도대로 렌더링하는 것**입니다. `velite`를 사용해 mdx 기반 콘텐츠를 nextJS 블로그에서 라우팅하고, mdx 렌더링 과정에 `codehike`를 적용해 코드블록을 멋지게 표현하는 것이 8월 22일의 목표였습니다.

# 첫 시작 - codex는 실수하는 AI

처음에는 v0가 생성한 프로젝트에 codex에게 `/init`을 시키고, 필요한 라이브러리 설치 후 Getting started 설정만 해두고 맡겼습니다.

> 현재 프로젝트 설정을 기반으로 Placeholder가 아닌 mdx 콘텐츠 기반으로 렌더링 되게 프로젝트를 바꿔줘

명확한 가이드 없이 모호한 프롬프트를 주었고, 상황에 맞지 않는 셋업으로 여러 문제에 부딪혔습니다. 예를 들어, `content/posts/hello.mdx`라는 샘플 글을 만들었는데, 페이지 빌드를 위해 `hello`라는 필드와 페이지를 하드코딩한 객체를 만들고, 이를 지적하자 하드코딩 파일을 생성하는 스크립트를 만들고 package.json에 등록하는 등 복잡해졌습니다. 원하는 대로 렌더링되지 않는 문제도 많았습니다.

이때 GIGO 원리를 다시 떠올리고, 최소한 일을 잘 시키려면 라이브러리를 직접 공부해야 한다고 판단했습니다.

# AI가 일을 잘하게 하려면

AI든 사람이든, 일을 잘하려면 명확한 지시가 필요합니다. codex의 작업 효율성을 높이기 위해 라이브러리를 공부했습니다.

## velite 라이브러리 학습

처음 velite를 단순히 mdx를 페이지로 연결하는 도구로 오해했습니다. 그러나 velite는 md(x) 콘텐츠의 스키마를 점검하고 mdx 컴파일 과정을 처리하는 '중간 처리기' 역할로, nextJS와 직접 관련이 없습니다.

framework agnostic한 라이브러리라 Getting started 설명이 nextJS와 맞지 않는 부분이 있었고, 별도의 nextJS용 셋업이 필요함을 알게 되어 제대로 구성할 수 있었습니다.

이후 아래 프롬프트로 다시 AI에게 작업을 맡겼습니다.

> velite로 mdx 콘텐츠 nextJS SSG를 하면서, codehike를 사용해 mdx를 조물조물 할 수 있는 예제를 만들어줘

chatGPT5로 간단한 설정 파일을 만들었고, 결과는 다음과 같습니다.

```ts
import { defineConfig, s } from "velite";
import { remarkCodeHike, recmaCodeHike } from "codehike/mdx";

// Code Hike 설정: 컴파일 시 하이라이팅(테마 자유 변경 가능)
const chConfig = {
  components: { code: "CHCode" }, // MDX 코드블록을 CHCode 컴포넌트로 렌더링
  syntaxHighlighting: { theme: "github-dark" },
} satisfies import("codehike/mdx").CodeHikeConfig;

export default defineConfig({
  mdx: {
    remarkPlugins: [[remarkCodeHike, chConfig]],
    // Code Hike는 recma 플러그인도 필요함
    // @ts-expect-error: 타입 경고 무시 가능
    recmaPlugins: [[recmaCodeHike, chConfig]],
  },

  collections: {
    posts: {
      name: "Post",
      pattern: "posts/**/*.mdx",
      schema: s.object({
        title: s.string(),
        date: s.date(),
        slug: s.string(),
        code: s.mdx(),
      }),
    },
  },
  output: {
    data: ".velite",
  },
});
```

date 필드를 string으로 수정한 것 외에는 큰 변경 없이 사용할 수 있었습니다.

## mdx, codehike 라이브러리 학습

md는 마크다운으로, 기술 문서화에 특화된 마크업 언어입니다. mdx는 마크다운에 React 컴포넌트를 직접 삽입할 수 있게 확장한 형태로, React 환경에서 표준처럼 사용됩니다. 이와 관련된 라이브러리 중 하나가 codehike입니다.

jekyll처럼 단순히 마크다운 문서와 간단한 설정으로 자동 렌더링되지는 않습니다. 특히 '커스텀 컴포넌트'는 직접 import 등 별도 설정이 필요하며, codehike의 `CHCode`가 대표적인 외부 컴포넌트입니다.

이런 사전 학습을 마치고 추가 설정을 올바르게 적용해 생성형 AI 결과물을 제대로 활용할 수 있었습니다. 아래는 mdx, codehike 관련 설정 파일입니다.

**components/mdx-content.tsx**

```tsx
import * as runtime from "react/jsx-runtime";
import type { ComponentType } from "react";

// Velite의 function-body 코드를 컴포넌트로 변환
const useMDXComponent = (code: string) => {
  const fn = new Function(code);
  return fn({ ...runtime }).default as ComponentType<any>;
};

type MDXProps = {
  code: string;
  components?: Record<string, ComponentType<any>>;
};

export function MDXContent({ code, components = {} }: MDXProps) {
  const Mdx = useMDXComponent(code);
  return <Mdx components={components} />;
}
```

**components/CHCode.tsx**

```tsx
// Code Hike가 전달하는 하이라이트된 코드를 렌더링하는 컴포넌트
import { Pre } from "codehike/code";
import type { HighlightedCode } from "codehike/code";

export default function CHCode({ codeblock }: { codeblock: HighlightedCode }) {
  return <Pre code={codeblock} style={codeblock.style} />;
}
```

# codex의 활약 - 인간보다 빠른 퍼블리싱

파일럿이 기술을 습득하고 올바른 셋업을 제공한 후, 같은 프롬프트로 codex에게 작업을 의뢰했습니다.

> 현재 프로젝트 설정을 기반으로 Placeholder가 아닌 mdx 콘텐츠 기반으로 렌더링 되게 프로젝트를 바꿔줘

프롬프트는 같지만 상황이 달라 결과물이 깔끔하고 기능이 구현되기 시작했습니다. 에러 로그가 나오면 코드를 복사해 붙여넣고, 의도에 맞지 않으면 거부하는 등 관리자로서 역할만 하면 3~4분 내에 기능을 완성했습니다. mdx 스타일링을 위해 v0 플레이스홀더를 가져왔을 때는 단순 번역에 그쳤으나, 프롬프트를 정제하며 다시 한번 GIGO를 염두에 두고 작업을 지시했습니다. 기능은 건드리지 말라는 단서도 달고, 거부 인터랙션을 통해 의도를 읽는 codex가 인상적이었습니다.

가장 돋보였던 점은 퍼블리싱 측면에서 codex의 역량입니다. 라이트모드/다크모드에서 코드블록 가독성 문제, 패딩 부족, 보더 스타일 등 추상적인 요구에도 잘 부합해 퍼블리싱 시간을 약 70% 줄였습니다. 일반적으로 이런 작업은 여러 시행착오를 거치는데, codex 덕분에 효율적이었습니다.

codex는 워크스페이스 파일을 열심히 확인하며 작업하지만, 생성형 AI 특유의 환각으로 API에 맞지 않는 코드를 작성하는 경우도 있습니다. 이런 부분은 사전 학습으로 보완하면 AI와 '함께하는' 페어 프로그래밍이 가능합니다.

# 맺으며

에이전트형 AI를 직접 사용해본 첫 프로젝트였지만, 강력한 도구임을 확인했습니다. 파일럿인 인간이 잘 알고 있고, 작업 내용이 명확하다면 인간의 공수를 크게 줄여줍니다.

개발자 혼자 하루 만에 프로토타입 마크업을 완성하고, 또 다른 하루 만에 MVP 기능을 구현하는 시대가 왔습니다. 앞으로도 꾸준히 발전시켜 블로그 분관을 빠르게 완성하고, 알찬 PS글로 채워갈 계획입니다.

아직 남은 마일스톤(댓글 추가, 검색 구현, 테스트 코드 작성)도 있으니, 앞으로의 개발일지에도 AI와 함께하는 페어 프로그래밍의 흔적을 남기겠습니다.
