---
layout: post
title: 기술면접을 계기로 다시 본 클로저
subtitle: 기초를 아는 신입이 되어야겠지?
category: javascript
image: /images/thumbnails/JS.png
---

# 도입

2025년 1월 초. 바야흐로, 제가 모 회사의 체험형 인턴 면접을 보았을 때 입니다. 면접이라는 것이 1시간 남짓한 시간에 지원자를 평가해야 하는데, 기업 입장에서는 가장 해당하는 자리에 적합한 사람을 뽑으려고 할 것 입니다.

그 면접 시간때에 클로저에 대한 정의부터 시작해서, 클래스도 클로저인가 라는 질문까지 받게 되었고, 그때의 저는 제대로 답을 하지 못했습니다. 프론트엔드 개발자를 지망하면서, 본인이 사용하는 기술의 본질에 대해서 모른다는 것은 그닥 좋은 일은 아니었고, 뜬구름 잡는 지식 탐독보다, 실제로 기록을 하면서 지식을 정리하는 쪽이 도움이 되리라 생각하였습니다.

이번 글에서는 우선 클로저의 개념적 설명을 해보겠습니다. 클로저와 클래스를 비교하는 여러 주제의 글들도 대략적으로 준비가 되어 있지만, 일반적인 컴퓨터공학 학부 과정에서 다루는 클래스와 OOP와 다르게, 함수형 프로그래밍 언어 쪽에서 많이 쓰이는 클로저의 개념은, 조금 더 낯서리라 생각이 들기도 하거든요

그리고 기술 면접때의 저의 이야기와 함께, 다시 면접의 기회가 왔을 때, 적어도 클로저 때문에 발목 잡히는 일 없게끔 정리하는 기회로 삼으려 합니다.

# 배경지식

## 렉시컬 스코프

첫째는 렉시컬 스코프 입니다. JS는 함수를 어디서 호출 했는지가 아니라, **어디서 정의** 했는지에 따라 상위 스코프를 결정하고, 이를 렉시컬 스코프(정적 스코프) 라고 한다.

