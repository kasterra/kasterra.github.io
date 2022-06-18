---
title: GraphQL용 apollo-server 개발 환경 구축 (feat TypeScript + yarn berry)
layout: post
subtitle: 반짝반짝한 모던한 환경으로 개발 환경을 만들어 보자구요
image: /images/thumbnails/apolloTSSetup.png
category: backend
---

# 들어가며

요즘 저는 제대로 된 개발자가 되기 위해서, 프로젝트도 하고, 인턴 면접도 보면서 스스로의 실력도 체크하면서 역량 성장에 집중하고 있습니다. 학습을 한 것을 정리하면서, 나중에 헷갈릴 때 제가 찾아보고, 혹시나 비슷한 고민을 하는 분들에게 도움을 주기 위해서 블로그 글 작성에 언제나 심혈을 기울이고 있습니다.

이번 글의 주제는 GraphQL 학습을 위한, 개발환경 셋업입니다. 프론트엔드에서 실제 데이터를 가져오기 위해 서버로 데이터를 fetching 하기 위해 REST API를 주로 사용합니다. REST API 자체는 깔끔한 기술입니다. HTTP Method와 명사로 된 URL들을 이용해서 서버에 요청을 하면, 서버가 그 요청을 받아서, 프론트의 요청을 깔끔하게 처리를 해 주지요. 하지만, DB에서 data를 fetching 할 때, depth가 깊어지거나, 필요한 데이터보다 더 많이 받아오는 overfetching의 문제, 필요한 데이터보다 덜 받아오는 underfetching의 문제를 REST API는 가지고 있습니다. 이런 문제를 해결하기 위해서 나온게 GraphQL 이라고 합니다.

GraphQL 자체는, 서버가 어느 언어로 구성되어 있든 쓸 수 있습니다. 우리가 RDBMS에서 데이터를 fetching 하기 위해서 SQL query문을 날릴 때, 서버가 Java Spring으로 짜여있던, Node express로 짜여있던 상관 없는것 처럼요. 이번 글에서는 node.js + Apollo-server를 활용해서 서버 구성을 해보려고 합니다. 이유는 블로그 저자인 제가 javascript 환경에 익숙하기 때문이지, 더 특별한 이유는 딱히 없습니다. 이번 글에서는 맨땅에 헤딩하듯 개발환경을 만들면서 겪은 꽤나 많은 에러를 제가 다시 겪지 않기 위해, 그리고 이 글을 읽고 계시는 여러분이 겪지 않길 바라는 마음에서 써보는 글입니다. 자 이제 출발해 봅시다~!

# 사용할 환경

이때까지 제가 써온 글을 보면 아시겠지만, 저는 javascript 생태계에 무척이나 익숙합니다. 프론트엔드 공부를 계속 하다보니, 당연하지요... 그리고 자동완성 등의 DX를 위해서 TypeScript를 사용할 것이고, 많은 기업들이 실무에서도 사용하는 [yarn berry](../setting-yarn-berry/)를 사용할 것입니다. 네, 이 글의 썸네일에 있는 그 친구들 입니다.

# 본격적인 환경 설정

## 필수 패키지 설치 + TypeScript

이제 본격적으로 설정을 시작해 봅시다. 이 글에서는 graphQL이나 apollo-server에 대한 깊은 고찰 같은것이 아닌, 단순한 개발 환경 설정 글이기에 그렇게 길지 않을것 같습니다.

우선 폴더를 하나 만들고, npm으로 init 해줍시다.

```bash
mkdir graphql-api-server
cd graphql-api-server
npm init -y
```

yarn berry를 사용한다고 했으므로, yarn berry를 위한 작업들을 해줍시다. 또, TypeScript를 사용하므로, typescript 관련 세팅까지 해주겠습니다. 이게 좀 낮설다 싶은 글은, [제가 이전에 썼던 yarn berry 세팅 관련 글](../setting-yarn-berry)를 참고해 주세요.

```bash
yarn set version berry
yarn plugin import typescript
yarn dlx @yarnpkg/sdks vscode
```

그리고 우리의 환경에 핵심이 되는 apollo-server와 graphql을 설치해 줍시다.

```bash
yarn add apollo-server graphql
```

TypeScript를 사용해서 작업할거라고 했으니까, 설치해 줍시다!

```bash
yarn add typescript -D
yarn add @types/node -D
```

TypeScript를 쓸 때 없어서는 안되는 `tsconfig.json`도 작성해 줍시다. 정해진 정답은 없지만, 저는 아래와 같이 작성했습니다. 여기서 target을 **꼭!!** `es5` 또는 이하로 해야지 이후에 정상적인 작동이 보장됩니다. 이유를 간단하게 설명하자면, target이 `es6` 이상이면 esmodule이 그대로 나오고, `es5` 이하여야지 commonjs 기반 모듈 번들링으로 전환됩니다. esmodule이 현재 yarn berry의 pnp와 잘 안맞는것 같더라고요... 그래서 es5 이하로 설정을 해주셔야 겠습니다!

