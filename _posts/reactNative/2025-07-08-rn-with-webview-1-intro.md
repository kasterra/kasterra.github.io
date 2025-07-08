---
layout: post
title: 웹뷰와 React Native의 조화 ① - 개요
subtitle: React Native와 웹뷰, 실제 사용 경험을 중심으로
category: react native
image: /images/thumbnails/RN-with-webview.png
---

# 도입 - React Native에서 웹뷰의 필요

React Native는 **크로스플랫폼 네이티브** 애플리케이션을 개발하는 도구입니다. 현대 웹, 특히 웹앱은 브라우저만 설치되어 있다면 유사한 사용자 경험을 제공할 수 있어, 이 둘은 자연히 비교 대상이 되곤 합니다.

그렇다면 이렇게 비슷한 두 기술을 굳이 함께 써야 할 이유가 있을까요? 실제 모바일 앱 개발 현장에서는 웹과 네이티브 기술을 **혼합해 사용할 필요가 있는 상황**이 자주 발생합니다. 외부 SDK가 제공되지 않거나(PASS 인증 등), 지도·결제·로그인 같은 기능을 이미 만들어진 웹 페이지로 대체할 수 있을 경우, 웹뷰는 빠르고 효율적인 해결책이 됩니다.

웹뷰가 필요하다는 점에 공감했다면, 이제는 ‘어떻게 사용할 것인가’를 구체적으로 고민해야 할 시점입니다.

# 어떻게 사용할 것인가? - 데이터 흐름

React 기반 웹 앱 개발 경험과 바닐라 웹 개발까지 고려하면, 브라우저에서 렌더링되는 웹 페이지는 단방향 또는 양방향으로 사용자와 서버 간 데이터를 주고받아 왔습니다. 프론트엔드 웹 개발 경험이 있다면 익숙할 용어들로 설명해보겠습니다.

## 단방향 데이터 전달

가장 기본적인 웹 기술 중 하나는 `SearchParams`입니다. 예를 들어 `/user` 페이지로 이동할 때, URL 뒤에 `?id=150`을 추가해 `150`이라는 id를 가진 사용자 페이지를 요청하는 단순한 데이터 전달 방식입니다.

React를 이용한 웹 앱 개발 경험이 있다면, `prop`을 통한 데이터 전달도 익숙할 것입니다. 직렬화 가능한 데이터뿐 아니라 JS 함수 인자로 넘길 수 있는 다양한 값도 전달할 수 있어 여러 용도로 활용됩니다. prop을 통한 데이터 전달 역시 대표적인 단방향 데이터 흐름의 예입니다.

HTTP 통신을 통해 API 서버 등에서 데이터를 받아오는 것도 단방향 데이터 전달의 특성을 갖습니다. 실제 사례에서는 단 한 번의 요청-응답으로 끝나지 않고 상황에 맞게 호출되므로 다소 변형되어 사용됩니다.

## 양방향 데이터 교환

HTTP 통신을 언급한 이유는 웹 생태계에서 양방향 데이터 교환 방법인 **웹소켓**과 비교하기 위해서입니다. 웹소켓의 핵심 특징은 ‘위계 없는 데이터 교환’입니다. 전통적인 HTTP 통신은 서버가 클라이언트 요청에만 응답하지만(물론 SSE(Server Sent Event) 같은 기술도 있으나 여기서는 제외), 웹소켓은 서버가 필요할 때 즉시 알림을 보낼 수 있습니다. 예를 들어 채팅 클라이언트에서 사용자가 지속적으로 폴링하지 않아도 새 메시지가 도착하면 바로 받을 수 있도록 설계되어 있습니다.

웹뷰와 네이티브 앱 간 단방향 통신은 지도 임베드처럼 한 번 데이터를 전달하고 끝나는 경우에는 문제가 없지만, 실시간 데이터 교환이 필요한 상황에서는 어려움이 발생할 수 있습니다. 예를 들어 웹뷰에서 파일 다운로드를 눌렀을 때, 네이티브에서 다운로드가 진행되고 오류가 발생하면 웹뷰 페이지에서 오류 안내를 해야 하는 상황 등이 그렇습니다.

# 다시 보는 CS 지식 - 소켓 통신

웹뷰와 네이티브 앱이 메시지 기반 통신을 구현하는 방식을 살펴보면서, 학교에서 배운 ‘소켓 통신’과 닮은 점이 많다는 것을 확인했습니다. 이에 별도의 섹션으로 분리해 간단히 다루겠습니다.

소켓은 사전적 의미 그대로 연결 단자의 역할을 합니다. 우리가 일상에서 전기 콘센트를 socket이라고 부르는 것과 유사합니다.

![영어권에서 판매중인 socket](/images/reactnative/rn-with-webview-1/socket.png)

네트워크 통신에서 소켓도 이와 같이 접합부를 통해 전기 신호를 주고받듯, 소켓 통신에서도 같은 개념이 적용됩니다.

아래는 UNIX 환경의 C언어로 소켓을 생성하는 예시입니다.

```c
int sockfd;
struct sockaddr_in server_addr;

// 소켓 생성 (IPv4, TCP)
sockfd = socket(AF_INET, SOCK_STREAM, 0);
if (sockfd < 0) {
    perror("socket 생성 실패");
    exit(EXIT_FAILURE);
}

// 서버 주소 설정
memset(&server_addr, 0, sizeof(server_addr));
server_addr.sin_family = AF_INET;
server_addr.sin_port = htons(8080); // 포트 번호
server_addr.sin_addr.s_addr = inet_addr("127.0.0.1"); // 루프백 주소

// 서버에 연결 시도
if (connect(sockfd, (struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
    perror("서버 연결 실패");
    close(sockfd);
    exit(EXIT_FAILURE);
}

printf("서버에 연결되었습니다.\n");
```

