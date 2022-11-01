---
layout: post
title: VanilaJS의 Middleware. Proxy
subtitle: expressJS에서 쓰이는 middleware. 바닐라JS에도 비슷한게 있다는걸 아셨나요? Observable을 위한 빌드업...
category: javascript
image: /images/thumbnails/JS.png
---

# 들어가며

express.js로 서버 개발을 해봤다면 '미들웨어' 라는 키워드를 들어 보셨을 겁니다. 중간에 값들을 가로채서 여러 처리들을 할 수 있는 함수 조각들을 의미하는것을 알고 계실겁니다. 그런데 바닐라 JS에서도 이와 비슷하게 연산 중간에 값을 '가로챌' 수 있는 객체가 있다는 것을 알고 계셨나요? 이번 포스팅에서는 그 역할을 하는 Proxy 객체와 그 응용예시들을 살펴볼까 합니다. 생각보다 다양한 일을 할 수 있는것에 아마 깜짝 놀랄지도 모르겠습니다.

이 글은 <https://ko.javascript.info/proxy>의 내용을 참조하여 작성되었습니다.

# `Proxy` 개요

영단어 'proxy'는 '대리자'를 뜻합니다. JS에서의 `Proxy`도 이 단어의 뜻과 비슷하게, 여러 연산을 '대신' 수행하는 역할을 합니다. `Proxy`가 대신 할 수 있는 연산을 대략적으로 나열하면 아래와 같습니다.

- 읽기 연산
- 쓰기 연산
- `in` 연산자
- `delete` 연산자
- 함수 호출
- `new` 연산자 동작(생성자)
- `Object.getPropertyOf`
- `Object.setPropertyOf`
- `Object.isExtensible`
- `Object.preventExtensible`
- `Object.defineProperty`
- `Object.getOwnPropertyDescriptor`
- `Object.getOwnProertyNames`
- `Object.getOwnPropertySymbols`
- `for ... in`
- `Object.keys/values/entries`

모든 연산은 아니지만, JS에서 우리가 사용하는 여럿 연산에 대한 가로채기 연산을 지원해줌을 확인할 수 있습니다. 이 모든 연산을 가로채는 법과 그 실사용례를 다 적기엔 지면이 부족하므로, 혹시 관련 내용이 궁금하다면, [mdn 독스](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Proxy)에서 `Proxy`의 `handler`의 메소드들을 확인해 보면 좋을듯 합니다.

# `Proxy` 기본 문법

`Proxy`는 `Date` 처럼 전역으로 선언된 객체이며, `new` 연산자를 이용해서 인스턴스를 생성할 수 있습니다. 생성자 문법은 아래와 같습니다. **반드시** `new`를 통해서만 `Proxy()`를 호출 할 수 있습니다.

```js
new Proxy(target, handler);
```

- `target` : `Proxy`로 래핑할 원본 객체입니다. 어떠한 종류의 객체도 들어갈 수 있습니다. `Proxy` 또한 객체기 때문에 이중 프록시 같은것도 가능합니다.
- `handler` : `Proxy`의 동작을 정의하는 객체입니다. 해당 객체의 속성은 **함수** 입니다. 해당 속성들은 대상 객체에 대한 호출을 가로채기 때문에 **트랩**이라고도 합니다.

트랩의 이름은 언어 차원에서 정의되어 있으며, 이번 포스트 시리즈에서 다룰 트랩들의 이름과 역할에 대한 간단한 개요는 아래와 같습니다.

- apply : 함수 호출에 대한 트랩
- construct : `new` 연산자에 대한 트랩
- deleteProperty : `delete` 연산자에 대한 트랩
- get : 속성 값을 가져오는(읽는) 연산에 대한 트랩
- has : `in` 연산자에 대한 트랩
- set : 속성 값을 설정하는 연산에 대한 트랩

---

## 주의사항

