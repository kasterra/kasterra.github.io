---
layout: post
title: 본격 유저 친화적 React 바텀시트 만들어보기
subtitle: 웹앱 이라는 말에 어울리는 애니메이션, 제스처를 장착한 바텀시트를 React로 구현 해 봅시다
category: react
image: /images/thumbnails/React.png
---

# 들어가며

모바일 UX가 확산되면서, 웹앱에서도 바텀시트 UI가 꽤 자연스러운 요소가 되었네요. 단순한 클릭을 넘어서 스와이프 같은 제스처까지, 이제 모바일 웹에서는 당연한 흐름이 된 느낌입니다.

이번 글에서는 React 환경에서 제스처 기반 바텀시트를 직접 만들어보려고 합니다. 단순히 보였다 사라지는 수준을 넘어서, 사용자 경험을 고려한 자연스러운 애니메이션과, 비동기 흐름 처리, 그리고 재사용성과 구조적인 유연성까지 챙겨보려 합니다.

핵심 키워드는 아래와 같습니다

- 드래그 제스처 인식 (`@use-gesture/react`)
- 자연스러운 애니메이션 (`@react-spring/web`)
- 비동기 종료 흐름 (`Promise`)
- 함수형 children 패턴
- 안정적 DOM 구조 (`createPortal`)

# 전체 구현

사용자가 스와이프하면 내려가는 바텀시트를 구현했고, 외부 터치 시에도 닫히도록 했습니다.
자식 컴포넌트는 `handleClose` 함수를 통해 닫을 수 있으며, 비동기 애니메이션 종료까지 기다릴 수도 있습니다.

이번 포스트에서 살펴볼 구현체의 전체 코드를 공개하겠습니다.

````tsx
import { useSpring, animated } from "@react-spring/web";
import { useDrag } from "@use-gesture/react";
import { useRef } from "react";
import { createPortal } from "react-dom";
import {
  appBarContainer,
  appBarHandle,
  appBarHandleContainer,
  background,
} from "./style.css";

interface BottomSheetProps {
  children: (actions: {
    handleClose: (callback?: () => void) => void;
    handleCloseAsync: () => Promise<void>;
  }) => React.ReactNode;
  isOpen: boolean;
  onClose: () => void;
}

/**
 * 드래그 가능한 하단 바텀시트 컴포넌트입니다.
 *
 * `isOpen`이 true일 때만 표시되며, 드래그 또는 외부 클릭으로 닫을 수 있습니다.
 * 닫힐 때 애니메이션을 수행하고, 완료 시 `onClose` 콜백을 호출합니다.
 *
 * 자식 컴포넌트는 `({ handleClose }) => ReactNode` 형태의 함수형 children을 통해 전달되며,
 * `handleClose()` 호출 시 바텀시트가 닫히는 애니메이션을 실행합니다.
 *
 * @example
 * ```tsx
 * <BottomSheet isOpen={isOpen} onClose={() => setIsOpen(false)}>
 *   {({ handleClose }) => (
 *     <MyForm onSubmit={() => {
 *       doSomething();
 *       handleClose();
 *     }} />
 *   )}
 * </BottomSheet>
 * ```
 */
export const BottomSheet = ({
  isOpen,
  children,
  onClose,
}: BottomSheetProps) => {
  const sheetRef = useRef<HTMLDivElement>(null);
  const [{ y }, api] = useSpring(() => ({ y: 0 }));
  const openHeight = 0;
  const closeThreshold = 100;
  const bind = useDrag(({ down, movement: [, my] }) => {
    if (down) {
      api.start({ y: my > 0 ? my : 0, immediate: true });
    } else {
      if (my > closeThreshold) {
        api.start({ y: 400, onRest: onClose });
      } else {
        api.start({ y: openHeight });
      }
    }
  });
  const closeSheet = (callback?: () => void) => {
    api.start({
      y: 400,
      onRest: () => {
        onClose();
        callback?.();
      },
    });
  };
  const closeSheetAsync = (): Promise<void> => {
    return new Promise((resolve) => {
      api.start({
        y: 400,
        onRest: () => {
          onClose();
          resolve();
        },
      });
    });
  };

  if (!isOpen) return null;

  return createPortal(
    <div
      className={background}
      onClick={(e) => {
        if (e.target === e.currentTarget) {
          api.start({ y: 400, onRest: onClose });
        }
      }}
    >
      <animated.div
        className={appBarContainer}
        ref={sheetRef}
        style={{ y }}
        {...bind()}
      >
        <div className={appBarHandleContainer}>
          <div className={appBarHandle} />
        </div>
        {children({
          handleClose: (cb) => closeSheet(cb),
          handleCloseAsync: closeSheetAsync,
        })}
      </animated.div>
    </div>,
    document.body
  );
};
````

