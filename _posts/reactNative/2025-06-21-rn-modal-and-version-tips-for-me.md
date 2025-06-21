---
layout: post
title: RN에서 업데이트 모달창 만들때 유의점
subtitle: 제가 매번 깜빡거리는 몇몇 유의사항들의 정리입니다
category: react native
image: /images/thumbnails/React.png
---

# 들어가며

서울 회사에 입사해서 정신없이 기능을 개발하면서 정리하고 싶은 주제들은 참으로 늘어나고 늘어났지만, 시간이 잘 나지 않아서, 오랫동안 글을 쓰지 못하고 방치해둔 상태였습니다.

더 멋진 주제들이 많지만, 다시 블로그 감각을 되살린다는 마음으로, 제가 이때까지 겪은 종종 쓰이는 RN 개발의 유즈케이스를 하나 정리해 보고자 합니다. 바로 **(강제) 업데이트 다이얼로그** 입니다.

<https://ditoday.com/기획자의-모바일-앱-뜯어보기/> 같은 글들을 보면 알 수 있듯, 우리가 사용하는 모바일 앱 중에서 이렇게 업데이트 모달을 띄우는 경우들이 종종 있습니다. 현재 제가 테헤란로 모 빌딩에서 열심히 만들고 있는 서비스에도 이 기능이 들어가서, 제가 간단하지만 괜히 헤맸던 부분들을 정리하면서 저 스스로에게, 그리고 혹여나 비슷한 방황을 하고 있는 분들에게 도움이 되고자 이 글을 남깁니다.

# 버전 확인

업데이트 다이얼로그를 띄우기 위해서는, '로컬 버전'과, '최신 버전'을 알 필요가 있습니다.

## 로컬 버전 가져오기

[`react-native-version-number`](https://github.com/APSL/react-native-version-number)라는 훌륭한 라이브러리가 있습니다. 직관적으로 작동하나 `npx pod-install ios`를 잊지 맙시다.

이외에도 [`react-native-device-info`](https://github.com/react-native-device-info/react-native-device-info)와 같은 라이브러리들도 사용할 수 있겠지만, 단순히 버전만 확인하기에는 약간 무거울 수 있다는 점을 생각하면 좋을 듯 합니다.

## 최신 버전 가져오기

[`react-native-version-check`](https://www.npmjs.com/package/react-native-version-check)와 같은 라이브러리를 사용하면, 현재 배포되어 있는 앱의 최신 버전을 알 수 있고, 그에 기반한 플로우를 설계하는데에 최적화된 라이브러리 입니다.

다만 저의 케이스에서는, 각 버전별로 강제로 업데이트를 해야하는지, 선택적으로 업데이트를 할 수 있게 해준다는지 하는 유즈케이스가 있어서 회사 API 서버에서 버전을 받아와 처리하는 방식으로 구현하였습니다.

# 모달 띄우기

이제 모달을 띄워야 할지에 대한 조건을 알 수 있었습니다. 그러면 어떻게 모달을 띄워야 할까요?

## 모달 구현체 선택

기본적으로 React native에서도 `Modal`을 제공하긴 합니다. 하지만, **backdrop 클릭에 대한 제어**등 모달이라면 기대할 수 있는 기능들이 빠져 있어 아쉬웠기에, [`react-native-modal`](https://github.com/react-native-modal/react-native-modal)이라는 커뮤니티 라이브러리를 사용하였습니다.

## 모달 스타일링

이것도 상당히 직관적이지 않은 부분입니다만, 모달의 크기를 `<Modal>`의 스타일을 적용하는 것으로 제어할 수 없고, 가장 가까운 `View` 자식의 스타일로 제어해야 한다는 점입니다. 저 말고 다른 개발자들도 혼동이 왔는지 [관련한 github issue](https://github.com/react-native-modal/react-native-modal/issues/358)도 찾아볼 수 있었습니다.

# 마치며

오랜만에 글을 써 보았습니다. React native와 React 웹을 개발하면서, 겪은 여러 재미있는 기술 주제들이 많지만, 우선 가장 작은 것부터 시작해서 다시 짬짬이 이야기를 풀어보려 합니다. 끝까지 읽어주셔서 감사합니다.
