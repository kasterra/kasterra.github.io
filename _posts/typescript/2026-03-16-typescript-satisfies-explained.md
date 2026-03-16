---
title: TypeScript satisfies 키워드 제대로 이해하기
subtitle: 타입 계약은 검증하면서도 값의 실제 타입을 잃지 않는 방법
layout: post
image: /images/thumbnails/TS.png
category: typescript
---

## 들어가며

[infer와 extends를 설명한 글](/typescript-infer-explained), 그리고 [as const와 as 타입단언을 설명한 글](/typescript-controlling-type-inference) 다음으로 이어지는 세번째 글입니다.

이전 글들에서는 TypeScript가 타입을 "어떻게 추론하고 계산하는지"에 초점을 맞추었습니다. `infer`, `extends`, `as const` 같은 문법들을 활용하면 상당히 복잡한 타입 관계도 표현할 수 있고, 대부분의 상황에서는 이것만으로도 충분히 타입 안정성을 확보할 수 있습니다.

하지만 실무에서 코드를 작성하다 보면 한 가지 미묘한 문제가 등장합니다. **런타임에서 실제로 사용되는 값의 타입과, TypeScript가 알고 있는 타입 사이에 어긋남이 생기는 상황**입니다. Typescript는 런타임 정보를 알고 있지 않고, 컴파일 타임에만 검사를 하기 때문이죠.

이 문제는 처음에는 다소 엣지 케이스처럼 느껴질 수 있습니다. 그러나 `zod` 같은 런타임 스키마 검증 라이브러리를 사용하기 시작하면 꽤 자주 마주치게 됩니다. 특히 "이 값의 구조는 이 타입을 만족해야 한다"는 것을 **검증은 하되, 타입 자체를 바꾸고 싶지는 않을 때**가 그렇습니다.

이 글에서 다룰 `satisfies` 키워드는 바로 이런 상황을 해결하기 위해 등장한 도구입니다. 실제 코드 예시를 통해 살펴보겠습니다.

```ts
import { z } from "zod";
import type { SomeDTO } from "./types";

export const someSchema = z.object({
  id: z.string(),
  title: z.string(),
  content: z.string(),
  isStar: z.boolean(),
}) satisfies z.ZodType<SomeDTO["res"]["get"]["getInstanceById"]>;
```

이 코드에서 `someSchema`의 실제 타입은 여전히 `ZodObject`입니다. 즉 zod 스키마로서의 기능은 그대로 유지됩니다. 다만 `satisfies`를 사용함으로써 **이 스키마가 DTO에서 기대하는 타입 구조를 제대로 만족하는지**를 TypeScript가 검사하게 됩니다.

