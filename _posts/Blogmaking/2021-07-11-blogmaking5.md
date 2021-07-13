---
title: 블로그 옮기기 ⑤:기타 블로그 관리
tags: 기타환경 블로그 Jekyll TeXt
layout: post
excerpt: 블로그를 관리할 때 필요한 여러 요소들을 익혀봅시다.
image: /images/thumbnails/blogmove5.png
series: 블로그 만들기
comment: true
category: BLOG
---
# 이번 포스트의 목표
지금까지의 과정으로, 블로그의 형태를 모두 만들어내는것은 끝이 났습니다만, 블로그 관리에 관해서 알면 좋을 여러가지를 마저 소개하는 것으로 이 시리즈를 마칠까 합니다. 사실 이 블로그를 만들기 전에 만든 체크리스트라서, 실제 블로그를 만들면서 사용을 하지 않는것들도 있지만, 그래도 알아서 나쁠 것 없는것들이기에 간략하게라도 소개하고 넘기겠습니다.

사실 여기서 다룰 주제들은 예전에는 해야지 하고 생각은 했었는데, 막상 해보니 안하는게 더 낫겠다라는 결론이 들었거나, 성공적인 결과를 거두지 못해서 많은 소개를 못하는 것들이라서, 내용이 좀 부실합니다...
<hr>
<h3>현재 진행상황</h3>
- [x] Github 블로그 첫걸음
  - [x] 화면을 띄우기(`본인계정ID`.github.io 접속 확인)
  - [x] `Jekyll` 기능 및 `TeXt` 자체 기능 탐방
- [x] 블로그 포스트 내용이 의도한대로 나오게 하기
    - [x] `Katex`를 활용한 수식 입력 기능
    - [x] 잘려서 나오는 목차 목록, 안잘리게 하기
    - [x] 코드 강조기능 커스텀하기
    - [x] `github-pages` gem 설치해서, 빌드환경과 테스트 환경 같게 만들기
- [x] 그냥 글 목록만 나오는 메인 화면, 좀더 예쁘게 만들기
    - [x] 블로그 대문이 예쁘게 나오게 만들기
    - [x] 포스트 커버 이미지가 나오게 만들기
    - [x] 내 블로그 로고 만들어서 장식하기
- [x] 원래 테마를 넘어서 : 새로운 기능들 추가 
  - [x] 일기장에서 블로그로. 댓글 기능 달기
  - [x] 관련 게시글 기능 만들기
  - [x] 시리즈 기능 추가
- [ ] 블로그 관리 기타
  - [ ] `Github action`을 통한 배포 자동화 
  - [ ] `Google Search Console` 사용하기
  - [ ] 광고 달기
  - [ ] 개인 도메인 연결하기

<div style="display:flex; justify-content:center"><strong>진행률</strong><br></div>
<div style="display:flex; flex-direction:column; align-items:center;"><progress value="12" max="16" style="width:80%"></progress><p>12/16</p></div>
<hr>

