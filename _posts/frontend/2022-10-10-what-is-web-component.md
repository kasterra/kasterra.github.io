---
layout: post
title: Web component란 무엇일까요?
subtitle: 많이 들어는 봤는데 제대로 알지 못하던 webcomponent. 알아봅시다.
category: frontend
image: /images/thumbnails/webcomponent.png
---

# 개요 - 컴포넌트

만약 본인이 프론트엔드쪽에 경험이 있다면 '컴포넌트' 라는 말을 분명 들어보셨을 겁니다. 특히 React 공부를 하셨다면, 컴포넌트라는 말을 듣지 않을 수 없죠. 리액트 입문 강의에서도 리액트의 장점으로 컴포넌트를 통한 재사용성을 끌어올려서 개발에 이점을 가져갈 수 있다는 설명을 분명 들어보셨을 겁니다.

틀린 말은 아닙니다. 하지만, 이번 글에서 다뤄볼것은 컴포넌트의 개념보다는, `Web Component`라는 기술에 대해서 다루어 보고자 합니다. 부스트캠프 미션에서 새로운것을 해보고 싶다는 막연한 생각으로 시작했던 공부였는데, 생각보다 이 지식의 깊이가 꽤나 깊다라는것을 알게 되고, 이 지식을 온전히 저의 것으로 만들기 위해서 이렇게 글을 써봅니다.

# Web component란?