```json
{
  "compilerOptions": {
    "target": "es5",
    "moduleResolution": "Node16",
    "strict": true,
    "alwaysStrict": true,
    "noImplicitAny": true,
    "allowSyntheticDefaultImports": true,
    "outDir": "dist",
    "listEmittedFiles": true
  },
  "include": ["src/**/*"]
}
```

이제 `package.json`에 들어가서 간단하게 스크립트를 작성해 줍시다.

```json
  "scripts": {
    "build" : "tsc"
  },
```

`src` 폴더를 만들고, 그 안에 `index.ts` 파일을 만들어서 아래와 같이 돌아가는 Apollo-server 코드를 작성해 줍시다.

```ts
import { ApolloServer, gql } from "apollo-server";

const typeDefs = gql`
  type Query {
    text: String
    hello: String
  }
`;

const server = new ApolloServer({ typeDefs });

server.listen({ port: 3000 }).then(({ url }) => {
  console.log(`Running on ${url}`);
});
```

이제 ts 파일을 빌드해서 확인해 봅시다.

```bash
$ yarn build
TSFILE: /home/kasterra/Project/graphql-api/dist/index.js
```

이런 결과를 확인할 수 있고, `yarn node dist/index.js`를 실행해서, 우리의 빌드 결과물을 확인할 수 있습니다. 왜 `node dist/index.js`를 하지 않냐면, 현재 `yarn berry`를 사용하고 있기에, npm이나 yarn classic 처럼 node_modules에 바로 모듈이 담겨있지 않고, zip 파일 안에 묶여있습니다. `yarn node`로 실행하면, 그런 모듈 resolving 과정을 해결해 주는게 아닌가... 싶습니다.

## nodemon 적용하기

create-react-app 기반으로 react개발을 해봤다면, 저장할 때마다 hot reloading이 되서 개발시 상당히 편했던 경험이 있을 것입니다. nodejs 기반 프로젝트에서 그런 hot reloading 기능을 nodemon이라는 프로그램이 담당하고 있습니다.

기본적으로 nodemon은 `.js`, `.mjs`, `.coffee`, `.litcoffee`, `.json` 확장자의 파일만을 지켜보다가 이들이 변하면, 서버를 재실행 해 주는 역할을 합니다. 우리는 TypeScript를 사용하기 때문에, nodemon에게 `.ts` 파일 역시 watch 해달라고 요청해야 합니다. `-e` 옵션을 통해서 확장자를 명시해 주면 됩니다. 당연한 이야기지만, ts파일이 변할때마다, 실행되어야 할 js 파일도 변경되어야 하니, 스크립트는 아래처럼 작성해야겠지요?

```json
  "scripts": {
    "build" : "tsc",
    "dev": "nodemon -e js,ts,json --exec 'yarn build && node dist'"
  },
```

이렇게 해서 저장한 다음 `yarn dev`를 실행하고, 터미널을 살펴보면, 무한하게 돌고 있음을 확인할 수 있습니다. 이유는 간단합니다.

```text
ts파일이 변경됨 => nodemon 작동해서 js 파일을 새로 빌드 => js 파일이 변경된것을 nodemon이 확인 => 또 빌드 => 반복...
```

이건 절대 우리가 의도한 바가 아니므로, nodemon에게 dist에 있는건 확인할 필요 없다고 알려줘야 하겠네요... `--ignore`을 이용해서, 무시할 패턴을 지정해 줄 수 있습니다.

```json
  "scripts": {
    "build" : "tsc",
    "dev": "nodemon -e js,ts,json --ignore dist/ --exec 'yarn build && node dist'"
  },
```

이제 무한루프를 돌지 않고, 우리가 ts 파일을 수정할 때만 다시 빌드가 되는것을 확인할 수 있습니다!

### nodemon.json에 담기

지금은 옵션이 몇개 없어서 원라이너로 처리가 가능해지지만, 만약에 옵션이 더 많아진다면? 상당히 읽는데 피곤해질 것입니다. 이런 상황에 대비해서 nodemon은 `nodemon.json`이라는 파일에서 옵션을 불러오는 기능을 갖추고 있습니다! 위의 명령을 json으로 바꾸면 아래와 같습니다.

```json
{
  "ignore": ["dist/"],
  "exec": "yarn compile && node dist",
  "ext": "js,json,ts"
}
```

