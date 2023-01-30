---
layout: post
title: React-beautiful-dnd 알아보기 ③
subtitle: Drag and drop 제어 + Column간 item 이동 만들기
category: react
image: /images/thumbnails/rbd.png
---

# 들어가며

[저번 글](../react-beautiful-dnd-2)에서는 실제로 작동하는 UI와, 드래그 이벤트가 발생했을 떄, 스타일을 수정하는 그런 작업을 했습니다. 이번 글에서는 예고했던, Drag and drop의 제어를 만들어 보고, 지금까지 우리가 만들었던 하나의 Column에서 벗어나서, 여러개의 Column에서의 Drag and drop의 처리를 구현해보도록 하겠습니다. 

# Drag handle 따로 지정하기 ( ~🏷️ specificDrag)

현재 우리의 컴포넌트는 `Task` 어디에서 클릭을 해서도 Drag 이벤트를 발동시킬 수 있습니다. 특정 부분만 Drag에 반응하도록 간단히 만들어 봅시다. 

```tsx
const Handle = styled.div`
  width: 20px;
  height: 20px;
  background-color: orange;
  border-radius: 4px;
`;
```

`Handle` 컴포넌트를 만들어 주고, `Container` 컴포넌트에 `display:flex`와 `gap:8px`를 줘서, 아래와 같은 컴포넌트가 만들어질 수 있도록 해봅시다. 

![handle added](/images/frontend/handleAdded.png)

당연한 이야기지만, 지금은 껍데기만 만들었지, 이 껍데기에 기능을 불어넣어주지는 않았습니다. 저번에 간단하게 이야기하고 넘어갔는데, 실제로도 간단하니까, 간단하게 적용하고 넘어갑시다. `Handle` 컴포넌트에 `{...provided.dragHandleProps}`를 적용합니다.

```tsx
const Task = ({ task, index }: ITaskProps) => {
  return (
    <Draggable draggableId={task.id} index={index}>
      {(provided, snapshot) => (
        <Container
          {...provided.draggableProps}
          ref={provided.innerRef}
          isDragging={snapshot.isDragging}
        >
          <Handle {...provided.dragHandleProps} /> // Handle에 dragHandleProps 적용됨
          {task.content}
        </Container>
      )}
    </Draggable>
  );
};

export default Task;

```

![실제로 동작하는 모습](/images/frontend/handleWorking.gif)

이제 이 모습을 보면, 실제로 오랜지색으로 색칠된 부분을 드래그 하는 부분이 아니면, 드래그가 되지 않는 모습을 볼 수 있습니다. 

Todo 앱의 사용성을 생각해 봤을 때에, 특정 부분만 드래그 되게 설정하는것 보다, 영역 전체가 드래그 가능하도록 만드는것이 UX 측면에서 더 좋을듯 하므로, 해당 부분은 따로 브랜치를 만들어서 커밋해두겠습니다.

# 여러 Column간 item 이동 ( ~🏷️ multipleColumn)

지금까지는 우리는 `Column` 한개에 갇혀 있었습니다. 이제 `Column` 여러개를 만들어서, `Column`간 이동을 구현해봅시다. 우선, 우리가 데이터를 불러오는 `inital-data.ts` 파일에서 데이터를 더 만들어 봅시다.

```ts
const initialData = {
  // 동일하므로 생략
  columns: {
    "column-1": {
      id: "column-1",
      title: "To do",
      taskIds: ["task-1", "task-2", "task-3", "task-4"],
    },
    "column-2": { // 추가됨
      id: "column-2",
      title: "In progress",
      taskIds: [],
    },
    "column-3": { // 추가됨
      id: "column-3",
      title: "Done",
      taskIds: [],
    },
  },
  // Facilitate reordering of the columns
  columnOrder: ["column-1", "column-2", "column-3"], // 추가됨
};
```