`Web component`는 어떠한 라이브러리나 프레임워크가 존재하지 않는 바닐라 JS 환경에서 컴포넌트를 구현하기 위한, **명세의 집합체** 입니다. React나 Vue, Angular 등에서도 컴포넌트를 각자 자기네 방법으로 구현하지만, 해당 라이브러리나 프레임워크를 설치 하지 않은 환경에서는 사용할 수 없지요. 반면 Web component는 Vanila JS의 명세의 집합이기 때문에, 외부 의존성 없이 사용할 수 있고, 위에서 언급한 라이브러리나 프레임워크들도 결국은 JS 코드의 응용들이기 때문에, Web component로 작성된 컴포넌트는 위에서 언급한 라이브러리나 프레임워크 등지에서도 사용할 수 있습니다. (예시 : [React에서 사용하는 web component](https://ko.reactjs.org/docs/web-components.html))

# Web component를 이루는 기술

web component는 4가지 스펙의 집합체입니다.

- HTML Template
- Custom Element
- Shadow DOM
- ES Module

단순히 이렇게 키워드만 던지면 무슨말인지 당연히 알 리가 없겠죠. 각 키워드별로 알아보도록 하겠습니다.

## 1. 렌더링 되지 않는 HTML : `<template>`

HTML에는 생각보다 많은 태그가 존재합니다. 이번 섹션에서 다룰 것은 그 많은 태그 중 하나인 `<template>` 입니다. 일반적으로 HTML은 렌더링을 위한 기초 뼈대를 만들어내는 마크업의 기능을 가지고 있지만, 이 `<template>`라는 친구는 **렌더링이 되지 않습니다**. 브라우저에서 읽기는 하지만, 렌더링을 위해서 읽는 것은 아니고, 나중에 사용될것에 대비해서 유효성을 검증하기 위해 미리 한번 읽어 보는 수준 정도지요. 그래서 페이지를 렌더링 할 때 DOM 구조에도 포함되지 않습니다.

그럼 렌더링 하기 위해서 사용하는 HTML에 렌더링 되지 않는 이 태그는 과연 왜 존재하는 것일까요? template라는 단어 의미 그대로 입니다. 나중에 생성될것을 대비해서 미리 템플릿을 만들어 두는 용도입니다. React에 익숙하시거나, VanilaJS에 좀 빠삭하다 하시는 분은 아마 fragment라는 개념을 들어보셨을 것입니다. React에서는 `<React.Fragment>`나, 축약한 형태인 `<></>`을 보셨을 것이고, Vanila JS에서는 `document.createDocumentFragment()`라는 DOM API를 보셨을 것입니다. 이것들은 DOM 구조를 생성하지는 않지만, DOM에 append 될 때, 자연스럽게 그 DOM에 합쳐지기 위해서 존재하는 것이지요.

`<template>`도 그러한 용도입니다. `id`를 지정해두고, `querySelector`등의 방법을 통해서 해당 마크업을 가져와서 사용할 수 있게 하는것이죠. [mdn 독스](https://developer.mozilla.org/ko/docs/Web/HTML/Element/template)에서는 테이블에서 사용할 열들을 손쉽게 만들기 위해서 `<template>`을 사용하는 예제를 보여주고 있습니다. 그리고 `<template>` 태그와 함께 사용되는 짝꿍 태그인 `<slot>`에 관한 내용들을 다루는 [mdn 독스(en)](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_templates_and_slots)역시 첨부합니다.

## 2. 내가 만들어서 쓰는 태그 : Custom Element

웹 서비스가 아니더라도, 코드를 작성하다 보면, 여러 곳에서 동일하거나 비슷한 코드를 반복해야 할 일이 생길 수 있습니다. 이러한 코드들을 우리는 함수나 클래스라는 이름으로 분리를 해서, 여러 곳에서 사용할 수 있도록 하죠. 정확한 설명은 아니지만, 코드 덩어리들에 이름을 붙이는 작업이지요. HTML에서도 이런 '이름을 붙이는 작업'을 할 수 있으면 좋지 않을까요? 그런 수요를 해결해주는 기술이 Custom Element 입니다.

Custom Element는 생각보다 다양한 일들을 할 수 있습니다. 단순히 여러 마크업 뭉치를 하나로 묶는 역할만 하는것처럼 위에서 설명하였지만, 사실 web component가 component라고 불릴 수 있는 가장 핵심적인 역할을 제공하는 기술이라고 생각합니다.

Custom Element는 아예 새로운 Element를 만들 수도 있고, 기존의 Element를 확장할 수도 있습니다. 그리고, React 등의 UI 라이브러리나 SPA용 프레임워크를 사용하셨던 분들이라면 들어봤을 **생명주기 메소드** 같은것도 가지고 있지요. Vanila JS에서 작성하는 Custom Element는 React에서 작성하는 클래스형 컴포넌트와 유사한 형태를 가지고 있습니다.

Custom Element의 사용법부터 생명주기 메소드까지 모든것을 다루기 위해서는 내용이 너무 많아져서 간략하게만 다루고, 넘어가겠습니다. 우선 custom Element는 `HTMLElement` 또는 그 자식들을 상속하는 `class`의 형태로 JS에서 선언되어야 합니다. 그리고 안에 들어갈 마크업과 스타일들을 적용하고, 해당 컴포넌트가 기능하기 위한 스크립트들도 작성해서 넣을 수 있지요. 아까도 언급했듯, 생명주기 메소드(`connectedCallback`, `disconnectedCallback`, `adoptedCallback`, `attributeChangedCallback` 등)를 통해 특정한 상황에서 컴포넌트가 반응할 수 있도록 하는 로직 등도 작성이 가능합니다.

주의할 사항이라면, 이렇게 class 문법으로 작성된 코드를 Babel, Typescript등의 트랜스파일러를 통해서 ES5이하로 변환시 제대로 트랜스파일이 되지 않을 수 있음에 유의하여야 합니다. ES6 이상으로 컴파일 하거나, [babel-plugin-transform-builtin-classes](https://www.npmjs.com/package/babel-plugin-transform-builtin-classes) 라는 것을 이용할 수 있다고 하는데, 요즘 브라우저들은 ES6이상 다 지원하기 때문에(돌아가신 IE11도 지원합니다) 굳이 ES5 이하로 트랜스파일 할 이유가 있지 싶네요.

좀 많이 줄인 이 내용이 아쉬우신 분은 [mdn 독스](https://developer.mozilla.org/ko/docs/Web/Web_Components/Using_custom_elements)를 읽어보시는게 어떨까 싶네요.

## 3. 무지성 전역 멈춰! : Shadow DOM

프론트엔드 개발을 하다 보면, 참 많은것이 전역(global)이라는 것을 느낄 수 있을것 같습니다. CSS도 전역이고, document에서 발생하는 이벤트들도 root까지 버블링 되죠. 이러한 문제점을 해결하기 위해서 참 많은 방법론들과 기술이 개발되어 왔습니다. CSS의 경우에는 BEM 이라는 컨벤션이 만들어지기도 하고, CSS module이라는 기술이 만들어지기도 했죠.

Shadow DOM도 이러한 전역성(?)에 대처하는 방법 중 하나입니다. web component는 언제 어디서나 쓰일 수 있는 컴포넌트를 만드는데에 사용되는데, 다른데서 사용하는 CSS나 이벤트가 해당 컴포넌트에 들어가서 오염이 된다면 해당 부분을 처리하기 참 곤란할 것입니다. 해당 컴포넌트를 사용하는 쪽이 곤란해지건, 아니면 해당 컴포넌트를 작성하는 쪽이 곤란해지겠죠. 해당 DOM API를 사용하는 방법에 관해서는 [mdn 문서](https://developer.mozilla.org/ko/docs/Web/Web_Components/Using_shadow_DOM)를 첨부해 두도록 하겠습니다.

## 4. JS를 분리해 주세요 : ES Module

개발을 하면서 '모듈화' 라는것이 참 중요하다라는것을 부정하실 수 없을겁니다. 하나의 monolithic한 파일보다는, 각자 자신의 일만 잘하는 여러 함수로 분리된 코드가 읽기도 좋고, 유지보수 하기도 쉽죠. JavaScript는 10일만에 뚝딱 해버린 근본리스한 언어 답게 처음 탄생할때는 제대로 된 모듈 시스템이 없었습니다. 그래서 참 많은 모듈 시스템이 탄생하였습니다. Amd, CommonJS, RequireJS 등등... 이러한 명세들이 군웅할거 하는것은 언어 생태계 차원에도 그닥 좋지 않은 일입니다.

JS의 표준을 정의하는 ECMA 인터네셔널에서 ES Module이라고 불리는 사양을 발표하였고, 여러 프로그래머에게 익숙할 `import`, `export` 등을 사용하는 그런 모듈 시스템을 사용할 수 있게 되었죠. `node.js` 에서는 `package.json`에 `type:module`을 정의해서 사용할 수 있고, 브라우저 사이드 JS에서는 `<script type="module">`을 이용해서 ES Module을 사용할 수 있습니다.

찝찝하다고요? 그럼 읽을거리 몇개를 남겨놓겠습니다.

- ECMAScript? JavaScript? 이게 무슨 말인가요? : [링크](https://wormwlrm.github.io/2018/10/03/What-is-the-difference-between-javascript-and-ecmascript.html)
- 참 많은 모듈 시스템이라고 압축해 놨지만, 각각의 특징을 알고싶어요 : [링크](https://d2.naver.com/helloworld/12864) (2015년도 글이라서 좀 옛날 글입니다.)

---

이 기술들은 모던 브라우저에서 전부 stable로 지원이 되는 기술들입니다! 이제 이 4가지 명세들을 이용해서 실제로 개발에 어떻게 써먹는지에 대한 이야기들을 해봅시다!

# Web Component 생태계

## web component는 처음이야? 우리가 알려줄게

아무리 명세를 기가 막히게 만들어놔도, 이걸 쓰는 사람이 없다면, 이 기술을 기반으로 사용하기 좋은 라이브러리를 만들어 놓은것이 없다면, 해당 기술의 가치는 높게 평가되기 힘들 것입니다. [webcomponents.org](https://www.webcomponents.org/)라는 곳에서는 webcomponent를 사용하는 여러 라이브러리들을 소개해주고, web component에 대해서 쉽게 사용할 수 있도록 여러 자료들을 소개해 주고 있습니다.

## 귀찮은 보일러플레이트들 슥삭! 배포도 슥삭! @open-wc

[open-wc.org](https://open-wc.org/)라는 곳에서도 비슷한 것을 제공해 주는데, 여기서 배포하는 `@open-wc` 라는 친구가 web component기반의 페이지나 라이브러리로 배포할 수 있는 컴포넌트를 만드는데 필요한 코드베이스를 만들어주는 generator를 배포하고 있습니다. web component를 연습하는데에 유용했던걸로 기억합니다.

## 좀더 편하게 쓰자! 라이브러리

위에서 말헀던 open-wc에서 web component를 기반으로 구성된 [ui 라이브러리를 소개](https://open-wc.org/guides/community/component-libraries/)해주고 있습니다. 저는 이것들을 다 사용해보지는 않았지만, Google에서 만든 LitElement라는 라이브러리가 참 마음에 들었었는데, TypeScript Decorator 등의 문법을 이용해서 코드 양을 비약적으로 줄이는데 도움을 받았던것 같습니다. 바닐라 JS로 개발하는것보다 개발 생산성이 더 올라가는 느낌이 확실히 있습니다.

# 이걸 왜 배워야 할까?

이걸 앞쪽에 배치할까 하다가, 내용을 구상하면서 생각해보니, 좀 찬물을 끼얹는듯한 내용들이 많아서 약팔이를 위해(?) 일부로 뒤에 배치했습니다. Web component는 아까도 말했듯이 **특정 라이브러리에 종속되지 않습니다**. 그래서 한번 만들어 놓으면 다양한 개발 환경에서 돌려쓸 수 있는 라이브러리를 만들어 놓을 수 있지요. [Github 서비스도 web component로 만들어져있고](https://github.blog/2021-05-04-how-we-use-web-components-at-github/) github에서 사용하는 web component도 [오픈소스로 배포](https://github.com/github/github-elements) 하고 있습니다. Google의 일부 서비스도 web component로 작성되어 있다고 하고요.

사실 이 글을 읽으시는 분들은 한국어를 하시는 분들이기 때문에, '국내에서 사용하느냐'가 많은 관심 사항일것이라고 생각하는데, 여러 기업 기술블로그 등을 구글링 해봤을 때, web component를 사용한다고 나오는 글은 지금 당장은 본 적이 없고, 현업 개발자 한분에게 여쭤본 결과, web component를 통해서 디자인 시스템을 만들어보자고 *건의*는 해봤지만, 이게 실질적으로 수리되지는 않았다 라고 합니다...

그래도 마냥 부정적으로 볼 수 만은 없는것이, 몇년전 웹 개발의 표준이라고 불렸던 Jquery는 95% 정도의 점유율을 해먹었지만, 지금은 지는해가 되고 있고, 현재의 대세인 React도 지금은 정말 승승장구 하고 있지만, React라고 해서 Jquery처럼 지는 해가 되지 않으리라는 보장은 어디에도 없습니다. **변하지 않는 기술**인 VanilaJS에 들어있는 web component를 알아서 나쁠것 없다는 생각이 듭니다. 그리고 지금 당장은 국내에서 쓰이지 않는것 같다고 하지만, 평생 국내에서만 일해야 한다고 규칙이 정해져 있는것도 아니고, 미래의 일은 100% 맞다고 예측할 수 없는 것이니까요.

# 마치며

이번 글은 제가 잘 알고 있던 내용을 작성했던 이전 글들과 달리, 이번주에 개발 시간을 줄여가며 공부한 결과물을 작성한것 이기 때문에, 상대적으로 지식의 깊이가 얕다 라는 생각이 많이 드는 분야입니다. 위에 Custom Element 부분을 보시면 아시겠지만, 지면상의 문제로 압축을 한것도 사실이지만, React Lifecycle처럼 잘 알고 있는것도 아니라서 사실 mdn 문서 링크하고 넘긴것도 부정 할 수 없는 사실이기도 하거든요... 그렇기 때문에 이 글에서 틀린 부분도 있을 수 있고, 부족한 부분이 있을 수도 있습니다. 만약 잘못된 부분이 있으면 댓글로 알려주시면 감사하겠습니다.

부족한 글 끝까지 읽어주셔서 감사합니다.
