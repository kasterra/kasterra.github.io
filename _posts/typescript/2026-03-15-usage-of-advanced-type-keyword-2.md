---
title: TypeScript as const, readonly, as 타입단언 제대로 이해하기
subtitle: widening 문제부터 타입 단언의 위험성까지
layout: post
image: /images/thumbnails/TS.png
category: typescript
---

## 들어가며

[지난 TS 고급 타입 글](/typescript-infer-explained)에서는 단순한 원시 타입을 넘어서, 타입에서 새로운 타입을 이끌어내는 **제네릭 타입**에 대한 이야기를 주로 다루었습니다. 해당 제네릭 타입에서 템플릿 변수를 제약하는 `extends` 키워드와, 타입에서 타입을 이끌어내는 `infer` 키워드도 다루었죠.

이번 글에서는 그렇게 정의한 타입들을, 실제 TS 런타임 변수들에 어떻게 잘 넣을 수 있을지에 대한 이야기를 해보려고 합니다.

## 가장 간단한 방법 - 타입 주석 (type annotation)

우선 변수 자체에 '너는 어떤 타입을 받아' 라고 알려주는 타입 주석이 있습니다. 아래의 코드에서 `myVal`이라는 변수에 `string`이라는 타입을 설명하는 것이 타입 주석 이지요.

```ts
let myVal: string;

myVal = "hello type";
```

간단한 문법이고, 많은 경우 큰 문제 없이 잘 동작합니다. `string`같은 원시 타입이 아니고, [지난 글](/typescript-infer-explained)에서 언급했던 제네릭 타입 에서도 문제없이 동작하고요

```ts
type ApiResponse<T> = {
  data: T;
  success: boolean;
  error: string | null;
};

type User = {
  id: number;
  name: string;
};

const userResponse: ApiResponse<User> = {
  data: {
    id: 1,
    name: "kasterra",
  },
  success: true,
  error: null,
};
```

> 이렇게 괜찮은 도구가 있으니, 잘 써봅시다. 읽어주셔서 감사합니다

라고 끝났을거면 이 글을 발행할 문제가 없었겠지요. 타입 주석에는 몇몇 상황에서 생길 수 있는 문제들이 있습니다. 어떤 문제인지 알아보고, 해당 문제들을 해결하는 도구가 어떤 것인지에 대해 살펴봅시다.

### 친절한 TS 타입 시스템의 '오해' - widening 문제

TS의 타입 시스템은 C++ 같은 다른 언어의 `auto` 타입처럼, 적극적으로 타입 추론을 하려고 합니다. 다만 가끔 그 추론이, 코드 작성자의 의도와 맞지 않을 때가 있지요. 간단한 예시를 들어보면 다음과 같습니다.

```ts
// UI의 스타일을 위한 객체
const config = {
  theme: "dark",
  // ... 기타 여러 속성 lang:'ko' 같은것이 있을 수 있겠음
};

type Theme = "dark" | "light";

function setTheme(theme: Theme) {}

setTheme(config.theme);
// ❌ string은 Theme에 할당 불가
```

분명 `config` 객체의 `theme` 속성은 `"dark"`이고, 타입 추론 하는데에 도움이 되라고, `const`도 붙였는데 이게 무슨 일일까요?

JS에서 `const` 키워드의 오해하기 쉬운 특징을 이해하고 있으시다면, 당연한 것이라는 결론을 얻을 수 있습니다. JS에서의 `const`키워드는, **할당된 변수의 주소값이 바뀌지 않는다** 인것이지, 내용물이 바뀌지 않는다는 보장은 해줄 수 없다는 사실을 리마인드 하면 바로 알 수 있습니다. 그래서, 정말로, '값도 변하지 않게 하기'를 하려면 `Object.freeze()`등의 방법을 사용함을 공부해보셨으리라 생각합니다. 실제로 config선언한 다음에 `config.theme = 'light'`를 작성해도 에러가 나지 않으니까요!

참고로 TypeScript는 객체의 속성이 이후 변경될 가능성이 있다고 가정하기 때문에 `"dark"` 같은 리터럴 값을 그대로 유지하지 않고 `string`으로 타입을 넓혀(widen) 추론합니다.  
이러한 동작은 대부분의 상황에서는 유용하지만, 지금처럼 리터럴 타입을 유지하고 싶은 경우에는 개발자가 의도를 명확하게 전달해 줄 필요가 있습니다.

