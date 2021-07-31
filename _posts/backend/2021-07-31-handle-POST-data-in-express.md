---
title: express.js 에서 POST 데이터 처리하기
layout: post
subtitle: express.js에서 다양한 종류의 POST 요청들을 어떻게 처리해야 할까요?
image: /images/thumbnails/express.png
category: backend
tags: node.js express POST
---

# 들어가며

express.js로 간단한 공부들을 해보면서, 대략적으로 express로 간단한 CRUD 서비스를 만드는 법은 배웠지만, 제대로 express.js를 알고 있다고 하기에는 수준이 많이 부족하고, 언젠가 공모전 등의 실제 개발을 하게 될 때, 도움이 되기를 바라면서, express.js 공식 문서를 보면서 번역을 하며, 몰랐던것을 새롭게 정리한 글입니다. 1대 1 번역은 아닙니다. 원본은 필요한 부분만 클릭해서 볼 수 있는 형태였다면, 이 포스트는 제가 공부한걸 제가 보기에 적합한 형태로 바꾼 것이라서요.

# express()

express.js로 서버를 구성할떄,

```javascript
import express from "express";
const app = express();
//other configs...
```

로 시작합니다. 여기서는 `express` 자체에 딸린 메소드들을 설명할 예정입니다.

## 서버로 오는 payload를 분석해 req.body에 붙여주는 메소드들

