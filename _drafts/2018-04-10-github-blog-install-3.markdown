---
layout: post
title:  "깃헙 블로그 구축하기 - 블로그 소스 게시하기"
series: 3/4
excerpt: npm와 gulp-gh-pages을 이용해 블로그 페이지를 빌드하고 사이트 소스를 깃헙 블로그에 업로드하는 법을 배웁니다.
date:   2018-04-10 23:15:00 +0900
categories: jekyll
---
소스는 별도의 브랜치를 만들어 커밋합니다.

\# source라는 브랜치라는 생성하고 해당 브랜치로 변경합니다.

git checkout -b "source" 

\# 이미 만든 브랜치로 변경하려면

git checkout source

\# 변경된 파일들을 스테이지에 추가합니다. 

git add *

\# 커밋 메시지를 작성하고 커밋합니다.

git commit -m "커밋메시지"

\# git 원격 저장소에 푸시합니다.

git push origin source

npm(패지지매니저)을 이용해 지킬로 작성한 사이트의 빌드 과정에서 생성된 빌드 결과물을 작업 디렉토리에 해당하는 git 리모트 깃헙에 업로드 합니다. 이때 브랜치는 gulpfile.js에서 gulp-gh-pages의 옵션인 'branch'에 지정한 브랜치로 커밋됩니다.

npm run deploy