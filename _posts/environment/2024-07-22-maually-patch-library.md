---
title: 내 손으로 node_module 패치하기
subtitle: 개발자가 농사 지으러 가버려서 유지보수를 안한다면 내가 고쳐서 써야지...
layout: post
category: environment
image: /images/thumbnails/patch-package.png
---

# 들어가며

요즘은 졸업 요건을 채우기 위한 현장실습을 하러 대구에 있는 모 회사에서 열심히 일을 하고 있습니다. 최근 하고 있는 일은, React Native로 회원 관리 어플리케이션을 만드는 일인데, 요구사항 중에 이미지 캐로샐이 있어서, React Native 기본 컴포넌트에 없는 것이고, 웹 프로그래밍과 다르게, AOS, iOS 네이티브 코드와 함께 대화를 해야하는 RN 개발 특성상, [react-native-snap-carousel](https://github.com/meliorence/react-native-snap-carousel) 이라는 외부 라이브러리를 사용하기로 했습니다.

문제라면 3년이라는 기간동안 업데이트가 끊기면서, React Native도 변화했고, AOS와 iOS도 많은 변화가 있어왔다는 것 입니다. 그래서 `node_modules` 안에 들어있는 외부 의존성 파일을 수정해야 하는데, `node_modules`는 일반적으로 `.gitignore`안에 넣어서 수정을 해도, 그 변화가 당장 내가 코딩하는 '내 컴퓨터' 안에서만 유지가 되고, 내 동료의 컴퓨터나, CI/CD 서버에 반영이 안된다는것이 꽤나 심각한 문제입니다...

라이브러리 메인 스트림 코드에 PR을 날려서 머지가 되고, 그게 다시 배포가 된다면 문제가 되지 않겠지만, 중요한것은 결국 지금 당장 돌아가는 코드입니다. 이번 포스트에서는 이러한 상황에서, 어떻게 해야할지에 대해서 간단하게 알아보는 글을 써보려고 합니다.

# react-native-snap-carousel 패치

아까도 말했듯이, 처음 `patch-package`를 꺼내게 된 이유는 이 친구 입니다. 더이상 유지보수가 되고 있지 않지만, 이미지 캐로셀은 누군가가 반드시 만들어 줘야 하기에, 이미 검증 받은 친구를 쓰는 것이었죠.

React Native 버전이 변화함에 따라서, 특정 prop이 작동을 안하게 되었고, 그 결과로 이슈가 생겼습니다. 다행히도, 오픈 소스 정신을 가진 다른 개발자분들이 [본인의 이슈 해결 방법을 공유](https://github.com/meliorence/react-native-snap-carousel/issues/961) 해주셨더라구요.

간단히 위의 내용을 요약하자면 다음과 같습니다.

1. node_module 폴더를 헤집으면서 필요한 수정을 가한다.
2. `npx patch-package react-native-snap-carousel` 을 하여 패치 파일을 만든다.
3. ```json
   "scripts": {
     "postinstall": "patch-package"
   }
   ```
   을 이용하여 패치가 `npm i` 이후에 반영되게 한다.

![patch-package-usage](/images/environment/patch-package-usage.png)

위 사진은 `patch-package`라이브러리의 readme.md 입니다. 나름 직관적이고, 떔빵하기 좋은 도구죠. 결국 최선의 패치는 적극적으로 유지보수 해줄 개발자. (나 또는 너 또는 우리) 가 되야겠지만...

# react-native-text-size 패치

RN은 웹의 css와 **비슷**한 문법을 통해서 스타일링을 합니다. **비슷** 하다는 점에 강조를 한 점에서 알 수 있듯이, 웹 식으로 사고를 하면 가끔씩 사고가 나는 경우가 있습니다 😂

이번에 개발하는 어플리케이션 중에서, 설명 텍스트가 한번에 들어가지 않을 정도로 길면, 우선 ... 으로 잘라내고(이건 RN Stylesheet로 가능) 접었다 펼 수 있는 기능을 넣어야 했습니다.(이건 RN Stylesheet로 불가능)

그렇기에, 폰트 관련 정보를 넣고, 실제로 렌더링 되면 너비나 높이가 얼마나 나올까 예측하는 [react-native-text-size라는 라이브러리](https://www.npmjs.com/package/react-native-text-size)를 사용해서 구현 하려고 했습니다. 라이브러리는 참 잘 만들어서 문제는 없었고, 크로스 플랫폼 특성상 에러가 안뜨면 좋았쓰 하고 넘기면 되지만, 에러가 나와서 빌드가 안되면 좋았쓰가 안됩니다.

![yoshi!!!](/images/environment/meme-cat.jpeg)

다행히 이번 문제는 단순했습니다. 3~5년이라는 시간 동안 텍스트 사이즈를 구하는 로직 자체는 바뀔 리 없었고, 위의 캐로셀 라이브러리와 다르게 뷰를 건드리지 않고 단순히 숫자만 추정하는 라이브러리였기에, 안드로이드 sdk 타겟만 들어가서 바꾸고 끝 했습니다. 제가 작성한 간단한 패치 파일을 공유합니다.

```diff
diff --git a/node_modules/react-native-text-size/android/build.gradle b/node_modules/react-native-text-size/android/build.gradle
index 2617bd7..661447d 100644
--- a/node_modules/react-native-text-size/android/build.gradle
+++ b/node_modules/react-native-text-size/android/build.gradle
@@ -8,10 +8,10 @@ def safeExtGet(prop, fallback) {
     return rootProject.ext.hasProperty(prop) ? rootProject.ext.get(prop) : fallback
 }

-def _buildToolsVersion  = safeExtGet('buildToolsVersion', '28.0.3')
-def _compileSdkVersion  = safeExtGet('compileSdkVersion', 28)
-def _targetSdkVersion   = safeExtGet('targetSdkVersion', 28)
-def _minSdkVersion      = safeExtGet('minSdkVersion', 16)
+def _buildToolsVersion  = safeExtGet('buildToolsVersion', '34.0.0')
+def _compileSdkVersion  = safeExtGet('compileSdkVersion', 34)
+def _targetSdkVersion   = safeExtGet('targetSdkVersion', 34)
+def _minSdkVersion      = safeExtGet('minSdkVersion', 23)

 buildscript {
     repositories {
```

# 마치며

회사 현장실습 일을 하면서 글을 쓸 수 있는 주제는 정말 풍성하게 늘어나지만, 글을 쓸 시간이 풍성하지 못하네요... 월차 쓴 기념으로 하나 써봤습니다. 끝까지 읽어주셔서 감사합니다!
