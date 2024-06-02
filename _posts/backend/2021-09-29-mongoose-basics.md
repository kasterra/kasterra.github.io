---
title: Mongoose 기초 정리
layout: post
subtitle: node.js에서 MongoDB를 쓸 때 도움을 주는 친구인 Mongoose에 대하여
image: /images/thumbnails/mongoose.png
category: backend
---

# 개요

몇달전부터 저는 제대로 된 주니어 풀 스택이 되기 위해서, 프론트엔드/백엔드 공부를 모두 하고 있습니다. 주니어 풀 스택 이라면, CRUD 서비스를 스스로의 힘으로 구현해야 한다는 말을 어디서 주워들었기 떄문에, mongoDB를 깔아서, [제대로 안되는 이슈도 극복](/mongodb-error-when-just-installed/) 해보고 여러 겅험을 해봤습니다.

이번 글에서는 실제 제 node.js 백엔드에서 mongoDB를 연결해주는 mongoose에 다루어 볼까 합니다.

# 0. mongoose란?

mongoDB와 node.js를 연결시켜주는 드라이버 입니다. 맨 처음 공부를 할 때는, node.js에서 mongoDB를 쓰려면, 얘말고는 대안이 없는줄 알았는데, mongoDB자체적으로 제공하는 node.js 드라이버가 또 있더라고요. 이 글을 쓰기 위해서 관련 정보를 더 찾아봤습니다.

mongoose는 mySQl같은 기존의 RDBMS에 익숙한 사람들에게 mongoDB를 쓰는데 좋은 드라이버라고 합니다. mongoDB는 태생이 noSQL이기 때문에, 정해진 스키마가 없어, RDBMS에서는 있을 수 없는, 한 데이터 셋에서 여러 종류의 데이터셋이 섞여있는 경우가 생깁니다. mongoose는 이런 일들을 방지하기 위해, 스키마를 정의하는 기능을 제공하고, 이를 필수로 규정해 놨습니다. 그리고 JOIN 문이 없어서, 데이터 중복 처리가 까다로운 경우가 있는데, mongoose는 `JOIN` 문에 해당하는 `populate` 기능을 제공합니다.

mongoose와 mongoDB native driver를 비교하는 글들은 구글에 검색했을 때 결코 적지는 않았지만, 어느것이 더 좋다 하는 그런 이야기를 하기보다는, mongoose의 메소드를 어떻게 활용하여, CRUD 서비스를 만드는데에 사용하는가를 이 글에서 더 중점적으로 다룰 것입니다.

우선 들어가기 전에 `npm`이건 `yarn`이건을 통해 mongoose를 설치하고 진행하기 바랍니다.

```bash
$ npm i mongoose # yarn을 사용한다면 yarn으로 설치...
```

혹시 컴퓨터에 mongoDB가 설치되어 있지 않다면, [제 글](/mongodb-error-when-just-installed/) 보시면서 설치하고 다시 글 읽어주시길 바랍니다.

# 1. DB와 연결하고 귀를 기울이기

mongoose를 통해서 mongoDB 서버에 연결하는 법은 생각보다 정말 간단합니다. mongoose 모듈을 `require`을 통해서든, `import`를 통해서든 일단 불러왔다면, `mongoose.connect()`로 mongoDB 서버와 연결 할 수 있습니다. 예를 들어서, mongoDB 서버의 주소가 "mongodb://localhost:27017/myapp" 라면,

```js
import mongoose from "mongoose"; //mongoose 불러오기
mongoose.connect("mongodb://localhost:27017/myapp"); //DB 연결!
```

