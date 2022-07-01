---
layout: post
title: React-beautiful-dnd 알아보기 ①
subtitle: React에서 손쉽게 drag and drop을 만드는 방법!
category: react
image: /images/thumbnails/rbd.png
---

# 들어가며

아무런 라이브러리나 프레임워크 없이 순수 100% Vanila JS로 웹 개발을 해보신 분이라면, 우리가 상당히 자주 사용하는 UI 요소중 하나인, 드래그 앤 드롭이 상당히 많은 손이 간다는 사실을 아마 알 것입니다. 그 뿐인가요? 마우스로 드래그 앤 드롭을 만들었는데, 터치 이벤트는 또 따로 구현해야하고, 그냥 아무 애니메이션도 없이 드래그 앤 드롭 처리만 하기에는 UX가 걱정되고, 브라우저에 기본으로 걸려있는것들도 해제해야하고... 속이 타들어 갑니다. 이런 여러가지 사항들로부터 고통받는 프론트엔드 개발자에게 한줄기 빛이 있었으니...! 그것이 바로 이 글에서 소개할 react-beautiful-dnd라는 이름 그대로 아름다운 라이브러리가 되겠습니다!

![gaviskon zzal](/images/frontend/dnd-gaviskon.png)

# 라이브러리 소개

우선 간단하게 라이브러리 소개부터 해볼까 합니다. react-beautiful-dnd(이하 rbd)는 협업 툴로 유명한 [Jira](https://www.atlassian.com/software/jira)와 일정관리 툴로도 널리 알려져있는 [trello](https://www.atlassian.com/software/trello)를 만든 [atlassian](https://www.atlassian.com/)이라는 회사에서 배포하는 오픈 소스 라이브러리 입니다. 글을 쓰는 현재 시점(2022-06-13)일 기준으로 해당 레포의 Readme를 살펴보면, 치명적인 보안 문제가 아닌 이상, 유지보수 관리를 열심히는 안하겠다고 합니다만, 라이브러리의 만듦새가 정말 괜찮고, 관련한 레퍼런스 자료도 많기 때문에, 해당 라이브러리에 대해 제대로 알아보는것이 나쁘지 않다고 생각하여 이 글을 써봅니다.

# 본격적인 설명을 시작하기 전에

참 서론이 깁니다만, 최대한 여러분께 글을 압축해서 전달하기 위함이니, 양해 부탁드립니다. 프론트엔드 개발에서 뷰를 떼어놓고는 설명을 할 수는 없지만, 이 글에서는 여러분께 이 멋진 라이브러리의 사용법 이라는 것에 더욱 더 초점을 맞추기 위해서, 뷰 개발 부분을 생략하겠습니다. 이 글의 출처가 되는 [공식 튜토리얼 비디오](https://egghead.io/courses/beautiful-and-accessible-drag-and-drop-with-react-beautiful-dnd) 에서는 클래스형 컴포넌트를 사용하고, 컴포넌트 스타일링에 `styled-component`를 사용하지만, 저는 제 입맛에 맞게 환경을 구성하여서 설명을 진행하겠습니다. 환경만 다를 뿐이지, 전하고자 하는 핵심내용은 최대한 손상시키지 않기 위해서 노력하겠습니다!

본격적인 설명을 시작하기 전에, [제 깃헙 레포](https://github.com/kasterra/dnd-study)에 들어가셔서 레포를 `clone`받으신 후에,

```bash
git checkout init
```

을 콘솔창에 입력하셔서, `init`으로 태그해놓은, 커밋으로 체크아웃 하셔서, 거기에 세팅되어 있는 코드들을 확인해보시기 바랍니다. 그 코드베이스로 이제 설명이 시작되거든요.

간단하게 파일 구조를 설명하겠습니다. 우리가 이 강의에서 만들어 볼것은 프론트엔드계의 hello world와 같은 예제인 Todo List입니다. Drag and drop을 사용하여, 할일 목록 안에 있는 할일들의 순서를 바꿀수도 있고, 다른 할일 보드로 옮길 수도 있는 트렐로와 같은 그런 어플리케이션을 작성한다고 생각하면 될 것 같습니다. 할 일 보드의 역할을 `Column`이 하고, 각 할 일 컴포넌트의 역할을 `Task`가 한다고 생각하면 될 것 같습니다. 이제 정말로 설명 들어갑니다.

# 라이브러리 개요 (🏷️init ~ 🏷️intro)

앞으로 (🏷️<태그명>)이라는 제목을 보시면, 위의 언급된 레포에서 <태그명>이라는 태그로 해당 섹션에 관련된 커밋을 태그해놨다 라고 이해하시면 될 것 같습니다.

길고 긴 서론을 읽으시느라 고생이 많으셨습니다. 우리의 Drag and drop을 책임져주는 이 아름다운 라이브러리는 3개의 요소로 구성되어 있습니다.

- DragDropContext
  - Drag and drop을 사용하고자 하는 어플리케이션의 영역을 감싸는 Wrapper 입니다
- Droppable
  - Drag and drop에서 drop을 할 수 있는 영역이자, Draggable을 감싸는 Wrapper 입니다.
- Draggable
  - Drag and Drop의 주체가 되는, Drag가 가능한 컴포넌트를 감싸는 Wrapper 입니다.

이제부터 자세히 알아봅시다.

## DragDropContext

역할은 앞에서 간단히 소개한 그대로입니다. DragDropContext는 3개의 콜백 prop을 갖습니다.

- onDragStart
  - drag가 시작되었을 때 호출됨
- onDragUpdate
  - drag 진행중에 새로운 위치로 이동하는 등, **새로운 변화**가 생겼을 때
- onDragEnd
  - drag가 끝났을 때 호출됨

이 세가지 중에서 없어서는 안되는 필수적인 콜백은 `onDragEnd` 입니다. 여러분의 state를 동기적으로 업데이트 할 책임을 가지고 있는 함수가 `onDragEnd`이기 때문이지요. 쉽게 말해서 drag and drop의 결과를 반영하는 함수라고 생각하시면 되겠습니다. `App.tsx`에 있는 `header`안에 있는 내용물들을 `DragDropContext`로 감싸주고, onDragEnd에는 일단 placeholder용으로 간단하게 채워넣어 봅시다.

```tsx
import React, { useCallback, useState } from "react";
import { DragDropContext } from "react-beautiful-dnd";
import Column from "./components/Column";
import initialData from "./initial-data";
interface IData {
  tasks: {
    [key: string]: { id: string; content: string };
  };
  columns: {
    [key: string]: { id: string; title: string; taskIds: string[] };
  };
  columnOrder: string[];
}

function App() {
  const [data, setData] = useState<IData>(initialData);
  const onDragEnd = useCallback((result: any) => {
    // TODO: reorder our column
  }, []);

  return (
    <div className="App">
      <header className="App-header">
        <DragDropContext onDragEnd={onDragEnd}>
          {data.columnOrder.map((columnId) => {
            const column = data.columns[columnId];
            const tasks = column.taskIds.map((taskId) => data.tasks[taskId]);
            return (
              <Column column={column} tasks={tasks} key={column.id} />
            );
          })}
        </DragDropContext>
      </header>
    </div>
  );
}

export default App;
```

<div style="display:flex; justify-content:center;">App.tsx의 모습</div>

## Droppable

Droppable에는 필수적인 prop이 한개 있습니다. droppableId라는 prop입니다. 해당 prop은 한 DragDropContext내에서 **유일해야** 합니다. `droppableId`에는 `column`을 상징하는 유일한 값이 `column`의 id를 넣어서, `Column.tsx`의 `TaskList`를 `<Droppable>`로 감싸 줍시다.

```tsx
import styled from "@emotion/styled";
import { Droppable } from "react-beautiful-dnd";
import Task from "./Task";

// 생략...

interface IColumnProps {
  column: { id: string; title: string; taskIds: string[] };
  tasks: {
    id: string;
    content: string;
  }[];
}

const Column = ({ column, tasks }: IColumnProps) => {
  return (
    <Container>
      <Title>{column.title}</Title>
      <Droppable droppableId={column.id}>
        <TaskList>
          {tasks.map((task) => (
            <Task key={task.id} task={task} />
          ))}
        </TaskList>
      </Droppable>
    </Container>
  );
};

export default Column;

```

<div style="display:flex; justify-content:center;">Column.tsx의 모습. 그런데 에러가 생긴다..?</div>

그런데 이상합니다. vscode에서는 빨간줄이 나오고, 브라우저에서도 제대로 렌더링이 되지 않고 에러가 발생하네요...
![DroppableError](/images/frontend/droppableError.png)

에러를 읽어보니, `Element`타입은` (provided: DroppableProvided, snapshot: DroppableStateSnapshot) => ReactElement<HTMLElement, string | JSXElementConstructor<any>>'` 타입에 할당될 수 없다고 합니다. `<Droppable>`의 `children`은 일반적인 컴포넌트처럼 `Element`를 받는것이 아니라, 위와 같은 함수 형태를 받는다 라는 뜻으로 추측해볼 수 있을것 같습니다. `<Droppable>`의 `children`은 Element를 반환하는 함수를 받는다고 합니다. 왜 그런 걸까요...?

강의의 설명에 의하면, rbd에서는 DOM을 렌더링 하지 않고, 기존의 구조에 달라붙는(latch)되기 때문에 그런 패턴을 사용한다고 합니다. 이 설계철학에 맞게끔 `children`에 들어가는 함수에 대해서 알아봅시다.

위의 에러를 보면 알 수 있듯, `children`자리에 오는 함수의 첫번째 인수는 `provided` 입니다. 이 인수는 중요한 정보를 담은 객체 입니다. 이 `provided`객체에는 `droppableProps`라는 프로퍼티가 있습니다. 해당 프로퍼티는 우리가 `droppable`로 사용할 컴포넌트에 적용이 되야 하는 props를 모아 놓은 것으로, 일일히 적용할수도 있고, monkeypath 할 수도 있지만, `...`이라는 스프레드 연산자를 이용해서 간편하게 적용할 수 있습니다. 그리고 그게 깔끔하기도 하고요. 

그리고 `innerRef`라는 프로퍼티 또한 있습니다. 해당 프로퍼티는 여러분의 컴포넌트가 rbd와 상호작용하는것을 도와주는 ref 콜백의 역할을 해줍니다. 

마지막으로 알아둬야 할것은 `placeholder`라는 프로퍼티 입니다. 해당 프로퍼티는 drag and drop중에, `droppable`의 면적이 변화해야 할 일이 생기게 되면, 해당 부분을 처리해주는 React Element 입니다.

자 이제 설명을 들었으니, 해당 부분을 적용해 봅시다.

```tsx
import styled from "@emotion/styled";
import { Droppable } from "react-beautiful-dnd";
import Task from "./Task";

// 생략...

interface IColumnProps {
  column: { id: string; title: string; taskIds: string[] };
  tasks: {
    id: string;
    content: string;
  }[];
}

const Column = ({ column, tasks }: IColumnProps) => {
  return (
    <Container>
      <Title>{column.title}</Title>
      <Droppable droppableId={column.id}>
        {(provided) => (
          <TaskList {...provided.droppableProps} ref={provided.innerRef}>
            <>
              {tasks.map((task) => (
                <Task key={task.id} task={task} />
              ))}
              {provided.placeholder}
            </>
          </TaskList>
        )}
      </Droppable>
    </Container>
  );
};

export default Column;
```

<div style="display:flex; justify-content:center;">완성된 Column.tsx의 모습.</div>

## Draggable

`Draggable`에는 필수 prop이 두개가 있습니다. `draggableId`와 `index` 입니다.

`Task` 컴포넌트를 Draggable 하게 만들어 봅시다. `Container` 컴포넌트를 `Draggable`로 감싸봅시다. `draggableId`에는 `task.id`를 넣고, `index`에는... 우리가 코드에서 index를 넘기지 않았네요. `Task`컴포넌트를 호출하는 `Column` 컴포넌트에 약간의 수정을 해봅시다. 

JS를 조금 빠삭하게 공부해 보신 분이라면, `map` 함수에서 두번째 인수로 배열에서의 인덱스인 `idx`를 제공해준다는 사실을 알고 계실 것 입니다. 해당 부분을 `index`를 넘기게끔 수정해 줍시다.

```tsx
            {tasks.map((task, idx) => (
              <Task key={task.id} task={task} index={idx} />
            ))}
```

<div style="display:flex; justify-content:center;">수정된 Column.tsx의 모습. 32번째 줄부터 이다.</div>

그리고, `Task.tsx` 또한 수정해 봅시다. 

```tsx
import styled from "@emotion/styled";
import { Draggable } from "react-beautiful-dnd";

const Container = styled.div`
  border: 1px solid lightgrey;
  border-radius: 2px;
  padding: 8px;
  margin-bottom: 8px;
`;

interface ITaskProps {
  task: {
    id: string;
    content: string;
  };
  index: number;
}

const Task = ({ task, index }: ITaskProps) => {
  return (
    <Draggable taskId={task.id} index={index}>
      <Container>{task.content}</Container>
    </Draggable>
  );
};

export default Task;
```

<div style="display:flex; justify-content:center;">Task.tsx의 모습. 그런데 에러가..</div>

`Column.tsx`에서 봤던 에러와 비슷한 에러가 또 나왔습니다. 여기서도 `children`을 함수 형태로 넘겨주어야 하는군요. 이뉴는 `Column.tsx`에서 설명한 그 이유와 같습니다.

여기서 사용하는 함수의 첫번째 인수는 `provided` 입니다. 용도는 `Droppable`의 그것과 매우 유사합니다. 첫번째로 소개할 `provided`의 프로퍼티는 `draggableProps` 입니다. 우리의 드래그를 하였을 때 반응하기 원하는 컴포넌트에 spread등의 방법을 통해서 적용해야 하는 prop들을 모아둔 프로퍼티 입니다.

두번째로 소개할 프로퍼티는 `dragHandleProps` 입니다. 해당 프로퍼티는 `draggableProps`가 적용된 컴포넌트를 잡을 수 있는 핸들의 역할을 하는 컴포넌트에 적용시킨다고 보면 됩니다. 실생활에서, 드래그 앤 드롭을 할 때, 특정한 부분에 드래그 상호작용을 해야 큰 컴포넌트가 움직이는 경우를 보셨을 수도 있을것 같습니다. 그런 상황에서는 `draggableProps`와 `dragHandleProps`가 각기 다른 컴포넌트에 적용된 상황으로 보면 되겠습니다. 우리가 만들 어플리케이션 에서는 할일 목록 어디를 클릭하던지 드래그 앤 드롭에 반응하기를 희망하므로, 같은 컴포넌트에 프로퍼티를 적용하겠습니다. 

여기에서도 `innerRef` 프로퍼티가 있습니다. `Droppable`의 그것과 개념적으로 동일하기에 설명은 생략하겠습니다. 

컴포넌트에 적용을 해봅시다.

```tsx
import styled from "@emotion/styled";
import { Draggable } from "react-beautiful-dnd";

const Container = styled.div`
  border: 1px solid lightgrey;
  border-radius: 2px;
  padding: 8px;
  margin-bottom: 8px;
`;

interface ITaskProps {
  task: {
    id: string;
    content: string;
  };
  index: number;
}

const Task = ({ task, index }: ITaskProps) => {
  return (
    <Draggable draggableId={task.id} index={index}>
      {(provided) => (
        <Container
          {...provided.draggableProps}
          {...provided.dragHandleProps}
          ref={provided.innerRef}
        >
          {task.content}
        </Container>
      )}
    </Draggable>
  );
};

export default Task;
```

<div style="display:flex; justify-content:center;">완성된 Task.tsx!</div>

# ⚠️ 어 왜 안돌아가요?

이 상태로 돌려볼려고 하면, **안돌아 갑니다.** 그리고 콘솔창에는, 해당 id를 가진 컴포넌트를 찾을 수 없다면서 에러가 뜨네요. 

해당 라이브러리의 유지보수가 끊긴것과, 리액트 18로의 버전업이 겹치는 바람에(?) 문제가 생겼습니다. 기본적으로 `create-react-app`으로 리액트 프로젝트를 만들면, `index.tsx`에서 `React.StrictMode`로 렌더링을 하는데, strictmode가 적용되면 **개발 빌드**에서 (production build는 해당 없습니다) 몇몇 제약사항이 생깁니다. 그중 하나가, 콜백 ref를 사용하지 못함인데, 아까도 설명했듯, rbd의 innerRef는 콜백 방식으로 동작하기 때문에 이 문제가 생깁니다. `index.tsx`를 수정하고 다시 돌려보면 정상적으로 돌아가는것을 확인할 수 있습니다.

```tsx
import ReactDOM from "react-dom/client";
import App from "./App";

const root = ReactDOM.createRoot(
  document.getElementById("root") as HTMLElement
);
root.render(<App />);
```

<div style="display:flex; justify-content:center;">index.tsx 수정</div>

# 다음에 할 것

이 길고 긴 과정을 통해서, 우리는 `Column`안에 있는 `Task`들을 드래그 앤 드롭 할 수 있게 되었습니다. 항목을 잡고 드래그를 했을 때, 다른 항목이 밀려나는 자연스러운 애니메이션까지 챙겼구요. 그런데 이 완성된 결과물을 확인해 보면, 드롭이 된 이후에, 다시 원래대로 돌아가는 것을 볼 수 있습니다. 이는 아까 placeholder용으로 `onDragEnd`를 구현하지 않고 끝냈기 때문인데요, 다음 포스팅에서는 이제 실제로 드래그 앤 드롭을 했을 때, 변경사항이 적용되게끔 구성을 해보도록 하겠습니다.

길고 긴 글 읽으시느라 고생 많으셨습니다. 오늘 작업한 사항은 강의 레포지토리에 🏷️intro 로 남겨두도록 하겠습니다. 끝까지 읽어주시느라 정말 감사드리고, 혹시나 이해가 안되시거나 에러사항이 있다면 주저하지 마시고 댓글 남겨주세요!!! 감사합니다. 🙏