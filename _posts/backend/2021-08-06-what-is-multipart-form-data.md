---
title: enctype="multipart/form-data" 가 뭐에요?
katex: True
layout: post
subtitle: multer을 쓸 때 &lt;input&gt;에 반드시 적어줘야 하는 이것. 뭔지 알아봅시다.
image: /images/thumbnails/HTML5.png
category: backend
tags: HTML enctype
---

# 개요

이번에 다뤄볼 주제는 HTML의 `<form>`으로부터 파일을 입력받을 때, 반드시 속성으로써 입력해줘야 하는 `enctype="multipart/form-data"`라는 것에 대해서 알아보고자 합니다.

공부를 하면서 제가 궁금했던 점 위주로 정리를 할까 하여, 간단히 끝낼 설명일지라도, 저의 꼬리에 꼬리를 무는 궁금증을 해결한 기록을 남겨보려고 합니다.

**파일을 업로드 하는 HTML form의 한 예시**

```html
<form method="POST" , action="" , enctype="multipart/form-data">
    <label for="myfile">Choose A file to Upload</label>
    <input type="file" id="myfile" />
</form>
```

그리고 express.js에서 파일 처리를 담당하는 미들웨어인 multer를 사용해보신 분이라면, multer를 사용하기 위해서라도 `enctype="multipart/form-data`가 익숙할 것입니다. 왜냐면 제가 이거 떄문에 이 궁금증을 가지기 시작했거든요.
![multer README](/images/backend/multerReadmeHead.png)
**뭔지 모르겠지만 아무튼 썼던 `enctype=multipart/form-data`**

# 1. enctype이 뭐야?

처음 든 궁금증은 이것 이었습니다. 위의 HTML코드에서 `<form>`도 알고있고, `<form>`의 `method`속성도 알고 있습니다. `POST`, `GET`, `PUT` 등을 쓸 수 있는것이라는것은, 간단한 웹 공부를 하신 분이라면 누구든지 알만한 간단한 것이죠. 하지만, 제가 HTML을 노마드코더 강의로 공부했을때는, `enctype`에 관한 언급은 딱히 없었기 떄문에, 가장 먼저 든 의문이 이것이었습니다. 노마드 코더의 1타강사 니꼴라스의 가르침대로, HTML에서 모르는게 생긴 저는 mdn을 뒤적거리다가 이런 mdn 문서를 발견했습니다. <https://developer.mozilla.org/ko/docs/Web/HTTP/Methods/POST>

HTML5에서 `enctype`은 총 세가지가 있다고 합니다.

1. application/x-www-form-urlencoded : 일반적인 `<form>`데이터를 보내는데 사용. &으로 값들을 구분하고 = 로 키와 값을 연결하는 방식을 취함. 영어 알파벳이 아닌 글자들은 퍼센트로 인코딩(percent encoding)되어, 이진파일을 보내는데에 적합하지 않음. (예시 : 캐리지 리턴 &rarr; `%0D%0A`)
2. multipart/form-data : 바이너리 파일을 보낼 때 사용한다고 함
3. text/plain : 디버깅용. 공백을 +로 인코딩함. 실제 서비스 에서는 사용을 지양해야 함.

이제 이걸 보고

> 음. enctype은 인코딩 타입이며, 영어 알파벳 이외의 문자들도 분명히 있을 이진파일을 POST로 보내기 위해서는 `multipart/form-data`를 이용해야 하는구나

하고 끝날 수도 있겠지만, 저는 몇가지 궁금한것들이 더 있었습니다. 일단 멀리 갈 필요 없이. `multipart`라는게 뭔뜻인지부터가 궁금하지 않나요?

# 2. multipart가 뭐야?

이 문제를 해결하기 위해서 what is multipart 라고 구글에 검색하니 상위에 뜬 [stack overflow 링크](https://stackoverflow.com/questions/16958448/what-is-http-multipart-request)가 하나 있었습니다. 재밌게도, 이 질문을 올린 질문자의 상태와 제 상태가 100% 일치하지는 않지만, 제 궁금증을 해결해 주리라는 생각이 들어서, 글을 읽어나가기 시작했습니다.

우선 첫번째 궁금증인 multipart에 대한 정의부터 입니다.

multipart란 말 그대로, 하나, 또는 하나 이상의 **다른 타입의 데이터**들이 하나의 body에 결합된 형태 입니다.
<br> 자세한 설명은 <https://www.w3.org/Protocols/rfc1341/7_2_Multipart.html> 참조
{:.info}

이 stack overflow의 답변들에 딸려있는 w3.org의 관련 문서를 읽고나서, 저는 저의 궁금증을 다 풀었던것 같습니다. 

<https://www.w3.org/Protocols/rfc1341/7_2_Multipart.html>

<https://www.w3.org/TR/html401/interact/forms.html#h-17.13.4.2>

이 두개의 문서인데, 이걸 다 번역하기에는 지금의 지식 수준이 충분치 않아서, 올바른 번역이 나오지 않을것 같아서, 쉬이 번역을 하지 못하겠습니다. (RFC 프로토콜에 관한 이야기도 중간중간 나와서 겁먹은것도 있고요).

그래도 이걸 읽으면서 제가 이해한것은 multipart라는 형태는 여러 part들이 각자의 경계를 두고(boundary 문자열) 내용을 기술해 놓은 것이고, `multipart/form-data`는 mutipart 형태에 맞게 form의 내용물들을 인코딩 하는 것이라 정도로 이해했습니다.

약간 글이 용두사미로 끝난것 같지만, 아쉽지만 이것이 현재로써의 최선인것 같습니다.