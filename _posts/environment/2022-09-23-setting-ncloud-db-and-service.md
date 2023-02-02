---
title: ncloud 서버 인스턴스에서 mySQL 세팅하기
layout: post
subtitle: 이미 만들어져있는 db 인스턴스가 있지만 한번 직접 해봅시다.
image: /images/thumbnails/ncloud-set.png
category: environment
---

# 들어가며

공부를 하면서 서버를 mysql이 돌아가며, express로 만든 물건을 배포할 서버가 필요해졌습니다. ncloud에서 이미 mysql관련한 세팅을 끝내놓은 밀키트같은 mysql 서버 인스턴스를 제공해 줬지만, 공부를 이왕 하는 김에 처음부터 제대로 만들어 보자 하는 생각으로, 해보려고 했는데, 생각보다 많은 삽질을 하게 되어서, 예전에 군대에 있을 때, [vps를 설정했던 글](/code-server-with-vps/)처럼 삽질을 정리해보고자 합니다.

# 인스턴스 만들기

학습용으로 사용할 것이기에 그렇게 많은 부하가 걸리지 않을것이지만, micro 특유의 버벅거림은 싫은 저는 compact로 서버를 만들고자 합니다.

![create server compact](/images/environment/ncloudStep1.png)

설명을 따라가면서 차근차근 세팅을 하고, 최종적으로 저는 아래와 같은 세팅을 하였습니다.

![server create result](/images/environment/ncloudCreateF.png)

이렇게 해서 몇분을 기다리면 서버 인스턴스가 준비됩니다. 만들어지고, 부팅되고, 세팅되는데에 많은 시간이 소모되므로, 차 한잔 하고 오셔도 좋습니다.

기다리고 있다보면 메일이 한통 옵니다.

![instance complete](/images/environment/server_complete.png)

# 네트워크 설정

클라우드 기반의 무언가를 사용하셨던 분이라면, 방화벽 세팅 같은것을 해줘야 외부에서 접속이 가능하다는 사실을 아마도 아실 겁니다

## ACG 설정

아까 우리가 만든 인스턴스에 ssh로 접속을 하고, mysql 서버에 접속하기 위해서는 특정 포트를 열어야 하죠. 그 설정을 해볼겁니다.

ncloud의 좌측 메뉴에서 server의 상세메뉴로 들어가면 ACG를 설정할 수 있는 탭이 있습니다.

![acg_setting_location](/images/environment/acg_setting_location.png)

ssh를 위한 22번, mysql을 서비스하기 위한 3306번 포트를 포함해서 개인의 필요에 맞게 세팅해주면 됩니다. 아래는 제가 사용하는 ACG 설정입니다.

![my-acg-setting](/images/environment/acg-setting.png)

## 공인 IP 발급

기본적으로 ncloud 인스턴스는 실제로 존재하는 IP에 할당되어 있지 않고, ncloud 내부에서만 쓰이는 사설 IP에 묶여 있습니다. 사설 IP로는 접근할 수 없기 때문에, ssh를 통한 서버 관리를 위해서는 포트 포워딩 설정을 해서 ssh 전용 IP를 발급받거나(무료), 공인 IP를 발급받아서 접속을 할 수 있습니다(월정액). 저는 공인 IP를 발급 받았는데, 이유는 별게 아니고, 어차피 웹 서비스를 제공하기 위해서는 외부에서 접속을 할 수 있게 공인 IP를 발급 받아야 하는데, ssh를 접속한다고 괜히 또 다른 IP를 발급받아서 그거 확인하기가 귀찮았거든요. ssh로 접속하기 위한 다른 IP를 발급받는 쪽이 보안상 더 좋으련지는 모르겠지만... 일단 저의 게으름으로 이정도로 할까 합니다.

공인 IP를 발급받는 곳인 ACG를 설정했을때와 비슷하게, ncp 좌측 메뉴에 Public IP에 들어가서 발급받을 수 있습니다. 공인 IP 신청을 누르셔서 발급 받으실 수 있습니다.

IP를 발급 받으시고, 서버 세팅 콘솔 화면에서 IP가 연결되어 있는지 확인합니다.

# ssh 연결 및 기초 설정

ssh연결을 하려면, 비밀번호를 우선 알아야겠죠. 관리자 비밀번호는 아래의 스크린샷에 나와있는 것처럼 서버 콘솔 화면에서 관리자 비밀번호 확인 메뉴를 통해서 확인할 수 있습니다.

![admin-pw-확인](/images/environment/admin-password.png)

해당 비밀번호 확인 절차에서 인스턴스를 만들때 안전한 곳에 저장하라고 했던 그 파일을 통해서 비밀번호를 확인할 수 있습니다.

![admin-pw-확인](/images/environment/admin-password-chk.png)

어디에 복사해두고 ssh 연결을 합시다. 그리고 `passwd` 명령어를 콘솔에 입력해서 비밀번호를 편한걸로 바꿔줍시다. 그리고 `apt update`와 `apt upgrade`를 통해서 기초적인 설정을 해줍시다.

아래와 같이 뭔가가 충돌난다는 창이 나왔을 때, 저는 배포자의 설정에 따름(Y)로 진행하였습니다.

