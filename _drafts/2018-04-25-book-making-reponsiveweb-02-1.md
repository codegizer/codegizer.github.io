---
published: false
layout: post
title: 반응형 웹 만들기 - 02-1. 가변 그리드
excerpt: 가변 그리드 이해하기
toc: true
tags: [책, 반응형웹]
categories: 책
---
### 가변 그리드 공식 이해하기

앞에서 간단히 % 값을 이용해 가변 크기의 박스를 만들어 보았습니다.  하지만 사실상 가변 그리드라는 기술은 정해져 있는 공식에 의해 정확학 가변 크기의 박스를 만드는 기술입니다.

**공식은 다음과 같습니다.**

(가변 크기로 만들 박스의 가로 너비 ÷ 가변 크기로 만들 박스를 감싸고 있는 박스의 가로 너비) × 100 = 가변 크기의 % 값

 ![enter image description here]({{ site.url }}/assets/article_images/2018-05-05-book-making-reponsiveweb/fluid_grid_formula.jpg)