### 이 변수는 안바뀐다고 내가 보장할게 - as const

이 상황에서 우리가 전달하고 싶은 의도는 사실 단순합니다.

> "이 객체의 값은 내가 의도적으로 고정한 것이고, 이후에 변경될 일이 없다"

이 의도를 타입 시스템에 전달하는 방법이 바로 `as const`입니다.

```ts
const config = { theme: "dark" } as const;
```

이렇게 선언하면 TS는 객체를 다음과 같은 타입으로 추론합니다.

```ts
{
  readonly theme: "dark";
}
```

여기서 중요한 변화는 세 가지입니다.

1. **리터럴 타입이 유지됩니다**

기존에는 `"dark"`가 `string`으로 widening 되었지만,

```ts
theme: string;
```

이 아니라

```ts
theme: "dark";
```

로 유지됩니다.

2. **속성이 readonly가 됩니다**

```ts
config.theme = "light";
// ❌ readonly property
```

이제 TS는 "이 값은 바뀌지 않는다"는 가정을 기반으로 타입을 추론합니다.

3. **배열이라면 readonly tuple로 변환됩니다**

예를 들어

```ts
const roles = ["admin", "user"] as const;
```

는 다음 타입이 됩니다.

```text
readonly ["admin", "user"]
```

---

이제 앞에서 문제가 되었던 코드도 정상적으로 동작합니다.

```ts
type Theme = "dark" | "light";

function setTheme(theme: Theme) {}

const config = {
  theme: "dark",
} as const;

setTheme(config.theme); // ✅ 정상 동작
```

정리하자면 `as const`는 단순히 "const처럼 보이게 하는 문법"이 아니라,

- **리터럴 타입 유지**
- **readonly 적용**
- **배열을 tuple로 고정**

이라는 세 가지 효과를 동시에 만들어 주는 **타입 추론 제어 도구**라고 볼 수 있습니다.

### 변수가 아닌것을 고정하려면 - Readonly

as const 설명에서 `readonly`라는 키워드가 등장했습니다.
이번 섹션에서는 이 readonly가 정확히 어떤 의미를 가지는지 조금 더 자세히 살펴보겠습니다.

변수 자체에 대해서는 `as const`를 통해 "이 값은 변경되지 않는다"는 의도를 전달할 수 있습니다. 그렇지만, 특정 속성은 수정을 허용하고, 특정 속성은 상수로 관리하고 싶을 수 있습니다. 아래와 같이요.

```ts
type AppConfig = {
  readonly appName: string;
  theme: "dark" | "light";
};

const config: AppConfig = {
  appName: "MyApp",
  theme: "dark",
};

config.theme = "light";
// ✅ 가능

config.appName = "OtherApp";
// ❌ readonly property
```

아무래도 테마는 바꿀 수 있는것이 좋지만, 어플리케이션의 이름이 런타임에 바뀌어 버리는 것은 여러 문제가 생길 수 있으니까요. `readonly` 키워드를 객체 필드의 이름 앞에 달아놓음으로서, TS 타입 시스템에 의도를 명확히 알릴 수 있습니다.

#### `Readonly<T>` 유틸리티 타입

새로운 타입을 선언하면서 `readonly`를 선언할 수 있지만, 기존의 타입을 참고하면서, 해당 타입에 불변성을 추가하고 싶을 수 있습니다. TS 전역에 선언해져있는 `Readonly<T>` 유틸리티 타입을 이용하면, 기존 타입을 재사용 하면서, 의도를 명확히 전달 할 수 있습니다.

```ts
type User = {
  id: number;
  name: string;
};

const user: Readonly<User> = {
  id: 1,
  name: "kasterra",
};

user.name = "other";
// ❌ Cannot assign to 'name'
```

## TS 컴파일러에게 '단언' 하기 - as 타입단언

지금까지 살펴본 `as const`, `readonly` 같은 도구들은 **타입 시스템이 더 정확하게 추론하도록 돕는 도구**였습니다.  
하지만 현실의 코드에서는 가끔 이런 상황이 생깁니다.

> "TypeScript가 모르는 정보를, 개발자인 내가 알고 있는 경우"

예를 들어 외부 API에서 오는 데이터, DOM API, 혹은 런타임 상황에 따라 달라지는 값들이 그렇습니다.  
이럴 때 사용하는 문법이 바로 **타입 단언(type assertion)** 입니다.

