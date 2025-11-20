---
title: express.js의 Request TS 타입에 대한 고찰
layout: post
subtitle: express.js의 Request에도 ResponseBody 타입 인자가 포함된 이유
image: /images/thumbnails/express.png
category: backend
---

# 들어가며

최근 블로그를 둘러보니 express 관련 글은 2022년 10월, 부스트캠프를 수료하던 시기에 작성한 글이 마지막이었습니다. 그 이후로는 주로 FE 기술을 다루었고, 연구실에서 풀스택 작업을 할 때에도 Python FastAPI를 사용했기 때문에, TypeScript 기반의 express.js를 다시 제대로 다루게 된 것은 꽤 오랜만이었습니다.

최근에 진행한 과제들 중 일부는 서버 구현까지 요구하는 풀스택 과제들이었습니다. 기능 자체를 구현하는 데에는 큰 어려움이 없었지만, 과제 리뷰를 하면서 한 가지 작은 의문이 생겼습니다. 바로 아래와 같은 express 라우팅 코드에서 `Request` 타입의 제네릭 구조가 왜 이런 형태인지였습니다.

```ts
router.post(
  "/signup",
  async (req: Request<{}, {}, SignupBody>, res: Response) => {
    const { email, password, name } = req.body;

    if (!email || !password) {
      return res.status(400).json({ message: "email and password required" });
    }

    const exists = users.find((u) => u.email === email);
    if (exists) {
      return res.status(409).json({ message: "user already exists" });
    }

    const passwordHash = await bcrypt.hash(password, 10);
    const newUser: User = {
      id: idCounter++,
      email,
      name,
      passwordHash,
    };
    users.push(newUser);

    return res.status(201).json({ id: newUser.id, email: newUser.email });
  }
);
```

express.js의 `Request` 타입 정의를 살펴보면 다음과 같았습니다.

```ts
interface Request<
  P = ParamsDictionary,
  ResBody = any,
  ReqBody = any,
  ReqQuery = ParsedQs,
  Locals = Record<string, any>
>
```

여기서 P는 경로 파라미터 타입, ReqBody는 요청 바디 타입, ReqQuery는 쿼리 스트링 타입, Locals는 로컬 변수(res.locals) 타입입니다. 그런데 두 번째 인자인 `ResBody`가 응답 본문의 타입을 가리킨다는 점에서 한 가지 의문이 생겼습니다. `Request` 타입인데 응답 타입을 왜 넣을 수 있도록 했을까요? 이 궁금증이 이 글을 쓰게 된 계기였습니다.

# Express.js의 특징과 역사적 배경

## 런타임 관점에서 바라본 이유

도구는 해결하려는 문제가 있기 때문에 탄생하고, 그 도구가 세상에 나온 이후 어떤 선택이 쌓였는지가 현재 구조를 이해하는 가장 중요한 단서가 됩니다. Express.js 역시 마찬가지로, Node의 http 모듈을 확장하는 가벼운 라이브러리로부터 출발했습니다.

Node의 http 모듈에서 기본 핸들러는 다음과 같은 형태를 가지고 있습니다.

```js
http.createServer((req, res) => {
  // req와 res가 항상 같은 요청 흐름을 공유합니다.
});
```

즉 Node는 애초부터 요청(req)과 응답(res)을 하나의 흐름에서 함께 움직이는 쌍으로 취급하는 모델을 가지고 있었습니다. Express는 이 구조를 그대로 계승했고, 요청과 응답을 “하나의 요청 단위(context)”처럼 바라보는 관성이 생기게 되었습니다.

이 과정에서 Express는 `req` 객체를 단순 요청 정보가 아니라 “요청과 관련된 거의 모든 것을 담아두는 컨텍스트 객체”처럼 사용하기 시작했습니다. 그래서 `req.params`, `req.query`, `req.cookies`, `req.session`, `req.app` 등 다양한 정보들이 모두 `req` 아래로 모이게 되었습니다.

