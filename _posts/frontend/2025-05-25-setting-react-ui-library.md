---
layout: post
title: React UI 라이브러리 만들고 배포하기
subtitle: 공통 디자인 시스템을 코드 레벨로 옮기기
category: frontend
image: /images/thumbnails/React.png
---

# 들어가며

서울에서의 첫 직장 글을 이후로 회사에서 열심히 회의도 하고 프로덕트를 만들기 위해서 여러 노력들을 하면서 블로그에 의미가 있는 글을 올리지 못한 것 같습니다. 스타트업에서 주니어 프론트엔드 개발자로 일하며 직접 기술적 의사결정에 참여했고, 그 과정에서 얻은 인사이트를 정리해 공유해보려 합니다.

회사에서 디자인 시스템을 코드화하는 과제를 맡게 되면서, 직접 UI 라이브러리를 구축해보게 되었습니다. 기존에는 단일 프로젝트 내에서만 쓰는 수준의 컴포넌트 정리만 경험해봤지만, 이번에는 조직의 아이덴티티를 유지할 수 있는 공통 라이브러리를 고민하며 만들게 되었죠. Vite, Typescript 등 익숙한 도구들을 활용해가며 설정을 다듬고, 실제 npm 패키지로 배포하는 과정까지 진행했습니다. 이 글에서는 그 과정을 정리해보려고 합니다.

# 도구들을 골라보기

라이브러리를 만들기 위해서 아무것도 없는 허허벌판에서 시작할 수는 없지요. 어떤 개발용 도구들을 사용해서 작업을 해볼까 하는 생각들을 하였습니다.

## vite (rollup) - 빠른 번들링과 쉬운 설정

React로 웹 앱을 작성하기 위해서는 기반 설정들이 몇 가지 필요합니다. 직접 구성 할 수도 있지만, 많이 사용해서 익숙한 도구를 굳이 마다할 필요 없지요.

JSX로 작성된 뷰나 SVG 같은 정적 애셋까지 포함하려면, 순수 Vanilla JS로 loader를 직접 구현하며 작업하는 것은 비효율적입니다.
Create react APP이 deprecated된 현재, 익숙한 vite를 사용하는 것은 좋은 선택이라 할 수 있겠습니다.

<https://victorlillo.dev/blog/react-typescript-vite-component-library> 를 참고하여서 vite 설정들을 했었습니다. 많은 부분들을 참고하였지만, 단순히 이것만 따라했다면, 제 시간을 들여서까지 블로그 글을 열심히 적진 않았겠지요.

## vanilla extract - 디자인 시스템에 최적화된 스타일 관리

