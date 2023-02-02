---
layout: post
title: Lit로 SPA 만들어보기 ① - BE
subtitle: 굳이 React 안쓰겠다고 삽질한 이야기
category: backend
image: /images/thumbnails/lit.png
---

# 개요

여느때처럼 과제를 하고 있는 평화로운 나날이 계속되던 중, 과제의 요구사항으로 SPA를 만들어라는 요구사항을 받았었습니다. 빠르게 구현만 하고 치워버릴거면 익숙하디 익숙한 Create React App(이하 CRA)를 통해서 빠르게 만들고, SPA에 필요한 라우팅도 React router 등으로 해서 끝낼 수 있었지만, 이 과제를 통해서 새로운 기술을 배워보고 싶다는 무언가의 오기가 생겨서, '최대한 바닐라 JS에 가까운 환경에서 바닥부터(from scratch) SPA를 만들어 보겠다' 라는 목표가 생겼습니다. 결국 만들기는 했지만, 많은 시간을 소모했고, 처음 배우는 지식들도 많았습니다. 그런 분량이 많은 지식들은 언제 휘발될 지 모르고, 나중에도 유용하게 쓰일 수 있을 지식인것 같아서 이렇게 기록을 남겨 놓으려고 합니다.

풀 스택으로 개발을 하다보니, 프론트엔드 사이드와 백엔드 사이드 해야할 일이 둘다 있고, 그 두가지를 모두 한 글에 우겨넣기엔 글의 용량이 너무 커질것 같아서 분리하고자 합니다.

# 사용할 스택

위에서도 간단히 언급했듯, 저의 목표는 '최대한 바닐라 스택 위에서 SPA를 만들어 보는것' 이었습니다. 그 목표를 기반으로 이런 저런 정보를 검색하였고, [전에 글을 작성하였던 web component](/what-is-web-component)를 공부하는 겸, 바닐라 JS에 대해서 더 깊은 이해를 가져야겠다라는 목표로 web component를 사용하자 라는 목표점을 잡았습니다.

web component가 무엇인지, 어떻게 사용하였는지를 학습을 하면서 가장 크게 느낀것은, '상당히 중복되고 긴 코드가 많다' 라는 것이었습니다. 프로그래밍을 하면서 그런 관례적인 코드, 즉 보일러플레이트를 작성하는것은 재미있는 학습이라기 보단, 지루한 노동이라는 생각이 더 많이 들었던 저였기에, 그런 보일러플레이트를 줄이기 위한 라이브러리를 찾던 중, 신뢰할 수 있는 Google에서 만들었고, 사용자도 꽤나 많아 생태계가 활성화 되어있는 [Lit](https://lit.dev/) 라는 라이브러리를 채택해서 사용하게 되었습니다.

Lit 자체에서도 TypeScript의 decorator을 사용해서 코드 양을 줄일 수도 있고, Typescript를 사용하면 여러 편리한 타입 가드 등의 DX를 높여주는 기능들을 누릴 수 있기에, TypeScript 역시 사용하기로 결정하였습니다.

FE 쪽의 스택은 대강 정해졌으니, 이를 웹에 뿌려줄 서버를 만들 필요 역시 있습니다. SPA는 단 하나의 `index.html`만 서빙하면 되기에, ngnix와 같은 정적 서버를 이용할 수도 있지만, SPA라고 해서 API를 당겨 쓰지 않는 것은 당연한 사실이기 때문에, 동적 웹 서버로 간단하면서도 쓸만한 `express`를 사용하기로 하였고, 서버를 개발하면서도 TS의 장점을 누리고 싶어서, 서버 사이드 코드에서도 TS를 사용할 수 있게 설정하고자 합니다. (서버 개발을 node로 하는것에 대해서 말이 많은 것을 알고 있습니다만, 과제의 제약사항이 node를 이용한 개발이었기 때문에 그러려니 하고 넘어가주세요 :), 그리고 어쩌다보니 상대적으로 제일 빠삭한 언어가 JS가 되어서기도 하구...)

# 서버 세팅

단순한 템플릿 엔진등을 사용하여서 만드는 고전적인 MPA는 `express-generator`라는 툴을 이용하면 금방 프로젝트 구조를 잡을 수 있지만, SPA를 구성하기 위한 설정을 만들기 위해서는 해당 툴을 이용할 수 없습니다. 해당 구조를 만들어 주는 툴이 있을수도 있지만, 학습을 위해서 라는 이유로 저는 직접 만들어 보기로 했습니다.

좀 예전에 썼던 글인데, [node 위에 apollo server를 설정하기 위해서 TypeScript를 사용하여 서버를 작성하기 위한 작업을 기록하였던 글](/setting-node-ts-apollo-server)이 있습니다. 많은 부분이 해당 글을 참고하여서 진행하였기 때문에, 위의 글을 가벼운 마음으로 한번 훑고 오시면 아래의 글 이해에도 도움이 되지 않을까 싶습니다.

이번 설정에서 다른 부분이라면, FE 쪽도 함께 개발을 하기 때문에, FE 쪽에서 사용될 webpack 설정이 있을 것이고, 해당 webpack을 express middleware로 실행시켜서 개발할 때 좀 더 편리한 환경을 구축하는 작업 정도가 포함이 되겠습니다.

이제 인트로 부분을 마쳤으니 실제로 어떻게 환경을 설정하였는지 보도록 하겠습니다.

## 1. 필요한 의존성 설치 + TS 설정