이런 문화 속에서 “응답을 보낼 수 있는 수단(res)”도 동일한 요청 컨텍스트에 포함시키는 것이 편리하다고 여겨졌고, 결국 Express 런타임에서는 `req.res`처럼 요청 객체 안에서 응답 객체에 접근할 수 있는 구조가 만들어졌습니다. [실제로 express 내부적으로 req.res에 많이 의존하고 있고요.](https://stackoverflow.com/questions/57598862/getting-original-request-url-from-express-response#:~:text=While%20they%20don%27t%20appear%20to,seems%20relatively%20safe%20to%20use) 이 구조는 ‘엄밀한 설계 철학’이라기보다, 당시 생태계에서 널리 쓰이던 실용적·편의적 선택의 누적에 가깝습니다.

다만 실제 어플리케이션 개발자 입장에서는 [req, res를 주어진대로 쓰는것이 좋다](https://www.reddit.com/r/expressjs/comments/1d17b4e/question_about_the_request_object_containing_res/)라는 의견이 있고, 그 편이 더 직관적인건 많은 사람들이 느끼는 바 인 것 같습니다.

이 섹션에서 가져갈 것은 `req.res`라는 기묘한 구조가 실제한다는 것이고, [공식 문서화](https://expressjs.com/en/api.html#:~:text=req.res)까지 존재한다는 것이지요

## 타입 설계 관점에서 바라본 이유

여기까지는 “왜 런타임에서 req.res 같은 구조가 생겼는가”에 대한 이야기였습니다. 이제 TypeScript 타입 설계 관점에서 왜 `Request` 제네릭에 `ResBody`가 포함되게 되었는지를 살펴보겠습니다.

핵심은 Express의 타입 정의가 `Request`, `Response`, `RequestHandler`를 서로 강하게 묶어 둔 형태라는 점입니다.

```ts
export interface RequestHandler<
  P = ParamsDictionary,
  ResBody = any,
  ReqBody = any,
  ReqQuery = ParsedQs,
  LocalsObj extends Record<string, any> = Record<string, any>
> {
  (
    req: Request<P, ResBody, ReqBody, ReqQuery, LocalsObj>,
    res: Response<ResBody, LocalsObj>,
    next: NextFunction
  ): void;
}
```

여기서 중요한 점은 세 타입이 **동일한 제네릭 시그니처를 공유하고 있다는 것**입니다. 즉, 라우터 선언 시 `<P, ResBody, ReqBody, ReqQuery, Locals>` 한 번만 넘기면 `Request`, `Response`, `RequestHandler` 모두가 자동으로 맞춰집니다.

```ts
const handler: RequestHandler<Params, ResBody, ReqBody, ReqQuery> = (
  req,
  res,
  next
) => {
  // req.body, req.query, res.json(...)이 모두 하나의 제네릭 셋에 연결됩니다.
};
```

그런데 요청 타입인 `Request`의 관점에서 보면 두 번째 인자인 `ResBody`는 실제로는 거의 사용되지 않는 정보입니다. 요청 객체는 본질적으로 “요청에 관한 정보”만 알면 되고, “응답 본문이 어떤 타입인지”는 반드시 알아야 하는 값이 아닙니다.

그럼에도 `Request` 제네릭에 `ResBody`가 포함된 이유는 명확한 당위(“Request가 응답 타입을 반드시 알아야 한다”)가 있었기 때문이 아니라, **런타임 구조(req.res)를 타입으로 정확히 옮기기 위해 필요한 정보였기 때문**입니다.

Express 런타임에는 실제로 `req.res`가 존재합니다. 따라서 TypeScript 타입 시스템에서도 `req.res`를 `Response<ResBody, Locals>` 형태로 표현하려면 `Request`가 `ResBody`와 `Locals` 정보를 알고 있어야 합니다. 그 결과 `Request` 타입도 `Response`와 동일한 제네릭 묶음을 갖게 되었고, 자연스럽게 `RequestHandler`까지 이 제네릭을 공유하도록 설계되었습니다.

요약하자면, `Request`에 `ResBody`가 포함된 것은 “요청이 응답을 알아야 해서”가 아니라, **런타임 구조를 타입 수준에서 일관성 있게 표현하기 위해 제네릭 묶음을 통일해 둔 결과물**에 가깝습니다.

**📌 핵심 요약**

- Express 런타임에는 req.res가 실제로 존재한다.
- TypeScript 타입도 이를 반영해야 했기 때문에 Request가 ResBody 제네릭을 알아야 한다.

# 마무리

정리하자면, Request 타입에 ResBody가 포함된 이유는 Express가 태생적으로 갖고 있는 구조를 TypeScript가 그대로 반영한 결과라고 할 수 있습니다. 다만 이 구조 자체가 요즘 기준에서 보면 다소 특수한 편이기도 합니다. 대부분의 프레임워크는 요청과 응답을 완전히 분리해 모델링하니까요.

Express는 Node.js 초창기 생태계의 문화—요청과 응답을 하나의 흐름으로 묶어 두고, req를 요청 컨텍스트 객체처럼 확장하던 방식—가 지금까지 이어진 사례입니다. 그 결과로 req.res 같은 구조가 생겼고, 타입 구조도 그 흐름에 맞춰 제네릭 묶음을 공유하게 되었습니다.

핸들러를 위한 함수 타입 시그니처가 있는 것을 몰랐던 것 때문에, 땜질을 하다가 이러한 지식도 알고 가게 되네요.
