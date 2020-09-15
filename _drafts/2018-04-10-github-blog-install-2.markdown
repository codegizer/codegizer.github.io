---
layout: post
title:  "깃헙 블로그 구축하기 - 로컬에 블로그 게시환경 구축하기"
series: 2/3
excerpt: 깃헙 블로그에 컨텐츠를 게시하기 위해 로컬 환경을 구축하는 법을 배웁니다.
date:   2018-04-10 23:15:00 +0900
categories: jekyll
---
# Thanks
hurderella님의 티스토리 블로그의 내용이 가장 많이 도움이 되었습니다. 그리고 이 글에는 hurderella님의 글에 있는 이미지를 포함하고 있습니다. 조금 다르다면 저는 지킬 3.5를 본의 아니게 사용하게 됨으로써 hurderella님의 문서 외에 발생한 문제(Deprecation)를 다루고 있습니다.

# 설치환경
OS: 윈도우10 Pro 64bit<br/>
CPU: x64

# 참고
http://hurderella.tistory.com/131<br/>
http://jekyllrb-ko.github.io/docs/installation/

# 준비물
- Ruby
(저는 루비는 ruby 2.4.1-2 (x64)를 설치했습니다. 지킬 공식 사이트의 설치문서를 보면 지킬에 따른 루비 버전은 Jekyll 2 사용 시 v1.9.3 이상, Jekyll 3 사용 시 v2 이상의 개발 패키지 포함이라고 되어 있습니다.)

- GitHub 클라이언트 프로그램
(http://desktop.github.com)

- Visual Studio Code 
(개인적으로 선호하는..사이트 컨텐츠 작업을 위해 사용하며 Git하고도 연동이 된다 그리고 마크다운 문법 및 미리보기가 제공된다. 자세한 내용은 구글링!)


# Step by Step
1. **루비를 설치한다.**
2. **GitHub 클라이언트 프로그램을 설치한다.**<br/>리포지토리를 Clone할 때만 쓰고 사이트 생성후 컨텐츠의 변경은 개인적으로 MS의 Visual Studio Code에 Git을 연동하여 변경하였다. 참고로 Visual Studio Code에서 마크다운도 지원 해준다.

4. **리포지토리 복제**<br/>
```
mkdir GitHub
cd GitHub
GitHub> git clone git://github.com/계정명/계정명.github.io.git
```
위와 같이 클론하게 되면 현재 GitHub에 '계정명.github.io' 폴더가 생긴다.

5.**원격저장소 연결하기**<br/>
GitHub> git remote add origin 계정명.github.io

6. **Jekyll를 설치하자**<br/>
Jekyll 설치를 위해 명령 실행을 위해 윈도우 메뉴에서 Start Command Prompt with Ruby를 실행하고 (그냥 윈도우 명령창에서도 되더라는..) 다음의 명령을 실합니다.(명령 실행시 경로는 상관없음)
```
gem install jekyll
``` 
Jekyll은 정적 사이트 생성툴 입니다. 이를 로컬에 설치하면 변경사항이 생겼을 때 느린 GitHub에 커밋해서 확인하지 않고 변경사항을 로컬에서 바로 확인할 수 있습니다.
![Jekyll설치]({{ site.url }}/assets/article_images/2017-07-15-install-guide/jeky_ruby_install_18.PNG)


7. **서버구동하기**<br/>
```
GitHub\계정명.github.io> jekyll serve
```
이때 만약 아래와 같은 내용의 빨간줄이 나올 때는 쫄지 말자.. 에러가 아니고 'gems' 설정 옵션명이 'plugins'로 변경되었다고 공지하는 내용입니다.
![변경사항]({{ site.url }}/assets/article_images/2017-07-15-install-guide/jekyll_serv_deprecation_gems.png)<br/>
이러한 문구는 **Jekyll의 버전이 3.5.0 일때 생긴다**(구글링에 의하면) 해결방법은 **_config.yml**에 보면 **gems:** 부분을 **plugins:** 로 변경해주면 된다.

8. **의존성을 갖는 패키지 설치**<br/>
```
gem install bundler 
gem install minima 
gem install jekyll-feed 
gem install tzinfo-data
```
하나라도 설치되어 있지 않은 경우 아래의 에러 로그를 만나게 됩니다.
<br/><br/>
**bundler가 없을때**
![bundler가 없을 때]({{ site.url }}/assets/article_images/2017-07-15-install-guide/jeky_ruby_install_10.PNG)<br/>
**minima가 없을때**
![minima가 없을 때]({{ site.url }}/assets/article_images/2017-07-15-install-guide/jeky_ruby_install_12.PNG)<br/>
**jekyll-feed가 없을때**
![jekyll-feed가 없을때]({{ site.url }}/assets/article_images/2017-07-15-install-guide/jeky_ruby_install_13.PNG)<br/>
**....이하 생략(위와 비슷한 유형의 에러가 발생시 해당 패키지를 설치하자)...**


9. **생성된 사이트 확인하기**<br/>
Jekyll이 내장하고 있는 서버를 구동시켜 브라우저로 확인해보자.
이때 서버의 주소는 http://localhost:4000이다. (4000이 기본 포트)
![정상적인 지킬 동작]({{ site.url }}/assets/article_images/2017-07-15-install-guide/jeky_ruby_install_16.PNG)<br/>