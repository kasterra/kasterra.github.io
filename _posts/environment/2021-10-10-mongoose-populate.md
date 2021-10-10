---
title: ObjectId를 실제 데이터로! populate() 에 대해서 알아보자!
layout: post
subtitle: RDBMS의 JOIN 문과 유사한, mongoose의 .populate()에 대해서 짚고 넘어갑시다.
image: /images/thumbnails/mongoose.png
category: environment
---

이번 글에서는, [저번 mongoose 글](/mongoose-basics)에서 나중에 다루겠다는, mongoose의 `populate`기능에 대해서 알아보겠습니다.

# 들어가며 : RDBMS의 JOIN이란?

지난 글에서도 언급을 했지만, 저는 MySQL 같은, RDMBS에 완전히는 아니지만, 어느정도 익숙한 경험이 있기에, NoSQL인 MongoDB와 Mongoose 공부를 하면서도, 해당 개념이 RDBMS의 어느 개념에 해당하는지 비교를 하면서 공부를 쭉 해왔습니다. 하지만, 이 글을 읽는 분들이 전부 RDBMS에 익숙하리라하고 예단하는것도 말도 안되는 것이기에, 본격적인 설명에 앞서서 이번에 설명할 `populate`기능과 비교될 RDBMS의 JOIN 기능에 대해서 설명을 먼저 해보려고 합니다.

이 게시물은 RDMBS의 JOIN을 설명하고자 하는 글이 아니기 때문에 정말 간단하게만 짚고 넘어가겠습니다. JOIN 연산은 두개 이상의 데이터베이스나 테이블을 연결해서 데이터를 검색하는 연산을 말합니다. 쉬운 이해를 위해서, 쇼핑몰에 사용되는 DB를 생각해 봅시다. 하나의 테이블에는, 고객의 **id**와, 물건을 배송받을 **주소지**가 있습니다. 그리고 다른 테이블에는, 이때까지 고객들이 주문을 한 **상품 정보**와, 해당 삼품을 주문한 고객의 **id**가 저장되어 있습니다.