웹 서비스를 구성하다 보면, 클라이언트에서 서버로 request body에 담아서 보내는 데이터가 있을 수 있습니다. 그러한 여러 데이터들을 express에서 바로 처리할 수는 없기에, [body-parser](https://www.npmjs.com/package/body-parser)라는 모듈을 사용해서 express에서 처리할 수 있도록 합니다. 클라이언트에서 서버로 데이터를 아예 보내지 않거나, POST 메소드만으로 데이터를 보내는 경우는 드물어서인지, express v.4.16.0 이후로 express에 내장된 메소드들 입니다.

http request body에 담겨저 오는 payload의 종류는 여러가지가 있고, body-parser에서는 총 4가지 종류의 미들웨어를 제공하고, 이것이 express에 내장되어 있다 보시면 됩니다.

해당 메소드들을 미들웨어로 사용하지 않으면 올바른 처리를 express에서 할 수 없고, `req.body` 는 `undefined`라고 나올 것입니다.

설명을 들어가기 전에, 몇가지 용어에 대해서 정리를 해볼까 합니다.

1. 페이로드(payload)란? : 데이터 전송부분 중에서 실제로 사용자가 필요로 하는 부분. [위키백과 링크](<https://ko.wikipedia.org/wiki/%ED%8E%98%EC%9D%B4%EB%A1%9C%EB%93%9C_(%EC%BB%B4%ED%93%A8%ED%8C%85)>)
2. MIME 타입이란? : 웹에서 전송되는 문서의 종류를 명시하는 방법. `text/plain` `application/json` 등등이 있다. [mdn 링크](https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/MIME_types)
3. 압축된 body? : 웹페이지의 속도를 올리기 위해서, 브라우저와 서버에서 자원들을 압축해서 주고받는 경우가 있습니다. HTTP에서의 압축에 대한 이야기는 [mdn 링크 참조](https://developer.mozilla.org/ko/docs/Web/HTTP/Compression)

`req.body`의 형태는 사용자가 만든 입력에 기반하기에, 해당 오브젝트의 값과 속성은 바로 신뢰할 수 없으며, 그 전에 검증할 필요가 있습니다. 예를 들어서 `req.body.foo.toString()`은 여러 이유로 인해서 작동하지 않을 수 있습니다.<br><br>
해당 `foo`가 존재하지 않을수도 있고, `.toString`이 함수가 아닐수도 있는 것이죠. 그래서 아래에 설명할 여러 메소드들에서는 검증에 관한 속성이 들어있습니다. 꼭 명심하세요.
{:.warning}

후술할 메소드들의 공통된 option은 아래의 표와 같습니다.

<table>
    <thead>
        <tr>
            <th>속성</th>
            <th>설명</th>
            <th>타입</th>
            <th>기본값</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code class="language-plaintext highlighter-rouge">inflate</code></td>
            <td>압축된 body를 처리할지에 대한 여부를 결정합니다. <br>비활성화 되었을 시(false로 설정) 압축된 body는 미들웨어에서 거부됩니다.</td>
            <td><code class="language-plaintext highlighter-rouge">Boolean</code></td>
            <td><code class="language-plaintext highlighter-rouge">true</code></td>
        </tr>
        <tr>
            <td><code class="language-plaintext highlighter-rouge">limit</code></td>
            <td>request body의 사이즈의 상한을 결정합니다.<br> 숫자로 입력이 되면, 상한값은 입력한 숫자만큼의 바이트로 설정이 되고, <br>문자열이면, <a href="https://www.npmjs.com/package/bytes">bytes 라이브러리</a>를 통해 파싱되어, 용량의 상한을 결정합니다</td>
            <td>mixed(numbers 또는 string)</td>
            <td><code class="language-plaintext highlighter-rouge">"100kb"</code></td>
        </tr>
        <tr>
            <td><code class="language-plaintext highlighter-rouge">type</code></td>
            <td>미들웨어가 어떤 타일을 처리할 것인지를 지정해 줍니다. 문자열, 문자열의 배열, 함수 중 하나로 사용합니다.<br> 함수가 아니면, <a href="(https://www.npmjs.org/package/type-is">type-is 라이브러리</a>를 통해서, 확장자 명인지(예시 : .bin) MIME 타입명인지(예시 : text/plain), 와일드카드를 사용한 MIME 파일 명인지, (예시 : text / \*)를 확인해서 반영합니다.<br>함수라면 <code class="language-plaintext highlighter-rouge">type</code> 옵션은 <code class="language-plaintext highlighter-rouge">fn(req)</code>로 호출되어, 참으로 해석될 수 있는 값(truthy value)이 반환되면 파싱 됩니다.</td>
            <td>Mixed(문자열, 문자열 배열, 함수)</td>
            <td>각 메소드 별로 상이</td>
        </tr>
        <tr>
            <td><code class="language-plaintext highlighter-rouge">verify</code></td>
            <td>해당 옵션이 제공되면, <code class="language-plaintext highlighter-rouge">verify(req,res,buf,encoding)</code>형태로 함수가 호출 됩니다. <br><code class="language-plaintext highlighter-rouge">buf</code>는 해당 raw request body의 Buffer이고, <code class="language-plaintext highlighter-rouge">encoding</code>은 http 요청의 인코딩 입니다. 이 단계에서 에러가 나면 파싱이 종료될 수 있습니다.</td>
            <td>Function</td>
            <td><code class="language-plaintext highlighter-rouge">undefined</code></td>
        </tr>
    </tbody>
</table>

### express.json([options])

이 메소드는 JSON 만을 파싱하고, request내의 `Content-type`과 해당 메소드의 `type`옵션이 일치하는 request 만을 처리하는 **미들웨어를 반환**합니다. 모든 형태의 유니코드 인코딩을 받아드리며, `gzip`이나 `deflate`인코딩으로 압축된 내용들을 알아서 압축 해제 합니다.

미들웨어로 처리되고 난 다음의 새로운 `body` 객체 내에는 `request` 객체 내에서 파싱 된 데이터들이 들어있을 것입니다. 만약에 파싱할 데이터가 없거나, `Content-type`이 일치하지 않거나 에러가 난다면 `body`객체는 빈 객체가 될 것입니다.

하단의 표는 `express.json()`에만 해당하는 옵션입니다. 위에서 명시한 표에 해당하는 내용에 덧붙여 보시면 됩니다.

<table>
   <thead>
      <tr>
         <th>속성</th>
         <th>설명</th>
         <th>타입</th>
         <th>기본값</th>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td><code class="language-plaintext highlighter-rouge">type</code></td>
         <td>위의 표에서 설명함</td>
         <td>위의 표에서 설명함</td>
         <td><code class="language-plaintext highlighter-rouge">&quot;application/json&quot;</code></td>
      </tr>
      <tr>
         <td><code class="language-plaintext highlighter-rouge">reviver</code></td>
         <td><code class="language-plaintext highlighter-rouge">JSON.parse()</code>의 두번째 인자로 전달됩니다. <code class="language-plaintext highlighter-rouge">JSON.parse()</code>의 두번째 인자에는 값을 반환하기 전에 가공하는 함수가 들어갑니다. <br>자세한 사항은 <a href="https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse">mdn 링크 참조</a></td>
         <td>Function</td>
         <td><code class="language-plaintext highlighter-rouge">null</code></td>
      </tr>
      <tr>
         <td><code class="language-plaintext highlighter-rouge">strict</code></td>
         <td>배열과 객체만 받아드릴지 결정합니다. 비활성화 되면(false로 세팅되면) <code class="language-plaintext highlighter-rouge">JSON.parse</code>가 처리할 수 있는 모든것을 처리하게 됩니다.</td>
         <td>Boolean</td>
         <td><code class="language-plaintext highlighter-rouge">true</code></td>
      </tr>
   </tbody>
</table>

해당 메소드를 사용해서 데이터를 파싱한 예시입니다.

**프론트엔드 자바스크립트**

```javascript
fetch("/", {
    method: "POST",
    headers: {
        "Content-Type": "application/json",
    },
    body: JSON.stringify({
        user: {
            name: "John",
            email: "john@example.com",
        },
    }),
});
```

**백엔드 express.js**

```javascript
app.use(express.json());
app.post("/", function (request, response) {
    console.log(request.body.user.name);
    console.log(request.body.user.email);
});
```

### express.urlencoded([options])

이 메소드는 urlencoded된 body 만을 파싱하고, request내의 `Content-type`과 해당 메소드의 `type`옵션이 일치하는 request 만을 처리하는 **미들웨어를 반환**합니다. 모든 형태의 유니코드 인코딩을 받아드리며, `gzip`이나 `deflate`인코딩으로 압축된 내용들을 알아서 압축 해제 합니다.

미들웨어로 처리되고 난 다음의 새로운 `body` 객체 내에는 `request` 객체 내에서 파싱 된 데이터들이 들어있을 것입니다. 만약에 파싱할 데이터가 없거나, `Content-type`이 일치하지 않거나 에러가 난다면 `body`객체는 빈 객체가 될 것입니다.

<i class="far fa-question-circle"></i> `urlencoded`이 무엇인가요?<br>
HTML `<form>`에서는 `enctype` 속성을 통해서, 여러가지 방법으로 form을 서버에 제출할 수 있도록 합니다. 기본적으로 `enctype`을 명시하지 않았을때는 `application/x-www-form-urlencoded`로 보내지며, 이는 '&'으로 분리되고 '='으로 키와 값을 연결하는 구조입니다.<br>
자세한 사항은 [mdn 링크](https://developer.mozilla.org/ko/docs/Web/HTTP/Methods/POST)를 참조해 주세요.
{:.info}
하단의 표는 `express.urlencoded()`에만 해당하는 옵션입니다. 위에서 명시한 표에 해당하는 내용에 덧붙여 보시면 됩니다.

<table>
   <thead>
      <tr>
         <th>속성</th>
         <th>설명</th>
         <th>타입</th>
         <th>기본값</th>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td><code class="language-plaintext highlighter-rouge">type</code></td>
         <td>위의 표에서 설명함</td>
         <td>위의 표에서 설명함</td>
         <td><code class="language-plaintext highlighter-rouge">&quot;application/x-www-form-urlencoded&quot;</code></td>
      </tr>
      <tr>
         <td><code class="language-plaintext highlighter-rouge">extended</code></td>
         <td>해당 옵션에서는 URL-인코딩 된 데이터를 <code class="language-plaintext highlighter-rogue">querystring</code> 라이브러리를 통해서 파싱할지(<code class="language-plaintext highlighter-rogue">false</code>일때 적용) <code class="language-plaintext highlighter-rogue">qs</code> 라이브러리를 통해서 파싱할지(<code class="language-plaintext highlighter-rogue">true</code>일때 적용) 결정합니다.<br>
         "extended"가 의미하는 바는, 더 풍부한 문법을 사용할 수 있게 <strong>확장되었다</strong> 라는 의미입니다. nested 된 형태들을 사용할 수 있는 등, 더욱 많은 문법을 지원합니다.<br>
         자세한 사항은 <a href="https://stackoverflow.com/questions/29960764/what-does-extended-mean-in-express-4-0/45690436#45690436">stack overflow 게시글 참조</a>
         </td>
         <td>Boolean</td>
         <td><code class="language-plaintext highlighter-rouge">true</code></td>
      </tr>
      <tr>
         <td><code class="language-plaintext highlighter-rouge">parameterLimit</code></td>
         <td>해당 옵션에서는 URL-인코딩된 데이터에서 처리할 수 있는 매개변수들의 상한을 설정합니다. 만일 request body에 해당 수를 초과하는 매개변수가 이를 초과할 경우 에러가 발생합니다.</td>
         <td>Number</td>
         <td><code class="language-plaintext highlighter-rouge">1000</code></td>
      </tr>
   </tbody>
</table>

해당 메소드를 사용해서 데이터를 파싱한 예시입니다.

**프론트엔드 HTML**

```html
<form method="post" action="/">
    <input type="text" name="user[name]" />
    <input type="text" name="user[email]" />
    <input type="submit" value="Submit" />
</form>
```

**백엔드 express.js**

```javascript
app.use(express.urlencoded());
app.post("/", function (request, response) {
    console.log(request.body.user.name);
    console.log(request.body.user.email);
});
```

### express.text([options])

이 메소드는 body를 문자열로 보고 파싱하고, request내의 `Content-type`과 해당 메소드의 `type`옵션이 일치하는 request 만을 처리하는 **미들웨어를 반환**합니다. 모든 형태의 유니코드 인코딩을 받아드리며, `gzip`이나 `deflate`인코딩으로 압축된 내용들을 알아서 압축 해제 합니다.

미들웨어로 처리되고 난 다음의 새로운 `body` 객체 내에는 `request` 객체 내에서 파싱 된 데이터들이 들어있을 것입니다. 만약에 파싱할 데이터가 없거나, `Content-type`이 일치하지 않거나 에러가 난다면 `body`객체는 빈 객체가 될 것입니다.

하단의 표는 `express.text()`에만 해당하는 옵션입니다. 위에서 명시한 표에 해당하는 내용에 덧붙여 보시면 됩니다.

<table>
   <thead>
      <tr>
         <th>속성</th>
         <th>설명</th>
         <th>타입</th>
         <th>기본값</th>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td><code class="language-plaintext highlighter-rouge">type</code></td>
         <td>위의 표에서 설명함</td>
         <td>위의 표에서 설명함</td>
         <td><code class="language-plaintext highlighter-rouge">&quot;text/plain&quot;</code></td>
      </tr>
      <tr>
         <td><code class="language-plaintext highlighter-rouge">defaultCharset</code></td>
         <td> 해당 옵션에서는 <code class="language-plaintext highlighter-rouge">Content-type</code> 헤더에서 기본 문자 셋(character set)이 지정되지 않았을 경우 문자 셋을 지정하는 역할을 합니다. 
         </td>
         <td>String</td>
         <td><code class="language-plaintext highlighter-rouge">"utf-8"</code></td>
      </tr>
   </tbody>
</table>

### express.raw([options])
이 메소드는 body를 `Buffer`로 보고 파싱하고, request내의 `Content-type`과 해당 메소드의 `type`옵션이 일치하는 request 만을 처리하는 **미들웨어를 반환**합니다. 모든 형태의 유니코드 인코딩을 받아드리며, `gzip`이나 `deflate`인코딩으로 압축된 내용들을 알아서 압축 해제 합니다.

미들웨어로 처리되고 난 다음의 새로운 `body` 객체 내에는 `request` 객체 내에서 파싱 된 데이터들이 들어있을 것입니다. 만약에 파싱할 데이터가 없거나, `Content-type`이 일치하지 않거나 에러가 난다면 `body`객체는 빈 객체가 될 것입니다.

<i class="far fa-question-circle"></i> `application/octet-stream`은 언제 쓰나요?<br>
RFC 2046에서 명시된 바로는 `application/octet-steam`은 임의의 2진 파일을 의미합니다. 대충 웹 요소가 아닌 무언가 정도로 해석하면 됩니다. 이게 언제 쓰이는지는 찾아봐도 잘 모르겠습니다.<br>
일단 브라우저 쪽에서 해당 헤더를 가진 HTTP POST를 수신했을 때, 파일을 다운로드 하는 창을 띄운다고 합니다. <a href="https://stackoverflow.com/questions/20508788/do-i-need-content-type-application-octet-stream-for-file-download">stack overflow 링크</a>
{:.info}

하단의 표는 `express.raw()`에만 해당하는 옵션입니다. 위에서 명시한 표에 해당하는 내용에 덧붙여 보시면 됩니다.

<table>
   <thead>
      <tr>
         <th>속성</th>
         <th>설명</th>
         <th>타입</th>
         <th>기본값</th>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td><code class="language-plaintext highlighter-rouge">type</code></td>
         <td>위의 표에서 설명함</td>
         <td>위의 표에서 설명함</td>
         <td><code class="language-plaintext highlighter-rouge">&quot;application/octet-stream&quot;</code></td>
      </tr>
   </tbody>
</table>

# 마치며
HTML `<form>`에서 POST 메소드로 정보를 송신할때, 정보를 처리하기 위해서 사용한다고 알고만 있었던 express의 body-parser 기반 미들웨어(정확히는 미들웨어를 생성하는 메소드)에 대해서 알아보았습니다. 아직 개발 짬이 부족한지라, 이러한 것들이 어떠한 가치를 가지는지는 100% 이해하지 못하지만, 향후에 HTTP POST 통신을 쓰면서 도움이 되지 않을까 하는 마음에 한번 정리 해 보았습니다.

끝까지 글 읽어주셔서 감사합니다. 