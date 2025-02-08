---
layout: post
title: 클래스와 클로저의 비교
subtitle: 이러이러한 상황에 클로저와 클래스 중 어느것을 써야할까요? 를 답변해보죠
category: javascript
image: /images/thumbnails/JS.png
---

# 도입

[이전 글](/closure-basics)에서도 언급했던, 저의 약점을 깨닫게 해주었던 그 면접 이후로, 곰곰히 생각해 보았습니다. 클로저와 클래스에 대해서도 언급을 했는데, 클로저와 클래스를 다른 측면에서 비교하는 질문도 할 수 있지 않을까 하고 말이지요.

그래서 가장 먼저 적고 싶었던 글은 이 주제의 글이었으나, 실행 컨텍스트와 클로저에 대한 명확한 설명 없이, 대뜸 비교하는 글을 적는다는 것이 상식적으로 말이 되지 않기에, 2월이 된 지금에서야 이렇게 글을 적고 있습니다.

2025년 현재, 대부분의 브라우저가 `#`를 활용한 private property를 정식 지원하는 현 시점에서, 5년 이내에 적힌 좋은 글들을 그대로 긁어오는 것은 별로 유효하지 못하다 생각했고, 단순히 번역 후 복사-붙여넣기 하는 글이 아닌, 저의 ‘연구 결과물’을 확인하는 글이 되었으면 합니다.

맨바닥에 헤딩하듯 적는 글이 아니지만, 이 글을 적기까지 참 많은 일이 있었던것 같습니다. 쭉 실행 컨텍스트와 렉시컬 스코프에 집착하다 보니, this도 함수 정의 시점에서 resolve 되지 않나 하면서 프로토타입 체이닝이 어떻게 동작할 수 있는거지 같은 실로 지금 보면 어처구니 없는 생각들도 하고… 이부분에 관한 것도 향후 정리를 해야겠군요…

# 성능의 비교를 하기 전에

## ‘은닉’을 할 수 있는 문법이 두개…?

클로저의 개념과 최대한 간단하게 설명한 응용에 관한 지난 글에서, 클로저의 특징인, ‘은닉’에 대해서 다루었습니다. 학부 과정에서 ‘객체 지향 프로그래밍’ 설명 할 때에도 자바 같은 프로그래밍 언어에서 `private` 한정자를 이용해, 변수를 은닉할 수 있다고 배웠습니다. 또 JS에서는 ES6때 `class` 키워드가 등장하고, [ES2022때 ,Private Property가 정식으로 등장하면서](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_properties) 자바 같은 여타 다른 OOP 언어에서 하는 것과 비슷한 것을 할 수 있게 되었지요.

클로저를 사용하는 이유 중 하나도 ‘변수의 은닉’ 이었는데, 어느쪽이 성능상 유리할까를 생각하는것은 그렇게 떠올리기 어려운 질문은 아닌 것 같습니다.

## 클로저의 특징 돌아보기 : 독립된 메모리 공간

[이전 글](/closure-basics)에서 알아봤던 클로저의 유의점 에서, ‘같은 함수로 만들어진 클로저라도 독립된 메모리 공간을 가진다’ 라고 언급했던 바를 다시 한번 되짚어 봤으면 좋겠습니다. 그 때 다루었던 예제 코드도 함께 첨부하겠습니다.

