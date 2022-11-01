---
layout: post
title: Proxy. 너 생각보다 다재다능 하구나?
subtitle: 이전 글에서 분량이 너무 많아서 다루지 못한 고급 사용법들을 정리해 보겠습니다
category: javascript
image: /images/thumbnails/JS.png
---

# 개요

[지난번에 쓴 글](/middleware-of-js-proxy)에서 JS 내장 객체인 `Proxy`에 대한 간단한 개요와 함께 간단한 예시와 함께, `Observable` 구현까지 함께 살펴 보았습니다. 이번 글에서는 지난 글에서 다루지 못했던 좀 더 고급 사용법 등을 한번 다뤄볼까 합니다. 물론 이번 글에서도 무작정 API 명세만 주르륵 늘어놓기 보다는, 실제적으로 어떻게 쓰이는지에 대한 예시를 우선적으로 살펴보고자 합니다. 이번 글에서 다룰 예제들은, 객체 디스크립터나 함수가 호출되는 방법 등 JS의 근본적인 내용들을 담고 있기 때문에, 이론적인 설명 후에 코드를 제시할 예정입니다. 아 그리고, **좀 길어요**. 앞에 썼던 글은 좀 가볍게 읽는 느낌이라면, 이번 글은 딥 다이브를 하는 느낌으로 다뤄볼 예정입니다!

이론 설명을 보기 전에 아래의 내용을 읽고 오면 좋을듯 합니다.

