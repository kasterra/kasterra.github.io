---
layout: post
title: 복잡한 form을 vanilaJS로 깔끔하게 정리해보기
subtitle: 결코 변하지 않는 기술인 vanilaJS로 복잡한 form을 관리해 봅시다
category: frontend
image: /images/thumbnails/HTMLJS.png
---

# 들어가며

구직 활동을 하다 보면 다양한 문제를 마주하게 됩니다. 그중에서도 개인적으로 의미 있었던 경험 하나를 이번 글에서 정리해보려 합니다.  
제가 맡았던 과제는 **React 없이**, 웹 앱 수준의 복잡한 폼 UI를 **오직 바닐라 JavaScript**만으로 구성해야 했던 프로젝트였습니다. 아래 스케치처럼 단순한 입력 폼이 아닌, 이미지 업로드부터 회차 관리, 실시간 검증까지 다양한 인터랙션을 포함하고 있었죠.

![UI image](/images/frontend/vanila-JS-complex-UI-sketch.jpeg)

사실 이 주제는 예전에 작성했던 **[“현대 프론트엔드 환경을 돌아보며”](/reviewing-contemporary-fe-tools)** 라는 글과도 맞닿아 있습니다. 그 글에서는 “SPA나 메타 프레임워크 없이도 MPA 환경에서 충분히 현대적 기능을 구현할 수 있다”는 관점을 이야기했죠.

이번 글은 그 연장선이지만 관점은 조금 다릅니다.

> 프레임워크 없이, 순수 웹 표준과 브라우저 기본 도구들만으로 복잡한 UI를 모던하게 구현할 수 있을까?

이 질문에서 출발해, 직접 구현했던 경험을 토대로 어떤 점들을 신경 썼는지 공유하려고 합니다.

# 프로젝트 개요

아래는 이번 글에서 다룰 폼 UI의 요구사항입니다. 사진으로 보면 단순해 보일 수도 있지만, 실제 기능 요구사항을 나열해 보면 꽤 복잡한 구조라는 걸 알 수 있습니다.

- **대표 이미지 업로드**
  - 업로드 시 이미지가 영역을 가득 채워야 함
  - 다시 클릭하면 새 이미지를 업로드할 수 있어야 함
- **보조 이미지 업로드**
  - 최대 4개 업로드 가능
  - 2x2 그리드 형태로 표시
  - 재업로드 가능
- **콘텐츠 제목 입력**
  - 입력과 동시에 실시간 글자 수 표시
  - 최소 글자 수 미달 시 시각적 경고
- **활동 방식 선택(온라인/오프라인)**
  - 버튼 기반 토글 UI
- **회차 정보 리스트**
  - 날짜, 시작 시간, 종료 시간, 활동 내역 입력
  - 활동 내역 textarea도 상태에 따라 시각적 반영 필요
- **제출 버튼**
  - 필수 요소들이 충족되기 전까지 비활성화
  - 모든 검증을 통과하면 활성화

이 글에서는 위 요구사항을 바닐라 JS만으로 어떻게 구현했는지, 그리고 그 과정에서 어떤 구조와 패턴이 도움이 되었는지 소개할 예정입니다.

# 기능을 어떻게 나눌 것인가? - DOM을 기준으로 생각해보기

사실 기능만 놓고 보면 `index.js` 하나에 모든 로직을 다 밀어 넣어도 동작은 합니다.  
하지만 그런 식의 **거대한 index.js**는 유지보수 단계에서 바로 한계를 드러냅니다. 특정 기능에 문제가 생겼거나 새로운 요구사항이 생겼을 때, 그 기능이 어디에 숨어 있는지 찾는 과정만으로도 이미 피로도가 높죠. 게다가 수정하다가 **해당 기능과 아무 상관 없는 외부를 건드려버릴 위험**도 커집니다.

이런 이유 때문에 자연스럽게 떠올렸던 접근이 React에서 익숙한 ‘컴포넌트 기반 사고’였습니다.  
웹 표준에도 Web Component라는 훌륭한 방식이 있고, 예전에 작성했던  
[Web component란 무엇일까요?](/what-is-web-component),  
[Lit로 SPA 만들어보기 ② - FE](/setting-litelement-spa-client)  
같은 글에서도 다뤘던 주제이죠.

하지만 이번 과업은 **시간 제약이 꽤 타이트**했고, Web Component는 초기 준비 비용이 적지 않기 때문에 적절한 선택은 아니었습니다. 그럼 자연스럽게 다음 질문이 생깁니다.

> 그렇다면 어떤 기준으로 기능을 쪼개야 할까?

이걸 고민하던 중 다시 떠올리게 된 것이 바로 **DOM 구조**였습니다.  
DOM은 원래부터 트리 형태로 구성되어 있고, 각 요소들은 뿌리에서부터 가지처럼 뻗어나가는 명확한 구조를 갖고 있습니다.