이제 배송을 위해서, 고객들이 주문한 상품 정보와, 고객의 주소지를 모두 함께 알 필요가 있습니다. 애초에 한 테이블에 싹다 몰아넣거나, 아니면 두 테이블 모두에 주소지 데이터를 넣어 버리면 되지 않을까 하는 생각이 들 수 있습니다. 안되는것은 아니지만, 테이블의 컬럼이 너무 많아지면 보기도 좋지 않고, 속도에도 영향을 미칩니다.([출처](http://www.dba-oracle.com/t_number_columns_affect_SQL_speed.htm)) 그리고 두 테이블 모두에 주소지 데이터를 넣어버린다면 현재의 문제는 해결이 되겠지만, 주소지가 아닌 다른 정보를 참조할때에도 컬럼을 추가하다 보면, 비슷한 문제가 생길 것이고, 중복된 데이터가 테이블에 있다면 갱신 등을 하는데에도 상당히 불편함을 감수해야 할 것입니다.

이럴때 사용하는 연산이 JOIN 연산입니다. 하나 이상의 칼럼이 공유(이 상황에서는 **id**)되어 있을 때, 다른 테이블의 데이터 검색을 할 수 있게 되어 하나의 테이블처럼 검색을 할 수 있게 해주는 쿼리 문법입니다. JOIN 문의 종류는 INNER JOIN, OUTER JOIN... 뭐 이런식으로 있는데, 그 부분까지는 이 글에서 다루고자 하는 부분과 멀어지기 떄문에, 대강 JOIN 문이 이런 것이구나 하는 느낌 정도만 여기서 가져가시면 좋을 것 같습니다.

# 1. mongoose.Schema.Types.ObjectId

지난 글에서 예시로 들었던 스키마를 만드는 코드를 한번 더 살펴봅시다.

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

`ObjectId`는 이름 그대로,다른 object의 `_id` 값입니다. `Video`의 `owner`는 `User` 데이터를 담고 있습니다.(`ref : User`를 보면 알 수 있지요.) 데이터를 말 그대로 다 담고 있는것은 아니고, 그 데이터의 `id`를 담고 있어서, 그 데이터를 참고할 수 있게끔 하는 것입니다. 제 db에 있는 video 정보를 한번 여기 가져와 보겠습니다. mongo shell에서 입력한 것이지만, mongoose에서 가져온것과 크게 차이는 없을것으로 생각되어 가져 왔습니다. 가독성을 위해서, 임의로 개행을 약간 하였습니다.

```bash
> db.videos.find({})
{ "_id" : ObjectId("61604bdc8c20231c18f03097"), "meta" : { "views" : 0, "rating" : 0 }, "hashtags" : [ "#hello #wow #video #doge" ],
"title" : "Hello Everyone", "description" : "Hello Hello Hello Hello Hello", "fileUrl" : "uploads/videos/9c7d5cbfb969564138d7dfefd1231670",
"thumbUrl" : "uploads/videos/3c8eecacc8790ef54e057819a6bfd661",
"createdAt" : ISODate("2021-10-08T13:47:08.593Z"), "owner" : ObjectId("615da0c6301b3828748b2b58"), "__v" : 0 }
```

`"owner"`를 자세히 보면, `ObjectId`로 무언가가 기록이 되어 있습니다. `_id` 처럼요. 이것 자체로도 그렇게 나쁘진 않지만, 해당 `owner` 정보를 얻기 위해서는 id를 얻어서 `findById`를 하던 해서 한번 더 쿼리를 날린 다음에, 합쳐야 합니다. 상당히 번거로운 일이지요.

# 2. populate()

`populate`는 이 번거로운 일을 mongoose가 알아서 처리해서, 우리의 손을 편하게 해줍니다. 스키마에서 `ref`로 지정된 테이블에서 정보를 얻어온 다음에, 그 정보를 `ObjectId`가 있던 자리에 붙여 줍니다. 제가 만든 서비스에서 populate를 사용하는 예시입니다.

```js
videos = await Video.find({
  title: {
    $regex: new RegExp(keyword, "i"),
  },
}).populate("owner");
```

`videos`에 `find`로 쿼리를 날린 다음에, `owner` 필드의 자세한 정보를 알기 위해서, populate를 하는 예입니다.

populate를 한 데이터를 한번 더 populate를 할 수도 있습니다. 아래의 코드를 봐주시기 바랍니다.

```js
const user = await User.findById(id).populate({
  path: "videos",
  populate: {
    path: "owner",
    model: "User",
  },
});
```

`User`안에 있는 `Video`데이터를 populate 한 다음, 또 `Video`내의 `User`데이터를 populate 할 수도 있음을 볼 수 있습니다.

제가 공부하면서 만든 서비스에서는 쓰이지 않았지만, 특정 부분만 populate 하는 방법 또한 존재합니다.

```js
// simple populate
User.findOne({ _id: userId }).populate("blogs", { name: 1 }); // get name only

// nested populate
User.findOne({ _id: userId }).populate({
  path: "blogs",
  populate: {
    path: "comments",
    select: { body: 1 }, // 1
  },
});
```

# 3. mongoose populate 공식 API 문서 번역

[원본 문서](https://mongoosejs.com/docs/api.html#document_Document-populate)

**Document.prototype.populate()**
매개변수

- path <<String \| Object \| Array>> : populate 할 경로, 혹은 모든 매개변수를 명시하는 Object, 아니면 그 모든것을 담은 배열
- [select] <<Object \| String>> : population 연산을 수행할 필드 선택 쿼리
- [model] <<Model>> : population에 사용될 Model. 명시되지 않았을 경우, 스키마의 `ref` 필드를 참고 합니다.
- [match] <<Object>> : population 쿼리에 사용될 조건
- [options] <<Object>> : population 쿼리에 사용될 옵션들 (sort 등)
  - [options.path=null] <<String>> : populate할 경로
  - [options.retainNullValues=false] <<boolean>> : 기본값으로는, mongoose는 populate 된 배열로부터, `null`과 `undefined`값을 쳐낸다. `populate()`가 `null`과 `undefined`값을 보전하기 원하면 해당 옵션을 수정하면 됩니다.
  - [options.getters=false] <<boolean>> : `true`로 지정이 되어 있으면, mongoose는 `localField`에 정의된 getter를 호출할 것입니다. 기본값, 즉 `false`일때는, `localField`에 있는 그대로의 값을 가져올 것입니다. `lowercase`등의 옵션을 `localField`에 지정했고 그러한 것들을 사용하기 원한다면, 해당 옵션을 사용할 수 있겠습니다.
  - [options.clone=false] <<boolean>> : `BlogPost.find().populate('author')`를 실행하면, 같은 author를 가지고 있는 blogpost들은 같은 author 문서를 공유 할 것입니다. mongoose가 populate 된 문서들에 각각 복제본을 할당하기를 원한다면 해당 옵션을 활성화 하면 됩니다.
  - [options.match=null] <<Object \| Function>> : populate쿼리에 추가전인 필터를 더합니다. MongoDB 쿼리 문법을 담고 있는 Object 또는, 필터 Object를 반환하는 함수가 올 수 있습니다.
  - [options.transform=null] <<Function>> : mongoose가 populate된 데이터들에 대해서 호출할 함수입니다. populate된 문서들을 가공할 수 있는 옵션을 제공해 주는 셈이죠
  - [options.options=null] <<Object>> : `limit`와 `lean`과 같은 추가적인 옴션
- [callback] <<Function>> : 콜백 함수

# 4. populate()는 무적이 아니다...

꽤나 유용하게 쓰이는 `populate` 이지만 유의해야 할 사항들이 있습니다. populate는 ObjectId로 조회를 해서 자바스크립트 단에서 합쳐주는 것입니다. DB단에서 처리해주는 JOIN문과는 대조적이죠. 남용하면 성능 문제가 생길 수 있습니다. 특히, populate가 중철되면 될수록, 성늘 문제가 생길 문제도 더욱 커지므로, 필요할 때만 쓰는 태도가 필요합니다.
