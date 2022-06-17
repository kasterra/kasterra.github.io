---
layout: post
title: React-beautiful-dnd 알아보기 ④
subtitle: 가로 방향 dnd + column 재정렬 + 최적화
category: react
image: /images/thumbnails/rbd.png
---

# 들어가며

[저번 글](../react-beautiful-dnd-3)에서는 rbd에서 Drag를 제어하는 방법에 대해서 알아 보았습니다. 아마 이번 글이 튜토리얼로서의 마지막 글이 될것 같습니다. 우리가 뷰를 만들면서, 꼭 수직방향의 dnd만 만든다는 보장은 없습니다. 수평방향의 dnd도 충분히 만들 수 있지요. 그리고, 지금까지는 `task`만 Drag 했었는데, trello 같은 서비스를 보면, `column`도 Drag의 대상이 됨을 확인할 수 있습니다. rbd의 `README.md`를 봐도 해당 예제가 표현되어 있습니다.

![rbd example](https://user-images.githubusercontent.com/2182637/53614150-efbed780-3c2c-11e9-9204-a5d2e746faca.gif)

<div style="display:flex; justify-content:center;">rbd로 할 수 있는 여러 예제를 포함한 공식 readme중...</div>

이제 정말 얼마 남지 않았습니다. 차례 차례 하나씩 가봅시다.

# 수평방향 dnd 만들기 ( ~🏷️ horizontalDnd)

column을 하나만 남기고, column내부의 task들을 가로로 정렬되게끔 스타일링을 해보고, 드래그를 시도해 봅시다.

![wrong horizontal](/images/frontend/wrongHorizontal.gif)

<div style="display:flex; justify-content:center;">drag and drop이 제대로 되지 않는다...</div>

예상한대로 작동하지 않는 모습입니다. 그렇다면, 수평 방향의 dnd를 만들려면, 새로운 라이브러리를 가져와야 하는 걸까요? 아닙니다! 그저 간단한 prop만 건드려 주면, 아주 쉽게, 수평 방향의 dnd를 만들 수 있습니다. `Column.tsx`에서 `Droppable`의 `direction` prop만 수정해주면, 아주 간단하게 수평 방향의 dnd를 만들 수 있습니다.

```tsx
<Droppable droppableId={column.id} direction="horizontal">
```

제대로 작동하는지 살펴봅시다.

![right horizontal](/images/frontend/rightHorizontal.gif)

제대로 작동하는군요! 아주 좋습니다.

# Column 재정렬 ( ~🏷️ columnReorder)

도입부 부분에서도 간단히 언급했지만, trello 같은 서비스에서는 column 또한 드래그를 이용해서 재정렬이 가능합니다. trello를 만든 회사에서 내놓은 라이브러리인 만큼, 한번 모방해서 만들어 봅시다!

직관적인 설명을 위해서, 원본 강의에서 내놓은 도식표를 가져오겠습니다.

![illustration for approach](/images/frontend/approachOfReorderableColumn.png)

우리가 만들 구현체의 모습이라고 생각하시면 되겠습니다. Column들을 Draggable로 감싸고, Column draggable들을 drop 할 수 있는 Droppable들로 Column draggable들을 감싸줍시다. 그리고 그 Droppable은 수평 방향으로 정렬되어야 하겠지요. 이제부터 만들 예제는 저번 예제의 지식까지 필요로 하기 때문에, 만약에 잘 이해가 되지 않는 부분이 있다면, 언제든지 이전 글로 돌아가서 확인해 주세요!

사실 말은 거창하게 했지만, 우리가 이미 저번 글들에서 rbd에 관해서 알아본 지식만을 사용해서 만드는 것이기 때문에, 설명할것은 그렇게 많지 않습니다.

> “Talk is cheap. Show me the code.” -Linus Torvalds-

라는 말이 있듯, 바로 코드를 보여드리겠습니다. 걱정하지 않으셔도 됩니다. 주석도 잘 달아놓을 테니까요.

```tsx
import styled from "@emotion/styled";
import React, { useCallback, useState } from "react";
import { DragDropContext, DropResult, Droppable } from "react-beautiful-dnd";
import Column from "./components/Column";
import initialData from "./initial-data";

// 기존과 동일하므로 생략

function App() {
  const [data, setData] = useState<IData>(initialData);
  const onDragEnd = useCallback(
    (result: DropResult) => {
      const { destination, source, draggableId, type } = result;
      if (!destination) return;
      if (
        destination.droppableId === source.droppableId &&
        source.index === destination.index
      )
        return;

      if (type === "column") {
        // column Drag 처리 부분 추가됨
        const newColumnOrder = Array.from(data.columnOrder);
        newColumnOrder.splice(source.index, 1);
        newColumnOrder.splice(destination.index, 0, draggableId);

        const newData = {
          ...data,
          columnOrder: newColumnOrder,
        };
        setData(newData);
        return;
      }
      // 기존과 동일하므로 생략
    },
    [data]
  );

  return (
    <div className="App">
      <header className="App-header">
        <DragDropContext onDragEnd={onDragEnd}>
          <Droppable // 새롭게 Droppable로 감쌈.
            droppableId="all-columns" // 다른 droppable과 상호작용하지 않으므로, 상관은 없지만 적어두자!
            direction="horizontal" // 수평 방향의 dnd이므로...
            type="column" // type을 명시함으로서, task들이 침범하지 못하게 함
          >
            {(provided) => (
              <Container {...provided.droppableProps} ref={provided.innerRef}>
                {data.columnOrder.map((columnId, index) => {
                  // index가 추가됨
                  const column = data.columns[columnId];
                  const tasks = column.taskIds.map(
                    (taskId) => data.tasks[taskId]
                  );
                  return (
                    <Column
                      column={column}
                      tasks={tasks}
                      key={column.id}
                      index={index} // droppable이 추가되므로, index prop을 추가로 넘기게 했음
                    />
                  );
                })}
                {provided.placeholder}
              </Container>
            )}
          </Droppable>
        </DragDropContext>
      </header>
    </div>
  );
}

export default App;
```

<div style="display:flex; justify-content:center;">App.tsx</div>

```tsx
import styled from "@emotion/styled";
import { Droppable, Draggable } from "react-beautiful-dnd";
import Task from "./Task";

// 생략

interface IColumnProps {
  column: { id: string; title: string; taskIds: string[] };
  tasks: {
    id: string;
    content: string;
  }[];
  index: number; // 추가됨
}

const Column = ({ column, tasks, index }: IColumnProps) => {
  return (
    <Draggable draggableId={column.id} index={index}>
      {" "}
      // 추가된 부분
      {(provided) => (
        <Container ref={provided.innerRef} {...provided.draggableProps}>
          <Title {...provided.dragHandleProps}>{column.title}</Title>
          <Droppable droppableId={column.id} type="task">
            {(provided, snapshot) => (
              <TaskList
                {...provided.droppableProps}
                ref={provided.innerRef}
                isDraggingOver={snapshot.isDraggingOver}
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
      )}
    </Draggable>
  );
};

export default Column;
```

<div style="display:flex; justify-content:center;">Column.tsx</div>

이제 정상적으로 작동하는지 확인해 봅시다.

![column Reorder](/images/frontend/columnReorder.gif)

잘 돌아가네요! 이제 우리는 trello와 같은 drag and drop 어플리케이션을 만들 수 있는 능력을 얻었습니다. 박수~ 👋👋👋

# `React.memo`를 이용한 최적화

튜토리얼에서 해야할 기능적인 구현 부분은 다 배웠지만, 사실 여기에서 더 개선될 여지가 남아 있습니다.

React.js를 공부를 하신 분이라면, `useCallback`과 `useMemo`를 쓰는 이유를 아마 알고 계실겁니다. React는 컴포넌트 안에 있는 요소들을 렌더링 될 때마다, 다시 연산하고, 이러한 기본적인 동작이, 비싼 연산을 여러번 일으켜서, UX를 저하하게 되는 원인이 되기도 하기 때문이라는것을 아실 겁니다. 그래서 `useCallback`이나 `useMemo`를 이용해서, 메모이제이션 처리를 함으로서, 불필요한 고비용의 연산을 줄이는 것으로 최적화를 하지요.

현재 우리의 앱에서는 Task들이 얼마 없어서 눈에 보이지 않지만, drag가 될 때, Task 컴포넌트의 상태가 계속 업데이트 되기 때문에, dnd 관련 컴포넌트에서 꽤나 많은 리렌더링들이 일어나게 됩니다. 이러한 동작은 버벅임을 야기시켜서, UX의 저해로 이어질 수 있게 되지요.

props가 변경되지 않는다면, 기존의 메모이제이션 된 컴포는트를 그대로 리턴하게 하는 `React.memo`를 사용하면 됩니다. `Task.tsx`의 `export`부분을 이렇게 바꿔줍시다.

```ts
export default React.memo(Task);
```

끝이냐고요? 네! 프론트엔드 개발을 이렇게나 편하게 해주는 React팀에게 언제나 감사합시다...

# 스크린 리더 메세지 커스터마이징

웹 개발을 할 때 고려해야 하는 부분은 여러가지가 있겠습니다만, 장애를 가진 사람들도 쉽게 접근해야 하는 웹 접근성을 이번 섹션에서는 한번 주목해 보겠습니다.

시각적인 장애를 가진 사용자는, 스크린 리더를 통해서 우리가 만든 웹 서비스에 접근할 것입니다. [원본 강의 영상](https://egghead.io/lessons/react-customize-screen-reader-messages-for-drag-and-drop-with-react-beautiful-dnd)14초부터 확인해 보시면, 별도의 처리 없이, rbd에서 기본적으로 제공해주는 스크린 리더 메세지를 확인할 수 있습니다. 이것 자체로도 훌륭하지만, 이 메세지 또한 커스터마이징이 가능합니다. 기본적으로는 영어만 제공해 주기 때문에, 영어가 모국어가 아닌 사람들을 위해서나, 아니면 별도의 메세지를 출력하게 하고 싶다면, 이걸 우리 마음대로 수정할 수 있다라는 소식은 꽤나 고무적인 소식이지요.

어떤 component가 focus 되었을 때의 메세지를 우리는 `aria-roledescription`이라는 태그를 활용해서 수정할 수 있습니다. 이 부분은 [공식 repo의 해당 부분](https://github.com/atlassian/react-beautiful-dnd/blob/master/docs/guides/screen-reader.md)을 참고하면 좋을 것 같습니다.

# 마치며

rbd 튜토리얼이라는 길고 긴 여정이 드디어 끝났습니다. 이것 자체 만으로 rbd의 모든 기능을 다 담고 있지 않지만, rbd를 사용하는데 있어서 많이 쓰일 기능들을 정리하는데는 충분하다고 생각합니다. 끝까지 따라오시느라 정말 수고 많으셨습니다. 박수👋👋👋
