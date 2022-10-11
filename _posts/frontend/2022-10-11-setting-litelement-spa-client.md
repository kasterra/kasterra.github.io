---
layout: post
title: Lit로 SPA 만들어보기 ② - FE
subtitle: 굳이 React 안쓰겠다고 삽질한 이야기
category: frontend
image: /images/thumbnails/lit.png
---

# 개요

[저번 글](/setting-litelement-spa-server/)에서 왜 이런 스택을 선정하였는가에 관한 이야기를 하였으니 이번 글에서는 무미건조 하지만 글의 분량을 위해서 빠르게 가보겠습니다.

클라이언트 사이드 세팅 이라는 이름으로 묶었지만, 꽤나 많은 정보들을 넣을 수 있을 것 같습니다. 크게 나누자면 웹팩 이야기와 SPA 세팅 이야기로 나눌 수 있을것 같네요.

# FE의 약방의 감초! webpack 설정

webpack이라는 친구는 상당히 많은 역할을 합니다. 원래 JS에서는 JS 파일만 모듈로 불러올 수 있지만, 그 한계를 풀어주고, 기존의 파일을 개발자가 원하는 방향으로 변형시켜서 사용할 수 있는 loader 들을 적용해서 사용할 수 있지요. 그리고 loader 뿐만 아니라 여러 plugin을 사용해서 더 많은 일도 할 수 있습니다. 제가 이 프로젝트를 세팅하는데 사용했던 설정에 대해서 이야기를 해보겠습니다.

webpack은 node에서 돌아가는 친구이기 때문에 cjs로 작성을 해야합니다. 우선 제가 작성한 코드를 보여드리고, 설정들을 설명드리겠습니다.

