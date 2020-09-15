---
published: false
layout: post
title: 반응형 웹 만들기 - 03-2. 뷰포트
excerpt: 가변 그리드 이해하기
toc: true
tags: [책, 반응형웹]
categories: 책
---
# 화면의 보이는 영역을 다루는 기술, 뷰포트
뷰포트는 아주 간단해 보이는 기술이지만 뷰포트가 없으면 반응형 웹도 없다는 말이 있을 정도로 중요한 기술 중 하나입니다. 01장에서 간단히 설명했듯이 뷰포트는 화면에서 실제 내용이 표시되는 영역으로, 데스크톱은 사용자가 설정한 해상도가 뷰포트 영역이 되고, 스마트 기기는 기본으로 설정되어 있는 값이 뷰포트 영역이 됩니다.

그런데 스마트 기기는 기본으로 설정되어 있는 뷰포트 영역으로 인해 미디어 쿼리가 정상적으로 작동하지 않는 문제가 발생할 수 있습니다. 이러한 문제를 방지하기 위해 뷰포트 메타 태그를 이용해서 화면의 크기나 배율을 조절해야합니다.

```html
<meta name="viewport" content="width=device-width. initial-scale=1, minimum-scale=1, maximum-scale=1, user-scalable=no"> 
```

## 뷰포트 속성

| 속성명 | 속성값 | 속성 설명 |
|   ------   |   ------   |  ------   |
|  width |  device-width, 양수  | 뷰포트의 너비를 지정합니다.
|  height |  device-height, 양수  | 뷰포트의 높이를 지정합니다.
|  initial-scale |  양수  | 뷰포트의 초기 배율을 지정합니다. <br/> 기본값은 1입니다. 1보다 작은 값을 사용하면 축소된 페이지를 표시하고, 1보다 큰 값을 사용하면 확대된 페이지를 표시합니다.
|  user-scalable |  yes, no  | 뷰포트의 확대/축소 여부를 지정합니다. <br/> 기본값은 yes입니다. 반대로 no로 설정함녀 사용자가 페이지를 확대할 수 없습니다.
|  minimum-scale |  양수  | 뷰포트의 최소 축소 비율을 지정합니다. <br/> 기본값은 0.25 입니다.
|  maximum-scale |  양수  | 뷰포트의 최대 확대 비율을 지정합니다. <br/> 기본값은 5.0 입니다.