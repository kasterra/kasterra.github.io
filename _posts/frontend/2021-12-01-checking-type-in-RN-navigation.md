---
title: React Navigation에서 TypeScript 사용하기
layout: post
subtitle: 우리에게 뭐든 타입을 지정하게 강제하는 TS에서 잘 살아남아 봅시다
image: /images/frontend/react-ts.png
category: frontend
---

# 개인적인 이야기

지금 제 상황은 주말에 올렸던 글에서도 잠깐 적엇듯이, 이때까지의 바쁜 일이 끝나고, 새로운 일을 시작하기에는 아직 디자인이 나오지 않았기 때문에, 어찌보면 출정을 대기하는 병사와 같은 상태입니다. 회사에서는 이러한 때에 향후에 RN으로 뭔가를 해야할 일이 분명 있을 것이니, RN 공부를 하면서 시간을 잘 보내보라면서 숙제를 받았습니다. 예전에 맨땅에 다이빙하듯, RN을 했을때 뭐가 뭔지 모르고, 위쪽에서 이렇게 짜놨으니 따라서 짜보자 하는 무지성 RN을 할때와는 조금 다른 느낌으로 나중에 RN을 할 수 있도록 공부를 해야긴 하겠는데, 동기부여가 너무 안되있는듯 하여, 어찌할가 곰곰히 생각을 해봤습니다. 블로그 글을 쓰면서, 내 공부 한다고 생각을 하면서 공부를 하면 좀 더 열심히 하지 않을까 하는 생각도 들기도 했고, 사수께서 예전에 무지성 으로 다이빙 할 때, 몇몇 내용들은 블로그에 정리를 해라고 하셨기 때문에, 지금 시간을 내어서 블로그 글을 적어볼까 합니다.

# 들어가며

React Native는 React를 하는 그 느낌 그대로, Native App을 만드는 프레임워크 입니다. 그리고 Typescript도 사용할 수 있지요. Typescript를 사용하면, 타입 실수를 예방할 수 있지만, 그냥 JS에서는 타입 명시를 하지 않고 넘어갈 수 있는 부분들에, 제네릭 등을 활용하여, 타입을 명시해줘야 합니다. 이번 글에서 다룰 React Native Navigation에서도 그러한 타입 정의들을 좀 해줘야 하는 부분들이 있고, 처음 보면 '여기는 왜 `undefined`를 쓰고, 여기에는 왜 제네릭에 들어가는 타입이 2개지?' 하는 생각이 들 수 있고, 그나마 좀 익숙해졌다고 생각한 지금도 계속 구글링을 하면서 매번 같은 페이지를 왔다갔다 하는 제 모습을 보니, 좀 정리를 해둘 필요가 있다고 생각이 들어서 이번 기회에 정리를 좀 해볼까 합니다.

# React Navigation은 무엇이고, 어떻게 사용하는가

React.js에서 주소에 따라서 서로 다른 컴포넌트를 연결하는데 `<Router>`, `<Route>`를 사용 하였던것을 기억할 것입니다. `react-router-dom`이라는 서드파티 라이브러리를 받아서, 사용했던것 처럼, React Native에서도 `@react-navigation` 에서 라이브러리를 다운로드 받아서 화면 전환을 진행하게 할 수 있습니다. 간단한 사용법부터 알아봅시다.
{% raw %}

```jsx
// In App.js in a new project

import * as React from "react";
import { View, Text } from "react-native";
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";

function HomeScreen() {
  return (
    <View style={{ flex: 1, alignItems: "center", justifyContent: "center" }}>
      <Text>Home Screen</Text>
    </View>
  );
}

const Stack = createNativeStackNavigator();

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

export default App;
```

<div style="display:flex;justify-content:center;"><https://reactnavigation.org/docs/hello-react-navigation> 에서 c-p한 예제 코드</div>

대략적으로 위의 코드를 소개하자면 이렇습니다. `createStackNavigator()`을 이용해서, `Stack`을 만듭니다. 그리고, 그 만들어진 Stack으로 하단의 `App` Component에서 `Stack.Navigator`과 `Stack.Screen`이라는 요소를 활용하는 것을 볼 수 있습니다. `Stack.Navigator`는 `react-router-dom` 에서, `<Router>`에 대응하고, `Stack.Screen`는 `<Route>`에 대응한다고 보시면 됩니다. `Stack.Screen`의 `name` prop에는 주소에 해당하는 이름을 넣고, `component` prop에는 렌더링 될 컴포넌트를 넣습니다. 이것 말고도 여러 옵션들이 있지만, 다루고자 하는 주제에서 좀 벗어나는 내용이기 때문에 넘기도록 하겠습니다.

중요한점은, 위 코드에서 `<NavigationContainer>`로 Stack 예하의 컴포넌트가 감싸져 있다는 사실입니다. 해당 작업을 해주지 않으면, RN 빌드할때 에러가 나오므로, 꼭 해줘야 합니다. `App` 자체를 감쌀 필요는 없지만, **`<Stack.Navigator>`를 반드시 감싸는 형태가 되어야 합니다.** 이 점만 유의하시면 큰 문제는 없으리라 생각됩니다.

