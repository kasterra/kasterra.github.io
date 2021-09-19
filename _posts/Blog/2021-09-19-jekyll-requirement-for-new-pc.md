---
title: 막 새로 설정한 컴퓨터에 jekyll 블로깅 환경 준비물 설치
tags: Jekyll Ubuntu
layout: post
subtitle: 방금 os만 설치한 따끈한 환경에 필요한 jekyll 준비물
image: /images/thumbnails/jekyll.png
category: blog
---

# 개요

개발 공부를 하다보면, 기존의 환경이 아닌 다른 환경에서 작업할 떄가 없을 수가 없습니다. 리눅스 환경이 필요해서 리눅스 듀얼부팅을 구성할 수도 있고, 듀얼부팅이 귀찮다면, wsl을 이용하시는 분들도 있을 것입니다. 그 환경에서 jekyll 설정을 다시 하다보면, 뭘 설치를 다시 해야할지 기억도 잘 안나고 해서, 구글 검색을 계속 했던 것 같습니다. 지금부터, 환경설정을 위해서 어떤 패키지를 설치해야 하는지 간단하게 정리해보도록 하겠습니다.

# bundler 설치 및 사용

node.js 기반 프로젝트가 npm으로 패키지를 관리하듯. 루비 기반인 jekyll은 bundler라는 친구가 gem이라고 불리는 패키지들을 관리해 줍니다.

```bash
sudo apt install bundler
```

npm으로 패키지를 관리하는 프로젝트에서 `package.json`이 있으면 설치할 패키지들을 알 수 있고, `package-lock.json`이 있으면, 사용했던 패키지 버전까지 정확히 알 수 있어서, `npm install`을 하면, 예전에 설정해놓은 패키지가 깔림을 알고 있을 것입니다. bundler 또한 그러하지만, 몇몇 선생조건이 필요합니다.

개발에 필요한 기본 라이브러리

```bash
sudo apt-get install build-essential
```

jekyll 공식 홈페이지에서 나온 [요구사항](https://jekyllrb.com/docs/installation/) 충족을 위한 준비물 설치

```bash
sudo apt install ruby-dev make gcc
```

이걸 설치하지 않으면 아래와 같은 오류를 보게 됩니다.

```bash
kasterra@kasterra-ZenBook-UX482EA-UX482EA:~/Project/kasterra.github.io$ sudo gem install jekyll
Building native extensions. This could take a while...
ERROR:  Error installing jekyll:
        ERROR: Failed to build gem native extension.

    current directory: /var/lib/gems/2.7.0/gems/ffi-1.15.4/ext/ffi_c
/usr/bin/ruby2.7 -I /usr/lib/ruby/vendor_ruby -r ./siteconf20210919-25528-zkifbx.rb extconf.rb
checking for ffi.h... *** extconf.rb failed ***
Could not create Makefile due to some reason, probably lack of necessary
libraries and/or headers.  Check the mkmf.log file for more details.  You may
need configuration options.
        --with-opt-dir
Provided configuration options:

```

이처럼 필요한 요구사항이 설치가 되지 않았을 떄에는 `Could not create Makefile due to some reason, probably lack of necessary libraries and/or headers.` 라는 말을 보게 되면서, 정상적으로 gem 설치가 되지 않음을 확인할 수 있습니다.

# 플러스 알파 : git 설정
저는 jekyll 블로그를 github pages를 통해서 호스팅 하기 떄문에, 글을 작성하고 github에 커밋을 날립니다. 가끔씩 가물가물한 git 사용자 설정 하는법도 정리해 둘까 합니다.

```bash
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```
물론 `user.name`과 `user.email`뒤에 있는 값은 본인의 이름과 메일로 설정하면 됩니다. 