```js
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

`counter`와 `counter2`는 서로 독립적으로 행동한다는 이야기를 했었지요. 가물가물 하다면, 원 출처인 [여기](https://ko.javascript.info/task/closure-variable-access)를 다시 한번 읽어보는것도 좋을 듯 합니다.

클래스는 반면, 프로토타입 체인과 유사하게 동작하기 때문에, 아래와 같은 도식으로 간단히 비교할 수 있을 듯 합니다.

![image.png](/images/javascript/closure-vs-class/class-vs-closure.png)

출처:<https://blog.logrocket.com/how-to-decide-between-classes-v-closures-in-javascript/>

# 메모리 효율성

어찌보면 정말 간단하다 싶습니다. 클로저는 각각 인스턴스마다 독립된 메모리 공간을 형성하지만, 클래스는 프로토타입 체인 위에 함수를 저장하고 있어서, 메소드의 중복 생성을 방지할 수 있지요. [이 글](https://medium.com/engineering-livestream/javascript-classes-v-closures-2-3-9fa3b1296b16)에서 3만개의 인스턴스를 만들어서 실제로 실행 결과를 비교하는 예제도 있고, [브라우저 버전](http://twitchard.github.io/classes-v-closures/browser/index.html) 또한 만들어져 있습니다.

## 퍼포먼스 테스트용 코드

### 클래스로 구현한것

```js
"use strict";
class Cow {
  constructor(lungCapacity) {
    this.lungCapacity = lungCapacity;
    this.airInLungs = 0;
  }
  getAirInLungs() {
    return this.airInLungs;
  }
  breathe() {
    this.airInLungs = this.lungCapacity;
  }
  moo() {
    let output = "m";
    let air = this.getAirInLungs();
    while (air-- > 0) {
      // The 'goes to' operator
      output += "o";
    }
    this.airInLungs = air;
    return output;
  }
}
const herd = [];
for (let i = 0; i < 30000; i++) {
  const cow = new Cow(i);
  cow.index = i;
  herd.push(cow);
}
console.log(process.memoryUsage());
const start = Date.now();
herd.map((cow) => {
  cow.breathe();
  cow.moo();
});
console.log("Finished mooing in " + (Date.now() - start) / 1000 + " seconds");
```

### 클로저로 구현한것

```jsx
"use strict";
function Cow(lungCapacity) {
  let airInLungs = 0;
  function breathe() {
    airInLungs = lungCapacity;
  }
  function getAirInLungs() {
    return airInLungs;
  }
  function moo() {
    let output = "m";
    let air = getAirInLungs();
    while (air-- > 0) {
      // The 'goes to' operator.df
      output += "o";
    }
    airInLungs = air;
    return output;
  }
  return { breathe: breathe, moo: moo };
}
const herd = [];
for (var i = 0; i < 30000; i++) {
  const cow = Cow(i);
  cow.index = i;
  herd.push(cow);
}
console.log(process.memoryUsage());
const start = Date.now();
herd.map(function (cow) {
  cow.breathe();
  cow.moo();
});
console.log("Finished mooing in " + (Date.now() - start) / 1000 + " seconds");
```

## 실행 결과

```bash
kasterra@kasterra-MacBook-Pro PS % node classes.js
{
  rss: 39944192,
  heapTotal: 6733824,
  heapUsed: 5046992,
  external: 1086369,
  arrayBuffers: 10515
}
Finished mooing in 1.222 seconds
kasterra@kasterra-MacBook-Pro PS % node closures.js
{
  rss: 55984128,
  heapTotal: 28229632,
  heapUsed: 12839528,
  external: 1086369,
  arrayBuffers: 10515
}
Finished mooing in 1.324 seconds
```

참고한 자료의 당시 저자의 컴퓨터 환경에서는 이렇게 나왔다던데, 격세지감이 드는군요…. 컴퓨터도 빨라지고, V8 엔진도 발전을 했나 봅니다…

```bash
$ nvm use 6
Now using node v6.9.1 (npm v3.10.8)
$ node classes.js
{ rss: 29143040, heapTotal: 11571200, heapUsed: 6064928 }
Finished mooing in 3.147 seconds
$ node closures.js
{ rss: 41410560, heapTotal: 29396992, heapUsed: 15649208 }
Finished mooing in 5.315 seconds
```

# CPU 성능 (실행시간)

결론부터 말해보자면 이것입니다.

> 많은 경우에 클래스 구현이 클로저 구현보다 더 빠른 경향이 있다.

[V8 컴파일러 엔지니어인 Slava Egorov씨가 직접 본인의 블로그에 적으신 글](https://mrale.ph/blog/2012/09/23/grokking-v8-closures-for-fun.html)을 확인해 보면, 자주 사용되는 프로퍼티 들을 naive 하게 명세 그대로의 체인을 타고 가는 대신, `hidden class` 라는 샛길을 통해서 더욱 빠르게 접근 할 수 있습니다. (참고자료 : <https://mrale.ph/blog/2012/06/03/explaining-js-vms-in-js-inline-caches.html>)

또한 V8 런타임은 자주 사용되는 메소드들에 대해서 함수를 직접 부르지 않고, 인라인화 하는 ‘인라인 캐싱’을 통해서 속도를 챙길 수 있기 때문에, 별도의 메모리 공간을 형성하는 클로저와 다르게 프로토타입 체인 안에 있는 함수를 부르는 클래스측 구현이 더 빨라 질 수 있지 않을까 하고 생각됩니다.

# 테스팅 관점(특히 mocking)에서의 비교

소프트웨어 개발이 단순 구현에서 끝나는 것이 아니고, jest와 같은 테스팅 라이브러리를 통해 테스트를 하는 상황 또한 있을 수 있으니, 이 부분에 대해서도 비교를 해봅시다. [이 섹션을 작성하는데 많은 참고가 되었던 글](https://medium.com/engineering-livestream/javascript-classes-vs-closures-3-3-9d9233eb0a5c)에서는 mock을 도와주는 라이브러리인 Sinon([https://sinonjs.org](https://sinonjs.org/))을 사용해서 예제들을 설명했네요.

테스팅을 하기 위해서는 여러 기법들을 사용하지만, 제목에서도 적은 것 처럼, mocking 부분에 대해서 초점을 맞추려고 합니다. 특정한 컴포넌트만 테스팅을 하기 위해서, 그 컴포넌트와 상호작용 하는 부분들을 다른 모의 데이터나 로직으로 감추는 것이지요.

특정한 메서드를 mock 하기 위해서는 테스트를 위해서 코드를 뜯어고치지 않으려면 그 메서드에 쓰기가 가능한 ‘접근’이 필요한데, [참조한 글](https://medium.com/engineering-livestream/javascript-classes-vs-closures-3-3-9d9233eb0a5c)에서는 접근이 ‘어려운’경우가 되기 위한 조건을 다루고 있습니다.

1. 의존성 주입(DI)를 하지 않았을 경우.
   - DI를 일관적으로 사용한다면, 테스트를 위해서 실제 로직 대신에 mock을 주입 할 수 있음
2. **클래스 대신에 클로저를 사용**했을 경우
   - 클래스를 사용한다면, 프로토타입에 메서드가 존재하고, 이를 테스트에서 참조하고 오버라이드 할 수 있지만, 클로저를 사용한다면 불가능함. nodeJS에서는 가능은 하지만, 상당한 흑마술을 요구함

이 두가지가 ‘모두’ 만족된다면 mock을 하기가 어려워 진다고 합니다. DI를 사용한 클래스는 학부 수업 시간 등에 너무나도 많이 다루어 봤기 때문에, 그 외의 경우를 통해, 어떻게 mock 테스팅이 이루어 지는지, node의 ‘흑마법’ 이라고 하는것이 무엇인지 한번 알아봅시다.

## 예제1 : DI를 사용한 클로저

`HttpClient` 라는 로직을 클로저 문법을 사용해서 구현했다고 해 봅시다.

```jsx
"use strict";
function HttpClient(config) {
  return {
    get: function (path, callback) {
      // 이 함수의 로직은 중요하지 않습니다
      // 테스트를 위해서 mock 할 예정입니다
    },
  };
}
module.exports = HttpClient;
```

그리고 `HttpClient`를 사용하는 의존성 주입을 통해 사용하는 모델이 있다고 합시다.

```jsx
// foo_model.js
"use strict";
function FooModel(fooClient) {
  return {
    getById: function (fooId, callback) {
      return fooClient.get(`/foo/${fooId}`, (err, result) => {
        if (err) return callback(err);
        const foo = JSON.parse(result.body);
        return callback(null, foo);
      });
    },
  };
}
module.exports = FooModel;
```

통상적인 상황에서는 인자를 통해서 넘겨서 여러가지의 Client들을 사용할 수 있게 하는 것이지요.

이제 `FooModel`로 넘길 client의 `get`을 mock하는 테스트를 만들어 보겠습니다.

```jsx
// test.js
"use strict";
const sinon = require("sinon");
const FooModel = require("./foo_model");
describe("Foo", () => {
  describe("#getById", () => {
    describe("when fooId is 10", () => {
      let mockFooClient;
      beforeEach(() => {
        mockFooClient = { get: sinon.spy() };
      });
      it("sends GET /foo/10 to the foo service", () => {
        // mock된 Client를 FooModel로 주입한다
        const fooModel = FooModel(mockFooClient);
        // 테스트 환경에서 호출
        fooModel.getById(10);
        // 실제로 원하는 형태로 호출되었는지 assert를 통해 확인한다.
        sinon.assert.calledWith(mockFooClient.get, "/foo/10");
      });
    });
  });
});
```

## 예제 2 : DI를 사용하지 않은 클로저

이번 상황에서는 `FooModel`이 DI가 아니고, 코드 내에 삽입 되어 있는 상황을 가정해 봅시다.

```jsx
// foo_model.js
const HttpClient = require('./http_client')
function FooModel (config)
    // 테스트 코드에서 fooClient를 접근할 수 없음!
    const fooClient = HttpClient(config)
    return {
        getById: function (fooId, callback) {
            return fooClient.get(`/foo/${fooId}`, (err, result) => {
                if (err) return callback(err)
                const foo = JSON.parse(result.body)
                return callback(null, foo)
            })
        }
    }
}
module.exports = FooModel
```

통상적인 방법에서는 이제 `FooModel` 안에 들어있는 client를 접근할 수 없습니다만, [nodeJS require cache](https://nodejs.org/api/modules.html#requirecache)를 사용하는 흑마법으로 이것을 어찌어찌 해내게 할 수 도 있습니다.

```jsx
"use strict";
const sinon = require("sinon");
describe("Foo", () => {
  describe("#getById", () => {
    describe("when fooId is 10", () => {
      let mockFooClient;
      beforeEach(() => {
        mockFooClient = { get: sinon.spy() };
      });
      let FooModel;
      beforeEach(() => {
        // 'foo_model'의 캐시가 이미 있는 상황에 대비해서
        // 캐시를 날려준다
        delete require.cache[require.resolve("./foo_model")];
        // 'http_client'가 캐시에 있게끔 해준다
        require("./http_client");
        // 우리의 mock으로 오버라이드 하기
        require.cache[require.resolve("./http_client")].exports = () =>
          mockFooClient;
        // foo_model을 다시 로드하면
        // 우리가 오버라이드한 캐시를 사용하게 됨
        FooModel = require("./foo_model");
      });
      it("sends GET /foo/10 to the foo service", () => {
        const fooModel = FooModel();
        fooModel.getById(10);
        sinon.assert.calledWith(mockFooClient.get, "/foo/10");
      });
    });
  });
});
```

상당히 장황하고, ‘이게 맞나’ 하는 생각이 듭니다. 테스트를 위해서 코드 외적인 부분의 캐시를 건드리는 것도 과연 ‘옳은’ 것인가에 대한 생각도 들고… 그래서 지금은 아카이브로 넘어가 버렸지만, [이런 라이브러리](https://github.com/mfncooper/mockery) 또한 있었습니다.

만약 클래스를 사용했다면 어떻게 할 수 있었을까요?

## 예제 3 : DI를 사용하지 않는 클래스

`HttpClient` 가 클로저 대신에 클래스 문법을 통해 쓰였다면, 테스트 하기가 조금 더 쉬울겁니다. 바로 `HttpClient`의 get을 프로토타입 단에서 오버라이드 할 수 있으니까요

```jsx
// http_client.js
"use strict";
const request = require("request");
class HttpClient {
  constructor(config) {
    this.config = config;
  }
  get(path, callback) {
    // 여전히 원래 내용물은 중요하지 않음
    // mock 할 부분이기 때문에...
  }
}
module.exports = HttpClient;
```

```jsx
// foo_model.js
// 이전 예제와 유일하게 차이점은, new 키워드를 사용한다는점 (HttpClient가 클래스니까)
"use strict";
const HttpClient = require("./http_client");
function FooModel(config) {
  const fooClient = new HttpClient(config);
  return {
    getById: function (fooId, callback) {
      return fooClient.get(`/foo/${fooId}`, (err, result) => {
        if (err) return callback(err);
        const foo = JSON.parse(result.body);
        return callback(null, foo);
      });
    },
  };
}