이 `Proxy`에는 한계점이 있는데, 사실 정말 직관적인 한계점입니다. `Proxy`는 **원본 객체와는 다른** 객체입니다. 그렇기 때문에

- 원래 객체의 프라이빗 속성에 접근 할 수 없습니다.
  - Proxy는 직접 접근할 수 없지만 `get` 연산의 `target` 매개변수를 통해서 접근하는 식으로 해결 가능합니다.
- `===` 연산자를 통한 엄격한 비교 연산에 대한 트랩을 작성할 수 없습니다.
  - 내부를 비교하는 연산 등은 [npm 라이브러리](https://www.npmjs.com/package/proxy-compare)등이 있는것 같습니다. (정확하지 않은 정보라면 코멘트 바랍니다.)

# 일단 부딛혀 보는 사용예시

새로운 개념을 볼 때, 공식 docs 부터 해서 bottom-up으로 배우는 것은 나쁜 방법은 아니지만, 시간이 많이 걸리고, 내용이 좀 깊어질 경우 파고둘 엄두가 안나는 단점이 있습니다. 이 글에서는 최대한 부담을 덜기 위해서 필요한 만큼의 지식과, 흥미 유발을 위한 실용적인(적어도 실용적으로 보이기라도 하는) 예제들로 글을 구성을 해보겠습니다.

## 값 검증

Express.js 에서 미들웨어를 사용해서 값들을 검증 할 수 있듯, `Proxy`도 비슷한 역할을 할 수 있습니다. 우선 코드부터 보여드리겠습니다.

```js
const validator = {
  set(obj, prop, value) {
    if (prop === "age") {
      if (!Number.isInteger(value)) {
        throw new TypeError("The age is not an integer");
      }
      if (value > 200) {
        throw new RangeError("The age seems invalid");
      }
    }

    // 값을 저장하는 기본 동적
    obj[prop] = value;

    // 성공 표시
    return true;
  },
};

const person = new Proxy({}, validator);

person.age = 100;
console.log(person.age); // 100
person.age = "young"; // 예외 발생
person.age = 300; // 예외 발생
```

나이를 나타내는 `age`라는 속성에 대해서 **200 이하의 정수** 만 들어갈 수 있도록 일종의 제약을 거는 코드라는 것을 대강 파악할 수 있을 것입니다. 이제 이 코드를 제대로 한번 파고 들어가봅시다.

### 쓰기 연산을 가로채는 `handler.set()`

`handler`는 위의 `Proxy` 명세에 간단히 지나가듯 적어놨듯, `new Proxy()`에 들어가는 두번째 인자인 객체를 의미합니다. 그리고 그 객체의 속성 이름이 `set`이라는 의미이지요. 앞으로 설명할 `Proxy`의 **handler**에 대해서도 다음과 같은 형식으로 설명할 것 입니다. mdn에서도 이런 형식을 사용하기도 하고요.

서론이 길었습니다. `handler.set()`은 다음으로 정의됩니다.

```js
new Proxy(target, {
  set(target, property, value, receiver) {
    // 여러분의 어썸한 로직
  },
});
```

#### 매개변수

- target : `Proxy`로 처리할 원본 대상 객체
- property : 설정할 속성의 이름 또는 `Symbol`
- value : 설정할 속성의 새 값
- receiver : 할당이 지시된 원래 객체... 라고 하는데, 프로토타입 체이닝을 통해서 해당 메소드가 불러진게 아니라면 프록시 그자체라고 보면 됨

#### 반환값

할당이 성공했을 때는 `true`, 실패했을때는 `false`를 반환해야 함. 엄격 모드에서 `false`가 반환되면 `TypeError`가 반환됨.

---

이제 다시 예제 코드의 `set()`을 사용하는 부분을 다시 봅시다.

```js
  set(obj, prop, value) {
    if (prop === "age") {
      if (!Number.isInteger(value)) {
        throw new TypeError("The age is not an integer");
      }
      if (value > 200) {
        throw new RangeError("The age seems invalid");
      }
    }
```

변경하려는 속성 이름인 `prop` 변수를 활용해서 `age`를 수정하려 할 때, 조건을 검사하고, 조건이 맞지 않으면 에러를 `throw`하는 간단한 로직임을 알 수 있습니다.

`handler.set`은 `[[Set]]` 연산을 가로채는 것이기 때문에, 내부적으로 `[[Set]]` 연산을 사용하는 `Array.prototype.push()` 등 또한 정상적으로 가로챌 수 있다는 점도 알아두면 좋을것 같습니다.

## 객체의 속성 기본 값 설정하기

기본적으로 존재하지 않는 속성을 읽으면 `undefined`를 받는것이 JS 세계의 기본 상식입니다. 하지만, `undefined`보다 다른 값, 이를테면 `0`을 받고 싶다면 어떡해야 할까요? 아래 코드는 그 작업을 아무 문제없이 해내는 코드입니다.

```js
let numbers = [0, 1, 2];

numbers = new Proxy(numbers, {
  get(target, prop) {
    if (prop in target) {
      return target[prop];
    } else {
      return 0; // 기본값
    }
  },
});

alert(numbers[1]); // 1
alert(numbers[123]); // 0 (해당하는 요소가 배열에 없으므로 0이 반환됨)
```

### 읽기 연산을 가로채는 `handler.get()`

긴 말 필요없습니다. 명세!

```js
new Proxy(target, {
  get(target, property, receiver) {
    // 여러분의 어썸한 로직
  },
});
```

#### 매개 변수

위의 `set`에서 다룬 그것과 사실 같습니다.

- target : `Proxy`로 처리할 원본 대상 객체
- property : 설정할 속성의 이름 또는 `Symbol`
- receiver : 읽기가 지시된 원래 객체. 프로토타입 체이닝을 통해서 이것이 불러지지 않았으면, 해당 프록시 자체.

#### 반환값

**아무거나** 정말 **아무거나** 반환할 수 있습니다. 여러분의 로직에 맞는 값을 반환해 주세요!

---

다시 한번 `handler.get`이 사용된 부분을 봅시다.

```js
  get(target, prop) {
    if (prop in target) {
      return target[prop];
    } else {
      return 0; // 기본값
    }
  },
```

`prop in target` 으로 `target`안에 `prop`이 있으면, 있는 값을 반환해주고, 아니라면 기본값으로 `0`을 반환해주는 간단한 로직이었음을 알 수 있습니다.

# 사실 이 글을 작성한 이유. `Observable`

컴퓨터 공학의 디자인 패턴을 공부하신 분이라면, 옵저버 패턴(또는 발행/구독 패턴)을 들어 보셨을 겁니다. 간단히 말하자면, 객체의 상태 변화를 관찰하는 관찰자, 즉 옵저버를 등록하여, 해당 객체가 변경된 사실을 옵저버(구독자)들에게 통지하는 패턴입니다. 자세히 알고 싶으신 분을 위해서 [위키백과 링크](https://ko.wikipedia.org/wiki/%EC%98%B5%EC%84%9C%EB%B2%84_%ED%8C%A8%ED%84%B4)를 달아 두도록 하겠습니다.

이 글에서 다룬 `Proxy`를 통해서 이 옵저버 패턴을 구현할 수 있습니다. 우선 예제 코드부터 살펴 보겠습니다.

```js
let handlers = Symbol("handlers");

function makeObservable(target) {
  // 1. 핸들러를 저장할 곳을 초기화합니다.
  target[handlers] = [];

  // 나중에 호출될 것을 대비하여 핸들러 함수를 배열에 저장합니다.
  target.observe = function (handler) {
    this[handlers].push(handler);
  };

  // 2. 변경을 처리할 Proxy를 만듭니다.
  return new Proxy(target, {
    set(target, property, value, receiver) {
      let success = Reflect.set(...arguments); // 동작을 객체에 전달합니다.
      if (success) {
        // 에러 없이 프로퍼티를 제대로 설정했으면
        // 모든 핸들러를 호출합니다.
        target[handlers].forEach((handler) => handler(property, value));
      }
      return success;
    },
  });
}

let user = {};

user = makeObservable(user);

user.observe((key, value) => {
  alert(`SET ${key}=${value}`);
});

user.name = "John";
```

임의의 객체 `target`을 넘겨받아서, 해당 객체를 observable로 만드는 코드입니다. 천천히 살펴봅시다.

중복을 방지하기 위해서 `Symbol`을 통해서 handler들을 담을 속성을 초기화 하고(`target[handlers] = []`), 핸들러 함수들을 등록하는 `observe` 메소드를 구현해줍니다.

**이제 우리가 이 글의 많은 지면을 잡아먹으면서 다룬 `Proxy`가 사용되는 시점입니다.** 해당 객체가 변경될 때 마다(수정 될 때 마다) `handlers`에 등록된 함수들을 호출할 수 있도록, `handler.set` 트랩을 작성해 줍시다.

```js
let success = Reflect.set(...arguments); // 동작을 객체에 전달합니다.
```

라는 코드가 낯설어 보이는군요. `Reflect.set`은 `handler.set`을 따로 떼놓은 것과 같다라고 생각하셔도 됩니다. `Proxy`의 `handler.set`과 받는 매개변수가 같으며, 동작도 매우 비슷해서 지금 이 코드를 이해할 때에는 같다 라고 생각하셔도 됩니다.

두번째로는 `argument`인데, 이것은 함수 내에서 해당 함수의 인수의 배열이라고 생각하시면 됩니다. `...` 스프레드 연산자를 통해서, `Reflect.set`에 넘겨주는것을 확인할 수 있지요. `set`연산이 성공적으로 되면 `true`를, 아니라면 `false`가 리턴 됩니다.

이제 이어지는 코드는 `set` 하는데에 문제가 없었다면, 등록된 콜백을 `forEach`로 실행해주는 로직임을 알 수 있습니다. 해당 객체의 변화를 구독하는 친구들에게 변화의 소식을 알려줄 수 있겠네요. 변경된 속성 이름(`property`)와 변경된 값(`value`)를 콜백에 넣어 줬으니까요.

Vanila JS에서 React의 Context API 처럼 전역 변수를 전달하고, 해당 변화를 구독할 수 있는 `makeObservable` 구현. `Proxy`만 제대로 이해한다면 그리 어려운 내용은 아니었습니다.

# 마치며

원래 처음에는 `Proxy`의 많은 내용들을 한 글에다가 담아보려고 했지만, 내용이 워낙에 방대하다 보니, 상태관리 라이브러리인 MobX 같은 곳에서 쓰이는 실전적인 예인 observable을 우선적으로 다루고, 향후의 글에서 조금 더 복잡한 사용 예들을 살펴볼까 합니다. 부스트캠프를 하면서 스스로 학습의 토끼굴에 빠져서 헤메고 있었는데, 이렇게 글을 하나라도 쓰면서 토끼굴을 조금 정리할 수 있어서 다행이었던것 같습니다. 글 끝까지 읽어주셔서 정말 감사합니다 :)

혹시라도 잘못된 정보나, 이해가 잘 되지 않는 부분이 있다면 코멘트 부탁드리겠습니다. 안녕~

# 참고 자료

- [proxy에 대한 javascript.info](https://ko.javascript.info/proxy) : 예시 참조에 사용 (CC-BY-NC)
- [mdn 문서](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Proxy#%EC%83%9D%EC%84%B1%EC%9E%90) : 예시 참조, 그리고 JS 개발자의 영원한 친구 MDN(퍼블릭 도메인!)
