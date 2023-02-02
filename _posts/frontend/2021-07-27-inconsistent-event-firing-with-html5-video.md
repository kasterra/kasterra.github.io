---
layout: post
title: HTML5 video의 비일관적인 이벤트 발생과 대처법
subtitle: HTML5 video 에서 발생할 수 있는 "왜 이벤트가 발생 안해요?" 상황을 알아보고, 해결책까지 알아봅시다.
category: frontend
image: /images/thumbnails/HTMLJS.png
---

<p class="info">
<strong><i class="fas fa-info-circle"></i> 정보</strong><br/>
<span>본 게시물은, <a href="https://dev.opera.com/">dev.opera.com</a>에 올라와있는 <a href="https://dev.opera.com/authors/simon-pieters/">Simon pieters</a>님의
<a href="https://dev.opera.com/articles/consistent-event-firing-with-html5-video/">게시글</a>을 번역한 내용을 포함하고 있습니다.
이 게시물은 CC-BY-NC-SA 3.0으로 배포됩니다.<img src="/images/ccl/by-nc-sa.svg" style="margin:0 0 0 auto;"></span>
</p>

# 개요

HTML5 에 있는 `<video>` 속성을 활용해서 JS로 이것저것을 제어하다 보면, 이벤트 발생에 관해서 문제를 느낄때가 종종 있습니다.

가끔씩은 몇몇 이벤트가 일어나지 않는것 같고, 로컬에서 테스트 할때와, 네트워크에서 실제로 서비스가 가동될 때, 다르게 작동하는것 같은 경험을 받죠.

이 포스팅에서는 그러한 문제에 대해서 알아보고, 문제에 대한 해결책까지 알아보도록 하겠습니다.

# 왜 이런일이 생기나요?

정말로 이유는 간단합니다. 스크립트에서 video의 이벤트들을 listen 하기 전에, video가 로딩이 다 되버리기 때문입니다. 아래의 코드를 주목해 주세요.

```html
<video src="test.webm" id="video"></video>
<script>
  var video = document.getElementById("video");
  video.onloadedmetadata = function (e) {
    alert("Got loadedmetadata!");
  };
</script>
```

`script`에서 `<video>`가 충분히 로딩되면 발생하는 `loadedmetadata` 이벤트를 listen 하고 있다가, 이벤트가 발생하면 `alert`창을 띄우는 간단한 코드입니다.

해당 코드를 로컬에서 테스트 하거나, 충분히 빠른 네트워크(한국의 인터넷은 충분히 빠릅니다)에서 테스트하면, alert가 뜰 수도 있고, 뜨지 않을 수 있습니다.

반면에 좀 느린 네트워크나 `<video>`의 용량이 커서 로딩하는데에 시간이 좀 더 걸리는 경우에는 `alert`창을 볼 가능성이 좀 더 커지게 됩니다.

네트워크 상태에 따라서 달라질 수도 있지만, 만약 `<video>`를 브라우저가 캐시하고 있다거나, `src`속성을 쓰지 않고 `<source>`요소를 쓰는가에 따라서 달라집니다.

이런 일은 `loadedmetadata`뿐만 아니라, `loadstart` `canplay`등등의 여러 이벤트에서 볼 수 있습니다.

# 브라우저에서 이 문제를 수정할 수 있을까요?

이 현상은 버그가 **맞습니다**. 하지만 브라우저 레벨에서 이 문제를 수정하기는 쉽지 않을겁니다. 브라우저는 비디오를 최대한 빠르게 로딩할려고 할 것이고, 우리가 특정한 이벤트를 listen 하고 싶어하는지 JS 코드를 다 해석하기 전까지 알 방법이 없습니다.

그러니까, 브라우저는 특정한 이벤트, 예를 들어서 `loadedmetadata`의 경우에 해당 이벤트에 대한 event listener 가 없으면, 따로 처리를 하지 않고, 해당 이벤트를 버려 버립니다. 나중에 event listener가 로딩 되면, 그 이벤트가 다시 생기지 않는 한, event listener는 실행될 일이 없게 되겠지요.

# 그럼 어떻게 이 문제를 수정하나요?

이 문제를 해결하려면, 당연히 브라우저가 비디오를 로딩하기 전에 event listener를 설정해야 한다는 자연스러운 결론을 얻을 수 있습니다. 한가지 하결책으로는, `<video>` 요소 안에 속성으로 event listener를 넣을 수 있지요.

```html
<video src="test.webm" onloadedmetadata="alert('Got loadedmetadata!')"></video>
```

아니면, `<video>`요소를 만드는 코드 내에서 event listener를 설정할 수 있습니다.

```html
<script>
  var video = document.createElement("video");
  video.onloadedmetadata = function (e) {
    alert("Got loadedmetadata!");
  };
  video.src = "test.webm";
  document.body.appendChild(video);
</script>
```

이 두가지 방법 모두 네트워크 상태에 관계 없이 잘 작동하는 방법입니다. 언제나 `loadedmetadata` 이벤트를 제대로 listen 할 수 있다는 뜻이지요.

javascript를 inline으로 쓰기 싫거나, JS가 로딩될 동안, 사용자들이 비디오에 대한 아무것도 못보고 기다리기 보단, 그래도 브라우저에 내장된 `<video>` 재생 기능을 사용할 수 있게 하다가, JS가 로딩되면 우리가 만든 멋진 동영상 재생 기능을 쓸 수 있게 하고 싶을수도 있습니다.

이 목적을 달성하기 위해서는, 브라우저에서 이벤트를 처리할 때 생기는 과정인 [버블링과 캡쳐링](https://ko.javascript.info/bubbling-and-capturing)을 활용해서 부모 요소에 event listener를 설정하는 방법도 있습니다.

이때, 부모 요소에 설정할 event listener의 두번째 인자를 true로 설정함으로써, 캡처링 단계에서 처리할 수 있도록 해야합니다. 미디어 요소에서 생기는 이벤트는 **버블링이 되지 않는** 이벤트이기 떄문에 꼭, 캡쳐링 단계에서 이벤트를 잡아줘야 합니다.

```html
<head>
  <script>
    window.addEventListener(
      "loadedmetadata",
      function (e) {
        alert("Got loadedmetadata!");
      },
      true
    );
  </script>
</head>
<body>
  <video src="test.webm"></video>
</body>
```

여기서는 단순히 `alert` 창을 하나 띄우는것으로 그쳤고, 하위 요소가 `<vided>`하나 밖에 존재하지 않아서 딱히 문제가 되지 않지만, 여러 미디어 요소가 있고, 그 요소들이 로딩이 될 때마다 `loadedmetadata` 이벤트가 발상할 것입니다.

우리가 원하는 이벤트만 잡아서 처리를 하기 위해서는, event listener의 매개변수인 `e` 에서 이벤트가 생긴 대상을 알려주는`e.target`을 논리적으로 판별해서, 우리가 원하는 요소의 이벤트만 처리를 할 수 있겠습니다.

# 참고 할만한 글

[https://ko.javascript.info/bubbling-and-capturing](https://ko.javascript.info/bubbling-and-capturing) : 위에서 링크로도 간략히 언급한 이벤트 버블링과 캡처링에 관한 글

[https://joshua1988.github.io/web-development/javascript/event-propagation-delegation/](https://joshua1988.github.io/web-development/javascript/event-propagation-delegation/) : 또 다른 이벤트 버블링과 캡처링에 관한 글. 조금 더 읽기 쉬운 문체로 되어 있고, 이를 활용한 예시인 이벤트 위임 패턴에 대해서도 다루고 있다.

끝까지 읽어주셔서 감사합니다.