이제부터 각 부분들을 역할 단위로 쪼개어서 살펴보고자 합니다.

## 제스처 인식을 위한 `@use-gesture/react`

[@use-gesture/react](https://www.npmjs.com/package/@use-gesture/react)는 리액트 환경에서, 드래그, 마우스 움직임, 스크롤 등의 동작을 `useDrag`, `useGesture`등의 hook을 사용해서 제어할 수 있게 하는 라이브러리 입니다.

본 예제에서는, 애니메이션을 위한 `@react-spring/web`의 `api`를 호출하여 네이티브 느낌을 주는 애니메이션을 실행시키는 역할을 하고 있습니다.

```ts
const bind = useDrag(({ down, movement: [, my] }) => {
  if (down) {
    api.start({ y: my > 0 ? my : 0, immediate: true });
  } else {
    if (my > closeThreshold) {
      api.start({ y: 400, onRest: onClose });
    } else {
      api.start({ y: openHeight });
    }
  }
});
```

그리고 이 `bind`변수는 애니메이션을 재생할 `animated.div`에 부착되게 됩니다.

```tsx
<animated.div
  className={appBarContainer}
  ref={sheetRef}
  style={{ y }}
  {...bind()}
>
  {/*중략*/}
</animated.div>
```

## 애니메이션을 위한 `@react-spring/web` 그리고 `Promise`

사실 단순하게 시각적으로 보임/보이지 않음 형태만 구현만 한다면, 필요 없는 부분 중 하나입니다. 하지만, 일반적인 모바일 앱의 바텀시트는 간단하게라도 닫히는 애니메이션이 있고, 핸들을 잡고 위 아래로 끌어당기면 일정 범위 내에서 따라오기도 하지요.

`Promise`에 대해서는 도입부에서도 간략히 언급한 바와 같이, 웹 환경에서의 '종료 시 애니메이션'을 구현하기 위해서는

> 애니메이션을 재생하기 -> 애니메이션이 끝나기 -> DOM에서 제거하기

라는 과정을 거쳐야 하기 때문에, 애니메이션이 종료되는 시점에서 실행 될 콜백을 호출할 수 있음을 이용해서, 개발자 입장에서 사용하기 편하도록 `Promise`로 래핑하는 내용을 다룰 것 입니다.

### `@react-spring/web`의 `useSpring`과 `api`

[@react-spring/web](https://www.react-spring.dev)은 물리 기반의 애니메이션을 다룰 수 있는 라이브러리로, 단순한 transition이나 keyframe 기반 애니메이션보다 더 자연스러운 움직임을 만들 수 있습니다.

본 예제에서는 `useSpring`을 통해 y축 위치를 제어하고 있으며, `api.start()`를 통해 위치 변경 및 애니메이션을 트리거하고 있습니다. 예컨대, 드래그 중에는 immediate: true 옵션을 사용하여 실시간 반응을 가능하게 하고, 드래그가 끝난 이후에는 위치에 따라 닫히거나 원래 위치로 복귀하는 애니메이션을 실행합니다.

```ts
const [{ y }, api] = useSpring(() => ({ y: 0 }));
```

`y`는 `animated.div`의 스타일로 적용되어 실제 UI에 반영되며, `api.start()`는 해당 스타일 값을 부드럽게 업데이트하는 역할을 합니다.

### fade-out 애니메이션과 `Promise`

React에서 컴포넌트를 “닫을 때” 보통 상태 값을 false로 바꿔서 DOM에서 제거하는 방식으로 처리합니다. 하지만 이렇게 단순하게 처리하면 애니메이션이 끝나기도 전에 컴포넌트가 사라져버리게 됩니다. 그래서 “닫기” 동작에 애니메이션이 필요한 경우 다음과 같은 흐름이 필요합니다

> 1. 애니메이션을 시작하고
> 2. 애니메이션이 끝날 때까지 기다린 다음
> 3. 실제로 닫기 처리 (예: 상태 업데이트)

이때 Promise는 이러한 “비동기 흐름”을 선언적으로 표현하는 데 매우 유용합니다. handleCloseAsync는 닫기 애니메이션이 완료된 후에만 resolve되므로, 다음과 같이 사용 가능합니다.

```ts
await handleCloseAsync();
// 이후 로직 실행
```

## 직접 컴포넌트를 넣지 않고, 함수를 자식으로 넘기게 한 이유

이 바텀시트 컴포넌트는 자식 요소를 일반적인 ReactNode가 아닌, 다음 형태의 함수로 받습니다

```tsx
children: (actions: { handleClose; handleCloseAsync }) => React.ReactNode;
```

이는 자식 컴포넌트가 바텀시트의 제어 함수(handleClose, handleCloseAsync)에 접근할 수 있도록 하기 위함입니다. 일반적인 방식대로 컴포넌트를 props로 넘기면, 바텀시트 내부에서 자식에게 제어 함수를 전달할 수 없기 때문이죠.

이런 패턴은 흔히 “Function as Child”나 “Render Props”라고 하는데요, 자식이 부모의 제어 함수를 바로 써먹을 수 있다는 장점이 있습니다. react-beautiful-dnd나 react-table 같은 유명한 라이브러리들도 이 방식을 사용하고 있습니다. [과거에 제가 react-beautiful-dnd 튜토리얼](/react-beautiful-dnd-1)을 적었을 때에도 이 내용을 짚고 넘어갔었습니다.

사실 JSX의 `children` 라는 것도, 특정 prop을 조금 더 HTML 스럽게 보이게 하기 위해서 넘기는 문법인것을 생각하면 딱히 세삼스러울 일도 없긴 합니다.

## 올바른 DOM 계층구조를 위한 `createPortal`

바텀시트는 보통 모달 형태로 동작하며, position: fixed 스타일을 이용해 화면 상단에 떠 있어야 합니다. 하지만 일반적인 React 컴포넌트 구조에 그대로 포함시키면, 부모 컴포넌트의 overflow나 position 등에 영향을 받을 수 있습니다.

이를 방지하기 위해 React의 `createPortal`을 사용하면, 실제 DOM 상에서는 `document.body` 하위로 바텀시트를 렌더링할 수 있습니다:

```tsx
return createPortal(<BottomSheet />, document.body);
```

조금 더 자세한 정보는 [react 공식 docs](https://react.dev/reference/react-dom/createPortal)를 참고해봅시다.

# 마치며

이번 글에서는 제스처 기반 바텀시트를 직접 만들어보면서 아래와 같은 내용을 다뤄봤습니다:

- `@use-gesture/react`로 드래그 감지하고
- `@react-spring/web`으로 부드럽게 움직이게 만들고
- `Promise`로 닫기 애니메이션 흐름 제어하고
- 함수형 children으로 바깥에서 제어 가능하게 만들고
- `createPortal`로 DOM 구조 안정적으로 유지하기

바텀시트는 그냥 모달보다 UX가 조금 더 까다롭긴 하지만, 이렇게 구성해두면 유연하게 재사용하기도 쉽고 실무에서도 충분히 써먹을 수 있습니다. 직접 만들어보면서 React에서 이런 고급 패턴을 익히는 재미도 느껴보셨으면 좋겠네요!

읽으면서 의문이 드는 부분이나 이해가 잘 되지 않는 부분이 있다면 댓글로 남겨주세요! 끝까지 읽어주셔서 감사합니다.