# `Github action`을 통한 배포 자동화
지금 블로그를 호스팅하기 위해서는 그저 커밋하고 github에 푸쉬하는것으로 끝났습니다만, [github pages에서 허용하는 플러그인](https://pages.github.com/versions/)이 아닌, **다른 플러그인을 사용**한다면 이야기가 달라집니다. github pages는 빌드되지 않은 jekyll 페이지가 있다면, 빌드를 해서 github pages로 서비스하고, 빌드가 이미 되있다면, 즉, HTML/CSS/JS 등의 바로 렌더링 할 수 있는 형태라면 그걸 github pages에 올려주는게, github의 역할입니다. github pages에서 공식적으로 제공하지 않지만 되게 유용한 플러그인인  [`jekyll-paginate-v2`](https://github.com/sverrirs/jekyll-paginate-v2)등을 사용하려면 빌드가 완료된 결과물을 github에 제출해줘야 정상적으로 플러그인이 적용된 블로그를 볼 수 있습니다.

`jekyll build`를 해서 제출해도 되고, 이런 수요가 높으니까, 빌드를 해서 `gh-pages`라는 브랜치를 만들어서 커밋까지 날려주는 [`gh-pages`라는 npm 패키지](https://www.npmjs.com/package/gh-pages)도 존재합니다.

로컬에서 `gh-pages -d build`를 하면 되긴 하지만, 매번 블로그 글을 갱신할때마다 이걸 쳐야한다면 너무나 귀찮을 것입니다. 그냥, 정해진 플러그인만 쓸 때 처럼 github에 커밋해버리면 알아서 해줬으면 좋겠다 라는 귀찮음이 생길것인데, 그 귀찮음을 해결해줄 도구가 `Github actions`입니다. 

[gh-pages를 github action에서 적용하는 방법을 알린 글](https://davidyang2149.dev/front-end/github-actions%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-gh-pages-%EC%9E%90%EB%8F%99-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0/)이 있어서, 링크 남겨둡니다.

현재 시점에서 제가 운영하는 `TeXt`테마로 만들어진 이 블로그는 이걸 쓸 일이 없어서, 추후에 쓰게 된다면, 제 케이스에 맞게 글을 올릴까 싶습니다.

# `Google Search Console` 사용하기
github pages로 블로그를 호스팅하면, 무료로 양질의 블로그를 호스팅 할 수 있다는 말은 사실이지만, 사실 구글 검색등에 노출되는것은 별개의 일입니다. [`Google Search Console`](https://search.google.com/search-console/about)에 접속해 본인의 사이트라는것을 증명하고, 사이트맵을 제출하면, **약 일주일 뒤**부터 사이트에 대한 정보를 반영해서 구글 검색결과에 반영될 것입니다. 

적어도 제 경우는 그랬습니다.
<div style="display:flex; justify-item:center"><img src="/images/blogmaking/sitemapCrawl.png"></div>

강조표시한 부분을 보면 알 수 있듯, 사이트맵을 읽는데 6,7일 걸렸고, 그리고 사이트맵이 읽어졌다고 해서 사이트맵에 명시된 모든 사이트가 짠 하고 검색 결과에 나오지도 않습니다. 사이트맵이 제출된지 일주일이 넘은 지금 글쓰는 시점에도 해당되는 말이란게 정말 코미디 입니다.
<div style="display:flex; justify-item:center"><img src="/images/blogmaking/urlCrawl.png"></div>

가장 하고싶은 말은 **인내심을 가져야 한다** 입니다. 정말로요.

그래도 이게 제대로 여러분의 사이트를 올바르게 긁어온다면, 검색어 유입이 어느쪽에서 많이 되는지에 도움이 되고, 후술한 광고를 붙이는데에도 도움을 준다는 카더라가 있습니다.

# 광고 달기
블로그를 운영하면서, 수익도 생겼으면 좋겠다는 욕심이 들 수도 있습니다. 저도 그런 욕심이 들어서, 제 블로그에 광고를 달아서, 한달에 커피 한잔이라도 나왔으면 좋겠다라는 생각이 들어서, Google Adsense를 신청했습니다만, 잘 안되더군요....
<div style="display:flex; justify-item:center"><img src="/images/blogmaking/adsenseFail.png"></div>

광고를 다는데 성공적이었으면, 유익한 정보를 실을 수 있었을 수도 있겠지만, 별 수익이 없었습니다.

이친구도 여러분의 사이트의 컨텐츠를 심사하는데 **약 일주일**정도 걸립니다. 쉽지 않군요...

# 개인 도메인 연결
`github.io`가 아닌, 나만의 도메인을 연결해서 쓰고 싶을 수도 있겠죠. 저는, github blog를 운영한다는 사실을 확실히 하고 싶어서, 제 개인 도메인 연결을 하지 않았지만, 다른 사유로 블로그에 개인 도메인을 연결하기를 희망하는 경우가 없는것은 아니기에, 여기에 공식 github pages docs [링크](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages)를 남기고 이 글을 마무리 짓겠습니다.

# 마치며
제 블로그를 만든 과정을 기록으로써 남겨보자고 생각해서 시작한 블로그 포스팅이, 마지막 챕터는 현재 제 상태도 그렇게 양호하지 않아(...) 용두사미처럼 끝난 감이 없지는 않지만, 추후에 블로그 테마를 [Jasper2](https://github.com/jekyllt/jasper2)로 바꾸면서 여러 작업을 할 것으로 예상됩니다. 테마 특성상, 여러개로 나눠져있는 테마들을 하나의 대표테마로 압축한다던지, 또 테마의 기본옵션 몇개가 마음에 안들어서 교체할 수도 있고, 할 일들이죠.... 어쨌거나, 이 시리즈의 완결을 낼 수 있어서 다행이라고 생각합니다. 끝까지 읽어주셔서 감사합니다. 혹시나라도 읽다가 질문이나 남기고 싶은 말이 있으시다면, 주저없이 댓글 남겨주세요.