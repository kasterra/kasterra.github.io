---
layout: post
title: 웹뷰와 React Native의 조화 ② - 라우터와 네이티브 사이드
subtitle: 웹뷰와 React Native를 연결하기 위한 메세지를 라우팅 해봅시다
category: react native
image: /images/thumbnails/RN-with-webview.png
---

# 도입 - 라우터 구현의 필요성

개발을 하다 보면 단순히 있는 기능을 그대로 쓰기보다는, 라이브러리를 도입하거나 특정 개념을 추가해 데이터 흐름을 관리하고 싶어질 때가 있습니다. 이럴 때는 '왜 해야 하는가', '어떤 실익이 있는가'와 같은 타당성을 따져볼 필요가 있습니다.

[이전 글](/rn-with-webview-1-intro)에서 메세지 송수신, 소켓이라는 배경지식도 간단하게 언급했습니다. 이 메시지들은 C언어의 소켓과 달리 타임아웃이나 기타 에러 처리를 직접 구현해야 하며, 다양한 요청들을 하나의 파일이나 함수에 인라인으로 관리하면 함수의 역할이 지나치게 커져 가독성과 유지보수에 문제가 생길 수 있습니다. 실제로 제가 개발 현장에서 해당 라우터를 개발한 이후에, 많은 부분을 추상화하여 뷰 개발에 조금 더 집중할 수 있었습니다.

# 라우터 구현 - `webviewMessageRouter.ts`

메시지의 종류가 여러 가지로 나뉘기 때문에, 이를 효율적으로 관리하기 위한 라우터의 도입이 필요했습니다. 사실 라우터 자체는 그렇게 복잡하지 않습니다. 아래는 가능한 라우터 구현의 한가지 중 하나입니다.

```ts
import { WebViewMessageEvent } from "react-native-webview";

type MessageHandler = (data: any) => Promise<void>;

const handlers: Record<string, MessageHandler> = {};

export function registerHandler(type: string, handler: MessageHandler) {
  if (handlers[type]) {
    console.warn(
      `[WebView Router] Handler for type "${type}" is already registered. Skipping duplicate registration.`
    );
    return;
  }
  handlers[type] = handler;
}

export function unregisterHandler(type: string) {
  if (handlers[type]) {
    delete handlers[type];
  }
}

export function createWebViewMessageHandler() {
  return async (event: WebViewMessageEvent) => {
    try {
      const data = JSON.parse(event.nativeEvent.data);
      const { type } = data;

      if (!type || typeof type !== "string") {
        console.warn("[WebView Router] Missing or invalid message type");
        console.warn(event);
        return;
      }

      const handler = handlers[type];
      if (!handler) {
        console.warn(
          `[WebView Router] No handler registered for type: ${type}`
        );
        return;
      }

      await handler(data);
    } catch (error) {
      console.error("[WebView Router] Failed to handle message:", error);
      console.error(event);
    }
  };
}
```

딱히 어려운 문법은 없는, 단지 핸들러를 등록하고 해지하고, 중복에 관련된 경고 등을 하는 간단한 코드입니다.

TypeScript 유틸리티 타입인 `Record`를 통해 간결하게 핸들러를 관리할 수 있으며, 결국에는 단순한 객체(Object)를 활용한 기본적인 구조입니다.

# 실제 라우터의 적용 - `useWebViewBridge.ts`

이 간단한 웹뷰 라우팅들을 React Native에 적용시켜야겠죠. 마치 redux를 react에 적용시키기 위해서 `react-redux`라는 일종의 연결용 모듈을 덧대서 쓰는것과 유사하다고 볼 수 있겠습니다.

이하의 코드는 가능한 구현 중 한가지의 예시입니다.

```ts
import { useEffect, useRef } from "react";
import WebView from "react-native-webview";
import { createWebViewMessageHandler } from "src/util/webView/webViewMessageRouter";

type HandlerSetup = (
  webViewRef: React.RefObject<WebView | null>
) => void | (() => void);

export function useWebViewBridge(setupHandlers: HandlerSetup[]) {
  const webViewRef = useRef<WebView>(null);

  useEffect(() => {
    const cleanups = setupHandlers
      .map((setup) => setup(webViewRef))
      .filter(Boolean);

    return () => {
      cleanups.forEach((cleanup) => {
        if (typeof cleanup === "function") {
          cleanup();
        }
      });
    };
  }, [setupHandlers]);

  return {
    webViewRef,
    handleMessage: createWebViewMessageHandler(),
  };
}
```