이해를 점검하기 위해서 좋은 예제를 가져왔습니다. (출처 : <https://ko.javascript.info/closure>)

```js
function makeWorker() {
  let name = "Pete";

  return function () {
    alert(name);
  };
}

let name = "John";

// 함수를 만듭니다.
let work = makeWorker();

// 함수를 호출합니다.
work(); // 무엇이 나올까요?
```

`makeWorker`의 리턴 함수에서 사용되는 `name`은 **정의 되었을 때** 결정 된 것이기 때문에, “Pete”가 출력 될 것 입니다.

또 하나의 간단한 예제를 풀어봅시다. (출처 : 모던 자바스크립트 Deep Dive)

```js
const x = 1;

function foo() {
  const x = 10;
  bar();
}

function bar() {
  console.log(x);
}

foo(); // ?
bar(); // ?
```

**함수가 정의 되었을 때 결정** 이라는 핵심 키워드만 이해하고 있다면, `foo`, `bar` 모두 1이 나올 것이라는 것을 간단히 알 수 있습니다. `bar` 는 전역 함수로 정의되어 있고, 이 때의 x는 전역 변수 x이기 때문이죠. foo 함수 안에 있는 x는 bar 함수 내부에 아무런 영향을 끼치지 못합니다.

## 실행 컨텍스트 스택이 관리되는 방법

이전 두 글에서 ‘클로저를 이해하기 위해’ 라는 말로, 어찌보면 상당히 로우 레벨인 요소들을 다루었습니다. 참 많은 내용들이 중요하지만, 굳이 하나를 꼽아 리마인드를 하자면, ‘실행 컨텍스트 스택에 있는 실행 컨텍스트는 함수의 실행이 완료되면 사라지지만, 실제로 변수 등의 정보를 저장하고 있는 환경 레코드는 그와 별개로 관리된다’ 가 되어야 한다 생각합니다. 또한 상위 스코프에 대한 정보는 함수 객체 자체의 `[[Environment]]` 라는 내부 속성에 의해 관리된다 라는 점도 기억했으면 좋겠습니다.

이것이 클로저라고 부르는 함수들이 동작할 수 있는 기본 원리가 되기 때문이지요.

# 클로저

## 클로저의 정의 + 나의 제대로 답하지 못했던 옛 질문

클로저의 정의는 “함수와 그 함수가 선언된 렉시컬 환경과의 조합” 이라고 MDN에 나와 있기는 합니다. 옛 성현들의 가르침을 달달 외우는 선비마냥 외우는 것도 좋지만, 짧은 경험이지만 제대로 알지 못하고 다만 외우기만 하면 기술면접장에서 제대로 이해 못하고 있었다라는 사실이 발각되었던 것 같습니다.

책에서 가져온 예제를 간단히 하나 들고오겠습니다.

```jsx
const x = 1;

function outer() {
  const x = 10;
  const inner = function () {
    console.log(x);
  };
  return inner;
}

const innerFunc = outer();
innerFunc();
```

다른 것 다 제쳐두고 ‘함수의 렉시컬 스코프’ 라는 원칙만을 생각했을 때, `inner` 안에 할당된 함수는 `outer` 함수 내에서 만들어 졌고, 그러면 `inner`에 할당된 함수의 `x`는 가장 가까운 스코프의 `x`인 10 이라는 값이 할당될 것 입니다.

그런데 `outer`의 실행이 끝났는데, `outer`의 `x`가 보존된다… 라는 것이 통상적으로 학부에서 다루는 C나 Java의 문법만 다루었던 저에게는 ‘내재화’가 되지 못했나 봅니다. ‘책’ 에서 다루는 정의는

> 외부 함수보다 중첩 함수가 더 오래 유지되는 경우 중첩 함수는 이미 생명 주기가 종료한 외부 함수의 변수를 참조할 수 있다. 이러한 중첩 함수를 클로저 라고 한다.

라고 하고, [javascript.info](http://javascript.info) 에서는

> 외부 변수를 기억하고 이 외부 변수에 접근할 수 있는 함수를 의미한다. JS에서는 `new Function` 이라는 예외를 빼고는 모든 자바스크립트 함수가 클로저이다.

라고 합니다. 결국 핵심은 ‘외부 변수를 참조’ 라는 것이고, 이것이 아니라면, 클로저라는 말을 쓰기 부적합 하다는 것입니다.

정의를 이렇게까지 강조하는 이유라면, 이전의 기술면접에서 클로저의 정의 다음으로 나온 질문이, ‘클래스도 클로저인가’ 였고, ‘그렇지 않을까요’ 라고 답했던 제가 **‘틀린’** 답변을 했다 라는것이 너무나도 안타까워서 이렇게 다시 정리를 해보고 싶었습니다…

덧붙여서, 클래스의 private property는 이전 포스팅에서 실행 컨텍스트 관련 포스팅을 작성하기 위해서 확인해 봤었을 때, ES2022에서 실행 컨텍스트에서 추가된 `PrivateEnvironment`이고, 이는 외부 변수의 참조가 아니고, 제 변수 참조니까 클로저와는 연관이 있다고 말하기 어렵다([이 velog글](https://velog.io/@shroad1802/Execution-Context-19pf2k6t?t) 의 ECMAScript 버전 별 변경점/ES2022)라고 답을 했으면 더 좋았을것 같네요

## 도식과 함께 알아보는 클로저 함수의 내부 동작

단순히, ‘JS라서 달라!’ 하고 마치기에는, 클로저의 실제 동작 원리를 보지 않고 넘어가는것은 별로 좋지 않다고 생각이 듭니다. 위의 예제에서, outer 함수가 평가되어 함수 객체 내부의 `[[Environment]]` 내부 속성에 참조가 기록된 상황을 그려보았습니다.

![outer 함수가 평가 되었을 때의 도식](/images/javascript/closure-basics/closure-1.png)

그리고 나서 중첩 함수 inner가 평가 되는 시점의 도식 역시 그려보겠습니다.

![inner 함수가 평가 되었을 때의 도식](/images/javascript/closure-basics/closure-2.png)

여기에서 outer 함수의 실행까지 종료되면, 실행 컨텍스트에서 outer의 실행 컨텍스트는 제거될 것입니다. 여기서, 환경 레코드는 아직 그대로 있음에 유의를 해 주세요.

![outer 실행 종료 후](/images/javascript/closure-basics/closure-3.png)

innerFunc가 실행되면, 실행 컨텍스트 스택에 inner 함수의 실행 컨텍스트가 추가 될 것이고, inner 함수의 렉시컬 환경, 그리고 환경 레코드도 이 코드의 렉시컬 환경에 기록될 것 입니다.

![innerFunc 실행](/images/javascript/closure-basics/closure-4.png)

inner 함수는 이미 실행이 종료되었지만, outer 함수의 렉시컬 환경을 여전히 참조하고(특정 자료에서는 ‘기억하고’ 라는 말을 쓰기도 합니다) 있습니다. outer의 렉시컬 환경이 도달할 수 없다면, 가비지 컬렉터에 의해서 지워지겠지만, inner 함수를 통해 접근할 수 있으므로, 연관 렉시컬 환경도 메모리에 살아있게 되는 것입니다.

# 클로저의 활용

## 상태의 은닉

클로저를 사용하는 이유라고 했을 때, 가장 먼저 떠오르는 것은, 상태의 은닉 입니다.

```jsx
const x = 1;

function outer() {
  const x = 10;
  const inner = function () {
    console.log(x);
  };
  return inner;
}

const innerFunc = outer();
innerFunc();
```

이 예제에서도, inner 함수가 가지고 있는 outer의 지역변수 x에 대해서는 inner를 통하지 않고는 접근할 수 없습니다. 단순히 로깅 하는 것만으로는 느낌이 오지 않으니, 숫자를 증감 시킬 수 있는 카운터 예제를 가져오겠습니다.

```jsx
const counter = (function(){
  let num = 0;

  return {
    // 여기에 num이 있다면 프로퍼티로 접근 가능했을 것
    increase() {
      return ++num;
    },
    decrease() {
      return num > 0 ? : --num : 0;
    }
  };
}());

console.log(counter.increase()); // 1
console.log(counter.increase()); // 2

console.log(counter.decrease()); // 1
console.log(counter.decrease()); // 0
```

`counter.num` 등의 방법으로 num에 접근할 수 없고, 오로지 increase, decrease 라는 방법만을 통해서, num 변수를 조작할 수 있습니다. 상태를 안전하게 변경할 수 있는, OOP언어의 private 비슷한 형태로 사용할 수 있지요. (물론 지금은 JS에서도 #로 시작하는 class private property를 통해 더욱 간단하게 할 수 있습니다)

이러한 함수를 활용하여 정보를 은닉하고, 필요한 부분만 뽑아 쓸 수 있다는 점이 클로저를 사용하는 이유중 하나가 되겠습니다.

## React Hook의 근본

[이 영상](https://www.youtube.com/watch?v=KJP1E-Y-xyo)에서는 React hook인 useState와 useEffect를 클로저를 이용해서 재현한 예제를 개념 설명 다 하면서 30분 이내에 컷을 해버렸습니다. 30분 만에 hook과 useState 만들어 보기라는 제목이 단순한 미끼용 문구가 아닐 정도로. 30분 안에 클로저 개념 설명 + 최소 구현 + 농담까지 다 하는 영상이니, 시간이 여유가 나신다면 꼭 봤으면 좋겠습니다.

React에 대한 설명까지 하면 글이 너무 길어지기에, React에서의 hook에 대한 설명 보다는 클로저에 초점을 더 맞춰서 설명을 해보겠습니다.

JS 클래스는 프로퍼티 라고 하는 상말 그대로 ‘상태’를 가지고 있을 수 있지만, 함수라는 것은 기본적으로 상태라는 것을 가지고 있을 수 없습니다. ‘상태’를 가지고 있는 컴포넌트를 ‘함수’의 문법으로 담아낼 수 있었던 데에는 `useState`와 같은 hook의 등장으로 가능했다고 할 수 있겠습니다.

상태 하나만을 담을 수 있는 useState를 아까 학습한 클로저의 개념을 통해서 뚝딱 하나 해보겠습니다

```js
const React = (function () {
  let _val;
  function useState(initVal) {
    const state = _val || initVal; // 초기값이 있으면 초기값 사용. 아니면 _val 사용
    const setState = (newVal) => {
      _val = newVal;
    };
    return [state, setState];
  }
  return { useState };
})();
```

`_val`은 useState의 스코프 밖에 있기 때문에, 익명 IIFE의 실행이 종료된 이후에도, `_val`에 대한 참조가 계속 남아있게 되고, useState라는 함수를 계속 불러도, 이전에 저장했던 값이 남아있게 됩니다.

하지만 너무나도 당연한 이야기지만, React hook 을 쓰는 환경에서 ‘단 하나’의 hook만 쓰는 경우는 상당히 찾아보기 어려울 것 입니다. 여러개의 상태를 달아놔야 하고, 가장 직관적인 해결책을 react가 채택했습니다. 바로 배열 이지요.

배열의 인덱스도 함수 형태의 컴포넌트가 여러번 호출되어도 매 호출마다 일관적으로 유지되어야 합니다.

```js
const React = (function () {
  let hooks = []; // hook들에 대한 정보를 달아둘 배열
  let idx;
  function useState(initVal) {
    const state = hooks[idx] || initVal; // 초기값이 있으면 초기값 사용. 아니면 _val 사용
    const _idx = idx; // [state, setState]가 처음 리턴되었을 때의, 해당 state를 위한 idx를 고정!
    const setState = (newVal) => {
      hooks[_idx] = newVal; // setter는 전역(?) idx가 바뀌어도, 기존 값을 들고 있어야 함
    };
    idx++;
    return [state, setState];
  }
  function resetIndex() {
    // 계속 idx가 증가하면 예전 값을 못찾을테니
    idx = 0; // 다시 '렌더링' 되면 idx를 초기화 할 것
  } // 자세한 사항은 향후 react hook을 제대로 다루는 딥 다이브 글에서 다루겠습니다!
  return { useState, resetIndex };
})();
```

이렇게 useState훅의 간단한 구현을 해볼 수 있습니다.

최대한 단순하게 설명을 한다고 했는데, 너무 덜어냈지 않는가 하는 생각이 계속 듭니다만… 유튜브 링크 하나만 던져두고, ‘이거 보세요’ 하는것 보다는 이게 그래도 조금 더 낫지 않을까 하여 최대한 클로저의 개념이 들어간 부분만 발췌하여 영상을 보기 위한 ‘밑밥’이라도 정리해보았습니다. 최대한 클로저에 대한 초점을 유지하면서, React 설명이 장황해지지 않게 하기 위해 요약하고 요약하다 보니, 뭔가 이도저도 아닌게 나온것 같습니다만… React hook에서도 클로저를 활용해서 상태를 보전하는구나 하는 느낌만 이해해 주세요.

제대로 된 설명은

- 유튜브
  - <https://www.youtube.com/watch?v=KJP1E-Y-xyo> (30분만에 useState및 hook 만들기 JSConf 영상)
  - <https://www.youtube.com/watch?v=1VVfMVQabx0&list=PLxRVWC-K96b0ktvhd16l3xA6gncuGP7gJ&index=3> (React 딥 다이브. #3 훅이 어떻게 동작하는가)

를 참조해주시고, 이 영상들을 포함한 여러 레퍼런스들을 모아서, ‘면접에서 대답할 수 있는 react 기초 지식’ 시리즈를 정리하는대로 이 글에 링크하도록 하겠습니다.

# 클로저에 대한 유의사항

이 역시 면접때 제가 받았던 질문 이었습니다. 다시 그 면접장에 돌아갈 수 있다면 할 수 있는 대답과, 해설을 한번 적어볼까 합니다.

## 클로저에서 ‘호출 형태’나 ‘호출 위치’는 의미가 없다.

- ‘렉시컬 스코프’ 설명하면서 입이 마르고 닳도록 이야기 했던것 같습니다.
- JS의 렉시컬 스코프는 ‘함수의 선언’시에 결정된다 라는 것을 강조하면 되지 않을까 싶습니다.

> 클로저의 스코프는 함수가 선언된 위치에서 결정되며, 호출 방식과 위치에 영향을 받지 않습니다.

## 외부 함수의 생명 주기가 중첩된 내부 함수의 생명 주기보다 짧아야 한다.

- 정확하게는 ‘내부 함수에서 참조하는 외부 변수’에 대한 것이긴 합니다.

```jsx
for (var i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i); // 3 3 3
  }, 1000);
}
```

너무나도 유명한 예제입니다. 왜 3 3 3이 나오느냐 하는 이유도 너무나도 유명합니다. var은 블록 스코프가 아니고, 함수 스코프만 있기 때문에, 여기서 i는 사실상 전역 변수이고, 이 콜백 함수 모두 3이 되어버린 전역 변수 i를 참조하고 있지요.

같은 원리로 아래의 코드도 3 3 3 이 출력됩니다

```jsx
let i;
for (i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i); // 3 3 3
  }, 1000);
}
```

0 1 2 를 출력하게 하려면

```jsx
for (let i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i); // 3 3 3
  }, 1000);
}
```

이런 식으로 i를 블록 스코프에 가두면 됩니다. 여기서 i는 블록 밖에 있는것 같지만, 블록 안에 있는 것으로 취급되며, 렉시컬 환경 생성도 매 블록이 반복문을 통해 실행될 떄 마다 새롭게 이루어 집니다. 기억이 잘 안난다면, 이 이전 글(링크하기)을 한번 읽어보세요

### 고차함수를 통해 문제 해결하기

제아무리 var 같은 것이라 하더라도, 함수 안으로 들어가면, 별도의 스코프를 가지게 된다라는 점을 활용해서, 고차함수 패턴을 이용해서 문제를 해결할 수도 있다(’책’에 나온 내용)

```jsx
const funcs = Array.from(new Array(3), (_, i) => () => i);
funcs.forEach(f => console.log(f()); // 0 1 2
```

매개변수 ‘i’를 넘겨서 그 값을 반환하게 한 함수 안에 들어있는 ‘i’라는 식별자는 Array.from의 두번째 인자로 들어가서 인덱스 값이 넘겨져서 ‘고정’되어 버리기에 클로저 패턴을 통해서 문제를 해결한 것이라고 볼 수 있을 것 입니다. (다만 함수형 신앙심을 잃은 지금은, 뭔가 ‘이거다!’ 하지는 않는것이 저자 본인의 생각이다)

## 같은 함수로 만들어진 클로저도 독립된 메모리 공간을 가진다

https://ko.javascript.info/task/closure-variable-access

어찌보면 당연한 이야기입니다. 힙 메모리 공간 어딘가에 참조를 저장한다는 점에서, C언어의 `malloc`과 비슷한 느낌이라는 느낌이 없잖아 들잖아요.

```jsx
function makeCounter() {
  let count = 0;

  return function () {
    return count++;
  };
}

let counter = makeCounter();
let counter2 = makeCounter();

alert(counter()); // 0
alert(counter()); // 1

alert(counter2()); // ?
alert(counter2()); // ?
```

`counter2` 는 `counter`와 독립적인 메모리 공간을 가지므로 0 1 이 각각 출력 되겠습니다.

# 마무리하며

면접에서 제대로 답하지 못한 실수를 반복하지 않기 위해서 여기까지 참 많은 자료들을 찾아보고, 토끼굴에도 빠져가면서, 지금까지 왔습니다. 다음 글은, 면접에서의 또 다른 질문이었던, ‘클로저와 클래스의 차이점’ 에 대해서 여러 관점으로 본 재미있는 글을 바탕으로 최대한 재미있는 글을 또 적어올까 합니다.

물론 react hook 섹션에서 언급했던 대로, 조만간에 react deep dive 영상들을 보고, 관련 기술 면접에도 대비할 수 있는 글들을 적기도 해야지요…

끝까지 읽어주셔서 감사합니다! 더 좋은 또 다른 글로 찾아뵐 때 까지, 안녕!

# 참조한 글

직접 출처 링크를 달지 못했지만, 내용을 적는데에 참고가 되었던 글의 목록입니다

- [https://velog.io/@minh0518/면접관-앞에서-클로저-closure를-왜-사용하는지와-주의사항에-대해-말해보자](https://velog.io/@minh0518/%EB%A9%B4%EC%A0%91%EA%B4%80-%EC%95%9E%EC%97%90%EC%84%9C-%ED%81%B4%EB%A1%9C%EC%A0%80-closure%EB%A5%BC-%EC%99%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EC%A7%80%EC%99%80-%EC%A3%BC%EC%9D%98%EC%82%AC%ED%95%AD%EC%97%90-%EB%8C%80%ED%95%B4-%EB%A7%90%ED%95%B4%EB%B3%B4%EC%9E%90)
- [React는 Hooks를 배열로 관리하고 있다](https://pozafly.github.io/react/react-is-managing-hooks-as-an-array/?t)
- [React Fiber 아키텍처 딥 다이브](https://velog.io/@alsgud8311/React-Fiber-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%EB%94%A5%EB%8B%A4%EC%9D%B4%EB%B8%8C?t)
