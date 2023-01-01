---
layout: post
title: 함수형 프로그래밍에서의 transducer
subtitle: 처음 보는 개념이라서 낯선 transducer를 친절한 말로 풀어봅시다
category: javascript
image: /images/thumbnails/functional.png
---

# 들어가며

요즘 저는 함수형 프로그래밍에 많은 관심을 가지고 공부하고 있었습니다. 멀티 패러다임 언어(좋게 말하면 그거고, 안좋게 말하면 줏대가 없는)언어인 JS를 메인으로 사용할 프론트엔드 개발자로서의 역량을 끌어올리기 위해서, 그리고 결정적으로 "재밌어 보여서" 입니다. 요즈음은 많은 언어에서 함수형 패러다임을 사용할 수 있는 여러 도구들을 제공해주고, JS도 `map`, `reduce`, `filter` 등을 기본 자료구조의 메소드로도 제공해줘서 함수형 프로그래밍의 여러 개념이 낯설지는 않은것 같습니다만... 이 `transducer`라는 것은 용어 자체도 처음 들어보는 것이기도 해서, 혹시나 해서 한국어로 "함수형 transducer" 같은 키워드로 찾아봤지만, 역시나 제가 원하는 정보를 빠르게 찾기가 어려웠습니다. 다행히도, 유튜브에 [멋진 강의](https://www.youtube.com/watch?v=SJjOp0X_MVA)가 있어서 개념을 익힐 수 있었습니다. 이제 제가 할 일은, 이 멋진 지식을 한국어 사용자들이 조금 더 찾기 편하도록 글을 하나 남기는 것이겠죠?

## 이 글의 대상

이 글은 함수합성, `map`, `filter`, `reduce`에 대한 이해를 가지고 있으며, `ramdaJS`등 함수형 라이브러리나 함수형 프로그래밍에 대한 기초적인 이해가 있는 사람을 대상으로 합니다. 글의 분량조절을 위해서이니 양해해 주세요 :)

# 함수 합성을 통해 메모리 아끼기

매우 간단한 코드 하나를 살펴봅시다.

```js
const nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 변형(transformation)
const add1 = (x) => x + 1;
const doubleIt = (x) => x * 2;
const add = (x, y) => x + y;

// 술어(predicates)
const isEven = (x) => x % 2 === 0;
const isOdd = (x) => !isEven(x);

const mapResult1 = nums
  .map(add1) // [2,3,4,5,6,7,8,9,10,11]
  .map(add1) // [3,4,5,6,7,8,9,10,11,12]
  .map(add1) // [4,5,6,7,8,9,10,11,12,13]
  .map(add1) // [5,6,7,8,9,10,11,12,13,14]
  .map(add1) // [6,7,8,9,10,11,12,13,14,15]
  .map(doubleIt); // [12,14,16,18,20,22,24,26,28,30]
```

주석을 달아놓은것에서 볼 수 있듯, `mapResult1`을 만들기 위해서 최종 단계에서는 쓰이지 않을 배열들을 계속해서 생성합니다. 이런 중간단계 배열들을 생성하지 않기 위해서는 함수 합성이라는 방법을 사용할 수 있지요.

```js
const transform = compose(doubleIt, add1, add1, add1, add1, add1);
const mapResult2 = nums.map(transform);
```

이렇게 함수 합성을 이용한다면, 중간 단계의 쓰지도 않을 배열들을 생성하는 일 없이, 메모리와 시간을 함께 아낄 수 있습니다.

`map` 뿐만 아니라 `filter`에서도 비슷한 결과를 보여줄 수 있습니다.

```js
const filterResult1 = nums.filter(isEven).filter(isOdd);

const excludeAll = (x) => isEven(x) && isOdd(x);
const filterResult2 = nums.filter(excludeAll);
```

`filterResult1`은 최종 결과를 얻기 위해서 중간 단계에서만 쓰일 배열을 생성하지만, `filterResult2`는 그렇지 않지요. 이런식으로 여러 메소드들을 통해서 리스트를 정제하고 가공한 이후에는 `reduce`를 통해서 값을 하나로 축약하는 경우가 많음을 함수형 프로그래밍에 관한 기초적 경험이 있는 여러분이라면 잘 알고 있을겁니다. `map`과 `filter`를 둘다 다 쓴 배열에 `reduce`로 값을 축약하는 아래의 예시를 한번 봅시다.

```js
const result1 = nums // -> [1,2,3,4,5,6,7,8,9,10]
  .map(add1) //         -> [2,3,4,5,6,7,8,9,10,11]
  .filter(isEven) //    -> [2,4,6,8,10]
  .reduce(add, 0); //   -> 30
```

