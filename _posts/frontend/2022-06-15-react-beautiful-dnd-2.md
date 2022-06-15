---
layout: post
title: React-beautiful-dnd 알아보기 ②
subtitle: Drag and drop 실제로 돌아가게 해봅시다. + 스타일링 까지
category: react
image: /images/thumbnails/rbd.png
---

# 들어가며

[저번 글](../react-beautiful-dnd-1)에서는 rbd에서 쓰이는 컴포넌트들과 사용법을 알아봤고, 그 사용법을 바탕으로, Drag and Drop의 UI를 작성하기까지 했습니다. 애니메이션까지 적용이 잘 되었지만, 사용자의 상호작용이 실제 데이터에 반영이 되지 않았음을 확인할 수 있었습니다.

이번 시간에는 실제적으로 기능하는 UI를 만드는 법에 대해서 알아봅시다.

# `onDragEnd` 작성하기 (🏷️intro ~ 🏷️resultPersistence)

`onDragEnd`라는 함수는 기본적으로 이벤트 리스너의 성격을 가지고 있습니다. 즉, 이 함수의 인수로는 Drag and drop 상호작용의 결과가 담긴 객체가 주어지게 됩니다. 해당 객체의 생김새는 아래와 같습니다.

```js
{
  draggableId: 'task-1', // 유저가 드래그 하고 있던 객체의 id
  type: 'TYPE', // 향후에 다룰 예정
  reason: 'DROP', // 'DROP' 또는 'CANCEL'인데, 일반적으로는 중요X
  source: { // draggable이 시작한 위치
    droppableId: 'column-1',
    index:0,
  },
  destination: { // draggable이 끝난 위치
  // 리스트 바깥에 drop되는 등 null이 될수도
    droppableId: 'column-1',
    index:1,
  },
}
```

이제 우리의 `onDragEnd` 함수를 작성해 봅시다. 일단 객체에서, `destination`, `source`, `draggableId`를 뽑아오고, 함수 작성을 시작해봅시다.

```ts
  const onDragEnd = useCallback((result: DropResult) => {
    const { destination, source, draggableId } = result;
    // 리스트 밖으로 drop되면 destination이 null
    if (!destination) return; 
    // 출발지와 도착지가 같으면 할 게 없다
    if (
      destination.droppableId === source.droppableId &&
      source.index === destination.index
    )
      return;
  }, []);
```

이제 할 일 목록들을 실제로 재정렬 하는 코드를 작성해야 겠네요. 먼저 출발지의 column을 얻어오고,

```ts
const column = data.columns[source.droppableId];
```

그리고 기존 state를 mutating 하는것을 지양하기 위해서, 새로운 배열을 만들어 줍시다.

```ts
const newTaskIds = Array.from(column.taskIds);
```

이제 `source.index`에 위치했던 원래의 원소를 제거하고 `destination.index`에 원소를 끌어놓아서 재배열 합시다.

```ts
newTaskIds.splice(source.index, 1);
newTaskIds.splice(destination.index, 0, draggableId);
```

`taskIds`가 변경되었으니, 해당 사항이 `column`에도 반영이 되야 하겠지요? 그리고 그 변경된 `column`도 우리의 `data` state에 반영이 되야할 것입니다. spread 문법을 사용해서 아래와 같이 코드를 작성해 봅시다.

```ts
const newColumn = {
  ...column,
  taskIds: newTaskIds,
};

const newData = {
  ...data,
  columns: {
    ...data.columns,S
    [newColumn.id]: newColumn,
  },
};

setData(newData);
```

즉 최종적으로 `onDragEnd` 함수는 아래와 같은 형태가 될 것입니다.

```ts
  const onDragEnd = useCallback((result: DropResult) => {
    const { destination, source, draggableId } = result;
    if (!destination) return;
    if (
      destination.droppableId === source.droppableId &&
      source.index === destination.index
    )
      return;

    const column = data.columns[source.droppableId];
    const newTaskIds = Array.from(column.taskIds);
    newTaskIds.splice(source.index, 1);
    newTaskIds.splice(destination.index, 0, draggableId);

    const newColumn = {
      ...column,
      taskIds: newTaskIds,
    };

    const newData = {
      ...data,
      columns: {
        ...data.columns,
        [newColumn.id]: newColumn,
      },
    };

    setData(newData);
  }, [data]);
```