위의 JS로 작성된 코드를 이해 할 수 있다면, 이제 이 코드를 Typescript로 전환할 때, 어떤 사항들이 변경이 되야 하는지 알아봅시다.

# React Navigation + Typescript

Typescript를 써보신 분이라면 아시겠지만, 온갖 것에다가 다 타입을 지정해 줘야 합니다. `useState`등을 사용할때도, generic으로 해당 state에서 사용할 타입을 지정할 필요가 있는 언어입니다. 뭐 그게 언어의 정체성이기도 하고, 이러한 철저한 타입 검사로 IDE에서 자동완성 같은것들에 대해서도 JS보다 더 많은 부분을 도움받을 수 있지요. React Navigation 에서도 Typescript를 사용하려면, 제네릭으로 넣어줘야 할 타입이 정말로 많지만, 그 넣은 타입들이 우리를 지켜주는 안전장치가 되고, 자동완성으로 돌아와서 우리에게 도움이 되니, 너무 귀찮게 생각하지 말고 찬찬히 공부해 봅시다.

## createStackNavigator에서 Type 정의 - Navigator에서 타입 검증

위에서 Stack을 만들기 위해서,

```jsx
const Stack = createNativeStackNavigator();
```

라는 코드를 사용했습니다. Typescript는 우선 이 코드에서 경고나 에러를 띄울 것입니다. `StackNavigator`를 생성하는것은 참 좋은데, 그 밑에 할당될 `Stack.Screen`의 prop으로 넘어갈 타입에 대한 검사를 안하고 넘어갈 수는 없기 때문이죠. 그렇기 때문에 아래와 같은 type을 만들어서 createNativeStackNavigator에 generic으로 넣어줍시다.

```tsx
type RootStackParamList = {
  Home: undefined;
  Profile: { userId: string };
  Feed: { sort: "latest" | "top" } | undefined;
};

// 중략

const RootStack = createnativeStackNavigator<RootStackParamList>();

// 중략
<RootStack.Navigator initialRouteName="Home">
  <RootStack.Screen name="Home" component={Home} />
  <RootStack.Screen
    name="Profile"
    component={Profile}
    initialParams={{ userId: user.id }}
  />
  <RootStack.Screen name="Feed" component={Feed} />
</RootStack.Navigator>;
```

위의 코드를 주목해서 봅시다. `Home`과 `Feed` 컴포넌트는 따로 렌더링을 할 때, prop을 던져주지 않습니다. 반면 `Profile`컴포넌트는 `initialParams`라는 prop으로 prop을 던져주는 모습을 볼 수 있지요. `RootStackParamList`라는 이름에서 유추할 수 있듯, 이 변수는 `RootStack`이 연결시켜주는 component들의 Param 즉, prop들의 리스트라고 생각하면 됩니다. `Home` 컴포넌트는 아무런 prop을 받지 않습니다. 그래서 아무것도 넣지 않으면 기본적으로 할당되는 `undefined`로 명시를 해두었고, `Feed`컴포넌트는 prop을 받을 수도 있지만, 안받을 수도 있습니다. 그래서 `Feed: { sort: 'latest' | 'top' } | undefined` 이런 타입도 받을 수 있다고 명시를 해두는 것이지요.

## 렌더링 된 컴포넌트에서 타입 검증

Navigator에서 타입을 검증할 수 있으면, 렌더링된 컴포넌트에서도 prop 타입을 검증할 수 있겠다는 생각이 들것입니다. 그리고 스크린 측에서 신경을 써야할게 몇가지가 더 있습니다. 다양한 네비게이션용 메소드를 담고 있는 `navigation` 프롭, 현재 페이지의 라우팅 정보를 담고 있는 `route` 프롭 이지요. 라우팅된 현재 페이지의 정보를 알기 위해서인지 뭔지는 정확히 모르겠지만, 여기서는 현재 렌더링 될 화면의 이름도 전달해 주어야 합니다. 실제 코드 예제는 아래와 같습니다.

```tsx
import { NativeStackScreenProps } from "@react-navigation/native-stack";

type RootStackParamList = {
  Home: undefined;
  Profile: { userId: string };
  Feed: { sort: "latest" | "top" } | undefined;
};

type profileProps = NativeStackScreenProps<RootStackParamList, "Profile">;
```

이렇게 정의를 해놓으면, `Profile.tsx`에서 해당 type을 쓰면 됩니다. export를 하던 해서 말이죠. 이런 타입들을 따로 파일을 빼두고 관리하기를 권장한다고 하네요. 회사에서도 일을 이런식으로 하고 있습니다.

```tsx
function ProfileScreen({ route, navigation }: profileProps) {
  // ...
}
```

이렇게 `route`, `navigation`을 받아올 수 있습니다.

`navigation`을 받아온 다음에, `navigation.navigate({이동할컴포넌트이름},{파라미터})` 형식으로 navigate가 필요한 버튼에 onPress등으로 넣어두면 되겠네요.

# 참고한 글

- <https://reactnavigation.org/docs/hello-react-navigation>
- <https://reactnavigation.org/docs/typescript/>
  {% endraw %}