이 코드도, 최종 결과를 얻기 위해서, 최종적으로 쓰이지 않을 배열들을 만들고 있습니다. 그럼 이것도 위의 `map`이나 `filter`예시처럼 함수 합성을 통해서 어찌 할 수 있지 않을까 하면서 아래와 같은 코드로 한번 시도해 봅시다.

```js
const transformFilterReduce1 = compose(add, isEven, add1);
const result2 = nums.reduce(transformFilterReduce1);
```

뭔가 그럴듯(?)해 보이는 이 코드를 한번 실행해 보면 `NaN`이라는 끝내주는 값이 우리를 맞아줍니다. 우리는 `add1`으로 `map`을 하고, `isEven`으로 `filter`를 하기 원했지만, `reduce`는 그걸 알 턱이 없죠. 그러니까 이런 느낌으로 작성을 해야 할거라는 뭔가 느낌이 들 수 있겠습니다.

```js
const transformFilterReduce2 = compose(reduce(add), filter(isEven), map(add1));
transformFilterReduce2(nums);
```

그러니까 해당 보조함수가 쓰이는 **맥락(context)**을 알 필요가 있다 라는 결론이 되겠지요. 그렇게 맥락을 알아내서 제대로 된 reducer가 되야 할 것이니까요!!

# `map`과 `filter`를 `reduce`의 reducer로 해석해보자

리스트에서 `map`을 실행한것과 같은 효과를 내기 위해서 `reduce`를 하는 함수를 아래와 같이 작성할 수 있을 것입니다.

> `concat(acc,f(val))`이라는 구문은 `acc`에 `f(val)`을 **덧댄다** 라는 뜻으로 이해하면 될 것 같습니다.

```js
function mapWithReduce1(xs, f) {
  return xs.reduce((acc, val) => {
    return concat(acc, f(val));
  }, []);
}

const mapWithReduceResult1 = mapWithReduce1(nums, add1);
```

현재 이 함수는 잘 작동합니다만, 약간의 수정을 해봅시다. 이 함수는 `xs`의 `reduce`를 직접 호출해서 작업을 수행하지만, 약간만 수정해서 `reduce`를 직접 호출하는것이 아닌, `reduce`에 넣을 **reducer**를 반환하는 함수로요

```js
function mapWithReduce2(f) {
  return (acc, val) => {
    return concat(acc, f(val));
  };
}

const mapWithReduceResult2 = nums.reduce(mapWithReduce2(add1), []);
```

`reduce`를 호출하는 부분, `acc`값만 호출하는 쪽으로 위임시켰기 때문에, 위의 코드를 이해했다면 이 코드도 그렇게 어렵지는 않겠습니다.

`filter`의 경우도 한번 살펴봅시다.

```js
function filterWithReduce1(xs, p) {
  return xs.reduce((acc, val) => {
    return p(val) ? concat(acc, val) : acc;
  }, []);
}

const filterWithReduceResult1 = filterWithReduce1(nums, isOdd);
```

이 역시 `reduce`에 들어가는 **reducer**를 반환하는 함수로 간단히 수정해봅시다.

```js
function filterWithReduce2(p) {
  return (acc, val) => {
    return p(val) ? concat(acc, val) : acc;
  };
}

const filterWithReduceResult2 = nums.reduce(filterWithReduce2(isOdd), []);
```

찬찬히 이들을 다시 읽어보면 `map`과 `filter`모두, `concat`라는 공통분모를 가집니다. 이 공통분모를 통해서, `map`과 `filter`의 역할을 하는 reducer를 성공적으로 합성 할 수 있지 않을까요?

함수 합성을 위해서는 최대한 일반화가 되어야 하니, 아까 만들었던 함수에서 `concat`도 분리해봅시다. reducer에서 쓰이는 함수(function)이라는 뜻에서 인자 이름을 rf로 지어보겠습니다.

우선 `map`

```js
function mapping(f) {
  return function (rf) {
    // 이 함수는 두개의 인자를 받아서 하나로 축약합니다.
    return (acc, val) => {
      return rf(acc, f(val)); // <-- rf는 'concat'을 대체했습니다.
    };
  };
}

const mapWithRf = mapping(add1);
const mappingResult = nums.reduce(mapWithRf(concat), []);
```

그 다음 `filter`

```js
function filtering(p) {
  return function (rf) {
    // 이 함수는 두개의 인자를 받아서 하나로 축약합니다.
    return (acc, val) => {
      return p(val) ? rf(acc, val) : acc; // <-- rf는 'concat'을 대체했습니다.
    };
  };
}

const filterWithRf = filtering(isOdd);
const filteringResult = nums.reduce(filterWithRf(concat), []);
```

