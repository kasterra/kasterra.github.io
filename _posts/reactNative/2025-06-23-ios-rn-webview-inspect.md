---
layout: post
title: react-native-webview iOS에서 inspect 안 될 때 디버깅 팁
subtitle: 웹뷰 앱을 만드는데 갑자기 냅다 죽어버리면... 디버깅 합시다
category: react native
image: /images/thumbnails/React.png
---

# 도입 - 문제제기

이번 글은 상당히 짧고 간단하게 쓸 예정입니다.

제목처럼 RN 껍데기의 웹 개발을 하는데 모종의 이유로 해당 페이지가 에러가 나서, 앱이 갑자기 하얗게 멈춘다면, 개발자 도구로 콘솔을 확인하며 문제를 추적해야 합니다. 문제는 웹뷰 내부에서 발생한 에러는 React Native의 디버깅 콘솔로 확인하기 어렵다는 점입니다.

그래서 안드로이드의 경우에는 [Chrome을 활용한 inspect](https://developer.chrome.com/docs/devtools/remote-debugging?hl=ko)를 하고, Safari에서도 비슷하게 설정해 inspect할 수 있습니다.

그런데 react-native-webview의 iOS 구현에서 inspect가 안되는 문제가 발생하였습니다. (RN 0.79.2, react-native-webview: 13.13.5 기준) 꽤 오래된 이슈였는지 [해당 레포 issue 탭](https://github.com/react-native-webview/react-native-webview/issues/2914)에서 저의 문제와 해결책을 찾을 수 있었습니다.

# 해결방법 공유

<https://github.com/react-native-webview/react-native-webview/issues/2914#issuecomment-1502086916>에서 찾은 단서를 참고하여 적용한 저의 커스텀 패치를 공유합니다.

```diff
--- a/node_modules/react-native-webview/apple/RNCWebViewImpl.m
+++ b/node_modules/react-native-webview/apple/RNCWebViewImpl.m
@@ -517,6 +517,7 @@ RCTAutoInsetsProtocol>
   if (self.window != nil && _webView == nil) {
     WKWebViewConfiguration *wkWebViewConfig = [self setUpWkWebViewConfig];
     _webView = [[RNCWKWebView alloc] initWithFrame:self.bounds configuration: wkWebViewConfig];
+    _webView.inspectable = YES;
     [self setBackgroundColor: _savedBackgroundColor];
 #if !TARGET_OS_OSX
     _webView.menuItems = _menuItems;
@@ -559,7 +560,7 @@ RCTAutoInsetsProtocol>
     __IPHONE_OS_VERSION_MAX_ALLOWED >= 160400 || \
     __TV_OS_VERSION_MAX_ALLOWED >= 160400
     if (@available(macOS 13.3, iOS 16.4, tvOS 16.4, *))
-      _webView.inspectable = _webviewDebuggingEnabled;
+      _webView.inspectable = YES;
 #endif

     [self addSubview:_webView];
@@ -594,7 +595,7 @@ RCTAutoInsetsProtocol>
 - (void)setWebviewDebuggingEnabled:(BOOL)webviewDebuggingEnabled {
   _webviewDebuggingEnabled = webviewDebuggingEnabled;
   if (@available(macOS 13.3, iOS 16.4, tvOS 16.4, *))
-      _webView.inspectable = _webviewDebuggingEnabled;
+      _webView.inspectable = YES;
 }
 #endif
```

inspectable 값을 모두 YES로 설정하여 개발 중에 문제없이 inspect가 가능했고, 에러 추적이 쉬워졌습니다.

webview 라이브러리의 특성상 생성된 코드들이 다량 첨가되어서 제가 실제 수정한 부분만 발췌하였음을 밝힙니다.

# 마무리

QA 중 앱이 갑자기 하얗게 멈추는 상황을 마주하고, 열심히 디버깅하며 자료를 찾고 실제로 수정한 내용을 기록으로 남깁니다. 첫째는 제가 나중에 다시 참고할 수 있도록, 둘째는 React Native 여정 중 같은 문제를 겪는 누군가에게 도움이 되길 바라서입니다. 읽어주셔서 감사합니다.
