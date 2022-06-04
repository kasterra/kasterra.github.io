---
layout: post
title: yarn berry로 React.js 프로젝트 시작하기
subtitle: 모던한 기술로 개발을 시작해 봅시다!
category: frontend
image: /images/thumbnails/yarn.png
---

# yarn berry의 필요성

오랫만의 포스팅입니다. 이런저런 일들이 있어서 블로그에 글을 남기지 못했는데, 잠시 여유가 나는 틈을 타서, 빠르게 '정리해야지'라고 생각했던 것들을 정리해보고자 합니다.

이번에 다룰 주제는 `yarn berry` 입니다. 저처럼 프론트엔드 개발을 하시는 분이거나, Node.js 기반으로 다른 무언가를 하시는 분이라면, `npm`과 같은 패키지 버전 관리 툴에 대해서 잘 알 수 밖에 없을 것입니다. 그리고, javascript 생태계에 관심이 있다면, npm이 패키지를 관리하는 방법에 문제가 꽤나 있다는 사실을 알고있는 분도 아마 있지 않을까 합니다. [토스 기술 블로그](https://toss.tech/article/node-modules-and-yarn-berry)에서 기존 npm의 문제점과, 사내 프로젝트에 yarn berry를 도입한 이야기에 관해 잘 정리해 놓았으니, 이 글을 읽기전에 읽어 보셨으면 좋겠습니다.

# 프로젝트에 실제로 적용하기

제가 링크한 글을 읽어보셨다면, yarn berry를 실제 프로젝트에서 써볼 마음이 조금이라도 드셨지 않을까 싶습니다. npm과 100% 호환되는 yarn classic 과는 다르게, yarn berry는(정확하게 PnP방식으로 모듈을 관리할 때에는) 몇몇 추가 설정이 필요하고, 여기에 대해서 일괄적으로 정리해놓은 한국어 자료가 검색해도 잘 보이지가 않아서, 저 스스로를 위해, 그리고 yarn berry를 React.js 프로젝트에 적용해보시려는 분들을 위해서 정리해보고자 합니다.

yarn berry는 기존 npm 또는 yarn classic 버전과 다르게, 모듈들을 `node_modules`에다가 관리하지 않습니다. 대신, 모듈들을 `zip`파일로 관리하여, `.yarn/cache`폴더에 두고, 해당 모듈들을 `.pnp.cjs`, `.pnp.loader.mjs`를 통해서 가져오게 됩니다. 이러한 특징 때문에, 몇몇 설정들이 추가적으로 필요하게 됩니다. 하나하나씩 알아보도록 합시다.

## 우선 프로젝트부터 만듭시다

React.js로 SPA 개발을 할 때에 필요한 보일러플에이트들을 자동으로 생성해주는 고마운 `create-react-app`을 이용해서 프로젝트를 시작해봅시다. 더욱 좋은 DX(Developer eXperience)를 위해서 `typescript`를 사용합시다.

```bash
npx create-react-app@latest my-proj --template typescript 
cd my-proj
```

`create-react-app@latest`를 한 이유는, `npx create-react-app`을 했을 때, global install을 지원하지 않는다고 하는 에러([참조](https://exerror.com/solved-we-no-longer-support-global-installation-of-create-react-app/))에 대응하기 위함입니다.

`npx`는 `npm`기반으로 돌아가는 명령어기 때문에, 너무나도 당연하게도 기존의 `node_modules`기반의 모듈 관리를 하게 됩니다. `yarn berry`기반이 아니죠. 그러므로 이렇게 생성한 프로젝트를 `yarn berry`기반으로 바꿔 줍시다.

기존의 npm 기반으로 되어 있는 의존성 관리들을 지우고...

```bash
rm -rf node_modules
rm -rf package.lock.json
```

yarn berry 버전으로 변경해 줍시다.

```bash
yarn set version berry
```

해당 명령어가 이상없이 적용이 되었다면, 레포지토리에 `yarnrc.yml`과 `.yarn/releases` 폴더 아래에 `yarn-berry.js` 또는 `yarn-(버전명).cjs`가 생성됩니다. 이후 yarn 버전을 확인했을 때, 1로 시작하는 classic 버전과 다르게 3 이상으로 시작하는 yarn berry가 설정되었음을 확인할 수 있습니다.

```bash
$ yarn --version
3.2.1
```

그리고 이제 `yarn install`을 해줍시다.

## yarn berry를 위한 vscode 추가 구성

### 의존성 파일을 열어보자. zipfs

yarn berry는 PnP 기능을 통해서 의존성을 관리합니다. 가끔 개발을 할 때, 타입이나 라이브러리 구현등을 잠깐 확인할 이유로 `node_modules`안에 있는 라이브러리 파일들을 열어볼 일이 가끔 있는데, PnP 에서는 zip 파일로 의존성들이 묶여있어서, 바로 학인할 수 없습니다. [zipfs](https://marketplace.visualstudio.com/items?itemName=arcanis.vscode-zipfs)Extension은 PnP를 사용하는 프로젝트에서 의존성 파일을 기존 방식처럼 쉽게 열어줄 수 있는 고마운 역할을 해줍니다.

### TypeScript를 위한 구성들

yarn berry의 PnP 기능을 사용할 때에는 TypeScript가 정상적으로 작동하도록 추가적인 구성이 필요합니다. 안전상의 이유로 VSCode에서는 TypeScript 설정을 명시적으로 활성화 해야 하고, yarn 측에서는 해당 Editor SDK를 배포하고 있습니다.

```bash
yarn dlx @yarnpkg/sdks vscode
```

이건 필수사항은 아니긴 한데, 자체적으로 types를 포함하지 않아서, `@types`로 시작하는 별도의 타입 정의를 설치해줘야 하는 작업을 자동화 해주는 플러그인을 설치하여 봅시다.

```bash
yarn plugin import typescript
```

이 설정들이 모두 완료가 되었다면, **에디터에서 TypeScript를 사용하는 파일을 연 상태에서** `ctrl + shift + p` 또는 `command + shift + p`를 눌러서 `TypeScript 버전 선택`을 검색하여, `작업 영역 버전 사용`을 선택하여 typescript sdk를 사용하도록 설정해 줍니다.

### eslint + prettier 설정

여럿이서 협업을 할 때에, 코딩 컨벤션과 스타일을 맞추기 위해서 사용하는 유용한 툴인 `eslint`와 `prettier`를 사용하기 위해서도 몇몇 추가 설정이 필요합니다. 우선 eslint 설정부터 합시다.

```bash
yarn dlx eslint --init
```

`yarn dlx`는 yarn berry에서 쓰이는 `npx`와 동치인 명령어 입니다. 여튼 해당 명령어를 통해서나 기존에 주로 사용하는 설정을 활용해서 eslint 설정을 해둡시다. 혹시, `node_modules`폴더나 `package-lock.json` 파일이 생성되었다면, 첫번째 세팅할때처럼, 해당 파일들을 지우고, `yarn`이나 `yarn install`을 실행하여서 yarn berry 버전에 맞게 의존성 관리를 하도록 합시다.

우리가 실제 개발을 할 때에는 prettier를 같이 사용할 경우가 꽤나 되는데, prettier에서 관리하는 스타일을 eslint에서도 확인하여서 충돌이 생기는 경우가 있을 수 있습니다. 이럴 때에 `eslint-config-prettier`를 사용하여, 중복 관리되는 스타일을 eslint에서 비활성화 할 수 있습니다.

```bash
yarn add eslint-config-prettier -D
```

그리고 해당 확장을 사용한다는것을 eslint에도 알리면 됩니다. `package.json`에다가 설정하는 방법이 있고, `.eslintrc.json`에 설정하는 방법이 있는데, 저는 개인적으로 후자를 선호합니다.

- `package.json` 파일에 설정하는 경우

```json
"eslintConfig": {
    "extends": [
      "react-app",
      "airbnb",
      "prettier" // 꼭 마지막에 추가
    ]
  },
```

- `.eslintrc.json` 파일에 설정하는 경우

```json
"extends": [
      "react-app",
      "airbnb",
      "prettier" //여기서도 꼭 마지막에 추가
],
```

#### prettier 확장이 안돌아 갈 때

PnP 기능을 사용할 때, eslint와 prettier의 바이너리 파일을 찾지 못하여(압축되어 있으니까...)prettier 확장이 올바르게 작동하지 않을 때가 종종 있습니다. 그리고 이는 prettier의 vscode 확장에 [이슈](https://github.com/prettier/prettier-vscode/issues/1502)로도 등록되어 있고, 해결 방법또한 제공되어 있습니다.

해당 글에는

```bash
yarn dlx @yarnpkg/pnpify --sdk vscode
```

라는 명령어를 사용하면 해결된다고 하는데, 사실 이거

```bash
yarn dlx @yarnpkg/sdks vscode
```

얘의 구버전 명령어 입니다. prettier, eslint 관련 설정을 하기 전에 해당 명령어를 입력해서 관련 설정이 되어 있지 않다면 한번 더 실행해주면 에러가 안납니다👍

### 간혹 생기는 문제 : Failed to load "react-app" to extend from...

프로젝트를 새로 만들면서 해당 문제를 재현하면서 같이 설명을 하려고 했는데, 어째서인지 재생산이 안되네요... 예전에 해당 문제를 겪었을 때, 도움을 받았던 [깃헙 이슈 링크](https://github.com/facebook/create-react-app/issues/10463) 정도 걸어두겠습니다...

## 마치며

React.js 등 node.js 생태계를 이용해서 하는 개발을 처음 배울 때에는 yarn berry에 대해서 배우지 않고, yarn berry의 특이한 특성 때문에, 개발 과정에서 간혹 에러를 마주하기도 합니다. 저도 처음에 yarn berry를 프로젝트에 도입하면서 참 많이 구글링을 했었고, 프로젝트를 여러개 만들어도 계속 헷갈려서 해당 문제를 해결하고, 커밋에 관련 링크들을 적어놨던 기억이 납니다. 현재 토스, 당근마켓 등의 많은 기업에서 프론트엔드 개발에 yarn berry를 사용하고 있어, 언제나 최신 기술에 대해서 거부감이 없어야 하는 프론트엔드 개발자로서 새롭게 yarn berry 도입을 고민하는 분들에게 조금이라도 도움이 되고자 이 글을 작성해 보았습니다.

끝까지 읽어주셔서 정말 감사드리며, 혹시 yarn berry로 프로젝트를 구성하는 중에 이 글에 명시되어 있지 않은 에러가 발생했다면 주저없이 댓글 부탁드립니다. 다시 한번 끝까지 읽어주신 여러분께 감사드립니다.🙇‍♂️🙇‍♂️