이전에 이런 상황에서는 보통 `as`나 타입 주석을 사용해 DTO 타입을 맞추곤 했습니다. 하지만 그렇게 하면 `someSchema`의 타입이 DTO 쪽으로 고정되어 버려, [zod에서 권장하는 확장 방식](https://zod.dev/api?id=extend)인 `someSchema.shape` 기반의 스키마 확장을 사용할 수 없게 됩니다.

`satisfies`는 이 두 가지 요구를 동시에 해결합니다. **zod 스키마 타입은 그대로 유지하면서도, DTO 타입을 만족하는지에 대한 정적 검사는 받을 수 있기 때문입니다.**

## 타입 인터페이스는 수정 안하지만 가드는 받고싶어 - satisfies

TS의 타입 시스템은 어떠한 변수를 특정 타입에 넣을 수 있을지 검사를 하고 넣습니다.

```ts
type Animal = {
  name: string;
};

type Cat = {
  name: string;
  talk: () => void;
};

type Bottle = {
  liquid: "water" | "coffee";
};
```

이 상황에서 TypeScript의 기본적인 타입 호환 규칙을 떠올려 볼 수 있습니다.

`Cat` 타입의 값은 `Animal` 타입으로 취급할 수 있습니다. `Animal`이 요구하는 속성인 `name`을 이미 가지고 있기 때문입니다.

```ts
const cat: Cat = {
  name: "nabi",
  talk() {
    console.log("meow");
  },
};

const animal: Animal = cat; // OK
```

반대로 `Bottle` 타입의 객체는 `Animal` 타입으로 취급할 수 없습니다.

```ts
const bottle: Bottle = {
  liquid: "water",
};

const animal: Animal = bottle; // ❌ name 속성이 없음
// Type 'Bottle' is not assignable to type 'Animal'.
// Property 'name' is missing in type 'Bottle' but required in type 'Animal'.
```

즉 TypeScript에서는 **목표 타입이 요구하는 구조를 만족하는 경우에만** 대입이 가능합니다. 이를 보통 _구조적 타입 시스템(structural typing)_ 이라고 부릅니다.

이 규칙은 대부분의 상황에서 매우 직관적으로 동작합니다. 하지만 실제 코드를 작성하다 보면 조금 미묘한 상황이 등장합니다.

예를 들어 우리가 어떤 객체가 **최소한 `Animal`의 구조는 만족하는지** 검사하고 싶다고 가정해 봅시다.

```ts
const cat = {
  name: "nabi",
  talk() {
    console.log("meow");
  },
};
```

이 객체는 분명 `Animal`이 요구하는 `name` 속성을 가지고 있으므로 `Animal`로 취급할 수 있습니다.

하지만 실제로 코드에서 사용하고 싶은 것은 `Animal`의 인터페이스가 아니라 **`Cat`이 가지고 있는 `talk` 메서드**일 수 있습니다.

만약 다음과 같이 타입 주석을 사용해 버리면 문제가 생깁니다.

```ts
const cat: Animal = {
  name: "nabi",
  talk() {
    console.log("meow");
  },
};
```

이렇게 작성하면 TypeScript는 `cat`을 **완전히 `Animal` 타입으로 취급**하게 됩니다. 그 결과 실제로 존재하는 `talk` 메서드를 사용할 수 없게 됩니다.

```ts
cat.talk();
// ❌ Property 'talk' does not exist on type 'Animal'
```

즉 우리는 다음 두 가지를 동시에 만족시키고 싶은 상황에 놓이게 됩니다.

- 이 객체가 `Animal`의 구조를 **만족하는지는 검사받고 싶다**
- 하지만 변수의 실제 타입은 **더 구체적인 형태 그대로 유지하고 싶다**

바로 이런 상황을 해결하기 위해 등장한 것이 `satisfies` 키워드입니다.

### satisfies vs as

이 내용을 보다 보면 이런 생각이 들 수 있습니다.

> "굳이 `satisfies`를 쓸 필요가 있을까? 일단 `Animal`로 받아 두고, 나중에 `as Cat`으로 단언하면 되는 것 아닌가?"

실제로 다음과 같은 코드는 작성할 수 있습니다.

```ts
const raw: Animal = {
  name: "nabi",
  talk() {
    console.log("meow");
  },
};

const cat = raw as Cat;
cat.talk();
```

겉보기에는 그럴듯합니다. raw를 먼저 Animal 타입으로 받고, 이후에 as Cat을 통해 다시 구체적인 타입으로 단언하는 방식입니다.

하지만 여기에는 중요한 문제가 있습니다. 처음 선언하는 순간 raw의 타입은 이미 Animal로 고정되어 버린다는 것이죠.

```ts
raw.talk();
// ❌ Property 'talk' does not exist on type 'Animal'
```

즉 TypeScript의 관점에서는 실제 객체에 talk 메서드가 존재하더라도, 그 변수의 타입은 이미 Animal일 뿐입니다.

그 다음에 작성하는

```ts
const cat = raw as Cat;
```

이는 타입을 다시 추론하거나 검증하는 과정이 아니라,
TypeScript의 타입 검사를 우회하는 타입 단언(type assertion)에 가깝습니다.

```ts
const raw: Animal = { ... };
const cat = raw as Cat;
```

즉 이 흐름은 처음에 타입을 Animal로 좁혀 버리고 이후에 as를 통해 개발자가 다시 Cat이라고 주장하는 패턴입니다.

반면 satisfies는 애초에 타입을 잃지 않습니다.

```ts
const cat = {
  name: "nabi",
  talk() {
    console.log("meow");
  },
} satisfies Animal;

cat.talk(); // OK
```

이 경우 TypeScript는

1. 이 객체가 Animal의 구조를 만족하는지 검사하고
2. 변수 cat의 실제 타입은 { name: string; talk: () => void } 형태로 그대로 유지합니다.

즉 satisfies는 검사는 받되 타입 정보는 잃지 않게 해주는 도구이고, raw as Cat 패턴은 한 번 잃어버린 타입 정보를 개발자 단언으로 덮어쓰는 방식이라고 볼 수 있습니다.

### satisfies의 실전 패턴

앞에서는 `Animal`과 `Cat` 예제로 `satisfies`의 기본 개념을 살펴보았습니다.

이제 이 개념이 실무 코드에서 왜 유용한지, 도입부에서 등장했던 `zod` 스키마 예제를 통해 다시 보겠습니다.

```ts
import { z } from "zod";
import type { SomeDTO } from "./types";

export const someSchema = z.object({
  id: z.string(),
  title: z.string(),
  content: z.string(),
  isStar: z.boolean(),
});
// 이 스키마는 DTO에서 기대하는 다음 타입과 구조가 맞아야 합니다.
// z.ZodType<SomeDTO["res"]["get"]["getInstanceById"]>
```

이 상황에서 선택지는 대략 두 가지입니다.

첫째는 타입 주석을 직접 붙이는 방법입니다.

```ts
const someSchema: z.ZodType<SomeDTO["res"]["get"]["getInstanceById"]> =
  z.object({
    id: z.string(),
    title: z.string(),
    content: z.string(),
    isStar: z.boolean(),
  });
```

이 방법은 스키마가 DTO 타입과 맞는지 확인할 수 있다는 장점이 있습니다. 하지만 변수의 타입이 `ZodType` 쪽으로 고정되어 버리기 때문에, `z.object(...)`가 원래 가지고 있던 더 구체적인 정보와 사용성을 충분히 살리기 어렵습니다.

예를 들어 [zod에서 권장하는 확장 방식](https://zod.dev/api?id=extend)인 `.shape` 기반 확장처럼, `ZodObject`일 때 활용할 수 있는 API를 쓰기 불편해질 수 있습니다.

반대로 타입과 관련된 검사를 아예 하지 않으면 `z.object(...)`를 자유롭게 다룰 수는 있지만, 실제 DTO 타입과 스키마가 어긋나더라도 그 사실을 바로 알아차리기 어렵습니다.

`satisfies`는 이 둘 사이의 균형을 잡아 줍니다.

즉 스키마의 실제 타입은 그대로 유지하면서도, 그 스키마가 DTO에서 기대하는 구조를 만족하는지는 TypeScript에게 검사받을 수 있습니다.

정리하면 `satisfies`는 다음과 같은 상황에서 특히 유용합니다.

- 특정 타입 계약을 만족하는지는 검사하고 싶고
- 그러나 값이 원래 가지고 있는 더 구체적인 타입 정보는 잃고 싶지 않을 때

`zod` 스키마처럼 런타임에서도 실제로 쓰는 객체를 다루는 코드에서 이 패턴은 꽤 자주 등장합니다.

## 마치며

유지보수 가능한 코드를 만들기 위해 아키텍처와 타입 시스템을 함께 공부해 왔고, 이번 글도 그 흐름 위에서 정리했습니다. 이 시리즈에서 계속 말하고 싶은 핵심은 하나입니다. **타입은 기교를 뽐내는 수단이 아니라, 코드의 예측 가능성을 높이는 도구**라는 점입니다.

이번 편의 `satisfies`도 같은 맥락에 있습니다. 타입 계약은 검증하면서도 값의 구체적인 타입 정보는 보존할 수 있어서, 실무에서 생각보다 자주 도움이 됩니다.

앞으로도 TypeScript를 단순한 문법 소개보다, 실제 문제를 푸는 관점에서 계속 정리해 보겠습니다.

읽어주셔서 감사합니다.