module.exports = FooModel;
```

```jsx
//test.js
"use strict";
const sinon = require("sinon");
const FooModel = require("./foo_model");
const HttpClient = require("./http_client");
describe("Foo", () => {
  describe("#getById", () => {
    describe("when fooId is 10", () => {
      let mockFooClient;
      let $get;
      beforeEach(() => {
        $get = HttpClient.prototype.get;
        HttpClient.prototype.get = sinon.stub();
      });
      afterEach(() => {
        // 테스트 종료 후에 원래 형태로 돌려놓기
        HttpClient.prototype.get = $get;
      });
      it("sends GET /foo/10 to the foo service", () => {
        const fooModel = FooModel();
        fooModel.getById(10);
        sinon.assert.calledWith(HttpClient.prototype.get, "/foo/10");
      });
    });
  });
});
```

아까 node 내부의 캐시를 어쩌고 저쩌고 하는 이상한 것보다는 훨 나은 듯 하군요. 단순 프로토타입을 끌어와서 수정하는 것으로, 어찌 테스트를 진행 할 수 있었습니다.

DI를 사용하는 쪽이 테스팅을 위한 mock의 측면에서 훨 깔끔하다 라는 것을 알 수 있었습니다.

# 마무리하며

실무에서 ~한 상황이 있을 때, 클로저를 사용하는 것이 좋을까요, 아니면 클래스를 사용하는 것이 좋을까요 라는 질문을 받는다면, 어떻게 답해야 할까. 라는 질문에서 시작된 포스트가 이렇게 끝이 났습니다. 솔직하게 말하자면, 아직은 이러한 단위 테스트에 익숙하지 않은 사람으로서, 테스팅 관련 섹션은 학부에서 스프트웨어 테스팅 이론이라는 과목을 들은 이후에 오랫만에 마주하는 것들이 꽤나 있었네요…

글을 4개나 잡아먹은 클로저 관련 시리즈가 이렇게 끝을 맺게 되었습니다. ‘재미있는 글을 써보겠다’ 라고 저번 글에서 다짐했었지만, 그래도 실제 환경에서 어떻게 돌아가는지를 다룬다는 점에서 조금이라도 흥미가 당기지 않았을까 하고 한마디 남기면서 글을 마치려고 합니다. 끝까지 읽어주셔서 감사합니다.

# 참고문헌

- 가장 많은 참고가 되었던 medium 글 :
  - <https://medium.com/engineering-livestream/javascript-classes-v-closures-2-3-9fa3b1296b16> (성능 측면 비교)
  - <https://medium.com/engineering-livestream/javascript-classes-vs-closures-3-3-9d9233eb0a5c> (테스팅 측면에서의 비교)
- V8엔지니어가 직접 말아주는 V8에서의 메모리 관리 및 빠른 접근 전략
  - <https://mrale.ph/blog/2012/09/23/grokking-v8-closures-for-fun.html> (클로저의 메모리 사용 + 클래스의 런타임 최적화 관련)
  - <https://mrale.ph/blog/2012/06/03/explaining-js-vms-in-js-inline-caches.html> (캐싱이 어떻게 돌아가는지)
