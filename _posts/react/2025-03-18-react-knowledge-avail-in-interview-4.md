---
layout: post
title: 면접에서 대답할 수 있는 React 지식 ④
subtitle: react의 state와 리렌더링에 대해서 되짚어보기
category: react
image: /images/thumbnails/React.png
---

# 들어가며

지금까지의 '면접에서 대답할 수 있는 React 지식' 시리즈 글들이 딥 다이브 느낌이었다면, 이번 글은 지금까지의 글을 마무리 하는 느낌일듯 합니다. 새로운 개념을 소개하기 보다는, 지금까지 나열해놓은 개념들을 다시 재조립 하는 느낌으로 말이지요.

이번 글의 주제는 'React의 상태는 무엇이며, 어떻게 돌아가는가'로 마무리 하려고 합니다.

# React의 업데이트 관리

컴포넌트 상태 등이 바뀌었을 때, react는 updater에 해당 사항을 전달합니다. 관련 사항은 [본 글의 1편의 Rendering과 setState 섹션](/react-knowledge-avail-in-interview-1#Rendering과-setState)에서 다루었으니 확인하면 좋을 것 같습니다. 내용이 꽤나 길기 때문에, 링크로 갈음하겠습니다.

그래도 간단하게 되짚어 봅시다. setState를 호출하면, React는 데이터를 큐로 보낸다.

업데이트 요청 들은 하나 하나 처리(대기열에 넣는것)되지만, 실제 변경은 모든 결과를 종합한 한큐에 됩니다. 배치 처리를 생각하면 좋을 듯 하네요. React 업데이트는 Fiber와 매우 밀접한 연관을 맺고 있다고도 언급 하였습니다. [이전 포스트](/react-knowledge-avail-in-interview-2)에서 Fiber의 두가지 phase에 대해서 설명했음. render 페이즈와 effect 페이즈. effect 페이즈에서 DOM 처리나, 생명주기 메소드 콜이 이루어진다고 한 내용을 다루었었죠.

# 생명주기 메소드로 살펴보는 두 페이즈

'두 페이즈' 라고 하는 것에서 그치는 것이 아니고, 실제 각 페이즈에서 어떤 역할이 수행되는지, 생명주기 메소드의 실행 과정을 통해서 살펴봅시다.

## getDerivedStateFromProps

prop이 바뀔때 마다 상태를 바꾸게 해주는 메서드 입니다. state를 prop에 기반해서 요리조리 해서 내놓는 함수입니다. 'derived'라는 이름에 걸맞게, '유도된' 상태를 내놓는 함수이지요.

이 함수는 특이하게 외부에서 오는 `prop`이 변경 될 때 뿐 아니라, state가 변경될 때도 호출이 됩니다. 또 특이한 부분은 `static` 즉, 정적 함수라는 점 또한 있습니다. 그래서 메서드 밖에 있는 `this.state`나 `this.props`에 접근 불가 하기 때문에, `state`와 `prop`을 매개변수로 받게 하지요.

## shouldComponentUpdate

react는 state나 props가 바뀌면 기본적으로 리렌더링 되지만, 성능 최적화 등의 이유로 렌더링을 건너뛰게 하는 역할을 합니다.

`nextProps`, `nextState를` 인자로 받아서 `boolean` 타입을 리턴하는 함수로, `static` 메서드가 아니라서 this.state등으로 현재상태에도 접근 할 수 있습니다.

## 두 페이즈에 관해서

다시 두 페이즈에 대해서 이야기 해보도록 합시다. `getDerivedStateFromProps`와 `shouldComponentUpdate`는 첫번째 페이즈에서 호출이 된다. 즉 side effect가 발생해서는 안된다. 그리고 사실 생각해보면 저 함수에서 부작용이 발생하는것도 어찌보면 꽤나 이상하기도 하고요.

`componentDidUpdate`는 두번째 커밋 페이즈에서 실행되고, side effect를 발생시키기에 적합. DOM 변경, 구독작업, 네트워크 요청 등이 이루어 집니다. 비동기적으로 배치 처리되는 페이즈와, 동기적으로 처리되는 페이즈의 차이가 실제로 렌더링에서 어떻게 이루어지는지 생각해 볼 수 있는 좋은 react를 사용하면서 확인할 수 있는 현상 중 하나라고 생각합니다.

## 고수준에서 보는 업데이트 과정

1. setState 호출
2. 데이터가 큐에 담김
3. 리액트가 업데이트 처리 시작
4. 새 상태가 계산되고, getDerivedStateFromProps 호출됨
5. shouldComponentUpdate가 최종 상태로 호출됨
6. true라면 render 호출됨
7. DOM change
8. componentDidUpdate 호출

# 마치며

장대하게 끌어온 이 주제도 마무리가 되었습니다. react core 부분을 다루는데 초점을 두었지만, 사실 react에 관해 다룰 수 있는 부분은 정말로 많이 남아 있습니다. 이번 시리즈에서는 'UI를 렌더링 하고, 상태에 맞는 UI로 갱신하는' react 라는 타이틀에 조금 더 가중치를 두어 봤습니다.

react에 관한 글들을 앞으로 계속 작성한다면, 최근 들어서 새롭게 정식 기능으로 편입 된, concurrent에 관한 글이라던가, react로 개발을 하면서 반 필수적으로 함께 사용되는 라이브러리의 핵심 원리를 파악하는 글들 등등이 나오지 않을까 싶습니다.

끝까지 읽어주셔서 감사합니다!

## 참고 자료

[How Does React State Actually Work? React.js Deep Dive #4](https://www.youtube.com/watch?v=2cSijEC_m7g&list=PLxRVWC-K96b0ktvhd16l3xA6gncuGP7gJ&index=4)