즉, DOM을 기준으로 기능을 나누면 다음과 같은 장점이 생깁니다.

- 특정 기능이 속한 DOM subtree만 다루면 되기 때문에 **애꿎은 사촌 노드**를 건드릴 일이 없다.
- 실제 UI 구조와 1:1로 대응되기 때문에 어떤 기능이 어느 위치를 담당하는지 빠르게 파악할 수 있다.
- 모듈을 물리적으로 분리했을 때도 DOM 구조 자체가 **자연스러운 “경계(boundary)” 역할**을 해준다.

이러한 이유로, 이번 프로젝트에서는 DOM 구조를 기반으로 기능을 나누고, 각 기능을 개별 모듈로 분할하는 방식으로 접근하게 되었습니다.

## DOM subtree 단위로 기능을 캡슐화하기

```js
export function initSomeModule(root) {
  if (!(root instanceof HTMLElement)) return;
  // 로직 구현 ...
}
```

와 같은 형태로 구현하고, 페이지 최상단의 js 파일에는 `document.querySelector` 등으로 해당 기능 모듈의 최상단 `Element`를 가져오면 됩니다. UI를 그리는 마크업만 분리되었을 뿐, 모듈화가 지향하는 목표들은 이루고 있다고 볼 수 있지요. 그리고 글자수 실시간 추적 등의 상태 관리도, 해당 함수 안에서 갇혀있게 되니, 변수를 관리하면서 생길 수 있는 위험성도 회피할 수 있게 되지요.

아래 예시는 ‘텍스트 영역의 실시간 상태 관리’처럼 작지만 독립적인 기능을 DOM 기반 모듈로 캡슐화한 사례입니다.

```js
/**
 * responsive-textarea 컴포넌트를 위한 기능 모듈 JS 입니다.
 * @param {HTMLElement} root
 * @param {number} [minChars=8]
 * @param {number} [maxChars=80]
 */

export function initResponsiveTextarea(root, minChars = 8, maxChars = 80) {
  if (!(root instanceof HTMLElement)) return;
  /** @type {HTMLTextAreaElement | null} */
  const textarea = root.querySelector(".responsive-textarea__textarea");
  const charCounter = root.querySelector(".responsive-textarea__char-counter");
  const help = root.querySelector(".responsive-textarea__tooltip");
  if (!textarea || !charCounter || !help) return;

  textarea.maxLength = maxChars;

  function update() {
    let length = textarea.value.length;

    if (length > maxChars) {
      textarea.value = textarea.value.slice(0, maxChars);
      length = maxChars;
    }

    if (length < minChars) {
      root.classList.add("responsive-textarea--red");
      root.classList.remove("responsive-textarea--green");
      help.textContent = `최소 ${minChars}자 이상 입력해주세요.`;
    } else if (length >= minChars && length < maxChars) {
      root.classList.remove("responsive-textarea--red");
      root.classList.add("responsive-textarea--green");
      help.textContent = "";
    } else {
      // 최대 길이 도달과 중간 구간 모두 동일한 UI 처리
      root.classList.remove("responsive-textarea--red");
      root.classList.add("responsive-textarea--green");
      help.textContent = "";
    }

    charCounter.textContent = `${length} / ${maxChars}자 (최소 ${minChars}자)`;
  }

  textarea.addEventListener("input", update);
}
```

이 패턴의 장점은 각 모듈이 DOM subtree 내부에만 관심을 갖기 때문에, 상태·이벤트·유효성 검증 등 모든 로직이 해당 영역 안에서 캡슐화된다는 점입니다. React의 컴포넌트만큼 강력한 구조는 아니지만, SPA 프레임워크 없이도 일관된 구조와 모듈성을 유지할 수 있습니다.

# Form의 실시간 검증 - 이벤트 버블링을 활용하기

각 기능들은 DOM subtree 단위로 캡슐화하여 해결할 수 있었습니다.  
그렇다면 이제 남은 문제는 하나입니다.

> **각 요소들의 값이 모두 유효한지 실시간으로 체크하고, 언제 제출 버튼을 활성화할 것인가?**

React 환경에서는 이런 문제를 해결하기 위해 `react-hook-form`을 자주 사용합니다.  
`useForm` 내부에 제약 조건을 넣고, 필드 변화가 일어날 때마다 최종 formStatus를 계산하는 방식이지요.

이번 프로젝트는 React가 아니기 때문에 동일한 구조를 그대로 가져올 수는 없습니다.  
하지만 여기서 가져올 수 있는 중요한 철학은 하나 있습니다.

> 각각의 input이 아닌, **그보다 상위에서 input들의 변화를 감지하고 검증을 통합한다**.

이 철학을 바닐라 JS로 구현하기 위해서는 "변화가 일어날 때마다 상위에서 그것을 알 수 있는 메커니즘"이 필요합니다.