그리고, 각 Column들이 적당히 보기좋게 보이도록 스타일링도 몇가지 해줍시다 ([커밋해 놓은 코드](https://github.com/kasterra/dnd-study/commit/a157ca7bcfd7f9c9244ed962b94c46973ec47bb4)를 참고하세요)

그리고, 스타일링이 완료된 이 리스트에서 드래그 앤 드롭을 시도하니...

![여러 column drag 실패](/images/frontend/multipleColumnDragFail.gif)
<div style="display:flex; justify-content:center;">방금 만든 화면에서 드래그를 시도하는 중</div>

안됩니다! 우리의 `onDragEnd` 함수에 있는 reorder 처리 로직에서 해당 부분에 대한 고려가 되어 있지 않기 때문입니다! 정확히 집자면 `onDragEnd`에서 `column` 변수를 정의하는 부분입니다.

```ts
  const onDragEnd = useCallback(
    (result: DropResult) => {
      const { destination, source, draggableId } = result;
      if (!destination) return;
      if (
        destination.droppableId === source.droppableId &&
        source.index === destination.index
      )
        return;

      const column = data.columns[source.droppableId]; // 여기 !!!!!!
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
    },
    [data]
  );
```

우리가 처음 작성할 때는, `column`이 하나 뿐이기에, 시작하는 `column`과 끝나는 `column`이 같을 수 밖에 없었습니다. 하지만, 이제 상황이 달라졌으니, 이렇게 기존 로직을 수정해줍시다.

```ts
  const onDragEnd = useCallback(
    (result: DropResult) => {
      // 동일하므로 생략

      const startColumn = data.columns[source.droppableId]; // 출발 column
      const finishColumn = data.columns[destination.droppableId]; // 종착지 column

      if (startColumn === finishColumn) {
        // 기존 로직
      }
      else{
        // 새로운 로직
      }
    },[data]
  )

```

기존 로직이라고 표시해 둔 부분은 복사-붙여넣기만 하면 되고, 새로운 로직이라고 표시한 부분도, 그렇게 완전히 새로운것을 요구하지 않습니다. 기존의 로직은 데이터를 넣고 빼는것을 하나의 column에서만 했다면, 이제는 서로 다른 column에서 이것이 진행되므로, 이것만 반영해주면 됩니다. 아래의 코드를 보시죠.

```ts
    const startTaskIds = Array.from(startColumn.taskIds);
    startTaskIds.splice(source.index, 1);
    const newStartColumn = {
      ...startColumn,
      taskIds: startTaskIds,
    };

    const finishTaskIds = Array.from(finishColumn.taskIds);
    finishTaskIds.splice(destination.index, 0, draggableId);
    const newFinishColumn = {
      ...finishColumn,
      taskIds: finishTaskIds,
    };

    const newData = {
      ...data,
      columns: {
        ...data.columns,
        [newStartColumn.id]: newStartColumn,
        [newFinishColumn.id]: newFinishColumn,
      },
    };
    

    setData(newData);
```

이제 확인을 해보면 정상적으로 적용이 됨을 확인할 수 있습니다.
![](/images/frontend/multiColumnDrag.gif)

서로 다른 column일떄와, 같은 column일때의 로직을 공통으로 적용할 수는 없을까 하는 생각이 들 수도 있습니다. 저도 처음 해당 부분 학습을 할떄, 그런 생각을 해봤지만, 그렇게 되면, `drag된 엘리먼트가 그대로 있는 startColumn` + `drag된 엘리먼트가 위치가 바뀐 finishColumn` 이 되어서, drag된 엘리먼트가 말 그대로 복사가 되는 상황이 생깁니다.

![로직을 공통으로 했을때](/images/frontend/dndDragCopyWow.gif)
<div style="display:flex;justify-content:center;">엘리먼트가 복사가 된다고!!</div>

# Drag and Drop 제어하기

## `isDragDisabled`를 이용해서 드래그 제한하기 ( ~🏷️ isDragDisabled)

현재 우리가 만든 이 앱은 아무런 제약사항 없이 `Task`들이 Drag 될 수 있게 허락해 줍니다. 하지만, 우리가 실제로 만들 앱에는 제약사항이 필요할 때도 있지요. 권한이 부족하다던가, 아직 할 일의 기한이 끝나지 않았다던가 하는 그런 이유도 있을 수 있으니까요. 일단 이 글에서는 그런 이유도 중요하지만, **어떻게** 구현하는지에 대해서 한번 초점을 맞추어 봅시다. 

우선 드래그가 되지 않는 컴포넌트를 어떻게 만드는가에 대해서 알아봅시다. `Draggable`이 가지고 있는 prop 중에는 `isDragDisabled` 라는 prop이 있습니다. 말 그대로 true로 설정되어 있으면, 드래그가 되지 않습니다. 간단하게 `task-1`이라는 `task`를 한번 비활성화 시키고, 이걸 styled된 컴포넌트에 전달시켜서 시각적으로 보여줍시다.

```tsx
// 생략

const Container = styled.div<IContainer>`
  border: 1px solid lightgrey;
  border-radius: 2px;
  padding: 8px;
  margin-bottom: 8px;
  transition: background-color 0.2s ease;
  background-color: ${(props) =>
    props.isDragDisabled
      ? "lightgrey"
      : props.isDragging
      ? "lightgreen"
      : "white"};
`;

// 생략

const Task = ({ task, index }: ITaskProps) => {
  const isDragDisabled = task.id === "task-1";
  return (
    <Draggable
      draggableId={task.id}
      index={index}
      isDragDisabled={isDragDisabled}
    >
      {(provided, snapshot) => (
        <Container
          {...provided.draggableProps}
          {...provided.dragHandleProps}
          ref={provided.innerRef}
          isDragging={snapshot.isDragging}
          isDragDisabled={isDragDisabled}
        >
          {task.content}
        </Container>
      )}
    </Draggable>
  );
};

export default Task;
```

![disabled drag](/images/frontend/disabledDrag.gif)

우리가 비활성화 시킨 첫번째 task는 드래그 되지 않고, 다른 task는 drag가 정상적으로 됨을 확인할 수 있습니다. 그런데 이걸 보면, 비활성화 시킨 task가 drag 되지는 않지만, 다른 task를 그 위치로 drag 함으로서, drag를 시행하는 효과를 낼 수도 있습니다. 여러분의 비즈니스 로직에 따라서, 적절하게 `isDragDisabled`를 설정해 주면 되겠습니다.

무조건적으로 drag를 막는 방법은 이제 알았습니다만, 특정 상황에서만 조건적으로 drag를 막기 위해서는 어떻게 하면 될까요?

## `TYPE`을 이용해서 drag를 더 자세히 제어하기 ( ~🏷️ controlWithType)

이 문제는 [첫번째 글](../react-beautiful-dnd-1)에서 나중에 다루겠다 하고 넘어간 `TYPE`을 사용하여 해결할 수 있습니다! `TYPE`은 `Droppable`과 `Draggable` 에 prop으로 부여할 수 있습니다. `Droppable`에서 기본적으로 `type`을 처리할 때, `Draggable`이 출발한 `Droppable`의 `type`과 도착한 `Droppable`의 `type`이 같지 않다면, Drag and drop을 허용하지 않습니다.

`Column.tsx`에 있는 `Droppable`을 아래와 같이 수정해 봅시다.

```tsx
<Droppable
  droppableId={column.id}
  type={column.id === "column-3" ? "done" : "active"} // 여기가 바뀜!!!
>
// 생략...
```

그러면 이제 `column-1`와 `column-2`는 `active`라는 `type`을 가지게 되고, `column-3`는 혼자 다른 `type`인 `done`을 가지게 됩니다. 실제로 rbd가 Drag and drop을 관리하는지 직접 확인해 봅시다.

![droppable의 type이 다를 때](/images/frontend/differentTypeDroppable.gif)
<div style="display:flex; justify-content:center;">droppable의 type이 다르니 drag and drop이 되지 않는다.</div>

실제로 드래그가 되지 않도록 rbd에서 관리해주어, 시각적으로도 이를 확인할 수 있습니다! 정말 갓-라이브러리가 아닐 수 없습니다.

하지만, `type`을 이용한 방법에도 한계점이 있습니다. type이 다른 column간 이동은 무조건적으로 제한한다는 것입니다. 무언가 동적으로 drag를 제어할 방법이 있다면 좋을텐데...

## isDropDisabled를 이용한 제어 ( ~🏷️ isDropDisabled)

trello를 만든 회사가 내놓은 라이브러리 답게 이것도 만들어 놨습니다! 바로 `Droppable`의 prop인 `isDropDisabled`를 이용하는 방법입니다. `isDropDisabled`를 이용하면, `type`이 같더라도, `isDropDisabled`가 `true`로 설정된 상황에서는 drag and drop을 비활성화 해줍니다. 우리는 간단한 예제로, task들을 현재 `column` 혹은 오른쪽에 있는 `column`으로만 옮길 수 있는 제약조건을 한번 만들어 보겠습니다.

이전 챕터에 만들어 놓은 코드는 지우겠습니다. (레포지토리 상에서는 별개의 브랜치를 파는것으로 표현해 놓았습니다!) 

`Column.tsx`에서 `isDropDisabled`라는 `boolean`형의 prop을 넘겨받게 해서, 이를 해당 컴포넌트 안에 있는 `Droppable`에 반영해 보겠습니다. 

```ts
interface IColumnProps {
  column: { id: string; title: string; taskIds: string[] };
  tasks: {
    id: string;
    content: string;
  }[];
  isDropDisabled: boolean; // 추가됨!
}
```

```tsx
<Droppable droppableId={column.id} isDropDisabled={isDropDisabled}> // isDropDiabled 추가됨!
```

그리고 index에 의해서 드래그의 가능여부가 변해야 하니, 저번 포스팅에서 배운 `onDragStart`를 이용해서 코드를 작성해 봅시다. 지면 공간상 코드에 대한 모든 설명은 하지 않겠습니다. 혹시 이해가 되지 않는 부분이 있다면 댓글 주시면, 알림 확인하는 대로 자세히 설명 드릴테니 걱정하지 말아주세요! 

```tsx
function App() {
  const [homeIndex, setHomeIndex] = useState<number | null>(null);

  const onDragStart = useCallback((start: DragStart) => {
    const newHomeIndex = data.columnOrder.indexOf(start.source.droppableId);
    setHomeIndex(newHomeIndex);
  }, []);

  const onDragEnd = useCallback(
    (result: DropResult) => {
      setHomeIndex(null);
      // 생략...
    }
    ,[]);

    return(
    <div className="App">
      <header className="App-header">
        <DragDropContext onDragStart={onDragStart} onDragEnd={onDragEnd}>
          <Container>
            {data.columnOrder.map((columnId, index) => { // map의 index를 이용!
              const column = data.columns[columnId];
              const tasks = column.taskIds.map((taskId) => data.tasks[taskId]);

              const isDropDisabled = homeIndex && index < homeIndex; //설정해주고

              return (
                <Column
                  column={column}
                  tasks={tasks}
                  key={column.id}
                  isDropDisabled={!!isDropDisabled}// prop으로 넘긴다.
                />
              );
            })}
          </Container>
        </DragDropContext>
      </header>
    </div>
  )
}

```

해당 코드가 실제로 동작하는지 확인해 봅시다.

![isDragDisabled 이용](/images/frontend/usingIsDragDisabled.gif)

우리가 원했던 대로, 오른쪽으로 움직이는것은 가능하지만, 왼쪽 방향으로 움직이는것은 되지 않음을 볼 수 있습니다. 

# 마치며

이번 포스팅을 통해서, drag and drop을 만들면서 drag를 제어해야 할 때, rbd를 활용해서 해당 기능을 어떻게 편하게 구현할 수 있는지 알아보았습니다. 내용이 좀 길었는데 끝까지 읽어주신데에 대해서 정말 깊은 감사를 표합니다.

지금까지는 수직 방향 drag and drop만 만들어 보았다면, 다음 글에서는 수평 방향 drag and drop을 만들어 보고, trello 처럼, column또한 drag and drop 가능하게 만들어 보는 시간을 가져볼까 합니다. 끝까지 읽어주셔서 정말 감사하고, 다음 글에서도 만나뵈었으면 좋겠습니다. 안녕~ 👋👋