---
layout: post
title: RN에서 애니메이션을 해보기 위한 박치기 기록
subtitle: 저만의 react-native-reanimated getting started 입니다.
category: react native
image: /images/thumbnails/React.png
---

# 들어가며

이전 글에서도 지나가듯 언급했는데, 저는 요즘 대구에 있는 모 회사에서 열심히 개발을 하고 있습니다. 하고 있는 일 중에서 비중이 정말로 큰것이 React Native 인데, 기획 담당인 부장님과 소통을 하면서, 개발하는 어플리케이션에 스크롤에 반응하는 헤더가 있어야 한다는 이야기를 들었습니다.

React Native를 학교 캡스톤 시간 등에 해 봤지만, 애니메이션과 같은 그런 화려한 작업 등은 해보지 않기도 했고, scroll 이벤트 등을 바로 WEB API를 통해서 끌어다 쓸 수 있는 웹 환경과 다르게 이런 스크롤을 어떻게 감지해야 하나 같은것도 전혀 몰랐기에, 이 지면을 빌려서, React Native에서 스크롤과 같은 **제스처**를 처리하게 도와주는 `react native gesture handler`와 그리고 애니메이션을 효과적으로 처리해주는 `react native reanimated`의 독스를 읽기 위한 박치기 과정을 간단하게나마 요약해보고자 합니다.

# native와 web의 차이점

라이브러리 박치기 이전에 짚고 넘어갈 가치가 있는 내용을 하나 말해보고자 합니다. 웹은 JS가 네이티브로 돌아지만, RN 환경은 안드로이드나 iOS 환경에서는 JS가 네이티브하게 돌아갈 리 없습니다.

그래서 React Native 에서는 Bridge 라는 것을 이용해서 JS 스레드와 네이티브 스레드를 연결시켜 줍니다.

![Bridge 도식]("/images/frontend/RNBridge.png")