그리고 script는 `nodemon`하나만 남게 되어서 아래와 같은 형태가 되겠습니다.

```json
  "scripts": {
    "build" : "tsc",
    "dev": "nodemon"
  },
```

## 빠른 빌드를 위한 babel 사용

위 코드는 정상 작동 합니다만, 솔직히 좀 답답하리만치 느립니다. 진짜 몇줄 되지도 않는 코드인데, 컴파일 하는데에 시간이 걸리지요. tsc가 타입 체킹을 하고, 현재 코드가 유효한 코드인지 검증하는 과정때문에 그렇습니다. 하지만, 우리가 개발 빌드를 만들때는, 일일히 타입 체킹을 하느라 시간을 쓰는것보다, 재깍재깍 코드의 결과물이 보이는 쪽이 좋지요. 그리고, 타입체킹은 vscode가 `tsconfig.json`을 보면서 라이브로 해주기 때문에, 사실 이렇게 되는것은 검사를 두번 하면서 시간을 낭비하는 꼴밖에 되지 않습니다!

react를 통한 프론트엔드 개발을 해오셨다면, 여러분은 여러분도 모르는 사이에 babel을 쓰고 계셨습니다. 정규 js 문법이 아닌 jsx를 브라우저에서 돌아가게 만들어준 프로그램이 babel 이거든요. 이 babel을 통해서 jsx는 물론이거니와, typescript와 같은 다른 언어도 정해진 규칙만 있다면, javascript로 변환되게 할 수 있습니다. 규칙만 정해지면 뭐든지 할 수 있어서, `F#`이라는 javascript와 상관없는 언어를 javascript로 바꿔주는 툴인 [Fable](https://fable.io/)이라는 툴도 있습니다.

여하튼 우리의 상황에서 babel을 사용하기 위해서 개발 의존성들을 설치해 줍시다. 꽤나 많으니 복붙해서 쓰시는걸 추천드립니다.

```bash
yarn add @babel/cli \
@babel/core \
@babel/node \
@babel/preset-env \
@babel/preset-typescript -D
```

이름이 상당히 직관적입니다. cli, core, node는 직관적이므로 생략하고, preset에 관해서 설명드리고 바로 설정파일 작성으로 넘어가겠습니다. 아까 babel은 '정해진 규칙에 따라' 코드를 변환한다고 했습니다. 그 정해진 규칙을 일일히 다 적는것은 너무나도 귀찮은 일이고, 같은 설정을 여러번 반복하는건 정말 끔찍한 일이기에 미리 세팅된 값들을 만들어서 사용하는데, 이것이 preset 입니다. 이제 우리가 설치한 이 개발의존성을 작업해 봅시다.

### .babelrc 사용하기

`nodemon.json`, `tsconfig.json`처럼 babel에서는 `.babelrc`라는 파일을 보고 설정을 읽습니다. 제가 작성한 `.babelrc`의 내용은 아래와 같습니다.

```json
{
  "presets": ["@babel/preset-typescript", "@babel/preset-env"]
}
```

그냥 typescript를 위한 preset과 최신 javascript코드를 위한 preset(preset-env)를 적용하면 끝!

### package.json에 적용하기

이제 babel 설정을 완료했으니 적용시켜 봅시다.

```json
  "scripts": {
    "build" : "babel --extensions '.ts' src -d dist",
    "dev": "nodemon"
  },
```

babel의 설정도 그렇게 어렵지 않습니다. `.ts`확장자인 파일을 src에서 찾아서 dist로 트랜스파일 하는 간단한 명령이 되겠습니다. 그리고 확인해보면, babel은 typescript 코드에 대해서 타입 체킹을 하지 않고, 그냥 재껴버리기 때문에, 훨 빠른 빌드 속도를 체험할 수 있습니다.

# 마치며

상당히 많은 시간을 쏟은 내용이었습니다. 사실 여기에는 나와있지 않지만, 저는 이 파이프라인을 만들기 위해서 `ts-node`라는 라이브러리도 사용을 검토해보고, esmodule로 작성을 하기 위해서 `package.json`에 `type:module`을 적었다가 무수한 `module not found` 에러를 마주치기도 하고(ts-node랑 type:module이랑 별로 친하지 않다 하더라고요...) 그리고 yarn berry라는 현업에서 분명히 쓰이지만, 기존 npm과 yarn classic 보다 자료가 적은 좁고 험난한 길을 택했기 때문에, 참고할 자료도 적었습니다. 삽질은 제가 다 했으니, 여러분은 제가 삽질한 시간만큼 소중한 가치창출을 더 할 수 있는 그런 시간절약을 위한 자료가 되었으면 좋겠다고 생각합니다. 끝까지 읽어주셔서 감사합니다. 🙏🙏