- [webpack.dev.js gist 링크](https://gist.github.com/kasterra/afee218b61cbb09d4761870fdfbac936)
- [webpack.prod.js gist 링크](https://gist.github.com/kasterra/f0b3ac69b77ecde461fb2a61b1bded93)

## 0. webpack entry, output 설정

```javascript
entry: {
    bundle: './src/client/index.ts',
    elements: './src/client/global/allComponents.ts',
},
output: {
    filename: '[name].js',
    path: path.resolve(__dirname, '../dist/static'),
    publicPath: `${process.env.ADDR}/static`,
    clean: true,
},
```

webpack은 특정한 파일에서 출발해서 해당 파일에서 의존성으로 등록된 친구들을 등록하는 기능을 하고 있다는 것을 알고 계실겁니다. webpack을 처음 시작할때 거의 국룰인 세팅으로 시작하고, 여기에 있는 `process.env.ADDR`은 이제 실제 배포와 dev 할때의 주소가 다를 것이니 그 부분에 대해서 대응할 수 있도록 환경변수로 빼놓았습니다.

## 1. FE 사이드 typescript 설정

우선 바로 이해할 수 있는 typescript 부분 코드입니다.

```javascript
{
  test: /\.ts$/,
  use:[{
    loader: "ts-loader",
    options: {
      compilerOptions: {
        target: "es6",
        moduleResolution: "node",
      },
    },
  }]
}
```

ts 확장자를 가진 typescript 파일을 `ts-loader`라는 로더를 통해서 트랜스파일링과 타입체크를 해주는 매우 간단한 로직이다. 그런데 여기 `options` 부분에, `compilerOptions`를 오버로딩하는 듯한 그런 옵션이 있습니다.

`target:"es6"`는 말 그대로, 타겟을 es6로 하겠다는 것입니다. [전에 작성했던 web component에 대한 글](/what-is-web-component)에서 지나가듯이, 트랜스파일링을 할 때, es6이상으로 변환해야 한다는 언급을 한적이 있었다. 클래스 문법 관련해서 생기는 이슈 때문이고, 여러분들이 잘 알듯이, JS에서 클래스 문법이 도입된것은 ES6때 부터이다. 그러면 `tsconfig.json`에서 `target`을 바꾸면 되지 않느냐는 질문을 할 수도 있지만, 아쉽게도 서버 사이드에서는 모듈 resolving 문제 때문에 ES5로 트랜스파일링 해줘야 했습니다. 물론 이는 나의 짬밥이 낮아서 생기는 문제일수도 있겠으나, 그 당시 나도 열심히 노력을 해보았지만, 잘 되지 않았던 기록을 다시 한번 [링크](/setting-node-ts-apollo-server/)해 놓을까 합니다.

`moduleResolution`은 말 그대로, `import` 등을 통한 module을 불러올 때, 이를 어떻게 resolve하는지에 대한 룰을 지정하는 옵션입니다. 기본적으로 typescript에서 `moduleResolution`을 지정하지 않으면, `Classic` 이라는 방법으로 module resolving을 실시하는데, 이는 `node_modules`를 검색하지 않고, module resolving을 시도하기 때문에, `node_modules`에 설치된 외부 의존성을 찾지 못하는 경우가 생깁니다. 여기에 대한 자세한 지식은, 내가 이 문제를 해결하는데 도움을 준 [블로그 글 링크](https://chiabi.github.io/2018/08/30/typescript/)를 참조하면 좋을 듯 합니다. 지면을 아껴야 하니까요.

## 2. styleSheet(scss) 설정

다음은 css 전처리기를 사용한다면 흔하게 볼 수 있는 styleSheet관련 webpack 설정입니다.

```javascript
{
  test: /\.scss$/,
  use: [
      'lit-css-loader',
      {
          loader: 'sass-loader',
          options: {
              sourceMap: true // dev 빌드의 경우
              sassOptions: { // prod 빌드의 경우
                  outputStyle: 'compressed',
              },
          },
      },
  ],
}
```

dev 빌드의 경우에는 원활한 디버깅을 위해서 `sourceMap`을 켜주고, prod 빌드의 경우에는 compress를 하여 더욱 날씬한 파일을 서버에서 제공한다라는 너무나도 간단한 이야기 입니다.

그런데 여기서 `lit-css-loader`라는 새로운 친구가 나옵니다. 일반적인 프로젝트에서 scss을 사용하기 위해서는 `sass-loader`다음에 `css-loader`가 나오고, 그 다음에 이 파일을 헤더에 넣기 위해서 `style-loader`를, 또는 별개의 파일로 분리하기 위해서 `MiniCssExtractPlugin.loader`을 사용하죠. 우리는 이 프로젝트에서 `lit`을 사용할 것이고, 지금은 lit에 대한 자세한 설명을 하지 않아서 자세한 이야기를 하기엔 좀 복잡하지만, 간단하게 설명하자면, lit에서 기본적으로 styleSheet를 다루기 위해서는 컴포넌트를 정의하는 js 또는 ts 파일 내에서 정의하지만, 파일 관심사 분리를 위해서 css, 혹은 scss 파일로 분리를 하기 위해서 사용되는 loader라고 보시면 되겠습니다.

## 3. html 파일을 만드는 `HtmlWebpackPlugin`

다음은 HTML 파일을 만드는 플러그인인 `HtmlWebpackPlugin` 입니다.

```javascript
plugins: [
    new HtmlWebpackPlugin({
        template: 'src/client/index.html',
        chunks: ['bundle'],
    }),
],
```

MPA로 개발을 할 때는 굳이 이런거 안만들어도 상관은 없었지만, SPA의 경우에는 이야기가 다릅니다. 처음 서버측에 던져줄 index.html이 필요하고, 이것은 서버에서 빌드할 때 함께 만들어 주면 좋거든요. (근데 막상 이 글을 적으면서 예전에 적었던 코드들을 읽어보니 굳이 적을 필요는 없었네요... 그냥 template에 다 적어서 static하게 서빙하면 되는데...)

`HtmlWebpackPlugin`을 통해 html 파일을 만들어 주고, `chunks` 옴션으로 지정된 번들 파일만 html에 삽입해 줍니다. 이 플러그인의 옵션에 대해서 더 알고 싶으신 분은 [해당 플러그인의 repo의 readme](https://github.com/jantimon/html-webpack-plugin#options)를 참조하면 좋을 듯 합니다.

## 4. 나머지 설정

```javascript
experiments: {
    topLevelAwait: true,
},
devtool: 'source-map',//dev 설정에서만
resolve: {
    extensions: ['.ts'],
},
```

module로 불러온 javascript는 top level에서 await를 시전할 수 있고, 이것이 상당히 코드 로직을 작성하는데에 도움이 되는 부분이 많기 때문에 명시적으로 켜주었습니다. `source-map`을 활성화 해줌으로서, 우리는 디버깅을 할 때, 트랜스파일된 우리가 적은적도 없는 JavaScript코드를 보는것이 아닌, 매핑된 typescript코드를 볼 수 있지요. 그리고 마지막에 `resolve`의 `extension`이라고 적힌 부분이 낯선 분도 계실겁니다. 이는 TypeScript 환경에서 ts 파일을 import 할 때, 확장자인 .ts를 붙이면, 에디터 차원에서 .ts를 떼라고 경고합니다. 그렇게 해서 떼놓으면, 확장자를 기준으로 의존성을 해석하는 webpack이 혼란에 빠지게 되지요. 이런 상황에서 혼란에 빠지지 말고, module resolving을 할 때, extension이 없으면 차례대로 아래의 것들을 시도해 보라면서, 후보군을 배열의 형태로 넘겨주는 것 입니다.

# 본격적인 SPA 설정(feat `lit`)

이 글의 제목이 lit로 **SPA 만들기** 인데, SPA를 위한 기초 설정에만 너무 많은 지면을 사용했습니다. 사실 이 설정들을 하는데 저도 너무 많은 시간을 소모했기도 했고요. 이 글을 읽는 여러분들은 저처럼 이런데에 시간을 소모하시지 마시고, 진짜 핵심 로직이라고 할 수 있는 부분에 시간을 투자할 수 있길 바라는 측면에서 최대한 담백하게 이 내용들을 적어 보았습니다. Vanila JS로 SPA를 만드는 글들은 다 있었는데, Web component를 사용한 SPA를 만드는 법에 대한 글을 너무나도 찾기 힘들었고, 그 결과로 저는 열심히 삽질을 했지요.

Vanila JS로 SPA를 만드는 것에 대한 글은, 프로그래머스에서 출제된 적이 있는 'VanilaJS로 SPA 쇼핑몰 만들기' 등의 검색 키워드들로 많이 찾을 수 있습니다. 저도 그런 글들 많이 읽어보았고, 해당 글에서 진행하는 흐름대로 이 글들을 진행해볼까 합니다. 그리고 많은 분들에게 익숙할 CRA(Create React App)에 대한 언급 역시 좀 잦을 예정이니 글을 읽는데에 참고가 되었으면 합니다.

## 5. 서버에서 날려줄 html 작성

CRA로 처음 부트스트래핑을 하면, 별 내용 없는 `index.html`이 `public` 폴더 내에 생성되는 것을 보실 수 있을 것 입니다. 우리도 날려줄 html 파일을 작성해 봅시다.

**index.html**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <script src="/static/elements.js" type="module"></script>
    <title>Single page app</title>
    <style>
      body {
        width: 320px;
        border: 1px solid black;
        margin: auto auto;
      }
    </style>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

여기에서 `moudle` 타입을 명시해서 넣어준 `elements.js`는 [위에서](/#0-webpack-entry-output-%EC%84%A4%EC%A0%95)언급한 각 element들이 의존성으로 등록되어 있는 파일들을 번들링한 물건입니다. 이 친구는 **꼭** `module`로 가져와 져야 하기 때문에 이러한 형태로 적게 되었죠.

## 6. SPA와 떼놓을 수 없는 CSR 구현

제목에도 적었듯, SPA를 구현하고자 하면 CSR역시 함께 구현하게 됩니다. SSR이 지원되는 SPA역시, 처음에 서버에서 날라오는 html 파일이 빈 파일이 아니라 뭔가 좀 더 들어있는 파일이 온다는 것이지, CSR을 아예 하지 않는다는 뜻은 아니거든요. (SSR이 포함된 SPA에 관해서는 나중에 글을 작성해보도록 하겠습니다)

### 6-1 router.ts 작성하기

우선 `router`가 무엇을 하게 될지 간단히 생각해 봅시다. 한줄로 요약하자면, **현재 페이지의 주소에 맞는 컴포넌트를 렌더링** 하는 역할 입니다. 그 명세 그대로 작성을 해보도록 하겠습니다.

**router.ts**

```typescript
export default function router(to?: string) {
  const root = document.getElementById("root") as HTMLElement;
  const routes = [
    { path: "/", view: "home-component" },
    { path: "/new-product", view: "new-product" },
    { path: new RegExp("\\/product\\/\\d+"), view: "product-component" },
    { path: "login", view: "login-page" },
  ];
  const viewName = routes.find((route) => {
    if (typeof route.path === "string")
      return route.path === (to || location.pathname);
    else if (route.path instanceof RegExp) {
      return route.path.test(location.pathname);
    }
  })?.view;
  root.textContent = "";
  if (!viewName) {
    root.appendChild(document.createElement("not-found"));
    return;
  }
  root.appendChild(document.createElement(viewName));
}
```

사실 이 코드는 완전히 건전한 코드라고 볼 수는 없지만, 이해하는데에는 딱히 문제가 없고, 이 코드의 문제를 해결하여야 겠다는 사고를 통해서, 많은 것을 얻을 수 있었기에 일부로 이 코드로 설명을 하겠습니다.

이 함수 내에서 `routes`라는 객체 안에 각 path에 대응되는 뷰들이 매칭되어 있습니다. path는 정적인 문자열일수도 있고, 특정한 패턴을 의미하는 정규 표현식일수도 있죠. 현재 URL인 `location.pathname`을 이 객체들과 비교해서 확인해서 알맞는 viewName을 찾아서 `root`에 `appendChild` 해주는것을 볼 수 있습니다.

문제가 몇가지 보이는군요. 첫째로는 `root`에 렌더링 하는 것까지 router가 책임지고 있다는 것입니다. `root.textContent=""`라는 코드를 통해서 root를 다시 렌더링하는것까지 router가 책임지고 있는 것을 볼 수 있죠. 이를 막기 위해서는 CRA에서 하는것처럼, `App`이라는 컴포넌트를 작성하여서, router에서는 해당 컴포넌트를 내어주기만 하고, 컴포넌트를 렌더링하는것은 `App` 컴포넌트의 책임으로 하면 됩니다.

두번째 문제로는 특정 경로로 이동하는 상황과, 현재 URL이 가리키는 페이지로서의 라우팅을 모두 `router`라는 한 함수에서 담당하고 있다는 것입니다. 이는 유명한 React의 서드 파티 Routing 라이브러리인 `React Router`에서 하는것처럼 `navigate()` 함수를 별도로 분리하는 식으로 해결할 수 있겠네요.

### 6-2 `router`가 호출될 eventListener 등록하기

이렇게 `router`를 잘 작동하게 만들어놔도, 불려지지 않으면 아무런 소용이 없습니다. 브라우저는 고전저긴 서버 사이드 렌더링에 맞춰져있기 때문에, 클라이언트 사이드 렌더링을 위해서 우리는 `router`를 만들었고, 이 `router`가 필요할 때 불려져야 할 필요가 있습니다. 페이지가 로딩되었을 때(`DOMContentLoaded`) 그리고, 브라우저의 이전/다음 버튼을 눌렀을 때(`popstate`)죠. 특히 `popstate`에 관해서는 **기본값이 지정**되어 있기 때문에, `e.preventDefault()`를 해줘야 한다는 것에 유의해 주세요

**index.ts**

```typescript
import router from "./lib/router";

window.addEventListener("popstate", (e) => {
  e.preventDefault();
  router();
});

document.addEventListener("DOMContentLoaded", () => {
  router();
});
```

### 6-3 spa에서 사용하는 `Link` 컴포넌트 만들기

CRA + React Router 조합으로 개발을 해보신 분이라면, 또는 `Next.js`로 개발을 해보신 분이라면, 라우팅을 할 때, `<a>`태그를 사용 지양해야 하는 이유를 분명 들어보셨을 것입니다. `<a>`로 라우팅을 하면, 기본적으로 페이지가 새로고침되고, 서버에서 정보를 불러오게 됩니다. 이는 CSR을 하는 웹앱에서 별로 좋지 않은 것이기에, CSR을 사용하는 솔루션에서는 별도의 `Link` 컴포넌트를 사용해라고 권고합니다.

우리가 만드는 spa 역시 그러합니다. 이유 역시 동일합니다. 우리의 spa에서 `Link`역할을 해줄 `<spa-link>` 컴포넌트를 만들어 봅시다.

**spaLink.ts**

```typescript
import { html, LitElement } from "lit";
import { customElement, property } from "lit/decorators.js";
import router from "../lib/router";

@customElement("spa-link")
class spaLink extends LitElement {
  @property()
  to = "/";

  handleClick = (e: MouseEvent) => {
    e.preventDefault();
    history.pushState(null, "", this.to);
    router(this.to);
  };
  protected render() {
    return html`<a href=${this.to} @click=${this.handleClick}
      ><slot></slot
    ></a>`;
  }
}
```

이 글은 Lit의 문법을 설명하는 글이 아니고 Lit을 이용해서 SPA를 설명하는 글이기 때문에 간략하게만 설명하겠습니다.`@customElement("spa-link")`는 해당 데코레이터 아래에 있는 클래스를 인수에 있는 이름인 `spa-link`로 등록하겠다는 일종의 Lit만의 shorthand 입니다. 그리고 `@property()`는 아래에 있는 클래스 프로퍼티를 customElement 상에서 attribute로 노출 될 수 있는 인터페이스를 제공하겠다라는 뜻입니다.

여기서 가장 중요한 부분은 **handleClick** 일 것입니다. 해당 앵커를 클릭했을 때, 기본적으로 브라우저에 할당된 그런 동작을 막고(`preventDefault`) 주소창을 바꾸어 줍니다(`history.pushState()`). 그런데 단순히 `pushState`를 하는것 만으로는 `popstate`이벤트가 일어나지 않아서, 명시적으로 라우터를 호출해 주고 있습니다. 만약에 위에서 router를 작성하면서 아쉬웠던 부분이 개선이 되었다면, `navigate()`라는 함수로 이 동작이 래핑되어 있겠죠.

# 개발은 계속되지만...

이제 우리는 SPA의 코어 부분을 구현한 상태입니다. 이제 할것은 직접 컴포넌트와 페이지를 작성해서 서비스를 점점 확장해 나가는 것이죠. 하지만, 그 내용은 Lit의 사용법에 더 많은 지면을 할애해야 하고, Lit의 기능은 한두가지로 간단히 요약될것이 아니고, 또 CustomElement의 명세와 함께 설명해야 할 것이 있기 때문에, 이 글에서 설명하기에는 좀 내용이 아니지 싶습니다.

조만간 시간과 여유가 난다면, 한국어로 쓰여진 글이 별로 없는 `Lit` 라이브러리의 사용법, 잘 사용하는 법, 배경 지식들을 다루는 글들을 이전에 React 관련해서 작성하였던 글처럼 작성해볼까 합니다.

제가 알기로서는 한국어로 된 아티클들 중에선 `lit`으로 SPA를 작성하는 글에 대해서 찾아 볼 수 없어서 제가 아마 선두주자 중 한이 된 영광의 선봉대원중에 한명이 아닐까 하고 감히 추측해 봅니다. 그 말인 즉슨, 비교할 대상이 없기 때문에, 정확하지 않은 정보가 있을 수도 있다는 사실입니다. 혹시 이 글을 읽으시다가, '어 내가 아는 지식이랑 다른것 같은데?' 하는 내용이 있다면 지체없이 코멘트를 남겨주시면 최대한 빠른 시간 내에 확인 해보고 답변을 드리고, 혹시 잘못된 내용이었다면 빠르게 이 글을 수정할 수 있도록 하겠습니다.

언제 돌아온다고 약속은 못드리겠지만, 다음 글에는 `Lit`을 중점적으로 다루는 글로 돌아오도록 하겠습니다. 끝까지 읽어주셔서 감사합니다!!!