그리고 다행히도 웹 브라우저는 이미 이 일을 아주 잘 해주는 기능을 가지고 있습니다. 바로 **이벤트 버블링(Event Bubbling)** 입니다.

표준 input 이벤트들은 기본적으로 버블링됩니다.  
그런데 "회차가 추가되었다", "활동 방식이 바뀌었다" 등과 같은 웹 표준에 없는 이벤트는 어떡하냐고요?  
이 역시 문제 없습니다.  
웹 표준에는 **커스텀 이벤트를 만들고 디스패치할 수 있는 API**가 있기 때문입니다.

해당 문서는 MDN에서 확인할 수 있습니다:  
[mdn EventTarget.dispatchEvent() 문서](https://developer.mozilla.org/ko/docs/Web/API/EventTarget/dispatchEvent)

아래는 이 방식으로 구현한 실제 예시입니다.  
각 기능 모듈이 상태가 바뀔 때마다 커스텀 이벤트를 상위로 올리고,  
최상단에서는 그 이벤트들을 모아 Form 전체의 validity를 계산합니다.

```js
const headerCtaBtn = document.querySelector(".header-cta-btn");

function hasMainImage() {
  const img = document.querySelector(".main-image-input__preview");
  return !!(img && !img.hidden && img.getAttribute("src"));
}

function validTitle() {
  const ta = document.getElementById("content-title");
  const v = (ta?.value || "").trim();
  return v.length >= 8 && v.length <= 80; // 최소 8자, 최대 80자
}

function hasActivityType() {
  const picked = document.querySelector(
    '.activity-style-picker [role="radio"][aria-checked="true"]'
  );
  return !!picked;
}

function parseTimeToMinutes(t) {
  if (!t) return null;
  const h = Number.parseInt(String(t.hour || "").trim(), 10);
  const m = Number.parseInt(String(t.minute || "").trim(), 10);
  if (!Number.isFinite(h) || !Number.isFinite(m)) return null;
  if (h < 1 || h > 12) return null;
  if (m < 0 || m > 59) return null;
  const base = (h % 12) * 60 + m;
  const isPM = String(t.ampm || "").toUpperCase() === "PM";
  return base + (isPM ? 12 * 60 : 0);
}

function validEventSchedule() {
  const root = document.querySelector(".event-schedule");
  if (!root) return false;
  const data = serializeEventSchedule(root);
  if (!Array.isArray(data) || data.length === 0) return false;

  for (const ses of data) {
    // 날짜 필수
    if (!ses?.dateYmd) return false;

    const s = parseTimeToMinutes(ses.start);
    const e = parseTimeToMinutes(ses.end);
    if (s == null || e == null) return false;
    if (s > e) return false;

    const activityText = String(ses.desc ?? "").trim();
    if (activityText.length < 8 || activityText.length >= 800) return false;
  }
  return true;
}

function recalcAndToggleCtas() {
  const ok =
    hasMainImage() && validTitle() && hasActivityType() && validEventSchedule();

  headerCtaBtn.disabled = !ok;
  headerCtaBtn.setAttribute("aria-disabled", String(!ok));
  footerCtaBtn.disabled = !ok;
  footerCtaBtn.setAttribute("aria-disabled", String(!ok));
}

// ---- 이벤트 연결 ----
// 각 모듈이 내보내는 커스텀 이벤트에 훅
document.addEventListener("main-image-input:change", recalcAndToggleCtas);
document.addEventListener("activity-type-change", recalcAndToggleCtas);
document.addEventListener("event-schedule:change", recalcAndToggleCtas);

// 초기 1회 계산
requestAnimationFrame(recalcAndToggleCtas);
```

이 구조의 장점은 명확합니다.

- 각 기능 모듈은 자신이 속한 DOM subtree만 관리한다.
- 변화는 커스텀 이벤트를 통해 상위로만 흐른다.
- Form 전체의 validity는 단 하나의 함수에서 일관되게 계산된다.

결과적으로 React의 단방향 데이터 흐름과 유효성 검증 접근을 바닐라 JS만으로도 충분히 재현할 수 있음을 확인할 수 있습니다.

# 마무리하며

이번 작업은 기술적으로도 흥미로웠지만, 개인적으로는 “도구에 앞서 사고가 먼저다”라는 점을 다시 확인한 시간이었습니다. 프레임워크에 익숙한 상태에서 바닐라 JS로 돌아오는 것은 때로는 불편하지만, 그 불편함 덕분에 ‘왜 이런 구조가 필요한가’를 더 깊게 이해하게 되는 것 같고, 어찌보면, 보다 앞선 엔지니어 분들의 생각을 들여다보는 재미있는 시간이었지 싶습니다.

앞으로도 개인의 경험 가운데에서 흥미로운 경험들을 공유하면서, 누군가의 인사이트에 도움이 되었으면 좋겠습니다. 끝까지 읽어주셔서 감사합니다.
