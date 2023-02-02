---
title: 문자열 처리의 해결사. 정규표현식을 알아보자 ④
layout: post
subtitle: 게으른 수량자인 겸허 수량자를 이용해, 캡쳐를 다루는 방법과 문자열 치환에 대해서 알아봅시다.
image: /images/thumbnails/regex.png
category: environment
series: 정규식
---

# 욕심 수량자와 겸허 수량자

`*`, `+`, `?`등의 수량자가 포함된 경우 서브 매치의 할당은 위에서 설명한, 왼쪽부터 오른쪽 이라는 간단한 원칙으로 설명할 수 있었지만, 이제 소개할 욕심 수량자와 겸허 수량자의 차이를 이해한다면, 이 원칙을 벗어날 수 있습니다. 캡처를 지원하는 엔진에서는 기본적으로 욕심 수량자(greedy quantifier)와 겸허 수량자(lazy quantifier)두가지를 지원합니다.

| 수량자 종류 | 욕심          | 겸허            |
| ----------- | ------------- | --------------- |
| 스타        | `*`           | `*?`            |
| 플러스      | `+`           | `+?`            |
| 물음표      | `?`           | `??`            |
| 범위 수량자 | `{n}`,`{n,m}` | `{n}?`,`{n,m}?` |

이 표를 보면 알 수 있듯이, 기본적으로 연산자는 욕심의 성질을 가지고 있고, 뒤에 물음표 하나 더 붙이면, 겸허 연산자가 됩니다. 욕심/겸허 연산자는 받아드리는 패턴은 동일하지만, 이 둘의 차이는 캡처를 대하는 동작입니다. 최대한 많은 패턴을 일치시키려는 욕심 수량자와는 다르게, 겸허 수량자는 최대한 적은 글자수를 본인에게 매칭시키려고 한다고 이해를 하면, 대강 맞는 설명이 됩니다.

## 욕심 수량자와 겸허 수량자의 동작 차이

'aaa'라는 문자열을 `(a*)(a*)`라는 정규 표현식에, 일치시켜 봅시다. 다만, 이것 그대로 한번만 일치시키는 것이 아니고, 2개의 스타 연산자를 욕심 수량자와 겸허 수량자로 가능한 경우 모두 일치시켜 보면서, 어떻게 이것이 돌아가는지 확인해 봅시다.

```javascript
const r1 = /(a*)(a*)/;
const r2 = /(a*?)(a*)/;
const r3 = /(a*)(a*?)/;
const r4 = /(a*?)(a*?)/;

const testString = "aaa";

const match1 = testString.match(r1);
const match2 = testString.match(r2);
const match3 = testString.match(r3);
const match4 = testString.match(r4);

console.log(`match[1] : ${match1[1]} match[2] : ${match1[2]}`); // 'aaa' ''
console.log(`match[1] : ${match2[1]} match[2] : ${match2[2]}`); // '' 'aaa'
console.log(`match[1] : ${match3[1]} match[2] : ${match3[2]}`); // 'aaa' ''
console.log(`match[1] : ${match4[1]} match[2] : ${match4[2]}`); // '' ''
```

자 이제 결과를 놓고 곰곰히 생각해 봅시다.

`(a*)(a*)`의 경우에는, 이전 포스팅에서도 다루었던, **왼쪽부터 오른쪽**이라는 원칙에 맞게, 왼쪽에 다 매칭이 되는 모습을 볼 수 있습니다.

한편, 왼쪽이 **겸허 수량자**일때는 이 원칙이 적용되지 않습니다. 왼쪽에 있는 겸허 수량자가, 최소의 패턴인 공백 문자열은 선택하고, 우측에 있는 욕심 수량자가, 이 남은것을 본인이 해먹는 것이 됩니다. 왼쪽이 욕심 수량자이고, 오른쪽이 겸허 수량자일때는, 첫번째 경우와 같이, 매칭이 되지요.

흥미로운 것은, 4번째 경우인 **양쪽 모두 겸허 수량자인 경우**가 되겠습니다. 다른 정규 표현식들과는 다르게, 어느것도 매치가 되지 않은 모습입니다. 이유를 간단히 설명하자면, `.match`메소드는 **부분 일치**를 하기에, 공백 문자열도 일치하게 되고, 최대한 본인이 적게 매칭을 가져가려는 겸허 수량자의 특성과 맞물려서, 아무것도 매칭되지 않는 상황이 된 것 입니다.

여기서 문제를 하나 내겠습니다. 4번째 정규 표현식에 완전 일치를 시키면, 어떻게 될까요? 직접 실행해서 확인해 봅시다.

```javascript
const r5 = /^(a*?)(a*?)$/;
const testString = "aaa";
const match5 = testString.match(r5);

console.log(`match[1] : ${match5[1]} match[2] : ${match5[2]}`); // '' 'aaa'
```