이제, 우리의 Drag and drop의 결과가 보존되는것을 확인할 수 있습니다. 그리고 `tab`, `space`, 방향키를 이용해서 드래그 앤 드롭도 수행할 수 있음을 확인할 수 있습니다. 이제 실제 서비스에 적용하려면, 우리의 변화를 해당 데이터들을 담고있는 서버 등에 보내주는 로직도 추가해야 겠네요. 이 부분은 `react-query`의 `useMutation` 등을 이용해서 처리할 수 있겠습니다. (현재 코드에서는 단순히 `useState`로 상태 관리는 하지만, 실무에서는 해당 state가 서버와 연동되어 있겠죠?)

# Drag에서 시각적인 효과 만들기(🏷️resultPersistence ~ )

우리가 만든 Drag and drop은 제 기능을 다합니다만, 몇가지 시각적 효과를 더하면 좋을것 같습니다. 이를테면, 드래그 중인 Element의 `background-color`를 다르게 해서, 현재 드래그 되고 있다라는 것을 시각적으로 보여준다면, UX에 도움을 줄 수도 있겠죠. `onDragEnd`에서 직접적인 mutation을 피하기 위해서, 새로운 객체를 반환하게끔 함수를 작성했는데, 사실 드래그 중에 `draggable`이나 `droppable`의 축의 내용(원문 : dimension)을 바꾸지 않는 한, 원본 객체를 mutate 해도 크게 상관 없습니다. React에서 상태 관리를 얕은 비교를 하기 때문에 이런 함수형 스러운 변경을 한다라는것을 아마 독자분들께서는 알고 계시리라 믿습니다.

## snapshot 인수를 이용한 스타일링 ( ~🏷️ styleUsingSnapshot)

여하튼 시각적 효과 반영을 위해서, 우리가 이때까지 사용하지 않았던, `<Draggable>`의 `children`의 두번째 인수 `snapshot`을 활용해 봅시다. 해당 객체에를 활용해 Drag 도중에 draggable 컴포넌트의 스타일을 바꿀 수 있습니다.

`DropResult`타입처럼, `snapshot` 객체에는 어떤 데이터가 들어가 있는지 살펴봅시다. 

```js
// Draggable
const draggableSnapShot = {
  isDragging: true, // 현재 drag 되고 있는가?
  draggingOver: 'column-1', // 현재 drag 되고 있는 droppable의 Id
  // 만약 droppable의 위에 있지 않다면 null이 됨
};

// Droppable
const droppableSnapShot = {
  isDraggingOver: true, // 현재 이 Droppable위에서 drag가 일어나는가?
  draggingOverWith: 'task-1' // 현재 이 Droppable위에서 drag되는 draggable의 ID
  // 없다면 null
}
```

이제 이것을 이용해서 실제 스타일링을 해봅시다. 

```tsx
import styled from "@emotion/styled";
import { Draggable } from "react-beautiful-dnd";

interface IContainer {// 추가됨!
  isDragging: boolean;
}

const Container = styled.div<IContainer>`
  border: 1px solid lightgrey;
  border-radius: 2px;
  padding: 8px;
  margin-bottom: 8px;
  background-color: ${(props) => (props.isDragging ? "lightgreen" : "white")};
`;

// 생략

const Task = ({ task, index }: ITaskProps) => {
  return (
    <Draggable draggableId={task.id} index={index}>
      {(provided, snapshot) => (
        <Container
          {...provided.draggableProps}
          {...provided.dragHandleProps}
          ref={provided.innerRef}
          isDragging={snapshot.isDragging} //추가됨!
        >
          {task.content}
        </Container>
      )}
    </Draggable>
  );
};

export default Task;
```

<div style="display:flex; justify-content:center;">Task.tsx의 모습</div>

`@emotion/styled`에 prop을 넘겨서 현재 drag가 진행되고 있으면, `background-color`를 수정하게끔 간단한 코드를 작성하였습니다. 

`Column.tsx`에서도 비슷하게 스타일링을 해줍시다.

```tsx
import styled from "@emotion/styled";
import { Droppable } from "react-beautiful-dnd";
import Task from "./Task";

// 생략

interface ITaskList {
  isDraggingOver: boolean;
}
const TaskList = styled.div<ITaskList>`
  padding: 8px;
  background-color: ${(props) => (props.isDraggingOver ? "skyblue" : "white")};