출처 : [https://brocoders.com/blog/hire-react-native-developer/]

그렇기 때문에 React native 환경에서 performant 한 작업을 하기 위해서는 JS 스레드에서가 아닌, Native 스레드에서 직접 작업을 할 필요가 있습니다. 이 간단한 상식을 염두해 두고, 앞으로의 글을 읽어주셨으면 좋겠습니다.

# `react-native-gesture-handler`

기존 React Native에서도 gesture를 감지하는 기능이 당연히 있었습니다만, 플랫폼의 기본 동작이라던가, gesture handler간의 관계를 정의하는 등의 동작을 위해서 이 라이브러리가 빛을 발한다고 하네요. 사실 저에게 가장 중요한것은, 이 라이브러리를 만든 단체인 software mansion에서 만든 `react-native-reanimated`와 매우 강하게 결합되어 있다 라는 점입니다. 종종 강한 커플링은 문제로 여겨지긴 하지만, 생태계를 강하게 구축하고 있다는 점은 매우 좋은 일이기도 하니까요.

## 설치

기본적인 설치는 여타 React native용 라이브러리와 엇비슷 합니다.

우선 yarn을 통해서 설치해 줍시다.

```sh
yarn add react-native-gesture-handler
```

그리고 iOS를 위한 pod linking도 해줍시다.

```sh
cd ios
pod install
cd ..
```

그리고 **모든** 엔트리 포인트를 `<GestureHandlerRootView>` 또는 `gestureHandlerRootHOC`로 감싸줘야 합니다. `react-navigation`등으로 여러 페이지를 렌더링 하는 경우에 빠짐없이 감싸줘야 함을 유의해 주세요.

특정 라이브러리를 사용하기 위해서 Context로 해당 라이브러리를 사용할 부분을 Wrap 하거나 했던 기억이 떠오르실 수 있겠습니다. 그것과 비슷하다고 보시면 됩니다. 물론 이것은 context가 아니고, 특정 제스처를 감지하기 위한 최상위 뷰라서 약간 다르긴 하지만요...

그리고 `index.js`에 import문을 최상위에 적어줘야 합니다.

```js
import "react-native-gesture-handler";
```

## 사용법

사실 저의 사용 용례는 `Animated.FlatList`에서 스크롤 이벤트만 사용하기 때문에 해당 프로젝트에서 직접적으로 사용하지는 않았지만, [공식 docs를 따라서 예제를 만들어 봤습니다](https://docs.swmansion.com/react-native-gesture-handler/docs/fundamentals/quickstart/).

# `react-native-reanimated`

아까 브릿지에 대해서 설명하면서, React Native 앱은 여러 스레드로 이루어져 있다고 간단하게 언급하였습니다. 이 라이브러리는 UI 스레드에서 애니메이션을 처리함으로서, RN에서 제공하는 animation 라이브러리보다 빠르고, 아까도 언급했듯 `react-native-gesture-handler`와의 연결로 특정 제스처가 트리거가 되어서 생기는 애니메이션도 선언적으로 쉽게 구현할 수 있다는 특징을 가집니다.

## 설치

기본적으로 yarn으로 설치를 합니다.

```sh
yarn add react-native-reanimated
```

**babel 설정을 수정해야 하는 라이브러리 입니다** babel 맨 마지막에 `react-native-reanimated`를 위한 플러그인을 붙여줍시다.

```js
  module.exports = {
    presets: [
      ... // 여기는 아니고
    ],
    plugins: [
      ...
      'react-native-reanimated/plugin',
    ],
  };
```

이것이 필요한 이유는, `react-native-reanimated`를 사용하는데 필요한 worklet을 적절한 위치에 삽입하기 위함입니다. worklet 같은 해당 라이브러리에 중요한 컨셉은 나중에 자세히 설명하겠습니다.

iOS를 위해서 pod도 설치해 줍시다.

```sh
cd ios
pod install
cd ..
```

## 핵심 개념

이 라이브러리에 중요한 개념들이 여럿 있겠지만, 박치기를 하기 위해서 꼭 필요한 개념 둘을 소개해보고자 합니다. worklet과 `useSharedValue` 입니다.

### worklet

부드러운 애니메이션을 위해서는 역시 60fps를 요구하겠지요. 이를 위해서는 1프레임이 16ms 안에 와야하고, RN에서 기본적으로 제공되는 `gesture`와 `Animated API`의 UI thread와 JS thread간 통신에 의존하다 보면, 이 둘의 통신은 비동기이기에 16ms에 한 프레임이 오리라는 보장을 하기 힘들어 집니다. 그래서 worklet이라는 개념을 `react-native-reanimated`에서 도입하여 UI thread에서 애니메이션 관련 로직을 모두 실행시켜 버리자는 개념이 도입 되었습니다.

### `useSharedValue`

worklet을 통해서 빠른 애니메이션을 얻었습니다만, JS 스레드도 애니메이션이 어떻게 돌아갔는지 정도는 알 필요가 있습니다. 학교에서 멀티스레딩 프로그래밍을 찍먹할 때, mutex라던가 하는 그런것들을 통해서 스레드간 값을 주고받고 했던 기억이 어렴풋이 났으면 좋겠습니다. `react-native-reanimated`에서는 `sharedValue`라는 개념을 사용하여 이를 해결한다고 보시면 됩니다.

서로 성격이 다른 두 스레드에서 공유되기도 하고, 두 스레드에서 이 `sharedValue`가 취급되는 방식이 상이함에 주목할 필요가 있습니다. 일반적으로 react 의 state는 **비동기적**으로 이루어짐을 react에 익숙하신 분들은 아시리라 믿습니다. async-await 등으로 조정할 수 있는 Promise와는 다르게 이 상태 변화의 관리는 react가 관리하기 때문에, 익숙하지 않은 분들은 이 함정에 빠진 경우가 상당 수 될 정도죠.

애니메이션이라는 특수한 용례에서는 이 원칙을 일부 부정할 필요가 있습니다. 당장 16ms마다 한 프레임이 나와야 하는데, 사용자의 잦은 입력으로 잦은 리렌더링을 방지한다는 그 개념은 여기에 어울리지 않음을 단박에 이해할 수 있지요. 하지만, 여러 애니메이션이 돌아갈 때, 16ms라는 납기일은 반드시 지켜야 하니, 이런 변화들을 배치(batch)로도 처리할 필요가 있습니다. 이 역할을 `sharedValue`가 해준다고 생각하면 될 듯 합니다. 그리고 `withTiming`, `withSpring`와 같은 `sharedValue`를 사용하는 친구들의 디펜던시 트랙킹을 하면서, 불필요한 재연산을 줄이는 역할을 한다고 이해하면 좋을 것 같습니다. (일단 저는 이렇게 이해했습니다)

그래서 간단히 요약하자면, 애니메이션 이라는 특수한 환경에서 부드러운 애니메이션을 위한 빠른 계산과, 특정 프레임마다 값이 나와야 하는 동기성을 충족시키면서, 같은 mutex를 공유하는 요소들의 동일함을 보장해주는, 애니메이션의 기저에 깔려 있는 기반암 이라고 이해하면 좋을 듯 합니다.

# 실제 사용

사실 앞의 이론적인 부분도 꽤나 흥미로울 법 하지만, 사실 결국 중요한것은 **실제로 돌아가는 코드** 입니다. 군말없이 제가 해당 프로젝트에서 썼던 custom hook을 공유하고자 합니다.

```ts
import {
  useSharedValue,
  useAnimatedStyle,
  useAnimatedScrollHandler,
  withTiming,
} from "react-native-reanimated";

export const useAnimatedHeader = (
  headerHeightInitial: number,
  collapsedHeight: number
) => {
  const translateY = useSharedValue(0);
  const headerHeight = useSharedValue(headerHeightInitial);
  const scrollY = useSharedValue(0);

  const scrollHandler = useAnimatedScrollHandler((event) => {
    scrollY.value = event.contentOffset.y;

    if (scrollY.value > headerHeightInitial) {
      translateY.value = withTiming(-headerHeightInitial - 50, {
        duration: 200,
      });
      headerHeight.value = withTiming(collapsedHeight, { duration: 200 });
    }
  });

  const animatedHeaderStyle = useAnimatedStyle(() => {
    return {
      transform: [{ translateY: translateY.value }],
      height: headerHeight.value,
    };
  });

  const handleMomentumScrollEnd = () => {
    translateY.value = withTiming(0, { duration: 200 });
    headerHeight.value = withTiming(headerHeightInitial, { duration: 200 });
  };

  return {
    animatedHeaderStyle,
    scrollHandler,
    handleMomentumScrollEnd,
  };
};
```

스크롤을 하면 줄어들어서 `FlatList`로 보는 아이템을 좀 더 넓게 볼 수 있게 하다가, 스크롤이 멈추면, 여러 기능들을 하는 헤더를 다시 노출 시켜서, 사용할 수 있게 하는 그러한 간단한 hook 입니다.

헤더 부분은 `Animated.View`로 해서 `animatedHeaderStyle`을 스타일로 적용시키고, 스크롤 이벤트가 발생하는 `FlatList`를 `Animated.FlatList`로 대체하여 아래의 prop을 적용시킨다고 생각하시면 될 것 같습니다.

```tsx
<Animated.Flatlist
  // 기타 필요한 여러 prop들
  onScroll={scrollHandler}
  onMomentumScrollEnd={handleMomentumScrollEnd}
/>
```

# 마치며

짧은 기간 동안에 처음 보는 라이브러리 박치기를 해보면서 정리 안하면 잊어버릴 법 한 개념들을 제 블로그 한켠에 정리 해 보았습니다. 사실 이 사용예가 이번의 처음이기도 해서, 간혹 잘못된 정보가 있을 수 있습니다. 혹시 잘못된 정보를 발견하셨다면, 댓글로 상세히 알려주시면, 잘못된 정보의 전파를 막기 위해서라도 즉각 수정하겠습니다. 감사합니다.

# 참고한 글

- [https://velog.io/@strongorange/React-Native-브릿지](https://velog.io/@strongorange/React-Native-브릿지)
- [https://mobileledge.medium.com/how-does-react-native-reanimated-usesharedvalue-work-77e3baae7aa3](https://mobileledge.medium.com/how-does-react-native-reanimated-usesharedvalue-work-77e3baae7aa3)
- [https://medium.com/crossplatformkorea/원리와-예제를-통해-react-native-reanimated-v2-입문하기-336e832f6ed6](https://medium.com/crossplatformkorea/원리와-예제를-통해-react-native-reanimated-v2-입문하기-336e832f6ed6)
