---
layout: post
title: 다시 마주한 JS 기초지식 - 원시값과 참조값
subtitle: 블로그 글을 위한 코드에서 다시 느낀 JS 기초의 중요성
category: javascript
image: /images/thumbnails/JS.png
---

# 들어가며

며칠 전에 블로그에 올린 [react query 만들기 글](/react-query-from-scratch)을 작성하면서 단순히 유튜브 강의의 코드를 복사-붙여넣기 하는 복제가 아닌, 의미 있는 재생산을 하기 위해서 코드를 열심히 패러프래이징 하듯이 빙글빙글 돌리고 있었습니다. 그러다가 코드가 돌아가지 않는 상황을 마주하기도 하고, 어떤식으로 개선하면 좋을까 라는 생각을 스스로 하기도 하고, 확률적으로 헛소리를 하는 LLM에게도 물으면서 '내가 이해할 수 있는 형태'로 만들기 위해서 노력을 했었습니다.

이번 글에서는 react query를 재현하면서 겪었던 이슈가, 잊고 지냈던 JS 기본 지식을 리마인드 하게 하여서 이렇게 글을 적어보고자 합니다.

# 문제의 코드

`createQuery.ts`를 처음 구성할 때 다음과 같이 구성하였습니다. 결론부터 말하자면, 이 코드는 원본 라이브러리의 동작을 재현하지 못합니다.