로 db에 연결할 수 있습니다. 원한다면, `connect()`의 두번째 매개변수에 옵션들을 담은 object를 넘겨서, 옵션들을 더 지정할 수 있습니다. 옵션의 종류들은 공식 API 문서([링크](https://mongoosejs.com/docs/api/mongoose.html#mongoose_Mongoose-connect))를 참조 바랍니다.

그런데, 이 코드 한줄로 DB와 연결이 된다고 하지만, DB 서버가 정상적이지 않는 등의 문제가 생겨서 연결이 되지 않을 수도 있고, 로그 등을 기록하는 목적으로 언제 DB에 연결이 되었는지를 알고 싶을 상황이 생길수도 있을 것입니다. 연결은 정상적으로 되었어도, 또 뭐가 잘못되어서 db에서 에러가 날 수도 있지요.

거기에 관련된 것들도 코드를 작성해 봅시다. 서버에 연결이 되었을떄에 실행되기를 희망하는 함수를 `handleOpen()`이라고 하고, db에서 에러가 났을 때, 실행되기를 희망하는 함수를 `handleError()`라고 하겠습니다.

```js
const handleOpen = () => console.log("connected to DB!");
const handleError = (error) => console.log("DB Error", error);
const db = mongoose.connection; //mongoose로 연결한 첫번째 연결을 의미합니다. 자세한건 후술
db.on("error", handleError);
db.once("open", handleOpen);
```

코드 주석에도 적어놨지만, `mongoose.connection`은 mongoose로 연결한 첫번째 연결을 의미합니다. mongoose로 DB 여러개를 연결 할 수 있다 하더라고요. DB 여러개를 연결해서 부하를 분산시킨다거나 하는 용도로 쓸 것 같다는 생각은 들긴 하지만, 일단 "CRUD를 만들자"라는 목적을 달성하는 데에는 DB 하나만 연결하는것도 문제는 되지 않죠. 혹시나 여러 db를 연결해서 connection이 여러개라면, `mongoose.connections[]`를 통해서 접근 가능하다고 하고, 아까 `mongoose.connection`이 첫번째 연결이라고 하였으니, `mongoose.connections[0]`으로도 접근이 가능하기는 합니다. (참고 : mongoose공식 API ([링크](https://mongoosejs.com/docs/api/mongoose.html#mongoose_Mongoose-connections)))

`db` 변수에 대한 설명을 마쳤으니, 이제 거기에 달려있는 `.on`과 `.once`에 대해서 설명해보겠습니다. 프론트엔드에서 JS를 사용할 때, 웹 페이지의 여러 이벤트('click', 'submit' 등등)을 listen하다가 이벤트가 발생하면 지정한 함수를 실행해주는 메서드인 `.addEventListener()`과 비슷하다고 생각하면 됩니다. mongoose의 connection 역시 프론트엔드의 요소들처럼 이벤트들을 발생시키고, Node.js에 내장된 `EventEmitter`클래스를 상속받아서 그런 이벤트들을 사용한다고 합니다.(참조 : mongoose공식 API([링크](https://mongoosejs.com/docs/connections.html#connection-events))). `EventEmitter` 계열의 이벤트들을 listen 하고 처리하기 위해서 `.on`과 `.once` 메소드를 사용한다고 합니다. 더욱 자세히 알고 싶다면, node.js에서 이벤트를 처리하는 방법을 설명한 구름 Edu 게시글([링크](https://edu.goorm.io/learn/lecture/557/%ED%95%9C-%EB%88%88%EC%97%90-%EB%81%9D%EB%82%B4%EB%8A%94-node-js/lesson/174362/event-%EB%AA%A8%EB%93%88))를 참조해 주세요.

# 2. 스키마와 모델 만들기

DB와 연결을 했으니, 이제 DB에 넣을 데이터들을 정의할 시간입니다. [위에서도](/#0-mongoose란) 간략히 얘기했는데, mongoose에서는 RDBMS에서 처럼 스키마를 만드는것을 강제합니다. 그리고 그 스키마를 기반으로 해서 모델을 만듭니다.

스키마는 `mongoose.schema({})`를 통해서 만들 수 있습니다. 제가 영상 CRUD 서비스를 만들때 사용했던 스키마를 함께 보면서 설명을 이어가도록 하겠습니다.

```js
import mongoose from "mongoose";

const videoSchema = new mongoose.Schema({
  title: { type: String, required: true, trim: true, maxLength: 80 },
  description: { type: String, required: true, trim: true, minLength: 20 },
  createdAt: { type: Date, required: true, default: Date.now },
  hashtags: [{ type: String, trim: true }],
  meta: {
    views: { type: Number, default: 0, required: true },
    rating: { type: Number, default: 0, required: true },
  },
  owner: {
    type: mongoose.Schema.Types.ObjectId, //_id에 대한 이해가 필요함
    required: true,
    ref: "User", //.populate 설명에서 더욱 자세히 다룰 것
  },
});
```

<div style="display:flex; justify-content:center;"><strong>`models/Video.js` 중 일부</strong></div>

위의 코드를 살펴보면 대략적으로 파악 할 수 있겠지만, `mongoose.Schema`의 매개변수로, 데이터의 요구사항을 담은 Object를 전달해 줍니다. `title`은 `String` 타입이며(사실 여기에는 부연설명이 약간 더 필요합니다, 여기 들어가는 type은 정확히는 `SchemaType` 라고 합니다.), `required` 속성을 가지고 있으며... 하는 것을 알 수 있지요. 그리고 `hashtags`를 보면, 배열 또한 사용할 수 있음을 알 수 있습니다. `default`로 따로 값을 지정하지 않았을 때, 자동적으로 들어가는 값도 설정할 수 있지요.

설명을 최대한 쉽게 넘어가기 위해서, 괄호치고 넘어간 `SchemaType`은 사실 되게 대단한듯 써놨지만 별 것 없습니다. `SchemaType`은 그냥 `type`과는 다르고, mongoose 내부에서 설정에 사용하는 object 입니다. `SchemaType`에 대한 자세한 설명은 공식 API 문서의 해당부분([링크](https://mongoosejs.com/docs/schematypes.html))를 참조 바랍니다.(사실 저도 정확하게 잘 모릅니다)

**지금 당장은 보이지 않지만 확실히 알아야 하는 한가지가 더 있습니다.** 우리가 적은 스키마 정의에는 들어있지 않지만, 실제로 스키마를 기반으로 모델을 만들면 자연스럽게 들어가는 요소인 `_id`(언더바가 **있음**) 입니다. mongoDB에서 데이터를 관리하는데에 사용되는 값으로 추측 됩니다. mongoDB에서 `_id` 즉, `ObjectID`가 어떤것인지 알려면, 잘 정리된 블로그([링크](https://koonsland.tistory.com/89))를 참조해 봅시다.

또, 비슷한 것으로 `id`(언더바가 **없음**)가 있는데, 이것은 `_id`의 getter 정도로 생각하면 됩니다. `ObjectID`는 2진 데이터인데, 이를 문자열 형태로 가져와주는 getter라고 보면 됩니다. 향후 CRUD 서비스를 만들 때, 각 컨텐츠의 고유번호로 기능하게 할 수도 있습니다. 마침 mongoDB에서는 `id`를 기준으로 검색을 할 수 있는 기능도 제공하고 있거든요.

mongoose에서는 스키마에 메소드를 추가로 더 장착 시킬 수도 있습니다. 아래의 예는 해시태그들을 처리하는데 쓰이는 코드인데, 로직 처리에서 꽤나 자주 등장하는 코드기에, copy-paste 보다는 일관적인 관리를 위해서 static 메소드로 해당 스키마를 통해서 사용할 수 있도록 하였습니다.

```js
videoSchema.static("formatHashtags", function (hashtags) {
  return hashtags
    .split(",")
    .map((word) => (word.startsWith("#") ? word : `#${word}`));
});
```

이렇게 만들어진 스키마는, 나중에 모델로 빌드 되었을 때, `모델명.static명()`형태로 불러 사용할 수 있습니다.

이렇게 스키마를 다 만들었으면, 모델로 빌드 한 다음, export 해서 다른 곳(Controller 같은 데..)에서 쓸 수 있게 하면 됩니다.

```js
const Video = mongoose.model("Video", videoSchema);
export default Video;
```

주의 사항 이라면, 스키마를 모델로 변환하는것은 일종의 컴파일 작업이므로, 스키마에 이것 저것을 추가할게 있으면 다 추가하고 모델로 전환해야 한다는 것 정도겠습니다.

`mongoose.model`의 첫번째 인수는 mongoose와 mongoDB내에서 쓰일 DB의 이름을 결정하는데에 사용됩니다. mongoose는 이 첫번째 인수로 온 문자열을, 앞글자를 소문자로 바꾸고, 복수형으로 만들어서 해당 이름으로 콜렉션을 찾고 없다면 만듭니다. 여기서는 Video 였으니, 콜렉션 이름은 videos 가 되는 식이지요.

# 3. DB에 쿼리 날리기(CRUD)

이제 우리가 저장할 데이터들을 다 설계하였으니, 실제로 CRUD 작업을 하는 쿼리를 다루는 방법에 대해서 다루어 보겠습니다. 쿼리를 날리는 문법은 mongoDB shell에서 하는것과 많은 방식을 공유한다고 합니다만, mySQl 같은 RDBMS에서 사용하는 `SELECT * FROM DBNAME`같은 SQL 쿼리에만 익숙한 분들(예를 들면 과거의 저)을 위해 정리를 해두도록 하겠습니다.

## 3-1 DB에 데이터 만들기

SQL 문에서 `CREATE` 에 해당하는 이 동작은 mongoose에서 두가지 방법으로 수행될 수 있습니다. 첫째는 위에서 만든 모델을 이용해서 `new 모델({...})`해서 객체를 만든 다음 해당 객체의 `.save()`메소드를 활용하는 것이고, 두번째는 `모델.create({...})`와 같은 방법으로 할 수 있습니다. 위에서 만든 `Video`모델을 이용해서 코드 예시를 보여보겠습니다.

1. 객체를 만든다음 `.save()`

```js
const video = new Video({
  title: "hello",
  description: "saying hello to everyone",
  createdAt: Date.now(),
  meta: {
    views: 0,
    ratings: 0,
  },
});

video.save(); //비동기 함수이기에, async-await나 .then, 콜백등을 이용해서 정상적인 코드 실행 순서를 보장할 필요가 있음
```

2. `.create()` 메소드 이용하기

```js
Video.create({
  title: "hello",
  description: "saying hello to everyone",
  createdAt: Date.now(),
  meta: {
    views: 0,
    ratings: 0,
  },
}); // 이 역시 비동기 함수임
```

## 3-2 DB에서 데이터 찾기

해당 모델의 `.find()` 메소드를 통해서 검색 할 수 있습니다. 검색을 할 때에는 여러 옵션을 사용할 수 있습니다. SQL 처럼 특정 필드의 값을 기준으로 찾을 수도 있고, 정규식을 활용할 수도 있고, 다양한 방법이 있습니다. 아래에 그 예시들을 정리해 두겠습니다.

1. 특정 필드의 값이 특정한 값과 일치하는가?

```js
Video.find({ title: "hello" }); // title이 "hello"인 데이터를 찾는다
```

같은 역할을 하는 SQL 문은

```SQL
SELECT * FROM VIDEO WHERE title = "hello"
```

정도가 되겠습니다.

2. 특정 필드의 값이 비교 연산자의 결과와 비교했을때 참인가?

SQL에서도 이러한 기능을 수행하는 연산자 중 하나인 `IN`연산자가 있는것을 아마 알 것 입니다. mongoDB에서도 `in` 연산을 쓰긴 하는데, `BSON`이라는 `JSON` 비슷한 형태로 자료를 저장하는 noSQL DB 특성상 완전히 문법이 같지는 않지만, 이해하는데에 도움이 될 것 같아서 한마디 해봤습니다.

```js
Video.find({title:{$in: ["hello" "goodbye"]}});//title이 "hello"나 "goodbye"인 데이터를 찾는다.
```

같은 역할을 하는 SQL 문은

```SQL
SELECT * FROM VIDEO WHERE title IN ("hello", "goodbye")
```

정도가 되겠습니다. `$in` 말고 다른 비교 연산자들도 정리해 보겠습니다.

- `$eq` : EQual. SQL의 `=` 연산자
- `$gt` : Greater Than. SQL의 `>` 연산자
- `$gte` : Greater Than or Equal. SQL의 `>=` 연산자
- `$lt` : Less Than. SQL의 `<` 연산자
- `$lte` : Less Than or Equal. SQL의 `<=` 연산자
- `$ne` : Not Equal. SQL의 `!=` 혹은 `<>` 연산자
- `$nin` : Not IN

3. AND, OR 조건 사용하기

SQL 에서는 AND 조건을 사용하려면, 조건들을 `AND`로 이어야 했지만, mongoDB에서는 그냥 해당 정보들을 쭉 이으면 됩니다.

```js
Video.find({ title: "hello", description: "saying hello to everyone" });
```

OR 연산은 `$or` 연산자를 통해서 할 수 있습니다.

```js
Video.find({
  $or: [{ title: "hello", description: "saying hello to everyone" }],
});
```

mongoDB에서 문서들을 관리하는데에 사용하는 `_id`를 활용하여 검색하는 함수인 `Model.findById()` 또한 있으니 기억해두면 좋습니다.

## 3-3 DB의 데이터 갱신하기

`Model.findOneAndUpdate()`를 사용해서, 검색 요건은 `.find()`에서 사용했던것과 동일하게 해서, 바꿀 데이터를 찾은 다음, 바꿀 내용을 집어넣으면 됩니다. `_id`를 이용해서 검색을 수행하는 `Model.findByIdAndUpdate()` 또한 있습니다. 아래의 코드는 제 CRUD 서비스에서 사용하는 `findByIdAndUpdate`를 사용하는 코드입니다.

```js
Video.findByIdAndUpdate(id, {
  title,
  description,
  hashtags: Video.formatHashtags(hashtags),
});
```

## 3-4 DB의 데이터 삭제하기

mongoose 에서는 데이터 삭제를 위해 아래와 같은 함수들을 지원해 줍니다.

- `Model.findOneAndDelete()`
- `Model.deleteOne()`
- `Model.deletMany()`

`deleteOne`과 `deleteMany`의 차이는 이름만 보고 알 것 같습니다. `findOneAndDelete`와 `deleteOne`의 차이는 무엇일까요? `findOneAndDelete`는 삭제한 데이터를 return 하지만, `deleteOne`은 그렇지 않다고 합니다. (해당 질문의 stack overflow [링크](https://stackoverflow.com/questions/42715591/mongodb-difference-remove-vs-findoneanddelete-vs-deleteone))

# 마치며

사실 위에서 언급했던 `populate` 기능까지 다룰려고 하였지만, 글의 분량이 생각보다 많이 길어져서, 이 글은 이쯤에서 정리를 해보려고 합니다. `populate` 설명하면서 DB구조도 설명하고 할려면, 내용이 더 길어질것 같거든요.

좀 글이 길어졌습니다. 끝까지 읽어주셔서 정말 감사하고, 혹시 잘못된 내용이나 설명이 더 필요한 내용이 있다면 댓글로 알려주시기 바랍니다.
