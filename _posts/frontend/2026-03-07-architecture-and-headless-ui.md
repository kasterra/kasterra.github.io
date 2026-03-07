---
title: 아키텍처를 고민하다 보니 Headless UI로 수렴한 이야기
subtitle: MUI 경험에서 시작해 FSD 구조와 Radix UI까지 이어진 이야기
layout: post
category: frontend
image: /images/thumbnails/Folder.png
---

## 들어가며 - 구현은 당연히 할 줄 알아야하죠. 그 다음은

이 글의 제목으로 꼭 쓰고 싶었던 문장이었습니다. 당연한 이야기지만, 개발자라면 주어진 기능을 구현해낼 수 있어야 합니다. 요리사가 칼을 다룰 줄 알아야 하듯, 구현 능력은 개발자의 가장 기본적인 도구이기 때문입니다.

하지만 제목에서도 말했듯이, 단순히 "돌아간다"는 것만으로 충분하지는 않습니다.

이론적으로는 유튜브나 인스타그램 같은 거대한 서비스도 하나의 `index.js` 파일 안에 모든 로직을 몰아 넣어 구현할 수 있습니다. 물론 실제로 그렇게 하지는 않습니다. 대신 기능별로 파일을 나누고, 계층을 만들고, 구조를 설계합니다.

왜일까요?

여러 이유가 있겠지만, 저는 그중에서도 **유지보수성**이라는 관점이 가장 중요하다고 생각합니다. 새로운 기능 하나를 추가하기 위해 수십만 줄의 코드가 뒤엉킨 숲을 헤매야 한다면, 개발은 금세 고통스러운 일이 되어버립니다.

조금 극단적인 예를 들긴 했지만, 이제 현실적인 이야기로 넘어가 보겠습니다.

### MUI의 시들함과 Headless UI의 부상

제가 학부 시절 동아리 프로젝트나 개인 프로젝트를 하던 때에는, MUI(Material UI)를 자주 사용했습니다. 구글의 Material Design 시스템을 React 환경에서 쉽게 사용할 수 있도록 제공하는 라이브러리였고, 덕분에 비교적 빠르게 UI를 구축할 수 있었습니다.

하지만 프로젝트를 몇 번 반복하다 보니 한 가지 문제를 느끼게 되었습니다.

**MUI가 제공하는 디자인 질서 안에서는 매우 편하지만, 그 질서를 벗어나려고 하면 코드가 빠르게 복잡해진다**는 점이었습니다.

커스터마이징이 점점 늘어나면서 스타일 오버라이드가 쌓이고, 결국에는 라이브러리를 쓰는 것인지 직접 구현하는 것인지 애매한 상태가 되기도 했습니다. 그래서 어느 순간부터는 MUI를 실제 컴포넌트로 사용하는 대신, Figma 디자인 키트 정도로만 참고하고 직접 UI를 구현하는 경우도 많아졌습니다.

여러 프로젝트를 거치며 또 하나 흥미로운 사실을 발견했습니다.

UI의 **겉모습은 매번 바뀌지만**, 컴포넌트 내부의 **동작 로직은 상당히 비슷하다**는 점입니다. 예를 들어 다음과 같은 로직들은 프로젝트가 바뀌어도 반복됩니다.

- Dialog의 열림/닫힘 상태 관리
- 접근성(ARIA) 속성 처리
- 키보드 포커스 관리
- ESC 키로 닫기
- 외부 클릭 감지

그래서 자연스럽게 이전 프로젝트의 코드에서 **로직 부분만 복사해서 재사용**하는 일이 생기곤 했습니다.

경험 많은 개발자들은 이 반복을 더 체계적으로 정리하기 시작했습니다. 그렇게 탄생한 개념 중 하나가 바로 **Headless UI**입니다.

Headless UI는 말 그대로 **스타일이 없는 UI 컴포넌트**입니다. 대신 다음과 같은 것들을 제공합니다.

- 상태 관리
- 접근성 처리
- 이벤트 로직

그리고 실제 UI 표현은 사용하는 쪽에서 자유롭게 구현할 수 있도록 합니다.

오늘날 많이 사용되는 `shadcn/ui` 역시 이러한 철학 위에서 만들어졌고, 그 기반에는 **Radix UI**가 자리 잡고 있습니다.

Radix UI는 UI의 시각적인 스타일을 제공하지 않는 대신, 복잡한 상호작용 로직과 접근성 처리를 안정적으로 제공하는 라이브러리입니다. 덕분에 개발자는 디자인 시스템에 맞는 UI를 자유롭게 구현하면서도, 이미 검증된 동작 로직을 재사용할 수 있습니다.

## FSD(Feature sliced design)과 헤드리스

FE 개발을 하다 보면 FSD(Feature-Sliced Design)라는 아키텍처를 들어보셨을 수도 있습니다. 간단하게 말하면 코드를 한데 모으는 것이 아니라 **책임과 역할에 따라 구조적으로 분리하는 설계 방식**입니다.

