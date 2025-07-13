---
layout: post
title: 웹뷰와 React Native의 조화 ③ - 웹뷰 사이드와 에러핸들링
subtitle: 클라이언트 입장인 웹뷰에서 핸들링 해야할 메시지 처리를 조금 더 알아봅시다
category: react native
image: /images/thumbnails/RN-with-webview.png
---

# 들어가며 - 클라이언트는 유저에게 알려줘야 한다

[첫번째 글](/rn-with-webview-1-intro)에서는 웹뷰를 RN앱에서 왜 써야 하는지, 쓰면서 어떤 문제가 있는지를 다루었고, [두번째 글](/rn-with-webview-2-router)에서는 웹뷰 사이드의 메시지를 받은 RN 앱이 어떤 행동을 취할지 결정하는 메시지 핸들러와, 그 핸들러들을 묶어내는 라우터와, 그 라우터를 React Native 환경에 통합하기 위한 과정을 간단히 설명하였습니다.

이번 시리즈의 마지막 글에서는, 클라이언트 역할을 하는 웹뷰에서 네이티브 사이드로 작업을 '의뢰'하고, 그 요청을 '처리'하고 '예외처리'하는 과정을 어떻게 구현할 수 있을지 살펴보겠습니다.

# 웹뷰측 구현 - PDF 다운로드 처리

코드가 조금 길지만, 이 한 예제 안에 이번 글에서 다룰 주요 개념이 모두 담겨 있습니다. 하나씩 나누어 설명해 보겠습니다.

```ts
import { useAuthStore } from "../store/authStore";
import { useState } from "react";

const TIMEOUT_MS = 10000;

type PendingRequest = {
  resolve: () => void;
  reject: (reason?: unknown) => void;
  timeoutId: ReturnType<typeof setTimeout>;
};

type PDFDownloadMessage = {
  type: "downloadPDF";
  requestId: string;
  message: "success" | "error";
  error?: string;
};

const pendingRequests = new Map<string, PendingRequest>();
function isValidPDFDownloadMessage(
  message: PDFDownloadMessage
): message is PDFDownloadMessage {
  return message.type === "downloadPDF";
}

export function usePDFDownLoadInWebview() {
  const [isDownloading, setIsDownloading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function downloadPDFInWebView(id: string): Promise<void> {
    const requestId = `${id}-${Date.now()}-${Math.random()}`;
    setIsDownloading(true);
    return new Promise((resolve, reject) => {
      const timeoutId = setTimeout(() => {
        pendingRequests.delete(requestId);
        reject(new Error("timeout"));
      }, TIMEOUT_MS);

      pendingRequests.set(requestId, { resolve, reject, timeoutId });

      window.ReactNativeWebView.postMessage(
        JSON.stringify({
          type: "downloadPDF",
          accessToken: useAuthStore.getState().accessToken,
          uri: `/api/v2/surveys/responses/${id}/download`,
          requestId,
        })
      );

      if (import.meta.env.DEV) {
        console.log(`[WebView] Sent PDF message: ${JSON.stringify({ id })}`);
      }
    });
  }

  function setupPDFDownloadUtilEventListener() {
    const isAndroid = /Android/i.test(navigator.userAgent);
    const whereToAddListener = isAndroid ? document : window;
    // @ts-expect-error webview message
    whereToAddListener.addEventListener("message", (event: MessageEvent) => {
      console.log(`[WebView] Received message: ${event.data}`);
      try {
        const response =
          typeof event.data === "string" ? JSON.parse(event.data) : null;
        if (!isValidPDFDownloadMessage(response)) return;

        const request = response
          ? pendingRequests.get(response.requestId)
          : undefined;
        if (!request) {
          console.error("PDF download request not found");
          return;
        }

        clearTimeout(request.timeoutId);
        request.resolve();
        pendingRequests.delete(response.requestId);
        setIsDownloading(false);
        switch (response.message) {
          case "success":
            break;
          case "error":
            setError(response.error ?? "");
            break;
          default:
            setError(`unknown msg : ${response.message}`);
        }
      } catch {
        console.warn("Invalid message format:", event.data);
      }
    });
  }

  return {
    isDownloading,
    error,
    downloadPDFInWebView,
    setupPDFDownloadUtilEventListener,
  };
}
```

## 타입 관련

우리가 TypeScript를 쓰는 이유는 무엇일까요? 변화무쌍한 타입 언어인 JS에서 최소한의 안전장치를 마련하기 위함입니다. 필요한 타입을 정의하고, `message is PDFDownloadMessage`처럼 타입 가드 함수를 선언해두면 이후 로직에서 타입 오류를 사전에 방지할 수 있습니다.

```ts
function isValidPDFDownloadMessage(
  message: PDFDownloadMessage
): message is PDFDownloadMessage {
  return message.type === "downloadPDF";
}
```