자 이제 **주목** 해주세요. 중요한 내용입니다.

- `rf`는 인자 두개를 받아서 하나로 만드는 함수입니다.
- `mapping(fn)`은 `rf`를 인자로 받을 수 있는 함수를 리턴합니다.
- `filtering(p)` 역시 `rf`를 인자로 받을 수 있는 함수를 리턴합니다.
- `mapping(fn)(rf)`는 `rf`처럼 인자 두개를 받아서 하나로 만드는 함수를 리턴합니다.
- `filtering(fn)(rf)` 역시 `rf`처럼 인자 두개를 받아서 하나로 만드는 함수를 리턴합니다.

이것을 곰곰히 씹어보면, `mapping(fn)(rf)`는 rf로 쓰일 수 있는 함수를 리턴한다는 것이고, 이는 `filtering(fn)`의 인자로 들어갈 수 있고, 반대 또한 그렇습니다(vice-versa)

이것이 transducer의 개념입니다. `rf`를 인수로 받고, `rf`를 리턴하여서, 서로 성공적으로 합성될 수 있도록 해주는 구조이지요.

아까의 목표를 혹시 기억하시나요? 이런것이 우리의 목표였습니다.

```js
const transformFilterReduce2 = compose(reduce(add), filter(isEven), map(add1));
transformFilterReduce2(nums);
```

이제 아까 학습한 transducer를 통해서 이것을 온전히 수행하는 로직을 만들어 보겠습니다.

```js
const transformFilterReduce2 = compose(mapping(add1), filtering(isEven));
const transformFilterReduceResult1 = nums.reduce(
  transformFilterReduce2(concat), // <--  concat은 rf고(두개의 값을 하나로) add 또한 그렇습니다.
  [] // concat은 배열을 반환하고 -- 이것은 배열을 위한 초기값(acc) 입니다
);
```

성공적이군요! 이제 이를 일반화 하면(그리고 함수형 라이브러리에서 사용하는 형태로 하면) 다음과 같습니다.

```js
function transduce(xf, rf, init, xs) {
  // 내부적으로 자료 구조에 reduce를 호출합니다. (추상화)
  // 합성된 transformation에 rf를 전달합니다.
  // init : 초기값을 전달합니다.
  return xs.reduce(xf(rf), init);
}
```

실제 현장에서 사용되는 라이브러리도 아래와 같은 구조를 지니고 있습니다. 물론 라이브러리들은 여기에서 몇몇 최적화를 더 거쳤긴 하지만, 전체적인 틀은 아래와 같다는 점을 코드를 찬찬히 읽어보면 알 수 있을 것입니다.

# RamdaJS로 실제 예 살펴보기

이때까지의 코드는 개념 설명을 위해서 직접 한땀한땀 작성하였지만, 우리가 실무에서 개발을 할 때는 이렇게 한땀한땀 짜지 않고, 훌륭하신 분들이 미리 짜놓은 라이브러리들을 활용을 하는 경우가 많습니다. 제가 사랑하는 함수형 프로그래밍 라이브러리인 `RamdaJS`에서 소개하는 예를 살펴보면서 이 글을 마무리 하고자 합니다.

```js
const numbers = [1, 2, 3, 4];
const transducer = R.compose(R.map(R.add(1)), R.take(2));
R.transduce(transducer, R.flip(R.append), [], numbers); //=> [2, 3]

const isOdd = (x) => x % 2 !== 0;
const firstOddTransducer = R.compose(R.filter(isOdd), R.take(1));
R.transduce(firstOddTransducer, R.flip(R.append), [], R.range(0, 100)); //=> [1]
```

`R.filp(R.append)`는 `R.append`의 형태가 (추가할 원소, 대상이 될 리스트) 이기 때문에, reducer의 형태인 `(acc,val)`의 형태를 준수하기 위해서 뒤집어 준 것입니다. 위의 글을 잘 이해를 했다면 아래의 코드도 그렇게 어렵지 않음을 아마 캐치 할 수 있을 것입니다.

# 마치며

새해 첫 글을 함수형으로 시작할 수 있음이 참 행복합니다. 깔끔하고, 디버깅하기 쉽고, 블록 단위로 쉽게 교체 할 수 있는 멋진 패러다임을 알게 된것도 행복하고요. JS를 멋지게 쓸 수 있는 여러 패러다임을 공부하면서, 뭔가 새롭게 정리할 필요가 있다 싶은게 있다면 또 정리하고 싶네요. 끝까지 읽어주심에 정말 감사드립니다!!!