- [객체 디스크립터를 다루는 mdn 링크](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
- [인터널 메소드(내부 연산)에 대해 다루는 블로그 링크](https://yceffort.kr/2020/09/understanding-ecmascript-1#objectprototypehasownproperty-%EB%AA%85%EC%84%B8)
  - (다 읽을 필요는 전혀 없음) 해당 블로그 링크에서도 소개되었지만, [표준 인터널 메서드의 목록](https://tc39.es/ecma262/#table-5)
- [함수 포워딩, 데코레이팅에 대한 링크](https://ko.javascript.info/call-apply-decorators) : 포워딩과 데코레이팅의 개념을 모르신다면 한번 읽어보세요

# 반복 작업 (`Object.keys`, `for ... in`) 인터셉트 하기

객체를 대상으로 반복 작업을 할 때, `Object.keys`나 `for ... in` 문을 통해서 작업을 많이 할 것입니다. `Proxy`라는 친구는 생각보다 다재다능 해서 이러한 연산 또한 인터솁트가 가능합니다.

`Proxy`는 내부 연산을 인터셉트 하는 친구입니다. 저번 포스트에서 소개한 친구들도 `[[Get]]` 연산과 `[[Set]]` 연산을 인터셉트 하는 친구였죠. 이번에 우리가 다루어 볼 반복 연산은 `[[OwnPropertyKeys]]` 라는 연산을 통해서 프로퍼티 목록을 가져오는 연산들 입니다. 그러면 `[[OwnPropertyKeys]]` 내부 연산을 인터셉트 할 수 있다면 위 속성들 또한 인터셉트 할 수 있다는 생각이 드네요.

아래의 코드는 `ownKeys` 트랩을 사용해서 `_`로 시작하는 프로퍼티는 객체를 대상으로 하는 반복문의 순환 대상에서 벗어나게 설정하는 코드입니다.

```js
let user = {
  name: "John",
  age: 30,
  _password: "***",
};

user = new Proxy(user, {
  ownKeys(target) {
    return Object.keys(target).filter((key) => !key.startsWith("_"));
  },
});

// "ownKeys" 트랩은 _password를 건너뜁니다.
for (let key in user) alert(key); // name, age

// 아래 두 메서드에도 동일한 로직이 적용됩니다.
alert(Object.keys(user)); // name,age
alert(Object.values(user)); // John,30
```

여기서 설명이 끝나지는 않습니다. 객체 내에 존재하지 않는 키를 반환하려고 하면 `Object.keys()`는 이를 반환하지 않습니다. 당연한 이야기 아니냐 할 수도 있지만, 아래의 코드를 확인해주시기 바랍니다.

```js
let user = {};

user = new Proxy(user, {
  ownKeys(target) {
    return ["a", "b", "c"];
  },
});

alert(Object.keys(user)); // <빈 문자열>
```

이렇게 된 이유는 간단합니다. `Object.keys()`는 프로퍼티 디스크립터에 `enumerable`가 `true`인 프로퍼티만 꺼내서 보여줍니다만, 위의 코드에서 작성한 `ownKeys` 라는 친구들은 따로 디스크립터를 작성해주지 않았기 떄문에 보이질 않는 것이죠. `getOwnPropertyDescriptor`라는 트랩을 이용해서 우리가 만들어낸 key들의 디스크립터들을 달아주는 코드를 확인해 봅시다.

```js
let user = {};

user = new Proxy(user, {
  ownKeys(target) {
    // 프로퍼티 리스트를 얻을 때 딱 한 번 호출됩니다.
    return ["a", "b", "c"];
  },

  getOwnPropertyDescriptor(target, prop) {
    // 모든 프로퍼티를 대상으로 호출됩니다.
    return {
      enumerable: true,
      configurable: true,
      /* 이 외의 플래그도 반환할 수 있습니다. "value:..."도 가능합니다. */
    };
  },
});

alert(Object.keys(user)); // a, b, c
```

## `[[OwnPropertyKeys]]`를 인터셉트 하는 `handlers.ownKeys()`

뭔가 그게 뭔데 싶은 내부연산이 점점 등장하고 있습니다. 간단히 말하자면, 위에서도 언급했듯이, 객체의 프로퍼티들을 반환하는 연산입니다. 명세는 무척이나 간단합니다.

```js
new Proxy(target, {
  ownKeys(target) {},
});
```

### 매개변수

- target : 대상 객체

### 반환값

열거 가능한 객체의 프로퍼티(해당 프로퍼티의 디스크립터의 `enumerable`이 `true`)를 반환합니다.

### 불변 조건

아래의 불변 조건이 위반된다면, 프록시에서 `TypeError`가 발생합니다. 아래의 조건을 살펴보면 상당히 상식적인 내용임을 볼 수 있을 것입니다.

- `ownKeys`의 결과는 배열이어야 함
- 각 배열 요소의 타입은 문자열 또는 `Symbol`이어야 함
- 결과 리스트에는 대상 객체의 설정할 수 없는(configurable 하지 않은)프로퍼티들이 모두 포함되어야 함
- 대상 객체를 extend 할 수 없는 경우, 결과 목록에는 대상 객체의 자체 프로퍼티가 모두 포함되어야 하고, 다른 값이 **포함되면 안됨**

### 가로채는 작업

- `Object.getOwnPropertyNames()`
- `Object.getOwnPropertySymbols()`
- `Object.keys()`
- `Object.values()`
- `Object.entries()`
- `for ... in`
- `Reflect.ownKeys()`

## `[[GetOwnProperty]]`를 인터셉트 하는 `handlers.getOwnPropertyDescriptor()`

`[[GetOwnProperty]]` 연산은 해당 오브젝트의 프로퍼티의 디스크립터를 리턴하는 연산입니다. `handlers.getOwnPropertyDescriptor()` 트랩은 인터널 메소드와 유사하게 동작합니다. 코드와 명세로 살펴봅시다.

```js
new Proxy(target, {
  getOwnPropertyDescriptor(target, prop) {},
});
```

### 매개변수

- target : 대상 객체
- prop : 검색할 프로퍼티의 이름

### 반환값

객체 또는 `undefined`

### 불변 조건

불변 조건을 어기면 프록시에서 `TypeError`가 발생합니다. 이 역시 상당히 상식적인 범위입니다.

- `getOwnProperty()` 트랩은 객체 또는 `undefined`를 반환해야 합니다.
- 속성이 대상 객체의 구성될 수 없는(configurable 하지 않은) 속성으로 존재할 경우, 존재하지 않는 것으로 보고될 수 없습니다. (`Proxy`역시 원본 객체를 존중해 줘야함)
- 속성이 대상 객체의 속성으로 존재하지 않고, 대상 객체가 확장 가능하지 않을 경우, 존재하는 것으로 보고될 수 업습니다.(`Proxy`역시 원본 객체를 존중해 줘야함 2222)
- 속성이 대상 객체의 속성으로 존재하고, 대상 객체가 확장 가능하지 않을 경우, 존재하지 않는것으로 보고될 수 없습니다.(`Proxy`역시 원본 객체를 존중해 줘야함 3333)
- 해당 연산으로 나온 결과물은 `Object.defineProperty()`를 사용하여 대상 객체에 적용할 수 있어야 하며, 예외를 발생시키지 말아야 합니다. (사실 위에 있는것들을 한번에 포괄하는 내용임)

### 가로채는 작업

- `Object.getOwnPropertyDescriptor()`
- `Object.keys()`
- `Object.values()`
- `Object.entries()`
- `for ... in`
- `Reflect.getOwnPropertyDescriptor()`

# 특정 프로퍼티 접근 제한하기

JS에서는 오랜 기간동안, 'private property' 라는 개념이 딱히 존재하지 않았습니다. `_`로 시작하는 프로퍼티는 private인걸로 하고 쓰지 말기로 하자 정도로 합의보면서 살고 있었죠. 하지만, 말 그대로 사람들끼리 본 함의이기에, 언어 차원에서 막을 수는 없었습니다. `Proxy`의 여러 트랩들을 사용해서 언어 차원에서 이러한 행위를 막을 수 있도록 설정할 수 있습니다.

`_`로 시작하는 프로퍼티의 접근을 제한하기 위해서는 아래와 같은 트랩이 필요하겠네요.

- `get` : 프로퍼티를 읽는것을 제한하기 위함
- `set` : 프로퍼티를 쓰는것을 제한하기 위함
- `deleteProperty` : 프로퍼티를 삭제하는것을 제한하기 위함
- `ownKeys` : `for ... in` 이나, `Object.keys()`같은 프로퍼티 순환 메소드에서 제외되게 하기 위함

해당 트랩들을 활용해서 private property를 구현해낸 코드를 확인해 봅시다.

```js
let user = {
  name: "John",
  _password: "***",
};

user = new Proxy(user, {
  get(target, prop) {
    if (prop.startsWith("_")) {
      throw new Error("접근이 제한되어있습니다.");
    }
    let value = target[prop];
    return typeof value === "function" ? value.bind(target) : value; // (*)
  },
  set(target, prop, val) {
    // 프로퍼티 쓰기를 가로챕니다.
    if (prop.startsWith("_")) {
      throw new Error("접근이 제한되어있습니다.");
    } else {
      target[prop] = val;
      return true;
    }
  },
  deleteProperty(target, prop) {
    // 프로퍼티 삭제를 가로챕니다.
    if (prop.startsWith("_")) {
      throw new Error("접근이 제한되어있습니다.");
    } else {
      delete target[prop];
      return true;
    }
  },
  ownKeys(target) {
    // 프로퍼티 순회를 가로챕니다.
    return Object.keys(target).filter((key) => !key.startsWith("_"));
  },
});

// "get" 트랩이 _password 읽기를 막습니다.
try {
  alert(user._password); // Error: 접근이 제한되어있습니다.
} catch (e) {
  alert(e.message);
}

// "set" 트랩이 _password에 값을 쓰는것을 막습니다.
try {
  user._password = "test"; // Error: 접근이 제한되어있습니다.
} catch (e) {
  alert(e.message);
}

// "deleteProperty" 트랩이 _password 삭제를 막습니다.
try {
  delete user._password; // Error: 접근이 제한되어있습니다.
} catch (e) {
  alert(e.message);
}

// "ownKeys" 트랩이 순회 대상에서 _password를 제외시킵니다.
for (let key in user) alert(key); // name
```

여기에서 눈여겨봐야 할 부분은 `get` 트랩의 아래 부분입니다.

```js
return typeof value === "function" ? value.bind(target) : value; // (*)
```

이는 객체 내부의 private property를 접근하는 함수들을 올바르게 처리하기 위함입니다. 외부에서 직접적인 수정은 되지 않아도, 공개된 메소드를 통해서는 private property들이 접근이 되야 하기 때문에, `proxy`를 통한 접근이 아닌 `this`를 올바르게 binding 하여 이러한 문제를 방지할 필요가 있는 것이지요. private property를 접근하는 코드의 예시를 하단에 간단히 적어놓겠습니다.

```js
user = {
  // ...
  checkPassword(value) {
    // checkPassword(비밀번호 확인)는 _password를 읽을 수 있어야 합니다.
    return value === this._password;
  },
};
```

bind가 되지 않았다면, `Proxy`를 통해서 `_password` 를 접근하려고 하였기에 아마 에러를 하나 받았을 것입니다. 하지만, bind 를 통해서 이런 문제를 해결했네요.

## 잠깐! 이런 형태의 프록시는 사용해서는 안됩니다

이 형태는 잘 작동하긴 하지만, 골때리는 문제가 있을 수 있습니다. 이렇게 프록시를 통해서 특정한 동작을 강제하기 위해서 객체를 프록시로 감쌌는데, 안의 메소드 어디선가 프록시로 감싸지지 않은 객체가 리턴 된다면 무언가 꼬이게 되겠죠?

그리고 한 객체를 여러번 프록시에 감싸게 된다면, 각 프록시마다 객체에 가하는 '수정'의 형태가 다를 수 있기 떄문에, 그런것들 고려를 하나 하나 하는것이 정말로 중노동이 됩니다... 제발 private은 내장 private인 `#`로 시작하는 프로퍼티 쓰세요~ 설명을 위해서 일종의 비문같은 예시라는것을 짚고 넘어가셨으면 좋겠습니다 :)

## `[[Delete]]`를 가로채는 `handlers.deleteProperty()`

제목에서 참 직관적으로 드러나듯, 오브젝트의 프로퍼티를 `delete` 하는 것입니다. 구문도 그렇게 어렵지 않습니다.

```js
new Proxy(target, {
  deleteProperty(target, property) {},
});
```

특이사항이라면, **`this`가 처리기에 바인딩**된다 라는 것 인것 같습니다.

### 매개변수

- target : 대상 객체
- property : 삭제할 속성의 이름 또는 `Symbol`

### 반환 값

삭제가 성공했는지 아닌지에 대한 Boolean 값을 반환해야 합니다.

### 불변 조건

다음 조건이 위배되면, 프록시에서 `TypeError`가 발생 합니다.

- 프로퍼티가 해당 객체에서 configureable 하지 않은 자체 프로퍼티일 경우

### 가로채는 작업

- 속성 삭제 : `delete proxy[foo]`와 `delete proxy.foo`
- `Reflect.deleteProperty()`

# 프록시를 이용해서 `in` 연산자 마개조하기

python 등의 언어에서 `range`를 이용해서 특정한 수가 어떠한 범위 내에 있는지 판별하는 코드를 작성할 수 있는것을 아실겁니다. JS에서도 프록시를 이용하면 실제로 그렇게 동작하는 `in`연산자를 만들어 낼 수 있습니다. 구현도 매우 간단하기에 간단히 살펴봅시다.

```js
let range = {
  start: 1,
  end: 10,
};

range = new Proxy(range, {
  has(target, prop) {
    return prop >= target.start && prop <= target.end;
  },
});

alert(5 in range); // true
alert(50 in range); // false
```

`start`와 `end` 프로퍼티로 범위의 시작과 끝을 지정해주고 `in`연산자를 사용해서 이렇게 체크 할 수 있는 간단한 코드입니다.

## `[[HasProperty]]`를 가로채는 `handlers.has()`

본래 `in` 연산자는 특정한 프로퍼티가 어떠한 객체안에 있는지 검사하는 연산자 입니다. 지금은 일종의 신비한 연산자 오버라이딩을 했지만요 ㅎ... 위에서 봤듯, `handlers.has()`도 막 복잡하지 않습니다.

```js
new Proxy(target, {
  has(target, prop) {},
});
```

이 친구도 특이사항으로 `handlers.deleteProperty()` 처럼 **`this`가 처리기에 바인딩 됩니다.**

### 매개변수

- target : 대상 객체
- prop : 존재 여부를 확인할 프로퍼티의 이름 또는 `Symbol`

### 반환 값

존재 여부에 해당하는 Boolean 값을 반환해야 합니다.

### 불변 조건

다음 조건이 위배되면, 프록시에서 `TypeError`가 발생 합니다.

- `prop`이 `target`의 configurable 하지 않은 자체 속성으로 존재하는 경우 속성이 존재하지 않는것으로 반환될 수 없습니다.
- 반대로 `prop`이 `target`의 configurable 하지 않은 자체 속성으로 존재하지 않는 경우 속성이 존재하는 것으로 반환될 수 없습니다.

### 가로채는 작업

사실 앞쪽에서는 간략하게 농담따먹기 하듯 다뤘지만, 나름 파보면 심도있는 트랩입니다.

- 단순 프로퍼티 질의 : `foo in proxy`
- 상속된 프로퍼티 질의 : `foo in Object.create(proxy)`
- `with` 확인 : `with(proxy){(foo);}`
  - `eval()`과 함께 사용하지 말라는 친구인 그 `with` 입니다.
- `Reflect.has()`

# 더 나은 함수 데코레이터 만들기

JS에서 함수는 일급 객체이기 때문에, 함수를 다른 함수의 인자로 추가해서 함수를 감쌀 수 있다는 사실을 아실겁니다. 모르셨다면 맨 위에 써놓은 데코레이터에 관한 내용을 참고하면 좋을 것 같습니다. 함수를 이용해서 데코레이터를 작성한 간단한 예시를 소개해 보겠습니다.

```js
function delay(f, ms) {
  // 지정한 시간이 흐른 다음에 f 호출을 전달해주는 래퍼 함수를 반환합니다.
  return function () {
    // (*)
    setTimeout(() => f.apply(this, arguments), ms);
  };
}

function sayHi(user) {
  alert(`Hello, ${user}!`);
}

// 래퍼 함수로 감싼 다음에 sayHi를 호출하면 3초 후 함수가 호출됩니다.
sayHi = delay(sayHi, 3000);

sayHi("John"); // Hello, John! (3초 후)
```

대체로 문제될 것 없는 코드이긴 합니다만, 몇가지 약점이 있습니다. 사실 그렇게 치명적인 약점은 아니긴 한데, 래퍼 함수로 감싸고 난 이후에는 기존 함수의 프로퍼티(`name`, `length`)등이 사라지는 문제가 있습니다. 이를 해결하기 위해 `Proxy`를 사용할 수 있습니다.

```js
function delay(f, ms) {
  return new Proxy(f, {
    apply(target, thisArg, args) {
      setTimeout(() => target.apply(thisArg, args), ms);
    },
  });
}

function sayHi(user) {
  alert(`Hello, ${user}!`);
}

sayHi = delay(sayHi, 3000);

alert(sayHi.length); // 1 (*) 프락시는 "get length" 연산까지 타깃 객체에 전달해줍니다.

sayHi("John"); // Hello, John! (3초 후)
```

기존의 함수 기반의 포워딩과 데코레이터는 호출만 넘겨주지만, Proxy는 원본 함수에 가하는 모든 연산을 포워딩을 하기 떄문에 `length`와 같은 것도 전달될 수 있습니다.

## 함수 호출을 가로채는 `handlers.apply()`

구문은 간단합니다.

```js
var p = new Proxy(target, {
  apply: function (target, thisArg, argumentsList) {},
});
```

이 친구도 특이사항으로 `handlers.deleteProperty()` 처럼 **`this`가 처리기에 바인딩 됩니다.**

### 매개변수

- target : 호출할 수 있는(callable) 대상 객체
- thisArg : 호출에 대한 this 인수
- argumentsList : 호출에 대한 인수 목록

### 반환 값

JS 세계에서 함수는 어떠한 값이든 반환할 수 있듯, `apply()` 트랩 역시 어떠한 값이건 반환될 수 있습니다.

### 불변 조건

다음 조건이 위배되면, 프록시에서 `TypeError`가 발생 합니다.

- 대상 객체는 callable 해야 합니다.

너무나도 단순한 것이라서 별 코멘트를 더 할게 없네요.

### 가로채는 작업

- `proxy(...args)`
- `Function.prototype.apply()`와 `Function.prototype.call()`
- `Reflect.apply()`

대략적으로 함수를 호출하는 그런 것들에 대해서 가로챈다고 생각하시면 좋을것 같습니다.

# 마무리 하며

정말로 긴 글이었습니다. `Proxy`의 **모든** 명세를 다루지는 못했지만, 나름 흥미로워 보이는 예제들과 함께 `Proxy`의 여러 API들을 살펴 볼 수 있는 글을 적을 수 있었던것 같습니다. 단순한 개발만이 아닌 JS 기초를 다지기 위한 시간들을 가지면서 배웠던 이것저것들을 글로써 정리를 해봅니다.

공부에 공부를 거듭할수록, JS가 상당히 뭐가 많다는것을 느끼게 됩니다. FE 개발의 짬이 쌓인 분들의 블로그를 보면, V8 엔진의 코드까지 접근할 정도로 JS를 딥하게 들어가시는걸 보게 됩니다. 지금 당장은 그 레벨에 들어갈 수 없지만, 그러한 레벨까지 갈 수 있도록, 꾸준히 노력하고, 공부하고 기록하는 삶을 이어나가겠습니다.
