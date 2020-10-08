---
layout: post
title: (오라클) 구분자를 사용한 문자열을 행으로 변환하기
date: 2020-10-07 23:00:00 +0900
categories: 개발노트
tags: [오라클,문자열구분,CONNECT_BY,행변환]
toc: true
---
구분자로 이루어진 문자열을 행으로 출력하는 SQL을 분석해보려 한다. 우선 완성된 쿼리는 아래와 같으며 하나하나 뜯어보며 알아가 보자.

```SQL
SELECT LEVEL, REGEXP_SUBSTR(A.TXT, '[^\,]+', 1, LEVEL) MGRT_NO
 FROM (
        SELECT 'A1,A2,A3,A4,A5' AS TXT FROM DUAL
  ) A
CONNECT BY LEVEL <= LENGTH(REGEXP_REPLACE(A.TXT, '[^\,]+',''))+1 
```

![실행결과](/assets/article_images/2020-10-07-note-oralce-split-text-convert-to-row/note-main.png)

<br/>

# 2. 뜯어보기

## 2.1. CONNECT BY

CONNECT BY 절을 계층적 쿼리라고 부른다. 
일반적으로 이 절을 이용하여 상위계층와 하위계층의 관계를 행으로 표현하기 위해 사용된다.

그외에도 다음 같이 조건이 충족하는 동안 원하는 만큼 행을 출력하는 것도 가능하며, <U>지금 살펴보는 쿼리도 이를 이용한다.</U>

"CONNECT BY [:CONDITION:]"

**[SQL]**

```SQL
 SELECT LEVEL FROM DUAL CONNECT BY LEVEL <= 4
```

**[결과]**

![CONNECT BY LEVEL 결과](/assets/article_images/2020-10-07-note-oralce-split-text-convert-to-row/note-result-connect-by.png)

## 2.2. regexp_replace('A1,A2,A3,A4,A5', '[^\,]+','')

**[SQL]**
```SQL
SELECT regexp_replace('A1,A2,A3,A4,A5', '[^\,]+','') FROM DUAL
```

**[결과1]**

![REGEXP_REPLCE 실행결과](/assets/article_images/2020-10-07-note-oralce-split-text-convert-to-row/note-result-regexp_replace.png)


콤마를 제외한 모든 문자를 빈문자로 치환한다.


**[결과2]**

![REGEXP_REPLCE 실행결과](/assets/article_images/2020-10-07-note-oralce-split-text-convert-to-row/note-result-regexp_replace-length.png)

4개의 콤마로 문자열을 구분하고 있다.

## 2.3. regexp_substr('A1,A2,A3,A4,A5', '[^\,]+', 1, 5)


### 파라미터 ###

- 문자소스
- 정규식패턴
- N번째 발생한 패턴의 구분값에서 검색을 시작할 위치
- N번쨰 발생한 패턴의 값

### 설명 ###
- 위 함수에서는 5번째 발생한 패턴의 문자 값의 1번째 위치부터 표현한다.
결과는 다음과 같다.

**[SQL]**

```SQL
SELECT regexp_substr('A1,A2,A3,A4,A5', '[^\,]+', 1, 5) FROM DUAL
```

**[결과]**

![REGEXP_SUBSTR 실행결과](/assets/article_images/2020-10-07-note-oralce-split-text-convert-to-row/note-result-regexp_substr.png)

5번째 발생한 패턴의 문자 값의 1번째 위치부터 표현한다.

<br/>           

# 3. 종합하여 분석해보자

**[SQL]**

```SQL
SELECT LEVEL, REGEXP_SUBSTR(A.TXT, '[^\,]+', 1, LEVEL) MGRT_NO
 FROM (
        SELECT 'A1,A2,A3,A4,A5' AS TXT FROM DUAL
  ) A
CONNECT BY LEVEL <= LENGTH(REGEXP_REPLACE(A.TXT, '[^\,]+',''))+1    
```

- 이제 이걸 해석하자면 CONNECT BY의 조건절에 있는 충족하는 동안 LEVEL값은 1씩 증가하게 될 것 이고 구분자수(4개)+1 하여 총 5번 행이 반복 할 것이다.

- 그리고 컬럼절에 있는 부분의 REGEXP_SUBSTR(A.TXT, '[^\,]+', 1, LEVEL) 에서 LEVEL이 증가함에 따라 
증가된 값에 해당하는 N번째 패턴이 발생한 값을 찾아 행으로 출력한다.

![실행결과](/assets/article_images/2020-10-07-note-oralce-split-text-convert-to-row/note-main.png)

  