연결이 완료되면 상대와 데이터를 주고받습니다.

```c
// 메시지를 서버로 전송
const char *msg = "Hello, Server!";
if (send(sockfd, msg, strlen(msg), 0) < 0) {
    perror("데이터 전송 실패");
    close(sockfd);
    exit(EXIT_FAILURE);
}
printf("서버로 메시지를 전송했습니다: %s\n", msg);

// 서버로부터 응답 수신
char buffer[1024];
int len = recv(sockfd, buffer, sizeof(buffer) - 1, 0);
if (len < 0) {
    perror("데이터 수신 실패");
    close(sockfd);
    exit(EXIT_FAILURE);
}
buffer[len] = '\0'; // 널 종료
printf("서버로부터 응답을 받았습니다: %s\n", buffer);
```

일부 요청에 대해서는 타임아웃 상황에도 대응할 수 있도록 설정할 수 있습니다.

```c
// 수신 타임아웃 설정 (5초)
struct timeval timeout;
timeout.tv_sec = 5;
timeout.tv_usec = 0;

if (setsockopt(sockfd, SOL_SOCKET, SO_RCVTIMEO, &timeout, sizeof(timeout)) < 0) {
    perror("타임아웃 설정 실패");
    close(sockfd);
    exit(EXIT_FAILURE);
}
printf("5초의 수신 타임아웃이 설정되었습니다.\n");
```

소켓 연결 종료 부분도 있으나 이번 주제와는 다르므로 간단히 언급하고 넘어가겠습니다.

```c
// 소켓 연결 종료 및 자원 해제
close(sockfd);
printf("소켓 연결을 종료하고 자원을 해제했습니다.\n");
```

# 실제 웹뷰와 통신하기

저수준 C 코드를 살펴봤으니, 이제 실제 코드 예제를 살펴보겠습니다. 분량 관계로 자세한 설명은 추후 게시물에서 다루고, 여기서는 최소한의 코드만 소개합니다.

## RN 측 (react-native-webview)

React Native 환경에서 웹뷰를 사용할 때는 `react-native-webview` 라이브러리를 주로 활용합니다. 이 라이브러리에서는 웹뷰와의 데이터 송수신을 위해 컴포넌트 자체와 `ref`에 있는 `onMessage` 속성을 사용합니다.

아래 예제는 개발자가 지정한 형식(JSON)에 맞게 데이터를 파싱하고, 설정한 프로토콜에 따라 RN 측에서 동작을 수행하는 예시입니다.

```jsx
<WebView
  onMessage={(event) => {
    const data = JSON.parse(event.nativeEvent.data);
    if (data.type === "nativeBack") {
      navigation.navigate(data.target);
    }
  }}
/>
```

RN에서 웹뷰로 메시지를 전달하는 예시도 소개합니다.

```js
// WebView의 ref를 통해 메시지 전달
const webviewRef = useRef(null);
const sendMessageToWebView = () => {
  const message = JSON.stringify({
    type: "showToast",
    message: "Hello from React Native!",
  });
  if (webviewRef.current) {
    webviewRef.current.postMessage(message);
  }
};
```

이렇게 생성한 `ref`를 실제 컴포넌트에 연결하면 정상적으로 동작합니다.

## 웹뷰 측

웹뷰에서는 `react-native-webview`에서 보낸 메시지를 수신할 수 있어야 합니다. JS가 이벤트 기반 패러다임을 많이 사용하므로 어렵지 않습니다. Android와 iOS 환경에 따라 이벤트 리스너를 부착할 위치만 신경 쓰면 됩니다.

```ts
const isAndroid = /Android/i.test(navigator.userAgent);
const whereToAddListener = isAndroid ? document : window;

// @ts-expect-error webview message
whereToAddListener.addEventListener("message", (event: MessageEvent) => {
  const parsed = JSON.parse(event.data);
  if (parsed.type === "mobile_auto_login") {
    console.log(`[WebView] Received message: ${event.data}`);
  }
});
```

Typescript에서는 `message` 이벤트가 정의되어 있지 않아 에러가 발생할 수 있으니, 별도 처리하거나 해당 줄에 대해 에러를 차단하면 됩니다.

메시지를 보내는 코드는 플랫폼에 상관없이 간단히 작성할 수 있습니다.

```js
if (window.ReactNativeWebView) {
  window.ReactNativeWebView.postMessage(
    JSON.stringify({
      type: "mobile_auto_login",
      message: "refresh_token",
      payload: JSON.stringify({
        refreshToken: loginResponse.refreshToken,
      }),
    })
  );
}
```

Typescript 환경에서 타입 문제를 해결하기 위한 간단한 라이브러리인 [react-native-webview-window-declaration](https://github.com/gridaco/dynamix/tree/main/lib/react-native-webview-window-declaration)을 사용하면 TS 에러 없이 편리하게 사용할 수 있습니다.

# 마무리하며

이번 글에서는 React Native 환경에서 웹뷰의 필요성, 웹뷰 환경에서 데이터 전달을 위한 소켓 프로그래밍 기초, 그리고 간단한 웹뷰와 React Native 간 통신 예제를 살펴봤습니다.

다음 글에서는 ‘저수준’ 표준만 존재하는 웹뷰 데이터 교환 환경에서 나름의 질서를 찾아가기까지의 과정을 다룰 예정입니다. 끝까지 읽어주셔서 감사합니다. 오류나 질문이 있으면 댓글로 남겨주시기 바랍니다.

감사합니다!
