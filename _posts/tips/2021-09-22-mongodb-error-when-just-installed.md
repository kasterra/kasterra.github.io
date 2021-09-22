---
title: MongoDB 설치 후 정상 작동 하지 않을 때
layout: post
subtitle: 분명히 공식 문서를 보고 올바르게 설치를 했는데 mongo 서비스가 안켜질 때
image: /images/thumbnails/mongoDB_Ubuntu.png
category: tips
---

# 개요

CRUD(Create, Read, Update, Delete)기능을 가진 웹 서비스를 만드는 공부를 한다고, 리눅스에 mongoDB를 설치해서 사용해 보려고 하였습니다. 분명, 몽고 db 공식 docs에 적힌 대로 차근차근히 따라했음에도, mongoDB 서비스가 올바르게 작동을 하지 않는 것입니다. [저번 문제](/fixing-audio-issue-in-ubuntu)처럼 구글링을 통해서 고치긴 하였지만, 다른 환경에서 작업할때도 반복적으로 생길 수 있는 문제이기에 제 블로그에 정리를 해둘려고 합니다.

# 발생한 문제
공식 docs에서 [지시한 사항](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/) 대로 설치를 진행하였습니다. 하지만, 몽고db 서비스를 `systemctl start mongod`로 실행하고, `systemctl status mongod`를 통해서 `mongod` 서비스가 올바르게 실행이 되있는가를 확인해 보았을 때, 터미널에서는 서비스가 제대로 실행이 되고 있지 않다는 상태를 보여주고 있었습니다. 

![status mongod failed](/images/tips/mongod-status-fail.png)
<div style="display:flex; justify-content:center;">mongod 서비스가 실행되지 못하는 모습</div>

이 상태는 mongodb 서버가 아예 실행이 되고 있지 않기 때문에, 로컬 몽고 서버에 접속하는 `mongo` 명령어도 제대로 들을 턱이 없습니다.

# 원인과 해결책
해당 문제의 원인은 제 경우에는 `mongodb-27017.sock` 파일의 권한 문제 때문 이었습니다. `.sock` 파일은 리눅스와 같은 유닉스계 운영체제에서 프로세스간 통신에 필요한 파일인데, `mongodb-27017.sock` 파일의 권한이 `mongodb`로 설정되어 있지 않아서 생기는 문제입니다.

아래와 같은 명령어를 이용해 파일 권한을 수정해 주면 정상 작동 합니다.
```bash
$ chown -R mongodb:mongodb var/lib/mongodb
$ chown mongodb:mongodb /tmp/mongodb-27017.sock
```

![status mongod sucess](/images/tips/mongod-status-success.png)

정상 작동하는것을 확인할 수 있습니다.