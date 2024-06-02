---
title: 커다란 csv 파일을 DB에 올리고 가공해보자
subtitle: 9GB 가량 되는 큰 데이터 파일을 DB에 넣고 잘 활용해 봅시다
layout: post
category: environment
image: /images/thumbnails/csv-mongo.png
---

# 들어가며

학부연구생으로 일하다 보면 다양한 과제에 직면하게 됩니다. 최근 글을 4번이나 연달아서 쓴 KOJ도 열심히 만들긴 했지만, 연구비를 받기 위한 원천인 "연구실 과제"도 소홀히 할 수 없지요.

작년에 완성해놨던 과제이지만, 서버 유지보수 작업 등을 하면서 기존의 작업물을 배포해둔 환경이 날라가서, 다시 과거의 저의 코드를 보게 되었고, 나름 정리할 가치가 있다고 생각해서 정리를 해봅니다.

# 배경

해당 과제에서 저의 역할은 수많은 데이터들 중에서 최적의 값을 구하는 로직의 인터페이스를 구축하는 것이었습니다. 데이터를 저장해둔 csv 파일의 용량이 9GB가 넘어갈 정도로 상당히 방대한 데이터들 이었죠. 당연히 그 수많은 데이터를 손으로 입력할 수 없고, 스크립트를 이용해서 올려야 했습니다.

하지만 파일이 원체 크다보니, 단순하게 파일을 메모리에 한번에 올리기도 쉽지 않고, 설사 올라간다 하더라도, 얼만큼 진행되었는지 모르는 채로 커서만 깜빡깜빡 거리고 있다보면, 작업이 제대로 되고 있는건지, 뭔가 문제가 생긴건지 하는것도 모르게 되어서 문제가 있어도 알 수가 없죠...

# 업로드 해결책 : pandas & tqdm

파일이 너무 큰것에서 나오는 문제는 한번에 파일을 읽지 않고, chunk 단위로 파일을 읽어서 올리면 됩니다. 그리고, 진행상황에 관해서는 파이썬 환경에서 진행상황을 표시해주는 `tqdm` 라이브러리를 사용하면 터미널에 나오는 진행바를 손쉽게 구현 할 수 있습니다.

```python
import pandas as pd
from pymongo import MongoClient
from tqdm import tqdm  # tqdm 라이브러리를 import합니다.
from dotenv import load_dotenv
import os

# 환경 변수 파일(.env) 로드
load_dotenv()

# 환경 변수에서 인증 정보 가져오기
MONGO_USER = os.getenv('MONGO_USER')
MONGO_PASSWORD = os.getenv('MONGO_PASSWORD')

# MongoDB 접속 설정
client = MongoClient(f'mongodb://{MONGO_USER}:{MONGO_PASSWORD}@localhost:27017/')
db = client['research_db']
collection = db['simulation_data']

# CSV 파일 읽기
file_path = 'very_large_simulation_data_file.csv'
chunksize = 1000  # 한 번에 처리할 로우의 수

# read_csv 함수를 chunksize 파라미터와 함께 사용
for chunk in tqdm(pd.read_csv(file_path, chunksize=chunksize), desc='Uploading chunks', unit='chunk'):
    records = chunk.to_dict('records')
    collection.insert_many(records)  # 한 번에 청크 단위로 업로드
```

# 데이터 정규화 및 인덱싱

DB에 데이터를 올렸다고 해서 끝이 아닙니다. 우리가 원하는 값을 찾기 위해서, 값들을 정규화 할 필요가 있었고, 그리고 정규화된 값을 빠르게 찾기 위해서 적절한 인덱싱 전략도 필요하지요. mongoDB는 SQL을 사용하지 않고, javascript를 통해서 데이터들을 관리합니다. 간단히 돌아가는 JS코드를 적어서 올려봅시다.

```js
// 정규화할 물성 목록
var properties_to_normalize = [
  "tenacity",
  "elongation",
  "ts_avg_temp",
  "ts_avg_str",
  "unevenness",
  "yarn_num",
  "crystallinity",
  "degradation_temp",
  "melt_temp",
  "glass_tran_temp",
];

// 최소/최대 값을 계산합니다.
var minMaxValues = {};
properties_to_normalize.forEach(function (prop) {
  var minVal = db.simulation_data
    .aggregate([{ $group: { _id: null, min: { $min: `$${prop}` } } }])
    .toArray()[0].min;
  var maxVal = db.simulation_data
    .aggregate([{ $group: { _id: null, max: { $max: `$${prop}` } } }])
    .toArray()[0].max;
  minMaxValues[prop] = { min: minVal, max: maxVal };
});

var batchSize = 100000;
var totalDocs = db.simulation_data.count();
var numBatches = Math.ceil(totalDocs / batchSize);

for (var i = 0; i < numBatches; i++) {
  var pipeline = [
    { $skip: i * batchSize },
    { $limit: batchSize },
    {
      $addFields: properties_to_normalize.reduce(function (acc, prop) {
        var min = minMaxValues[prop].min;
        var max = minMaxValues[prop].max;
        var newField = "norm_" + prop;
        acc[newField] = {
          $cond: {
            if: { $eq: [max, min] },
            then: "$" + prop,
            else: {
              $divide: [
                { $subtract: ["$" + prop, min] },
                { $subtract: [max, min] },
              ],
            },
          },
        };
        return acc;
      }, {}),
    },
    {
      $merge: {
        into: "simulation_data",
        whenMatched: "merge",
      },
    },
  ];

  db.simulation_data.aggregate(pipeline);
  print(`Batch ${i + 1}/${numBatches} processed.`);
}

properties_to_normalize.forEach(function (prop) {
  print(`Indexing ${prop}...`);
  db.simulation_data.createIndex({ ["norm_" + prop]: 1 });
  print(`Indexing ${prop} done.`);
});
```

순서대로 물성들을 정규화 하기 위해서 최소/최대 값을 계산한 다음, 정규화 과정을 실시합니다. 이 때, 정규화 파이프라인을 크게 하나만 둬서 하면 업로드 때와 마찬가지로 진행사항을 확인할 수 없는 답답함을 겪을 수 있으니, 이것도 batch들을 적당한 크기로 쪼개어 진행해 줬습니다.

# 마치며

별거 아닌 정말로 자그마한 스크립트지만, 나중에 혹시 필요할지 몰라, 그리고 혹여나 구글 검색창을 헤메는 누군가에게 도움이 될까 해서 저의 작은 지식을 공유하여 봤습니다. 감사합니다.