타입 단언은 **타입을 변환하는 것이 아니라**, 컴파일러에게

> "이 값은 내가 책임질 테니 이 타입으로 취급해 주세요"

라고 말하는 문법입니다.

가장 흔한 예시는 DOM API를 사용할 때입니다.

```ts
const input = document.querySelector("#name") as HTMLInputElement;

input.value = "hello";
```

여기서 `document.querySelector`의 반환 타입은 다음과 같습니다.

```text
Element | null
```

왜냐하면 실제 DOM에서는

- 해당 요소가 존재하지 않을 수도 있고
- 어떤 종류의 HTML 요소인지도 확실하지 않기 때문입니다.

하지만 우리는 코드의 맥락을 통해

> #name 은 input 요소다

라는 사실을 알고 있을 수 있습니다.  
이럴 때 `as HTMLInputElement`라는 **타입 단언**을 사용해 더 구체적인 타입으로 다룰 수 있습니다.

### 타입 가드와 함께 사용하는 경우

물론 TypeScript는 개발자가 제공하는 **런타임 단서**를 통해 타입을 좁힐 수도 있습니다.

예를 들어 다음과 같은 코드를 보면

```ts
const input = document.querySelector("#name");

if (input) {
  // ...
}
```

`if` 문을 통해 `null`이 아니라는 사실을 알 수 있으므로, TypeScript는 `input`을 `Element` 타입으로 좁혀 줍니다.  
이런 방식으로 타입을 좁히는 것을 **타입 가드(type guard)** 라고 합니다.

다만 이 경우에도 `input`이 정확히 `HTMLInputElement`라는 보장은 없습니다.  
그래서 특정 DOM API를 사용해야 하는 경우에는 여전히 타입 단언이 필요할 수 있습니다.

```ts
const input = document.querySelector("#name");

if (input) {
  (input as HTMLInputElement).value = "hello";
}
```

### as 타입단언은 함부로 사용해서는 안됩니다

지금까지의 설명만 보면 `as` 타입 단언이 매우 편리해 보일 수 있습니다. 하지만 이 문법에는 중요한 특징이 있습니다.

> 타입 단언은 **타입 검사를 통과시키는 문법일 뿐**, 실제 런타임 검증을 해주지는 않습니다.

예를 들어 다음 코드는 TypeScript 컴파일러를 통과할 수 있습니다.

```ts
const el = document.querySelector("#name") as HTMLInputElement;
```

하지만 실제 DOM에서 `#name` 요소가 `span`이라면 어떻게 될까요?

```ts
el.value = "hello";
// 런타임 에러 가능
```

`HTMLSpanElement`도 `Element`이기 때문에 TypeScript는 이 문제를 잡아낼 수 없습니다. 따라서 타입 단언은 **개발자가 책임지고 사용하는 도구**라고 볼 수 있습니다.

그래서 실무에서는 보통 다음과 같은 순서로 접근합니다.

1. 가능한 경우 **타입 가드로 해결한다**
2. 제네릭 추론이나 타입 설계를 통해 해결한다
3. **정말 필요한 경우에만 `as` 타입 단언을 사용한다**

실제로 많은 코드 리뷰에서 자주 나오는 질문이 바로 이것입니다.

> "이 `as` 없이 해결할 수 없을까요?"

그만큼 타입 단언은 강력하지만, 동시에 **타입 안정성을 약화시킬 수 있는 도구**이기도 합니다.

## 마치며

실제 실무 프로젝트를 진행하면서 느끼고 배우게 되는 TS 관련 지식을 정리하기 위해서 벌써 두번째 글을 작성하고 있습니다. TS 타입 시스템 자체가 그리 간단하게 넘어갈 수 있는 토픽이 아니기에, 다음 글로 넘겨야 하는 주제들이 계속 생기네요.

다음 글에서는 `as const`와 `as` 타입단언이 해결할 수 없는 문제와, 그 문제를 해결하는 도구로서의 `satisfies`키워드를 설명해볼까 합니다. 물론, 본 블로그의 성격에 맞게, 2026년 현재 런타입 검증 라이브러리로 정말 많은 지분을 가진 `zod`를 사용하는 코드의 예제와 함께요.

끝까지 읽어주셔서 감사합니다.
