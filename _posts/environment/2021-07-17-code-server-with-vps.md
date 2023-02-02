---
title: 사지방에서 개인 개발 공간을! VPS로 code-server 서버 구축기
layout: post
subtitle: 우리들의 친구 vscode를 언제 어디서나 쓸 수 있게 해봅시다.
image: /images/thumbnails/vultrVscode.png
category: environment
---

# 들어가며

군대 등의 사유로, 본인 PC에 프로그램을 따로 깔 수가 없어서, 개발환경 설정을 못하여, [구름 IDE](https://ide.goorm.io/)와 같은 서비스를 이용하는 분들이 꽤 있을것 입니다. 구름 IDE는 무료로 대중적으로 쓰이는 언어나 환경들을 잘 지원해주기 때문에, PS등의 간단한 코딩에 쓸만하고, 개발도 못해먹을 것 까지는 아닙니다.

하지만, 구름 IDE에도 한계가 있습니다.

- 무료 플랜은 성능이 떨어진다. → 물론 유료결제 해서 유료플랜 쓰면 되긴 합니다만, 하술할 이유로 결제할 마음은 별로 안들었습니다.
- vscode가 아니므로, vscode에 익숙한 저로써는 그 편의기능들이 너무나도 그리웠습니다.
  - vscode의 확장 플러그인. prettier, eslint 와 같은것들이 없으니까 많이 아쉽다.
  - `pug`와 같은 추가 템플릿 언어를 끌어다 쓰니까, 그런 것들은 코드 포맷팅을 구름 IDE에서 못해준다.
  - node.js로 개발하다보니, vscode 특유의 친절함이 그립다. (다른 모듈에서 `export` 함수를 불러오려고 하면 자동으로 윗줄에 `import`가 적절하게 추가되는것 등)
  - HTML 코드를 칠 일이 있으면, vscode등 대부분의 텍스트 에디터에서는 `!`을 치면 HTML 틀이 나와주고, `div.classname*3`이런식으로 입력하면 적절하게 만들어 주고 하는 기능이 있는데 구름 IDE에는 없었습니다.

사실 이렇게 이유를 써놨지만 vscode를 쓰고 싶다 한마디로 요약이 가능합니다. 여하튼, 저는 저에게 딱 맞는 개인 개발환경을 만들기 위해서 여러 정보들을 찾아봤습니다.

# code-server

`Visual studio code`는 `electron`기반으로 짜여진 프로그램 입니다. 아시는 분은 아시겠지만, `electron`은 `node.js`와 `chromium`으로 돌아가는 프레임워크이고, `Visual studio code`자체는 오픈소스 프로그램이기 때문에, vscode를 서버에 올릴 수 있도록 만든 [code-server](https://github.com/cdr/code-server)라는 오픈소스 프로그램이 있습니다.
개인적으로 사용할 수 있는 서버로 작동할 수 있는 무언가만 있다면, vscode를 사용할 수 있는 환경을 만들어 줄 수 있다는 것입니다.

단적인 예시로, 안드로이드 기기에서 리눅스 환경을 제공하는 [Termux](https://termux.com/)등의 환경에서도 해당 서버를 실행시키면, 인터넷에 연결되어 있지 않아도 `localhost:8080`을 주소창에 입력해서 vscode를 사용할 수 있고, 개발진 측에서도 이 방법을 [소개](https://coder.com/docs/code-server/v3.11.0/termux)하고 있습니다. 이런 세팅을 귀찮아하는 사람들을 위해서, [VHEditor](https://play.google.com/store/apps/details?id=vn.vhn.vsc&hl=ko&gl=US)라는 앱또한 만들어져 있습니다.

그렇다면 우리에게 `code-server`를 돌려줄 적당한 서버 한대만 있다면, 네이버에 접속하듯 개인 vscode 공간에 웹 브라우저로 접근 할 수 있다는 것입니다. 이제 그 **적당한 서버**를 구하는 작업을 해봅시다.

개인적으로 집에 굴리고 있는 서버가 있다면 그걸 써도 되겠지만, 아쉽게도 집에 그럴 서버가 없었습니다. 그래서 알아본게 가상 사설 서버 즉 VPS 였습니다. code-server를 돌려보겠다고 결심했을 당시에는, `Vultr`라는 업체가 서울 리전을 제공하고, 첫달에 거의 무료로 쓸 수 있을만큼의 크레딧을 줘서, 저는 `Vultr`를 골랐고, 지금 현재도 잘 쓰고 있습니다. 무료 스냅샷 기능이 있어서, 가끔씩 뇌절을 해서 서버가 예전처럼 잘 굴러가지 않을 때, 복구를 간단히 할 수 있다는 장점이 있는것에 상당히 만족하고 있습니다.

물론 본인이 선호하는 다른 서비스가 있다면 그걸 사용하셔도 됩니다. `Amazon lightsail`도 있고, `DigitalOcean`이라던가, `linode`등등 좋은 서비스는 많고, 여기에 적지 않았지만, 수많은 VPS 업체들이 있습니다.

<strong><i class="far fa-pause-circle"></i> 잠깐만요</strong><br/>
혹시 지금 글을 읽고계시는 분이 이 포스트를 보면서 code-server 설치를 따라해 볼 생각이 있으시면, "얘가 `vultr`라는 서비스로 했으니, 나도 그걸로 해볼까? 하고 따라오시지 말고, 다른 옵션들또한 꼭 확인해 보시기 바랍니다. [밑](#code-server-서버에-설치하기)에서 언급하겠지만, 몇몇 VPS업체들을 선택한다면 설치 과정을 좀 더 간단히 할 수도 있거든요
{:.info}

[vultr에서 소개하는 code-server 설치법](https://www.vultr.com/docs/install-code-server-on-a-ubuntu-18-04-lts-vps)을 참고해서 설치를 진행했지만, 최근에 code-server 신버전이 나와서 업데이트 했다가, 사지방의 구형 크로미움 브라우저에서 보이지 않는다는 치명적인 문제 때문에 최신 직전 버전으로 업데이트 아닌 업데이트를 하면서 느낀 점이 있기에 이를 반영해서, 과정 설명을 시작해 보겠습니다.

# 준비물

- code-server를 실행할 수 있는 인터넷에 연결된 서버 (제 경우에는 vultr VPS를 사용했습니다.)
  - 최소 1GB램에 2CPU 자세한 사항은 [공식 DOCS](https://github.com/cdr/code-server/blob/main/docs/requirements.md)참조
  - 해당 서버에 대한 root 권한
- HTTPS를 사용할 것이므로, 개인 용도로 쓸 수 있는 도메인

이거면 충분합니다. 몇몇 지식이 있으면 좋겠지만, 필요한 지식은 얼마 되지 않기 때문에 이 포스팅에서 다 다루기에 충분한 양입니다.

# code-server 서버에 설치하기

code-server의 `readme.md`를 보면

1. install script를 사용하기 <i class="fas fa-arrow-right"></i> 최신 버전으로만 설치됨
2. 직접 install 하기(deb 파일을 줬으니 dpkg 등을 이용해서 설치해라) <i class="fas fa-arrow-right"></i> 특정 버전을 명시해서 설치 가능
3. 특정 VPS업체를 위한 원-클릭 버튼을 활용하기 [해당 링크](https://github.com/cdr/deploy-code-server) <i class="fas fa-arrow-right"></i> vultr는 아직 해당사항이 없음

맨 처음에 vps를 알아볼때는 [vultr에서 소개하는 code-server 설치법](https://www.vultr.com/docs/install-code-server-on-a-ubuntu-18-04-lts-vps)만 봐서, vultr에서 나름 공식적으로 code-server를 어떻게 설치하는지 알려주고, 다른 업체들보다 더 싼것 같으니, 이걸로 해야겠다는 생각에 시작한 것이고, 맨처음에 확인해 볼때는 저 3번옵션의 존재를 몰랐습니다. 만약에 저 옵션의 존재를 알았다면, `digitalOcean`을 쓰지 않았을까 하는 생각이 들긴 하는데, 지금 서버를 여러 용도로 알차게 써먹고 있으니, 굳이 이사할 필요성을 느끼지 못해서 그냥 쓰고 있는 상태입니다.

군대 사지방에서 쓰기 위한 용도로 관련 정보를 알아본 저로써는 2번 선택지말고는 고려할 수가 없었습니다. 글을 쓰고 있는 시점에서의 최신버전인 `3.11.0`버전은 사지방에서 돌아가는 `78.0.3904.70`버전의 크로미움에서 정상적으로 보이지가 않습니다. `github docs`도 안보여서 휴대폰으로 보거나, 파파고 웹페이지 번역기능을 통해서 접속해서 봐야하는 신비한 브라우저 말고는 쓸 수 있는게 없는 환경에서는, 최신버전보다 바로 한단계 구버전인 `3.10.2` 버전으로 설치를 진행했습니다.

서론이 길었습니다. 이제 본격적인 설치에 들어가겠습니다.

```bash
curl -fOL https://github.com/cdr/code-server/releases/download/v3.10.2/code-server_3.10.2_amd64.deb
sudo dpkg -i code-server_3.10.2_amd64.deb
sudo systemctl enable --now code-server@$USER # 서비스로 등록
```

저는 우분투 20.04 버전을 서버에 설치하였습니다. 혹시나 우분투 16.04나 그 이전의 버전을 사용하신다면, `yarn`이나 `npm`을 이용해서 설치 할 수 있습니다. 그런데 2021년 시점에서 그정도의 구버전을 쓸 일은 잘 없지 싶어요.

```bash
yarn global add code-server #설치
code-server #실행
```

<strong><i class="fas fa-exclamation-triangle"></i> 잠깐만요</strong> <br/>
이 상태에서는 바로 접속을 해봐도 되지 않을것 입니다. 이유는 기본 `bind-address`가 `127.0.0.1:8080`으로 되어 있는데, `127.0.0.1` 다른 말로 `localhost`라는 주소는, **같은 기기에서만 접근이 가능** 합니다. VPS등의 서버에 접근할때는 `ssh` 등의 원격접속 프로토콜로 연결하므로 안되는것은 어찌보면 당연합니다. 그러므로 `bind-adress`를 `0.0.0.0`으로 수정하면 됩니다.
{:.warning}

```bash
vim .config/code-server/config.yaml
```

을 하여 `bind-addr`부분을 외부에서도 접근할 수 있게 `0.0.0.0`로 수정해주고, 서비스를 재시작 해줍시다.

```bash
systemctl restart --now code-server@$USER
```

사족으로 해당 서비스에 관한 명령어들은

```bash
systemctl stop code-server@$USER # 서비스 중지
systemctl start code-server@$USER # 서비스 시작
systemctl restart code-server@$USER # 서비스 재시작
```

입니다.

다시 http://\[IP주소\]:8080 로 들어가서 `.config/code-server/config.yaml`에 명시된 비밀번호를 입력하면 아래와 같은 창이 나올것 입니다.
![success with http](/images/environment/code-server first.png)
하단 오른쪽에서도 볼 수 있듯이, 보호되지 않은 연결이라서 몇몇 기능들이 동작하지 않을것이라고 합니다. 가장 크게 와닫는것은 **붙여넣기 기능**과 **확장 마켓플레이스 기능** 입니다. 이들을 사용하기 위해서는 안전한 연결, 즉 https를 사용해야 하는데, https 연결을 사용하려면 SSL 인증서를 발급받아야 합니다. IP주소에도 인증서를 발급받을 수 [있지만](https://www.koreassl.com/support/faq/%EB%8F%84%EB%A9%94%EC%9D%B8%EC%9D%B4-%EC%95%84%EB%8B%8C-Public-IP-%EA%B3%B5%EC%9D%B8IP%EC%97%90%EB%8F%84-%EC%9D%B8%EC%A6%9D%EC%84%9C-%EB%B0%9C%EA%B8%89%EC%9D%B4-%EA%B0%80%EB%8A%A5%ED%95%9C%EA%B0%80%EC%9A%94) IP주소에 인증서를 발급받으려면, **무료로는 안되고**, 더욱 비싼 돈을 내야 합니다.

도메인 하나 사서 `let's encrypt`등의 무료 서비스를 사용하는것이 더욱 저렴하게 먹히기 때문에, 저는 도메인을 하나 구입했고, 여기서도 그 기준으로 설명해볼까 합니다.

# nginx 이용하여 도메인 연결하기

이렇게 IP 주소와 포트번호로 접속을 할 수 있지만, 아까 말했듯, 보안 인증서를 넣으려면 더 많은 돈이 필요하고, 외우기도 쉽지 않습니다.
따라서, ngnix를 이용하여, 특정한 도메인으로 접속했을 때, code-server로 리다이렉션 시켜주는 **Proxy redirection**을 하도록 구성하면, 도메인주소로 연결할 수 있습니다.

ngnix를 설치하는것은 매우 간단합니다.

```bash
sudo apt install nginx
```

이제 ngnix가 읽어 처리할 파일을 생성해 주어야 합니다.

```bash
sudo vim /etc/nginx/sites-available/code-server.conf
```

내용은 대강 아래와 같이 채워줍시다.

```conf
    server {
    	listen 80;
    	listen [::]:80;
    	server_name <your domain>;
    	location / {
    		proxy_pass http://127.0.0.1:8080/;
    		proxy_set_header Upgrade $http_upgrade;
    		proxy_set_header Connection upgrade;
    		proxy_set_header Accept-Encoding gzip;
    	}
    }
```

여기서 `<your domain>` 부분은 사용할 도메인의 주소를 적어줍시다.

이 설정은, nginx가 80번 포트를 listen 하다가, 연결이 들어오면 `http://127.0.0.1:8080`으로 연결을 전달해라는 설정입니다. 8080번 포트는 아까 code-server가 먹고 있는 포트임을 아까 확인한 바 있지요.

이제 이 내용을 `sites-enabled`에도 복사해 주어야 합니다.심볼릭 링크를 하나 만들어 줍시다.
그리고, 기본 80포트로 listen 하는 `default`또한 존재합니다. 이 역시 제거합시다.

```bash
sudo ln -s /etc/nginx/sites-available/code-server.conf /etc/nginx/sites-enabled/code-server.conf
sudo rm /etc/nginx/sites-available/default
```

다음에는 ngnix 설정이 읽기에 문제가 없는지 체크하고, nginx를 재시작 합시다.

```bash
nginx -t #만약 여기서 에러가 나온다면 오타가 난것이므로, 수정하고 에러가 생기지 않음을 확인하고 다음 단계로 진행할것
systemctl reload nginx.service
```

이제 도메인의 dns 서비스에서 A 레코드 하나를 추가해 줍시다. 주소는 위의 `code-server.conf`에서 `server_name`으로 적어줬던 그 값을 적어줘야 합니다.
![dns A 레코드 만들기](/images/environment/code-server dns.png)
그리고 지정한 도메인으로 접속하면 정상적으로 접속되는 모습을 볼 수 있습니다.
![도메인 성공적 접속](/images/environment/domain successful.png)

# certbot으로 SSL 인증서 발급받기 (HTTPS)

이제 SSL 인증서를 발급받아서, HTTPS 연결을 사용시켜 봅시다. 무료 SSL 서비스인 `let's encrypt`를 활용하여 SSL 인증서를 발급받아 봅시다.

우선 `certbot`을 설치합니다. 정확하게는 nginx 플러그인 입니다만, apt가 의존성을 잘 해석해 주어서 certbot도 자동으로 설치해 줍니다.

```bash
sudo apt install certbot python3-certbot-nginx
```

그다음 certbot을 실행해서, 우리의 서버를 인증받읍시다.

```bash
certbot --nginx -d <your domain>
```

위 명령의 뜻은 nginx 설정을 자동으로 바꿔주는 플러그인(아까 설치한것)을 사용하고 \<your domain> 도메인을 인증받겠다는 뜻입니다. 물론 이 자리에는 우리가 쓸 도메인을 넣으면 됩니다.

차례로 이메일 주소를 입력해주고, 뉴스레터를 받을지 여부를 설정해 줍니다. 앞의것들은 좀 시시콜콜한 설정이라서 딱히 중요하진 않지만, **중요한 물음이 이제 나옵니다.**
![certbot 리다이렉션](/images/environment/certbot redirect.png)
HTTP 연결을 모두 HTTPS 연결로 바꿀것이냐를 묻는데, .dev 도메인 처럼 https가 강제된 도메인을 사용할 경우 반드시 2번 옵션을 선택해야 합니다.

이제 다시 접속을 해보면 https가 적용된 모습을 확인할 수 있습니다
![code-server https](/images/environment/code-server-https.png)