이전에 블로그에서 다루었던 KOJ 프론트 개발때는 css module을 사용해서 작업을 하였습니다. 당시 코멘트를 해줬던 [Dogdriip 선생님](https://github.com/Dogdriip)께서 zero runtime CSS-in-JS 라이브러리에 대한 긍정적 이야기도 해주셔서, 피상적으로 아는 데 그치기보다, 실제 프로젝트에 적용해보며 이해도를 높이는 것이 더 좋겠다고 판단했습니다. 또한 실제로 이러한 라이브러리들이 디자인시스템이 정의되어 있는 유즈케이스에서 사용하기 편하다는 평 또한 있었기에, 채택하게 되었습니다.

![zero runtime css-in-js quote](/images/frontend/dogdriip%20css%20module%20comment.png)

그리고 style variant 등을 미리 정의해서 `clsx`등의 css class util을 사용해서, 디자인 문서에 정의된 대로 구현할 수 있는 좋은 도구였던 것 같습니다.

```ts
import { style, styleVariants } from "@vanilla-extract/css";
import { colors } from "tokens/colors";
import { typography } from "tokens/typography.css";

export const base = style({
  display: "flex",
  gap: 4,
  justifyContent: "center",
  alignItems: "center",
  border: "none",
  borderRadius: 10,
  cursor: "pointer",
  ":disabled": {
    backgroundColor: colors.grey[100],
    color: colors.grey[500],
  },
});

export const variants = styleVariants({
  primary: {
    backgroundColor: colors.orange[500],
    color: colors.white,
    ":active": {
      backgroundColor: colors.orange[600],
    },
  },
  secondary: {
    backgroundColor: colors.orange[100],
    color: colors.orange[550],
    ":active": {
      backgroundColor: colors.orange[200],
    },
  },
  tertiary: {
    backgroundColor: colors.grey[200],
    color: colors.grey[800],
    ":active": {
      backgroundColor: colors.grey[400],
    },
  },
});

export const sizes = styleVariants({
  large: {
    padding: "14px 24px",
    ...typography.body1Semibold,
  },
  medium: {
    padding: "10px 19px",
    ...typography.body2Semibold,
  },
  small: {
    padding: "8px 13px",
    ...typography.caption1Semibold,
  },
});

export const expand = style({
  width: "100%",
});
```

이런식으로 스타일들을 정의해두고

```tsx
import type { ButtonHTMLAttributes } from "react";
import { clsx } from "clsx";
import { base, variants, sizes, expand } from "./style.css";

type ButtonVariant = keyof typeof variants;
type ButtonSize = keyof typeof sizes;

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: ButtonVariant;
  size?: ButtonSize;
  expandToContainer?: boolean;
  maxWidth?: string;
}

export const Button = ({
  variant = "primary",
  size = "medium",
  className,
  children,
  expandToContainer,
  maxWidth,
  ...props
}: ButtonProps) => {
  return (
    <button
      className={clsx(
        base,
        variants[variant],
        sizes[size],
        expandToContainer && expand,
        className
      )}
      style={{ ...props.style, maxWidth }}
      {...props}
    >
      {children}
    </button>
  );
};
```

prop들에 따라서 미리 정해진 스타일을 꺼내서 적용시키는 형태로 사용할 수 있었습니다.

## svgo - SVG를 최적화하여 번들 사이즈 줄이기

프론트엔드에서 정말 친숙한 벡터 이미지 타입인 svg는 사실 약간의 최적화를 거치면, 번들 사이즈를 줄여주는 꽤나 간단한 최적화를 진행 할 수 있습니다. [잘 설명된 velog글](https://velog.io/@sumi-0011/svgo-sprite)을 첨부합니다.

```ts
svgr({
  svgrOptions: {
    plugins: ["@svgr/plugin-svgo", "@svgr/plugin-jsx"],
    svgoConfig: {
      plugins: [
        {
          name: "preset-default",
          params: {
            overrides: {
              removeViewBox: false,
              removeUselessStrokeAndFill: false,
              cleanupIds: false,
              convertPathData: { floatPrecision: 2 },
            },
          },
        },
      ],
    },
  },
});
```

## private npm publish - 조직 내부용 라이브러리 배포

공개적으로 내놓고 싶지는 않지만, 회사 내에서 써야 하는 물건들이 있습니다. 핵심 비즈니스로직이 들어있다면 더더욱 그렇겠지요. 직접 npm 레지스트리를 회사 서버에 구성할 수 있겠지만, 조직의 규모와, 유지보수 복잡성들을 고려해서 npm private registry를 유료 결제해서 사용하기로 하였습니다. `npm login`을 통해서 유료결제한 계정으로 연결하여서 `npm publish` 하여주면 익히 알려진 npm 배포가 가능합니다.

publish 이후에 해당 라이브러리를 사용하기 위해서는 `.npmrc`라는 파일을 만들어서 아래의 내용을 작성한 후, 'yourPrivateOrg'자리에 배포한 조직 이름을 입력하고, authToken에는 npm 설정에서 발급한 토큰을 기입하면, public 라이브러리를 다운로드 받듯이 private 라이브러리를 사용할 수 있습니다.

```text
@yourPrivateOrg:registry=https://registry.npmjs.org/
//registry.npmjs.org/:_authToken=npm_placeyourawesometokenhere
```

# 마치며

현업 주니어 개발자로서, 회사 프로덕트에 멋진 기여를 할 수 있다는 것은 참으로 설레는 일이면서, 새로운 것을 공부할 수 있는 좋은 기회인 것 같습니다. 앞으로도 여러 지식들을 공유할 수 있었으면 좋겠습니다.

오랜 시간 키워온 기술 블로그를, 취업했다고 해서 멈출 수는 없겠지요. 끝까지 읽어주셔서 감사합니다.