```tsx
// createQuery.ts

/**
 * 이것은 결과적으로 사용자 입장에서 갱신이 되지 않는다
 * 클로저를 통해서 갱신이 되기는 하나, 이것은 함수 내에 있는 변수가 바뀌는 것이지
 * return된 복사된 값은 갱신이 되지 않는다
 * 원시값이 아닌 참조를 반환하게 되는 object를 통해서 작업을 하게 하면
 * 그것이 제대로 돌아가는 createQuery.ts의 결과물이다.
 */

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

코드 앞부분의 주석에서도 설명한 내용을 조금 더 자세히 풀면 다음과 같을 것 입니다.

`fetch`를 호출하면 `setState`에 의해서 `state`가 수정됩니다. 이것이 가능한 이유는, 실행컨텍스트와 클로저에 대한 개념을 이해하고 있다면, 충분히 가능할 것이라고 생각합니다. 처음 함정에 빠졌던 부분은, '객체 -> 참조타입 -> 식별자 달라도 같은 객체를 지칭함' 이라는 생각에 너무나도 몰두해 있었던 점 입니다.

# setState를 다시 돌아보기

react hook을 만들어보던[면접에서 말할 수 있는 react 지식 3편 글](/react-knowledge-avail-in-interview-3)에서 `useState`의 setter function은 값을 그대로 대입하는 것 이었습니다. state를 다시 set 할 때, 새로운 객체를 만드는 것은, 새로운 객체를 할당함으로서 값이 **달라졌다** 라는 것을 명확히 react에 전달하기 위함이라는 것도 우리가 배웠었지요.

곰곰히 다시 생각해봅시다. '새로운 객체'는 오해할 내용도 별로 없지만, '값이 달라진다' 라는 것에 대한 고찰이 저에게 부족하지 않았나 하고 생각이 들었습니다. JS라는 맥락에서 '값'에 대한 정확한 정의를 알고 있는가 라는 질문이 저에게 다가왔습니다.

# '값'에 대해서

JS 코어 컨셉에 대해서 모르는게 생겼다 하면 역시, '그 책'을 펼 때가 된겁니다. 도마뱀이 그려진 [모던 자바스크립트 딥 다이브](https://m.yes24.com/Goods/Detail/92742567), 정말 뼈가 되는 책이라고 할 수 있겠습니다.

이번에 초점을 맞출 부분은, 원시값과 객체값에 대한 정의와 차이점 입니다.

## 원시값과 객체란? 차이점은?

원시 타입의 값은, **숫자, 문자열, 불리언, null, undefined, 심벌** 이고, 간단하게 말하면 **객체 타입 빼고 나머지** 라고 볼 수 있겠습니다. 어떤 중대한 차이점이 있길래, 어떻게 구분이 '객체타입, 그리고 나머지' 로 구분이 될 수 있는지에 대해서 3가지의 커다란 차이점에 대해서 나열 해보겠습니다.

### 1. 원시값은 변경 불가능, 객체 값은 변경 가능

'변경 가능함' 이라는 말은 자칫 혼란을 줄 수 있습니다. 오히려 '불변성(immutability)'이라는 개념으로 접근하는 것이 더 혼란을 줄이는데 도움이 되기 때문에, 해당 관점으로 접근하겠습니다.

프로그래밍에서의 '불변'의 기준은, **변수의 할당이 아닌 메모리의 내용** 입니다. 숫자형 변수를 담을 수 있는 `let`으로 선언한 변수에, 여러가지 숫자를 담을 수 있다고 해서, 원시 값인 숫자를 여러가지 담을 수 있다고 해서, 숫자 타입이 변경 가능하다는 것이 아니라는 것 입니다. 불변의 기준은, **변수가 기준이 아닌 값이 기준** 이라는 사실을 염두해 주세요.

변수에 원시값을 저장하고, 재할당 하면, 메모리의 내용이 수정되는 것이 아니라, 변수가 가리키게 되는 메모리의 영역이 바뀝니다. 5에서 10으로 재할당을 했다고 하면, 5를 가리키던 변수가 10을 가리키게 되는 것이지요. 학부 1학년 기초 프로그래밍 시간 등에 배운 포인터를 생각하면 좋을 것 같습니다. 변수는 재할당을 통해서 변수 값을 변경(엄밀한 의미에서는 '교체') 할 수 있기 때문에 '변'수인 것 입니다.

반면 객체값은 변경이 가능합니다.

```js
let a = { value: 30 };
a.value = 20;
```

라는 코드가 있다고 할 때, 첫번째 줄의 a의 주소값과, 두번째 줄의 a의 주소값이 같다는 것 이지요. 내용물이 바뀌었는데도, 주소값이 같다는건 즉 메모리의 내용을 수정했다는 뜻이겠죠?

### 2. 원시값은 변수에 할당된 메모리에 실제값, 객체는 참조값이 들어있다

그림으로 그려보겠습니다. 첫번째에 다룬 변경 불가능과, 재할당을 하면 교체 된다는 이야기까지 한번에 정리 할 수 있습니다.

![원시값의 선언, 할당, 재할당 다이어그램](/images/javascript/primitive-vs-ref-in-js/1-primitive-imm.jpeg)

메모리 주소는 임의로 기입한 것 입니다. 중요한 것은 값이 달라질 때 마다, 변수의 주소값은 달라진다는 것이며, 해당 주소값에는 실제 값인 `undefined`, `80`, `90`등이 적혀있다는 것 입니다.

반면 객체값이 동작하는 방식은 원시값과 다르게 동작합니다.

![객체값의 선언, 수정 다이어그램](/images/javascript/primitive-vs-ref-in-js/2-object-m.jpeg)

원시 값 예제와는 다르게, `person`변수의 메모리 안에 들어 있는 값이, 실제 값이 아니고, 주소인 '참조값'입니다. 그리고 `person` 변수에 값을 재할당 하지 않고, 수정이 가능함을 확인할 수 있는 것도 확인 가능한 것을 알 수 있습니다. 원시 값과는 동작 방식이 다른것을 볼 수 있지요...

### 3. 원시 값은 값에 의한 전달, 객체 값은 참조에 의한 전달

해당 유형의 값을 가지는 변수를 다른 변수에 할당 할 때, 어떤 방식으로 전달되는지를 다루는 섹션입니다. 가장 직관적으로 다가오는 한가지 예시는, **'함수에 해당 값이 어떻게 넘어가는가'** 라고 볼 수 있겠네요.

간단한 예제를 보이자면, 아래의 코드의 실행결과를 예측하는 간단한 예제일 것 입니다.

```js
var score = 80;

function foo(scoreCopy) {
  scoreCopy = 90;
}

foo(score);
console.log(score);
```

답은 원본 그대로인 `80` 입니다. 매개변수에 score라는 변수가 할당되었지만, 매개변수와 원본 변수는 다른 변수이기 때문에 영향을 받지 않는다는 그런 간단한 이야기 입니다. 함수에 넘긴다는 특수한 상황이 아닌, 보다 일반적인 '변수에 변수를 할당하는 상황'을 아래에 도식으로 남겨놓겠습니다.

![원시값을 담은 변수가 다른 변수에 할당되는 상황을 나타내는 도식](/images/javascript/primitive-vs-ref-in-js/3-primitive-copy.jpeg)

이 동작이 JS엔진의 동작과 정확히 일치하지 않을 수도 있다라고 합니다만, mdn의 원시 값 페이지에서도 그런식으로 설명했다고 책에는 나와 있네요. 2025년 4월 6일 현재의 [mdn 원시값 한국어 docs](https://developer.mozilla.org/ko/docs/Glossary/Primitive)에서는 보이지 않으나, 복사 되면 같은 주소값을 가지다가, 다른 값이 할당되면 비로서 변경된다 라는것은, 적당히 추상화 시켜도 되는 내용이 아닐까 싶습니다.

객체의 경우에는 **얕은 복사** 라는 이름으로 많이 알려져 있습니다. 얕은 복사가 무엇인지 알고 있다면, 아래의 코드를 실행하면 어떤 결과가 나오는지 알고 있을 것 입니다.

```js
var person = { name: "lee" };
var copy = person;
person.name = "kim";