`;

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
        {(provided, snapshot) => (
          <TaskList
            {...provided.droppableProps}
            ref={provided.innerRef}
            isDraggingOver={snapshot.isDraggingOver} // 추가됨!
          >
            <>
              {tasks.map((task, idx) => (
                <Task key={task.id} task={task} index={idx} />
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
<div style="display:flex; justify-content:center;">Column.tsx의 모습</div>

`Droppable`위에서 drag가 발생하면 `background-color`가 바뀌는 간단한 스타일링을 해보았습니다. 실제로 잘 적용되었는지 확인해 보죠.

![](/images/frontend/dnd-visualize.gif)

잘 적용되었네요. 매우 좋습니다! 그리고 자세히 보시면 알겠지만, `Task`를 `Column` 밖에 `drag` 했을 때, `Column`의 `background-color`가 다시 `white`로 되는것도 확인할 수 있습니다.

## `DragDropContext`의 콜백을 이용한 스타일링( ~🏷️ callbackGlobalStyle)

`DragDropContext`에 3가지 콜백 prop이 있다고 [첫번째 글](../react-beautiful-dnd-1)에서 언급을 했습니다. 지금까지는 `onDragEnd`만 사용해봤지만, 이제 다른 콜백 prop에서 쓰이는 정보들을 한번 알아봅시다.

콜백 prop들의 인수로 들어오는 데이터들의 형식은 아래와 같습니다.

```js
// onDragStart
const start = {
  draggableId: 'task-1',
  type:'TYPE',
  soucre: {
    droppableId: 'column-1',
    index: 0,
  }.
};

// onDragUpdate
const update = {
  ...start,
  destination: {
    droppableId: 'column-1',
    index: 1,
  },
};

// onDragEnd
const result = {
  ...update,
  reason: 'DROP',
};
```

[위 섹션](#ondragend-작성하기-🏷️intro--🏷️resultpersistence)에서 `onDragEnd`에 들어가는 인수를 설명했으므로, 따로 설명은 필요없을 것 같습니다. `DragDropContext`에 있는 콜백 prop을 통해서 아까 `snapshot`을 이용해서 했던 특정한 컴포넌트에 대한 스타일링도 가능하지만, 콜백을 이용한 스타일링은 **전역** 스타일링이 가능하다는 점이 상당한 매력입니다.

```ts
  const onDragStart = useCallback((start: DragStart) => {
    document.body.style.color = "orange";
  }, []);

  const onDragUpdate = useCallback((update: DragUpdate) => {
    const { destination } = update;
    const opacity = destination
      ? destination.index / Object.keys(data.tasks).length
      : 0;
    document.body.style.backgroundColor = `rgba(153,141,217,${opacity})`;
  }, []);

  const onDragEnd = useCallback(
    (result: DropResult) => {
      document.body.style.color = "inherit"; // 초기값 재설정
      document.body.style.backgroundColor = "inherit"; // 초기값 재설정
      // 생략..
    }
    ,[]
  )
```

이렇게 설정해두고 `DragDropContext`의 prop에 적용을 하면 아래와 같은 결과를 확인할 수 있습니다.

![global styling](/images/frontend/dnd-global-style.gif)

원본이 되는 영상 강의의 강사는, 전역 스타일링이 따로 필요하지 않는 한, `snapshot`을 이용한 스타일링만 사용한다고 하네요. 각자 자기 컨벤션에 맞춰서 쓰면 될것 같습니다.

# 다음에 할 것

이번 포스팅에서는 실제로 데이터를 리스트에 반영하는 방법과, Drag 상황에서 사용자에게 정보를 알려줄 수 있는 스타일링에 관해서 알아봤습니다. 그렇게 어려운 내용은 없었으니 아마 이해하시는데에 어려움은 없었을 듯 합니다.(혹시 어려우셨다면 댓글 남겨주시면 더 설명드릴게요 ㅎㅎ)

이것 만으로 실생활에서 쓰이는 Drag and drop을 모두 구현했다고 할 수 있을까요? 실생활에서 쓰이는 상황을 생각해보면, 특정 항목은 드래그가 되지 않아야 하는 등의 **제어**가 필요한 경우 또한 있습니다. 이것도 rbd에서 제공해 주냐고요? **네!** 세상 좋은 라이브러리가 아닐 수 없습니다. 글의 길이가 대책없이 길어지면 곤란하니, 다음 글에서 뵙도록 하겠습니다. 끝까지 읽어주셔서 감사하고, 다음 글에서 봅시다. 안녕~👋👋