![setting config](/images/environment/config-file-check.png)

# mysql 설치 및 설정

mysql을 설치하는데에는 [digital ocean에서 제공하는 가이드](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-18-04)대로 진행하였습니다.

mysql을 설치하였으면, 외부에서 접속할 수 있는 사용자를 만들어야 할 것입니다. 이것도 참조한 링크가 있어서 공유합니다.

[https://cjh5414.github.io/mysql-create-user/](https://cjh5414.github.io/mysql-create-user/)

```bash
mysql -u root
mysql> CREATE USER '사용자'@'%' IDENTIFIED BY '비밀번호';
```

사용자, 비밀번호 부분은 당연히, 사용할 사용자명과 비밀번호 명으로 하시면 되고, 여기에 있는 `%`의 의미는 **모든 주소**입니다. mysql에서는 접속한 주소에 따라서 다른 유저로 인식하기 때문에 이 때문에 어떠한 외부 서비스에서도 접근할 수 있는 db 서버를 구축하기 위해서는 위와 같이 하면 됩니다.

그리고 유저를 만들었으면 당연히 권한을 줘야 할 것입니다.

```bash
mysql> GRANT ALL PRIVILEGES ON *.* TO '사용자'@'%';
mysql> exit;
```

# 외부에서 사용할 수 있게 설정하기

이제 우리는 외부에서 사용할 수 있는 유저를 만들었고, 해당 유저에게 권한을 주었지만, 지금 이 상태로는 db에 접근을 할 수 없습니다. 결론부터 말씀드리자면 mysql의 `bind-address` 설정이 `127.0.0.1`로 되어 있어서 외부에서 접근을 할 수 없기 때문이고, 이는 수정해 주면 됩니다. 관련 문서들을 공유합니다.

- [https://zetawiki.com/wiki/MySQL*ERROR_2003*(HY000):\_Can't_connect_to_MySQL_server_on](<https://zetawiki.com/wiki/MySQL_ERROR_2003_(HY000):_Can't_connect_to_MySQL_server_on>)
- [https://yoshikixdrum.tistory.com/217](https://yoshikixdrum.tistory.com/217)

```bash
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

여기에서 내리다 보면 `bind-address = 127.0.0.1` 라고 적힌 부분이 있는데 이 앞에 `#`문자를 더해줌으로서, 해당 속성을 주석으로 만들어서 없애면 됩니다.

그리고 mysql을 재시작 해줍니다.

```bash
sudo service mysql restart
```

그 다음에 `sudo service mysql status`를 입력해서 정상적으로 실행이 되는지를 확인해 보세요.

제대로 설정이 되었는지 아래의 명령어를 통해서 확인해 봅시다.

```bash
netstat -antp | grep mysql
```

모든 IP인 0.0.0.0에 대해서 열려있는지 확인이 되었다면, 이제 외부에서 접속을 시도해 봅시다. 해당 서버가 아닌 쉘에서 mysql CLI도구나, 기타 도구를 통해서 접속이 되는지 확인 해 봅시다. 전 잘 되더라고요.

# node 설치

저는 node위에 돌아가는 서버를 실행시키기 위해서 이 인스턴스를 구매했습니다. [공식 홈페이지에 소개된 가이드](https://github.com/nodesource/distributions/blob/master/README.md#debinstall)를 참조해서 아래의 명령어를 실행시켜 줍시다.

```bash
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs
```

`node`를 입력해서 만만한 `console.log()`한번 때려보면 제대로 되는것을 확인 할 수 있습니다.

# 혹시나를 대비! 이미지 따놓기.

지금은 잘 되지만, 나중에 프로그램 몇개 설치하다가 뭔가 설정이 꼬여서 DB가 재시작이 안된다거나, 하는 대참사가 일어날 수 있습니다. 리눅스를 처음 배울때 하드 리셋을 여러번 했지만, 이건 VM 서버이기 때문에 이미지를 따놓는 방법을 통해서, 대참사가 일어났을 때 이미지를 떠놨을 때의 시점의 서버를 새로 만들어 낼 수 있습니다.

![admin-pw-확인](/images/environment/admin-password.png)

여기 보시면 관리자 비밀번호 확인 밑에 내 서버 이미지 생성 이라는 탭이 있습니다. 이 탭을 클릭하여, 현재의 시점을 이미지로 따놓는다면, 나중에 대참사가 일어났을 때, 복구가 쉽습니다. :+1:

# 마치며

사실 참 경험있으신 분들에게는 이 인간은 이렇게 당연한 이야기를 구구절절 써놨나 싶겠지만, 백엔드나 이런 환경 쪽에 너무 둔감하던 저였어서, 정말 삽질을 많이 했었습니다. 그 삽질의 과정 속에서 배운것도 있었지만, 삽질을 반복하는것은 별로 좋은 경험이 아닌것 같아서, 다른분들이 검색 했을 때, 삽질을 좀 방지하고, 그리고 제 삽질을 방지(사실 이게 주 목적) 하기 위해서 이렇게 기록을 남깁니다. 끝까지 읽어주셔서 감사합니다!!!