console.log(copy.name);
```

`person`과 `copy`는 같은 객체를 가리키고 있기 때문에 한쪽의 변경사항이 다른 쪽에도 반영이 되는 것 이지요. 도식으로 나타내면 아래와 같습니다.

![객체 값의 얕은 복사 다이어그램](/images/javascript/primitive-vs-ref-in-js/4-obj-cpy.jpeg)

# 그래서 문제는 뭐였느냐

제가 마주했던 문제의 원인은, 값의 전달의 오해와 피상적으로 알고 있던 지식의 혼선에서 비롯되었던 것 이었습니다. React `setState`로 상태값을 갱신하는것과 마찬가지로, 알아서 잘 되겠거니 하였지만, 객체 그 자체를 수정한것이 아닌, 새로운 객체를 만들어서 재할당 하였기 때문에, 생긴 문제였습니다.

이전 코드에서는 `state` 객체 자체를 외부에 반환했습니다.  
하지만 내부에서 `setState`가 실행되며 **새로운 객체가 `state`에 재할당**되면,  
기존에 반환했던 `state`는 여전히 **예전 객체의 참조만** 갖고 있기 때문에, 변경된 내용을 감지하지 못합니다.

처음에 반환된 state객체는 전혀 수정되지 않고, 새로운 state객체가 함수 실행 컨텍스트 안에서 갱신이 될 뿐인 것이기에, 해당 컨텍스트 밖에 있는 코드 입장에서는 갱신을 확인할 수 있을 리 없지요

## 문제를 해결한 코드

```ts
// createQuery.ts

import { Query, QueryFn, QueryKey, QueryState, QuerySubscriber } from "./types";

export const createQuery = <T>({
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

    subscribers: [] as QuerySubscriber[],

    setState: (updater: (prev: QueryState<T>) => QueryState<T>) => {
      query.state = updater(query.state);
      query.subscribers.forEach((s) => s.notify());
    },

    subscribe: (subscriber: QuerySubscriber): (() => void) => {
      query.subscribers.push(subscriber);
      return () => {
        const index = query.subscribers.indexOf(subscriber);
        if (index !== -1) query.subscribers.splice(index, 1);
      };
    },

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
          lastUpdated: Date.now(),
        }));
      } catch (error) {
        query.setState((prev) => ({
          ...prev,
          error: error as Error,
          status: "error",
          isFetching: false,
        }));
      }
    },
  };

  return query;
};
```

해당 코드는 객체안에서 모든 변수를 관리하고 해당 객체를 반환하는 식으로 문제를 해결하였습니다. 이전 코드에서는 함수 내부 실행 컨텍스트에서만 반영이 되던게 문제였기 때문에, 그 실행 컨텍스트를 통째로 노출시키는 것이 어찌보면 가장 직관적이고 간단한 해결책 이었던 것이지요.

# 마무리

‘딥 다이브’, ‘기초 학습’이라는 말의 중요성은 익히 알고 있었지만,
이론서만 읽으며 마치 공자님의 말씀을 읊조리듯 공부하는 방식은 재미도 없고, 실감도 잘 나지 않았습니다.

하지만 이렇게 직접 코드를 작성하고, 예상치 못한 문제를 마주하며,
그 원인을 파고들어 다시 기본기를 되짚는 경험은 훨씬 더 생생하고 오래 남습니다.

이번 경험을 통해, 실전 속에서 기초를 다시 돌아보는 방식이야말로 가장 효과적인 학습 방법이라는 걸 다시금 느꼈습니다.

이 글을 읽는 여러분들도, 자신만의 코딩 프로젝트를 통해 기초 학습의 동기를 얻는 경험을 해보셨으면 하는 마음으로 이 글을 마무리합니다.

끝까지 읽어주셔서 감사합니다!
