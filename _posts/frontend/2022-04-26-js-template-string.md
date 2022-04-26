---
layout: post
title: JavaScript Template String의 모든것
subtitle: 단순히 문자열에 변수 넣는걸로 끝이 아니에요
category: Frontend
image: /images/thumbnails/JS.png
---

# 들어가며

현대 브라우저(IE는 빼고요 ㅎㅎ)에서 무슨 일 없으면 지원하는 ES6 이상 js를 사용하면, `` ` ``(esc위에 있는 backtick)을 이용해서 사용하는 template string에 익숙할 것입니다.

```js
let userName = "john";
console.log(`hello ${userName}`);
// => hello john이 console에 출력됨
```

js를 조금이라도 공부해보신 분들이라면 모르실 수 없는 문법입니다만, 최근에 `styled-component`라는 유명 라이브러리의 작동원리를 공부하던 중에, template string의 Tagged Template라는 기능을 사용해서 작동한다는것을 알게 되어서, 해당 포스팅을 작성하기 전에, 이해해야할 template string의 특징을 정리하고자 이 포스트를 작성하고자 합니다. 내용이 별로 길지 않을 것 같습니다.

# template string의 특징들

너무나도 당연해서 지루한 특징부터 살펴봅시다.

- 백틱 (`` ` ``)을 사용해서, 문자열을 감쌉니다.
- 플레이스홀더를 이용해서 표현식을 넣을 수 있는데 `${expression}`의 형태로 사용합니다.
  - template string 또한 표현식이기 때문에, template string안에 template string을 nestring 하는것도 당연히 가능합니다.
  ```js
  const classes = `header ${
    isLargeScreen() ? "" : `icon-${item.isCollapsed ? "expander" : "collapser"}`
  }`;
  ```
- template string 내에서 백틱 문자를 사용하려면, 이스케이프 해야합니다(`` \` ``)
- template string내에 삽입된 개행 문자는 template string의 일부가 됩니다.
  ```js
  console.log(`ab
  cd`);
  // => ab
  //    cd 가 console에 출력됨
  ```
- tagged template라는 기능을 사용하여, 함수로 template string을 조작(파싱)할 수 있습니다.

tagged template에 대한 내용은 위에 있는것처럼 간단히 몇줄로 되지 않기 때문에 다음 섹션에서 다루겠습니다.

# Tagged Template?

아까 간단히 소개할 때, 함수로 template string을 조작할 수 있다고 했습니다. 도입부에서도 언급한 `styled-component`라는 라이브러리를 써보셨으면, 아래와 같은 코드를 작성해 본적이 있을 것 입니다.

```js
const Box = styled.div`
  width:100px;
  height:50px;
  그리고 뭐 이것저것...
`;
```

이 코드에서 `styled.div`는 그저 js 함수입니다. 함수 뒤에 template string을 바로 붙여서 사용하는 이 문법이, tagged template입니다. 공식 홈페이지에서도 해당 내용에 대해서 언급하고 있습니다.

![공식 docs](/images/frontend/styled-component_docs.png)

태그 함수의 첫 번째 매개변수는 문자열 값의 배열, 나머지 인수는 표현식(`${}`에 감싸 넣었던 그거요)에 관한 매개변수 입니다.

말로 설명하는 것보다, 코드로 보여주는것이 이해에 더 도움이 될듯 하여, mdn에 있는 코드를 적당히 수정해서 가져오겠습니다.

```js
let person = "Mike";
let age = 28;

function myTag(strings, personExp, ageExp) {
  let str0 = strings[0]; // that
  let str1 = strings[1]; // is a

  let ageStr;
  if (ageExp > 99) {
    ageStr = "centenarian";
  } else {
    ageStr = "youngster";
  }

  return str0 + personExp + str1 + ageStr;
}

let output = myTag`that ${person} is a ${age}`;
console.log(output);
// => that Mike is a youngster가 출력됨
```

아까 태그 함수의 첫번째 매개변수에는 문자열의 배열이 들어간다고 하였습니다. template string안에 있는 문자열들을 표현식을 구분자로 파싱해서 넣었다고 생각하면 됩니다. 그래서 `str0`에는 `that`이 들어가있고, `str1`에는 `is a`라는 문자열이 들어가게 되지요.

# 마치며

글 자체가 길지도 않고, 어려운 개념도 아니어서, 쉽게 이해하실 수 있을 것 같습니다. 향후에 `styled-component`라이브러리의 원리를 파헤치는 글을 작성할 때, 이 글이 참고자료로 쓰이길 바라면서 적은 글이니까요...

# 참고 자료

- [mdn문서](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Template_literals)