여러 핸들러에서 인자로 받아서 처리하게 될 webview의 `ref`를 핸들링 하는 것, 그리고 해당 메세지 핸들러를 사용하는 컴포넌트들이 마운트 해제 될 때, 핸들러들의 `cleanup`을 실행함으로서, 불필요한 메모리 낭비를 막아주는, 매우 전형적인 react hook의 한 형태입니다.

# 메세지 핸들러의 예시 - `webViewStorageHandler.ts`

어플리케이션의 요구사항에 따라서 다양한 메세지 핸들러가 존재할 수 있지만, 이번 섹션에서는 네이티브의 `asyncStorage`를 메세지를 통해서 접근할 수 있게 하는 `webViewStorageHandler`를 살펴보도록 하겠습니다.

```ts
import AsyncStorage from "@react-native-async-storage/async-storage";
import { registerHandler, unregisterHandler } from "./webViewMessageRouter";
import { WebView } from "react-native-webview";

type GetItemRequest = {
  type: "GET_ITEM";
  key: string;
  requestId: string;
};

type SetItemRequest = {
  type: "SET_ITEM";
  key: string;
  value: string;
  requestId: string;
};

type RemoveItemRequest = {
  type: "REMOVE_ITEM";
  key: string;
  requestId: string;
};

type StorageRequest = GetItemRequest | SetItemRequest | RemoveItemRequest;

export function setupStorageHandlers(
  webViewRef: React.RefObject<WebView | null>
) {
  if (!webViewRef.current) {
    return;
  }

  function reply(type: string, requestId: string, value: string) {
    console.log("Replystorage", type, requestId, value);
    while (!webViewRef.current) {
      console.log("WebView is not ready yet");
    }
    webViewRef.current.postMessage(JSON.stringify({ type, requestId, value }));
  }

  const getItemHandler = async (data: StorageRequest) => {
    try {
      console.log(data);
      const value = (await AsyncStorage.getItem(data.key)) ?? "";
      reply(data.type, data.requestId, value);
      reply(data.type, data.requestId, value);
    } catch (e) {
      reply(data.type, data.requestId, "__ERROR__");
    }
  };

  const setItemHandler = async (data: StorageRequest) => {
    if (data.type === "SET_ITEM") {
      console.log(data);
      try {
        await AsyncStorage.setItem(data.key, data.value);
        reply(data.type, data.requestId, "OK");
      } catch (e) {
        reply(data.type, data.requestId, "__ERROR__");
      }
    }
  };

  const removeItemHandler = async (data: StorageRequest) => {
    try {
      await AsyncStorage.removeItem(data.key);
      reply(data.type, data.requestId, "OK");
    } catch (e) {
      reply(data.type, data.requestId, "__ERROR__");
    }
  };

  registerHandler("GET_ITEM", getItemHandler);
  registerHandler("SET_ITEM", setItemHandler);
  registerHandler("REMOVE_ITEM", removeItemHandler);

  return () => {
    unregisterHandler("GET_ITEM");
    unregisterHandler("SET_ITEM");
    unregisterHandler("REMOVE_ITEM");
  };
}
```

이 경우에서는 단지 네이티브는 메세지를 수신만 하고, 별도로 웹뷰측에 요청을 하지 않는, 일종의 '서버' 역할만 하기 때문에 타임아웃 등의 에러 핸들링이 존재하지 않습니다만, 향후 포스트에서 다룰 웹뷰 사이드의 메세지 핸들러 측에서는 요청을 했는데 감감 무소식일 경우를 핸들링 하기 위한 타임아웃 처리, 그리고 여러개의 메세지들이 오갈 때, 각각의 메세지들이 언제 어떤 내용으로 송신한 메세지들에 대한 응답을 의미하는지를 알기 위한 일종의 '메세지 큐'를 구현한 형태도 볼 수 있을 것 입니다.

# 마무리하며

이번 글에서는 웹뷰와 React Native 간의 통신 구조를 체계적으로 구성하기 위해, 라우터 패턴을 적용하고 이를 Hook 형태로 추상화하는 방식, 그리고 실제 스토리지 연동 예제를 통해 메세지 기반 통신이 어떻게 이루어질 수 있는지를 살펴보았습니다.

다음 글에서는 웹뷰 사이드에서 이 메세지를 수신하고 응답을 관리하기 위한 큐 처리, 타임아웃 처리 등, 보다 복잡한 상황에서도 안정적으로 데이터를 주고받을 수 있는 구조에 대해 이야기해보려 합니다.

읽어주셔서 감사합니다. 궁금한 점이나 제안이 있다면 댓글로 남겨주세요!