## 메시지 큐

1편에서 다룬 C언어에서의 소켓에서도, 내부적으로 구현된 여러 소켓 통신 요청들을 담는 메시지 큐가 있습니다. 다운로드 처리에서, 짧은 시간에 여러 개의 파일 다운로드 요청이 들어올 수 있으니, 그 요청들을 담아둘 메시지 큐를 JavaScript의 `Map`으로 간단히 구현한 것입니다.

성공, 에러, 타임아웃 상황에 적절히 대응하기 위해 필요한 정보들을 객체로 감싸 `Map`에 저장해 두는 구조입니다.

```ts
type PendingRequest = {
  resolve: () => void;
  reject: (reason?: unknown) => void;
  timeoutId: ReturnType<typeof setTimeout>;
};
const pendingRequests = new Map<string, PendingRequest>();
```

## 메인로직 - `usePDFDownLoadInWebview`

이 부분은 크게, 실제로 네이티브 사이드로 요청을 보내는 부분과, 요청을 보낸 이후의 여러 처리들을 하는 부분으로 분리해서 볼 수 있습니다.

차례로 살펴보겠습니다

### 다운로드 요청 보내기 - `downloadPDFInWebView`

타임아웃을 설정하고, 실제로 메시지를 송신하고, dev 환경일 때 원활한 추적을 위하여 로그를 남기는 부분만 있는 간단한 유틸 함수입니다. `setIsDownloading` 호출은 UI상에서 다운로드 중일 때 dim 처리와 같은 UI 요구사항이 있을 때 유용하게 사용할 수 있도록 설정한 부분입니다.

```ts
async function downloadPDFInWebView(id: string): Promise<void> {
  const requestId = `${id}-${Date.now()}-${Math.random()}`;
  setIsDownloading(true);
  return new Promise((resolve, reject) => {
    const timeoutId = setTimeout(() => {
      pendingRequests.delete(requestId);
      reject(new Error("timeout"));
    }, TIMEOUT_MS);

    pendingRequests.set(requestId, { resolve, reject, timeoutId });

    window.ReactNativeWebView.postMessage(
      JSON.stringify({
        type: "downloadPDF",
        accessToken: useAuthStore.getState().accessToken,
        uri: `/api/v2/surveys/responses/${id}/download`,
        requestId,
      })
    );

    if (import.meta.env.DEV) {
      console.log(`[WebView] Sent PDF message: ${JSON.stringify({ id })}`);
    }
  });
}
```

### 메시지 처리하기 - `setupPDFDownloadUtilEventListener`

요청에 대한 응답을 받을 수 있도록 메시지 리스너를 설정하는 간단한 구현입니다.

```ts
function setupPDFDownloadUtilEventListener() {
  const isAndroid = /Android/i.test(navigator.userAgent);
  const whereToAddListener = isAndroid ? document : window;
  // @ts-expect-error webview message
  whereToAddListener.addEventListener("message", (event: MessageEvent) => {
    console.log(`[WebView] Received message: ${event.data}`);
    try {
      const response =
        typeof event.data === "string" ? JSON.parse(event.data) : null;
      if (!isValidPDFDownloadMessage(response)) return;

      const request = response
        ? pendingRequests.get(response.requestId)
        : undefined;
      if (!request) {
        console.error("PDF download request not found");
        return;
      }

      clearTimeout(request.timeoutId);
      request.resolve();
      pendingRequests.delete(response.requestId);
      setIsDownloading(false);
      switch (response.message) {
        case "success":
          break;
        case "error":
          setError(response.error ?? "");
          break;
        default:
          setError(`unknown msg : ${response.message}`);
      }
    } catch {
      console.warn("Invalid message format:", event.data);
    }
  });
}
```

결국 이렇게 설정한 것들을 hook에서 꺼내 쓸 수 있도록 `usePDFDownLoadInWebview`의 리턴으로 넣어야겠죠

```ts
return {
  isDownloading,
  error,
  downloadPDFInWebView,
  setupPDFDownloadUtilEventListener,
};
```

# 마무리하며

이번 글에서는 웹뷰와 React Native 간의 통신 구조를 체계적으로 구성하기 위해, 그리고 뷰 역할을 하는 웹뷰에서, 사실상 서비스를 요청하고 응답받는 '클라이언트'로서의 기능을 구현하기 위한 '웹 측 네이티브' 코드를 살펴보았습니다. 클로저 패턴과 custom hook 등, 그동안 이론적으로만 알고 있던 React 관련 개념들이 실제로 어떻게 활용되는지 보여주는 좋은 예시였다고 생각합니다.

끝까지 읽어주셔서 감사합니다. 궁금한 점이나 피드백은 언제든 댓글로 남겨주세요!
