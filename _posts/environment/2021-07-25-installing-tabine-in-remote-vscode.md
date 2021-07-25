---
title: code-server에 AI 코딩 도우미 tabnine 설치기
tags: 
layout: post
excerpt: github Copilot의 경쟁자이기도 한 AI 자동완성기 tabnine을 한번 설치하고 설정까지 해봅시다.
image: /images/thumbnails/tabnine%20vscode.png
category: environment
---
# 들어가며
사실 처음부터 tabnine이라는 물건을 지금 개발공간으로 쓰고 있는 VPS 서버로 굴리고 있는 code-server(이하 code-server)에 설치할 생각은 그닥 없었습니다.

tabnine이라는게 있다는걸 알게 된 계기는 좀 어이없을수도 있지만, node.js 공부 하면서 이것저것 검색해 봤다가, 코드 예제를 보여줬던 사이트가 tabnine의 코드 모음집이어서 였고, AI가 코딩을 도와준다는 솔깃한 소식에 막무가내로 설치를 해보자 하고 마음을 먹었던 것이었습니다. 

하지만, 설치는 생각보다 순탄치 않았습니다. 로컬 환경에서 바로 Visual Studio Code를 돌리는 것이었으면 간단 명료 그자체인 홈페이지에 있는 설명을 보면서 설치를 마쳐버렸을 텐데, 저어기 서울에 있는 아무튼 서버, 즉 원격에서 돌아가는 환경이었고, GUI도 설치가 되어 있지 않고, 제가 모르는 부분이 한가득인 물건이라서 그런지, 설치에 난항을 겪었습니다.

간단하게 말씀드린 이 내용을 이제 최대한 생동감있게 풀어보려고 합니다.

# tabnine이 뭐고, 왜 설치했나요?
tabnine은 간단하게 말해서 좀 더 똑똑한 자동완성 입니다. 기본적으로 IDE에 내장된 자동완성은 정해진 코드 snippet, 키워드 정도를 자동완성 시켜주는데에 그치지만, tabnine은 문맥 자체를 해석해서 자동완성 제안을 띄워준다고 합니다.

아직까지 tabnine을 제대로 써보지를 못해서, 다른 분이 써놓은 tabnine 사용기를 첨부해 두도록 하겠습니다.