뒤에 나오는 `(a*?)`에 'aaa'가 매칭됨을 볼 수 있습니다. 왜 이렇게 되는지를 간단히 설명하면 아래와 같습니다.

두 겸허 연산자는 최대한 적은 문자열을 본인에게 매칭 시키기 위해서, 서로 양보합니다. 하지만, 정규 표현식의 매칭의 중요한 법칙인 **왼쪽부터 오른쪽**에 따라서, 왼쪽에 있는 겸허 연산자의 요청이 더 우선순위가 높게 적용되어서, 오른쪽의 캡쳐에 나머지가 다 할당 되는것입니다.

앞의 설명은 정확한 설명이 아닌, 일종의 비유 입니다. 왜 이런 현상이 생겼는지를 정확히 알려면, 심화 편에서 설명할 **백트랙**을 이해해야 합니다.

## 겸허 수량자의 사용 예

겸허 수량자의 활용 예는, [전 포스팅의 마무리 부분](/regex3-capture-and-its-priority/#마치며)에 나와있는 바로 그 문제상황 그자체 입니다. 큰따옴표(")로 감싼 문자열을 정규 표현식을 이용해서 파싱할 때, 겸허 수량자가 정말 편리하게 작용합니다.

`(".*")`라는 정규 표현식에 "apple,"banana","melon"이라는 문자열을 일치 시키면, 따로 따로 나오는 것이 아닌, 문자열 전체 덩어리가 캡쳐되어 나왔습니다. 이유는 정말 간단하고 명확합니다.

<div style="display:flex; justify-content:center">
    <span style="color:red;">"</span>
    <span style="color:yellowgreen;">apple", "banana", "melon</span>
    <span style="color:red;">"</span>
</div>

붉은 색으로 표시한 부분은 위 정규 표현식에 `"`에 일치하는 부분이고, 연두색으로 칠한 부분이 `.*`에 일치하는 것 입니다. 이 매치가 결코 잘못된 것은 아니지만은, 우리가 원하는 형태는 아님이 확실합니다. 욕심 수량자는 본인의 소임을 다 했기 때문에, 욕심 수량자를 탓할 수는 없죠. 여기서, 애꿎은 욕심 수량자를 탓하지 말고 최소한의 일치를 지향하는, 겸허 수량자를 사용하면 우리의 문제를 해결 할 수 있습니다.

<i class="fas fa-info-circle"></i> <strong>정규 표현식의 g 옵션</strong><br/>
일반적으로 문자열을 정규 표현식에 일치 시키면, 문자열을 쓱 훑다가, 매칭 되는 첫 경우만 찾고, 검색을 끝냅니다.<br/>
하지만, g 옵션을 사용하면, 정규 표현식에 대응하는 모든 문자를 검색합니다. <br/>
`/regex/`에 g 옵션을 사용하려면 `/regex/g` 처럼, 감싸는 `/` 뒤에 g를 덧대면 됩니다.
{:.info #goption}

<i class="fas fa-exclamation-triangle"></i> <strong>g옵션 사용시 주의사항</strong><br/>
지금까지 javascript를 통해서, 정규 표현식을 확인 할 때, `.match`메소드를 활용했었습니다. 하지만, g 옵션이 활성화가 되면, `.match()`메소드는 캡처된 결과를 **반환하지 않습니다**.<br/>
`Regexp.exec()`을 사용하거나, `String.prototype.matchAll()`을 사용해야 하는데, 이 글에서는 `String.prototype.matchAll()`을 이용해서 설명하겠습니다.<br/>
자세한 사항은 [mdn 문서](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/match) 참조 바랍니다.
{:.warning}

이제 g 옵션과, `Regexp.exec()`를 활용해서, 실제 테스팅을 해봅시다.

```javascript
const testString = '"apple", "banana", "melon"';
const regexp = /(".*?")/g;
const match = [...testString.matchAll(regexp)];
console.log(
  `match[0] : ${match[0][0]} match[1] : ${match[1][0]} match[2] : ${match[2][0]}`
);
//"apple" "banana" "melon"이 따로 따로 나옴을 볼 수 있음
```

`.matchAll()`메소드는 `.match()`메소드와 출력이 다르기 때문에 코드를 일부 수정했습니다. 문법적인 요소를 잠깐 제쳐두고, 중요한 것은, 겸허 수량자를 통해서, 서브 매치를 어느정도 제어할 수 있다는 사실입니다.

# 정규 표현식을 활용한 문자열 치환

이전 포스트 에서도 지나가는 말로나마 간단히 다루었지만, 정규 표현식은 문자열을 치환하는데도 사용 됩니다. 프로그래밍 언어에 내장된 정규 표현식에서도 이를 지원하고, 리눅스 쉘 등의 CLI 환경에서도 쓸 수 있는 도구들이 존재합니다. 이 툴들이 사실, 지금까지 CLI가 살아있는 이유인 '생산성'을 맡고 있는 중요한 부분들 중 하나이기 때문에, 효율적으로 사용할 수 있었으면 좋겠습니다.

## 문자열 치환 - javascript

javascript에서 String에서 쓸 수 있는 메소드 중 하나인 `String.prototype.replace()`는 정규 표현식을 이용한 문자열 치환 기능을 제공합니다. 해당 메소드 사용법은 아래와 같습니다.

```
var newStr = str.replace(regexp|substr, newSubstr|function)
```

첫번째 매개변수에는 변경할 문자열 리터럴(substr)이나, 정규 표현식(regexp)를 넣고, 두번째 매개변수에는, 치환 이후의 문자열(newSubstr)이나, 해당 문자열을 가공할 함수(function)을 넣습니다. 함수를 넣었을 경우에는, 첫번째 매개변수에서 지정한 문자열을 함수의 인자로 넣어서 반환된 값을 이용해서, 문자열 치환을 합니다. 간단히 설명하면 이렇지만, 두번째 인자에도 여러 기능들이 더 들어 있습니다.

### 두번째 인자에 String을 사용했을 때

대체할 문자열에는 아래의 문자들을 사용해서, 리터럴 뿐만이 아닌, 여러 기능들을 쓸 수 있습니다.

| 패턴                                                         | 기능                                           |
| ------------------------------------------------------------ | ---------------------------------------------- |
| `$$`                                                         | $라는 문자열 자체를 삽입합니다.                |
| `$&`                                                         | 매치된 문자열을 삽입합니다.                    |
| <code class="language-plaintext highlighter-rouge">$`</code> | 매치된 문자열에 선행하는 문자열을 삽입합니다.  |
| `$'`                                                         | 매치된 문자열에 후행하는 문자열을 삽입합니다.  |
| `$n`                                                         | n번째 캡처를 삽입합니다.(단 $$n \le 99$$)      |
| `$<name>`                                                    | 이름 있는 캡처로 가져온 서브패턴을 삽입합니다. |

vscode에서 파일을 편집 할 때에, **파일에서 `ctrl+f`를 눌렀을 때** 정규식 사용 옵션을 선택했을 때에도 사용할 수 있는 문법들 이기에, javascript를 주로 사용하지 않더라도 알고 있으면 분명 도움이 되리라 생각합니다.
![vscode](https://i.stack.imgur.com/PTvcq.png)

<div style="display:flex; justify-content:center;"><strong>vscode에서는 javascript의 정규 표현식을 사용한다</strong></div>

![vscode](https://i.stack.imgur.com/Lo9Ce.png)

<div style="display:flex; justify-content:center;"><strong>치환을 사용하기 전</strong></div>

![vscode](https://i.stack.imgur.com/tBPeI.png)

<div style="display:flex; justify-content:center;"><strong>치환을 사용한 후 javascript 문법에서 소개한 `$` 기능이 올바르게 작동함을 볼 수 있다</strong></div>

이 부분은, 제가 블로그를 옮길 때, 사용했던 기능이기도 해서, 적을 때, 더 각별한 마음이 들었습니다. 예전에 쓰던 velog라는 블로그 에서는 인라인 수식을 `$` 하나로 감싸서 표현했었는데, jekyll에서 해당 문법은 `$`2개로 감싸야 하는 것이었습니다. 그래서 vscode의 치환 기능을 이용해서 `$([^$])$`을 `$$$$$1$$$$`라는 정규 표현식으로 치환해서 했던 경험도 있었거든요. 위에서 말했듯 정규 표현식이라는 도구 덕분에 생산적으로 일 할수 있었던것 같습니다. 몇백개에 가까운 수식들을 일일히 검색해 가며 `$`한개 더 씌우는 것보다, 버튼 짤각 한번 하는게 더 쉬운건 결코 부정할 수 없는 사실이니까요

### 두번째 인자에 function을 사용했을 때

위에서 간단히 다루었듯이, 두번째 매개변수에 함수를 지정할 수 있습니다. 해당 함수는, 정규 표현식 일치가 성공적일 때마다 호출이 되며, 이 함수의 반환값은 치환 결과 문저열이 될 것입니다. 주의할 점은 위에서 소개했던, 특수 문자열은 **적용되지 않는다**라는 것 정도입니다.

함수의 매개변수로 사용할 수 있는것은 아래와 같습니다.

| 매개변수 이름   | 의미                                                            |
| --------------- | --------------------------------------------------------------- |
| `match`         | 매치된 문자열. (윗쪽의 $& 표현식으로 매치된 경우와 동일합니다.) |
| `p1`, `p2`, ... | 윗쪽의 $n 표현식과 동일합니다. 캡처로 가져온 서브패턴 입니다.   |
| `offset`        | 조사 대상의 문자열에서 찾은 매치된 문자열의 인덱스              |
| `string`        | 조사 대상 문자열 전체                                           |

다음은 함수를 사용해서, 텍스트를 조작하는 javascript 코드의 예시입니다.

```javascript
function replacer(match, p1, p2, p3, offset, string) {
  // p1 is nondigits, p2 digits, and p3 non-alphanumerics
  return [p1, p2, p3].join(" - ");
}
var newString = "abc12345#$*%".replace(/([^\d]*)(\d*)([^\w]*)/, replacer);
console.log(newString); // abc - 12345 - #$*%
```

## 문자열 치환 - sed, perl -pe

아까 설명하였듯이, GUI가 널리 통용되는 현재 사회에서 CLI 환경을 아직까지 살아있게 해준 수많은 이유중 하나인 **효율성**을 담당하고 있는 도구들 입니다. sed는 grep과 함께 유닉스를 대표하는 툴로, 정규 표현식을 사용해서, 문자열을 치환하기 위한 툴 입니다. 사용 방법은 간단합니다.

```
sed s/regex/replacement/g
```

라는 형식만 지키면 됩니다. `regex`는 치환 대상이 될 문자들을 명시하는 정규 표현식이고, `replacement`는 치환 후 나욜 문자열을 의미합니다. sed에서는 javascript와 다르게, 캡쳐된 서브매치를 가져올 때, `$n`이 아닌 `\n`이라는 형식으로 가져온다는것만 알면 됩니다. 저기 맨 끝에 붙어있는 `g`는 [위에서 소개한](#goption) 그 g 옵션과 같습니다.

사용 예시는 아래와 같습니다.

```bash
$ echo 'Hong Gildong, Kim Chulsu' | sed -E 's/([a-zA-Z]+) ([a-zA-Z]+)/\2 \1/g' # g option을 사용했을 때
Gildong Hong, Chulsu Kim
$ echo 'Hong Gildong, Kim Chulsu' | sed -E 's/([a-zA-Z]+) ([a-zA-Z]+)/\2 \1/g' # g option을 사용하지 않으면?
Gildong Hong, Kim Chulsu # 처음에 매칭된 것만 반환됨을 볼 수 있다.
```

sed의 `s/regex/replacement/g`라는 형식은 다른 언어에서도 사용할 수 있고, 그 중 하나가 펄(perl) 입니다. 펄 에서도 해당 문법을 그대로 사용할 수 있고, PCRE 정규식에서 지원하는 더욱 많은 기능들을 쓸 수 있으므로, perl로 hello world 하나 못찍는 사람이더라도, 정규 표현식 도구로써의 사용법은 알고 있으면 좋을것 같아, 이 지면에 담아 봤습니다.

```bash
$ echo 'Hong Gildong, Kim Chulsu' | perl -pe 's/([a-zA-Z]+) ([a-zA-Z]+)/$2 $1/g'
Gildong Hong, Chulsu Kim
```

# 마치며

좀 글이 길어졌습니다. 정규 표현식을 사용해서, 문자열을 치환하는 방법에 대해서 알아보았습니다. 이 내용들을 한번에 이해하려는 기대는 사실 누구에게도 하면 안될 기대가 아니자 않나 싶긴 하지만, 코딩 인생에 있어서 엄청난 생산성 향상을 가져올 내용들이기 때문에, 이 내용들을 곰씹으면서 꼭 본인의 지식으로 만들기를 바랍니다.

다음 포스팅에서는 아직 설명하지 못한 정규 표현식의 기능들과, 주요 정규 표현식 엔진에서의 문법 사용 가능 여부등을 정리해 볼까 합니다.

끝까지 읽어주셔서 정말 감사하고, 혹시 궁금증이 있거나, 틀렸다고 생각하는 내용이 있으시면, 주저없이 댓글 부탁드립니다. 감사합니다.

# 참고한 자료

[다양한 언어로 배우는 정규표현식 최신 엔진 구현과 이론적 배경을 배우다(신야 료마 , 스즈키 유스케 , 타카타 켄 지음)](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9791185890456)

javascript에서의 `String.prototype.replace()`참고 자료 : [mdn 링크](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/String/replace)

[https://stackoverflow.com/questions/42179046/what-flavor-of-regex-does-visual-studio-code-use](https://stackoverflow.com/questions/42179046/what-flavor-of-regex-does-visual-studio-code-use) : vscode에 관한 부분을 적을 때 참고한 stackoverflow 글