예를 들어 다음과 같은 계층 구조를 사용합니다.

- `shared` : 프로젝트 전반에서 공통적으로 사용하는 유틸, UI, 훅
- `entities` : 도메인 데이터 단위
- `features` : 특정 기능을 수행하는 로직
- `widgets` : 여러 feature를 조합한 UI 블록
- `pages` : 실제 페이지 단위

이 구조의 핵심은 **"UI와 로직을 어디에 둘 것인가"**에 대한 명확한 기준을 만든다는 점입니다.

제가 현재 진행하고 있는 프로젝트에서도 FSD 구조를 사용하고 있는데, 실제로 작업을 하다 보니 흥미로운 점이 하나 보였습니다.

바로 **Headless UI 패턴과 자연스럽게 맞물린다**는 점입니다.

예를 들어 다음과 같은 구조를 생각해볼 수 있습니다.

```text
features/auth
  ├─ model
  │   └─ useUserForm.ts
  ├─ ui
  │   └─ UserForm.tsx
```

여기서 `useUserForm` 같은 훅은 다음과 같은 역할을 합니다.

- 상태 관리
- 폼 로직
- API 호출
- validation 처리

그리고 `UserForm` 컴포넌트는 **순수하게 UI만 담당**합니다.

```tsx
function UserForm() {
  const { register, submit } = useUserForm();

  return (
    <form onSubmit={submit}>
      <input {...register("name")} />
      <button type="submit">submit</button>
    </form>
  );
}
```

이 구조를 조금 더 확장해 보면 자연스럽게 다음과 같은 구조가 됩니다.

```text
widgets/user
  └─ UserWidget.tsx

features/user
  └─ useUser.ts

shared/ui
  └─ Input
```

즉 **로직은 feature에**, **표현은 widget이나 ui에** 위치하게 됩니다.

이런 구조에서 widget을 작성하다 보면 자연스럽게 이런 규칙이 생깁니다.

> widget은 최대한 "헤드(head)"만 담당한다.

즉 widget은 실제 비즈니스 로직을 직접 가지고 있기보다는, 이미 정의된 feature 훅들을 가져와서 **조립만 하는 역할**을 합니다.

이렇게 되면 결과적으로 다음과 같은 구조가 만들어집니다.

- 로직 → hooks / features
- 상태 → model
- UI → widget / shared/ui

그리고 이 구조는 **Headless UI 패턴과 굉장히 비슷한 방향**으로 흘러가게 됩니다.

Headless UI 역시 동일한 철학을 가지고 있습니다.

- 로직 제공
- 상태 관리 제공
- 접근성 처리 제공

하지만 **스타일이나 UI는 제공하지 않습니다.**

예를 들어 Radix UI의 Dialog는 다음과 같이 사용할 수 있습니다.

```tsx
import * as Dialog from "@radix-ui/react-dialog";

export function ExampleDialog() {
  return (
    <Dialog.Root>
      <Dialog.Trigger>Open</Dialog.Trigger>

      <Dialog.Portal>
        <Dialog.Overlay />

        <Dialog.Content>
          <Dialog.Title>Edit profile</Dialog.Title>
          <Dialog.Description>
            Make changes to your profile here.
          </Dialog.Description>

          <button>Save</button>
        </Dialog.Content>
      </Dialog.Portal>
    </Dialog.Root>
  );
}
```

여기서 Radix가 제공하는 것은 다음과 같습니다.

- focus trap
- ESC close
- ARIA attributes
- accessibility

하지만 실제 스타일은 하나도 제공하지 않습니다. 그래서 Tailwind나 CSS-in-JS, 혹은 디자인 시스템과 자유롭게 결합할 수 있습니다.

이 지점에서 저는 한 가지 흥미로운 점을 느꼈습니다.

**좋은 아키텍처를 고민하다 보면 자연스럽게 Headless UI 같은 패턴에 가까워진다**는 점입니다.

즉 이것은 단순히 "요즘 유행하는 UI 라이브러리"라기보다는, **복잡한 UI를 유지보수 가능한 형태로 만들기 위한 자연스러운 설계 방향**에 가깝다고 생각합니다.

## 마치며

프론트엔드 개발을 하다 보면 "어떻게 구현할 것인가"라는 질문에만 집중하게 되기 쉽습니다. 하지만 일정 규모 이상의 프로젝트에서는 그보다 더 중요한 질문이 등장합니다.

> "이 코드를 앞으로도 유지할 수 있는 구조인가?"

Headless UI, 컴포넌트 설계, 그리고 아키텍처에 대한 고민은 결국 이 질문에서 시작한다고 생각합니다.

이 글에서는 그 중에서도 Headless UI라는 개념이 왜 등장했는지에 대해 개인적인 경험을 바탕으로 정리해 보았습니다.

이 포스트의 썸네일 아이콘은 [flaticon](https://www.flaticon.com/free-icons/project)에서 가져왔음을 밝힙니다. 끝까지 읽어주셔서 감사합니다.