[코드 자동완성 플러그인 tabnine 3일 사용기](https://nookpi.tistory.com/entry/%EC%BD%94%EB%93%9C-%EC%9E%90%EB%8F%99%EC%99%84%EC%84%B1-%ED%94%8C%EB%9F%AC%EA%B7%B8%EC%9D%B8-tabnine-3%EC%9D%BC-%EC%82%AC%EC%9A%A9%EA%B8%B0)

최근에 github 에서 [github copilot](https://copilot.github.com/)이라는 특이점이 와버린듯한 미친 툴을 공개했습니다. 
![github copilot](/images/environment/github%20copilot.png)
<div style="display:flex; justify-content:center;"><small>https://copilot.github.com/ 캡쳐</small></div>
위의 이미지에서 보듯이 적당히, 유추할 수 있는 함수의 형태를 대강만 적어놓으면, 알아서 완성해 버리는 모습을 보여주어, 흔히 "땔감질" 이라고 불리는 작업을 수동으로 안하고 자동으로 하는, "땔깜의 종말"이 올것이라고 주변 친구들과 이것저것 이야기 해보았지만, 아직 얘는 `Technical Preview`라서, 정해진 인원만 사용할 수 있고, 저는 아직도 대기열에만 콕 박혀 있는것이 현실입니다. 
![copilot waiting](/images/environment/copilot%20waiting.png)
구글 검색에서 비록 자기네들이 올린 광고지만, 본 서비스보다 더 위에서 **우리는 `github copilot`의 대체품이다** 하고 당당히 외치는 모습을 보니, '한번 설치하면 좋을지도?' 라는 생각이 들었습니다.
![copilot alternative](/images/environment/copilot%20alternative.png)

그리고 가장 중요한것은, 학생 신분임을 증명하면, 할인도 아닌 무려 **무료**로 쓸 수 있다는 점이 제일 컸습니다. 돈드는것도 아닌데 안해볼 이유 없잖아요?
![tabine is free](/images/environment/tabnine%20students.png)
<div style="display:flex; justify-content:center;"><small>tabnine 홈페이지 캡처</small></div>
이러한 이유로 저는 tabnine을 제 VPS에서 돌아가는 `code-server`에 설치 해보기로 했습니다.

# 역시나 찾아온 문제들
tabnine을 설치하는 방법은 Extension 탭에서 tabnine을 검색해서 설치한 다음, vscode를 재시작(code-server의 경우에는 새로고침) 하는것이 끝입니다. 

진짜 이게 끝이에요. 근데 이게 끝이었고, 문제가 있었다면, 이 글을 적을 이유는 없었겠죠. 기본적으로 tabnine을 확장 마켓플레이스에서 설치만 하면, **Free** 버전으로 설치가 되어서 제약이 몇몇개 생깁니다. 

![tabnine prices](/images/environment/tabnine%20price.png)
<div style="display:flex; justify-content:center;"><small>tabnine 홈페이지 캡처</small></div>

Pro의 기능을 사용하려면, 일반적으로는 결제를 해야 하지만, 위에서도 간략하게 언급했듯, 학생임을 증명하면 무료로 쓸 수 있습니다. 

<https://www.tabnine.com/students>에서 본인의 학교 이메일을 입력하고, vscode 하단의 tabnine 탭을 클릭해서, 아까 입력한 학교 이메일로 가입을 진행하면 된다.... 라는 것이었는데, 제 경우에는 아래 이미지와 같은 문제가 발생했습니다.

![recjected](/images/environment/rejected127.png)

대략 느낌으로는, 127.0.0.1 즉 `localhost`에서 서버를 실행시켜서 거기에 접근하는것 같은데.... 무슨 문제가 있어서인지 접속이 되지를 않습니다. free 요금제로 계속 쓸 수는 없다고 생각해서, 저는 몇가지 문제해결책을 생각해 봤습니다.

# Browser Preview 확장 이용
vscode에는 browser preview라는 확장이 있습니다. 말 그대로 웹 브라우저로 결과물을 봐야하는 프로젝트를 작성할 때 도움을 주는 플러그인인데, vscode가 실행되는 현재 환경에서 브라우저를 같이 띄워주는 것이라서, 외부로 포트 개방을 안했어도, localhost내에서 접근은 할 수 있으리라 여겨서 한번 시도해 봤습니다. 

![iframe](/images/environment/tabnine%20iframe.png)
F12를 눌러서 개발자 도구를 실행해서 자세히 보면, tabnine hub을 실행했을때 에러를 내는 페이지가 `iframe`으로 띄워져있고, 주소를 확인할 수 있음을 볼 수 있습니다.

browser preview로 해당 주소로 접근했을때, 접속이 일단 가능은 함을 볼 수 있습니다.

![browser privew](/images/environment/tabnine%20set.png)
하지만 막상 써보면, 정상적인 사용이 불가한만큼 속도가 버벅거리기에, 이건 쓸 물건이 못됩니다. 그래서 다른 방법을 강구했습니다.

# 127.0.0.1:5555로 리다이렉트 되는 ngnix 서버 설정하기
전에 올렸던 포스팅 중에서, [vps에 code-server 설치하기](/code-server-with-vps/) 라는 글이 있었는데, 외부에서 `127.0.0.1:8080`으로 접근하기 위해서 도메인 설정 후, 외부에서 성공적으로 접근한 적이 있음을 보여드린 적이 있습니다. 이걸 여기에서도 그대로 활용해볼까 합니다.

`/etc/nginx/sites-available`과 `/etc/nginx/sites-enabled`에 `.conf`파일을 하나 만들고, 외부에서 접속해보는 방법을 써보기로 했습니다.

상기 디렉토리에 작성할 **panel.conf** 내용
```conf
    server {
    	listen 80;
    	listen [::]:80;
    	server_name <domain name>;
    	location / {
    		proxy_pass http://127.0.0.1:5555;
    		proxy_set_header Upgrade $http_upgrade;
    		proxy_set_header Connection upgrade;
    		proxy_set_header Accept-Encoding gzip;
    	}
    }
```

여기서 server_name은 본인의 서버로 접속되게할 도메인을 입력하고, proxy_pass에는 위의 내용을 그대로 복사-붙여넣기 **하지 마시고** 직접 code-server 환경에서 열린 페이지의 주소를 확인하고, 붙여넣기 바랍니다. 가끔 5555번 포트가 아닌 다른 포트가 열릴때도 있거든요.

위 내용을 `/etc/nginx/sites-available`과 `/etc/nginx/sites-enabled`에 전부 입력하고 `nginx -t`로 오타 문법 검사를 한 다음에 `systemctl restart ngnix`를 해서 nginx를 재시작 하고, `certbot`으로 https 인증서까지 발급 받읍시다.(.dev 도메인은 https를 강제함)

![panelOk](/images/environment/panelok.png)

성공적으로 접속됨을 확인할 수 있습니다. 저는 지금 로그인이 된 상태라서 이렇게 보이는 것이고, 처음 설정을 하는 분이라면, 우측 상단에 sign up/sign in 비슷한 내용의 글자가 있을 것입니다. 

여기에 아까 학생 할인 페이지에 입력한 학교 이메일로 가입을 진행하시면, 이제부터 tabnine을 사용할 준비가 끝났습니다.

# 마치며
처음에 이게 panel page가 접속이 되지 않았을때는 꽤나 당황했습니다. 홈페이지의 FAQ에는 ssh로 리다이렉트 해서 사용해라고 했었는데, 군대 환경의 특성상, 함부로 ssh 접속을 하면, 비인가 통신이라고 뭐라고 할 까봐 제대로 하지도 못하고... 어떡하지 생각을 많이 했었는데, 생각보다 간단하게 문제가 해결되서 다행인것 같습니다.

언젠가 tabnine 사용 후기 글도 올릴까 생각도 하고 있습니다. 끝까지 제 글 읽어주셔서 정말로 감사드립니다. 혹시 설정을 하다가 나는 이 글 대로 진행했는데 안된다 하는 부분이 있다면 댓글 남겨주시면 최대한 쁘르게 확인하고 도와드릴 수 있도록 하겠습니다.