TypeScript위에서 돌아가는 express 서버를 세팅하기 위한 의존성을 설치 해보겠습니다.

```bash
npm i express dotenv cookie-parser # express를 사용할 것
npm i typescript @types/node -D # TypeScript
npm i @babel/cli @babel/core @babel/node @babel/preset-env @babel/preset-typescript -D # babel용
npm i nodemon morgan @types/morgan @types/cookie-parser @types/express
```

`morgan`은 서버 사이드에서 발생하는 로그를 편하게 확인할 수 있게 해주는 개발용 미들웨어이며, `cookie-parser`는 클라이언트 사이드에서 날라오는 쿠키를 파싱하게 도와주는 미들웨어 라이브러리라고 생각하면 됩니다. `nodemon`은 서버 파일이 수정될 때 마다 서버를 재시작해주는 개발 편의성을 높여주는 라이브러리 입니다.

TypeScript를 사용하기 위해서는 너무나도 당연하게 `tsconfig.json`을 설정해 주어야 하는데, 이에 관한 것은 전에 작성한 [apollo server 세팅 글](/setting-node-ts-apollo-server)에서 자세한 이야기를 했으니 간단히 제가 사용한 `tsconfig`설정만 남겨두고 넘어가겠습니다.

```json
{
  "compilerOptions": {
    "target": "es5",
    "strict": true,
    "alwaysStrict": true,
    "noImplicitAny": true,
    "allowSyntheticDefaultImports": true,
    "outDir": "dist",
    "listEmittedFiles": true,
    "experimentalDecorators": true
  },
  "include": ["src/**/*"]
}
```

## 2. `webpackDevMiddleware`을 설정하기 위한 발버둥

`webpackDevMiddleWare`는 **개발 시** 쓰이는 미들웨어로, 서버가 실행 될 때마다, webpack을 실행시켜서 서버가 재시동 될 때, 클라이언트 사이드 코드도 함께 refresh 될 수 있게 해주는 미들웨어 입니다. webpack을 실행하기 위해서는, 참 당연한 이야기지만, `webpack.config.js`를 필요로 합니다. 근데 우리는 TypeScript를 사용한다고 하였고, 위 typescript 설정에서 `strict`를 설정해줬기 때문에, `webpack.config.js`의 타입에 관한 문제가 당연히 수면위로 떠오르게 됩니다.

TypeScript 생태계에 함께 쓰이는 파일이다 보니 처음에는 `.ts` 확장자로 작성을 해보려고 하였지만, 뭔가 녹록치가 않아서, TypeScript 생태계에서 JavaScript 파일을 타입과 함께 사용하는 방법인 ,`d.ts` 즉 타입 정의 파일을 선언해서 사용하기로 하였습니다.

webpack의 내용물에 관해서는 FE 사이드에서 더 자세히 설명할 예정이기 때문에, 간단하게 설명하고 끝내도록 하겠습니다. webpack은 TS 친화적이라고 해서 이런저런 타입들을 선언해두고 그것을 이용해서 TS 파일로 config를 작성할 수 있다고 하는데, 뭔가 문제가 생긴 저는 d.ts 파일을 아래 처럼 작성하였습니다.

```typescript
import { Configuration } from "webpack";

declare const configuration: Configuration;

export default configuration;
```

해당 파일을 같은 디렉토리에 같은 파일 이름으로 확장자만 `d.ts`로 수정을 해서 하면 에디터의 TypeScript가 이를 읽을 수 있습니다.

그리고 전에 작성한 [apollo server 세팅 글](/setting-node-ts-apollo-server)에서 파일을 직접 emit 해서 사용하였기 때문에, 관련 내용은 링크해둔 글에서 찾아보시기 바랍니다. 아래에는 webpackDevMiddleware을 설정하기 위한 제 코드의 일부를 첨부하도록 하겠습니다.

```typescript
// 전략 ...
import webpack from "webpack";
import webpackDevMiddleware from "webpack-dev-middleware";
import webpackConfig from "../webpack/webpack.dev.js";
// 중략 ...

const app = express();

const compiler = webpack(webpackConfig);
//중략 ...

if (process.env.NODE_ENV == "development") {
  app.use(
    webpackDevMiddleware(compiler, {
      publicPath: webpackConfig.output!.publicPath,
      writeToDisk: true,
    })
  );
}

// 하략 ...
```

## 3. 이제 다 끝났다. 간단한 라우팅

제목에서 말했듯, 이제 복잡한것은 다 끝냈습니다. 이제 정적인 에셋을 서빙하고, 클라이언트 사이드에서 렌더링할 기초가 되어줄 간단한 `index.html`을 보내주는것 까지가 CSR에서 서버의 책임이니 해당 부분만 구현을 해둡시다.

```typescript
// 전략 ...
app.use("/static", express.static("dist/static"));
app.route("/*").all((req, res) => {
  res.sendFile(path.resolve("dist/static", "index.html"));
});
// 후략 ...
```

# 마치며

사실 이번 과제를 하면서 FE 쪽에 집중을 하여서 BE 쪽의 작업량은 별게 없고, 웹팩 쪽 설정을 이해하지 않고는 사실 이것들을 이해할 수 있을까 하는 생각까지 드네요. 정리하니까 참 별일 없는데, 관련해서 찾아 본 자료들은 참 많았습니다. 제가 삽질을 할 때, 참고했던 자료들을 첨부하면서 글을 마무리 하려고 합니다. 끝까지 읽어주셔서 감사합니다.

- [https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-d-ts.html](https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-d-ts.html)
