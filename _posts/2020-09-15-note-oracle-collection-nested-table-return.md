---
layout: post
title: (오라클) 테이블 형식의 컬렉션을 리턴하는 함수
date: 2020-09-14 23:00:00 +0900
categories: 개발노트
tags: [ORACEL,오라클,object,collection,nested-table]
---

돈을 계좌로 송금을 요청할 때 아래의 같은 제한이 있다고 가정해보자
> 건당 최대 이체 가능한 금액이 5천만원이다. <br/>
예를 들어, 1억의 요청에 대해 두번에 걸처 5천만씩을 분할하여 조회가 되어야 한다.

이때 다음의 테이블을 내역을 재생성하는 방법을 이용할 수 있으며 이 콜렉션을 테이블 형식을 리턴받아 조회가 가능하다.


### 1. 객체유형 및 테이블 형식 정의

```sql
--멤버를 가지는겍체타입을 선언한다.
CREATE OR REPLACE TYPE TYP_OBJ IS OBJECT (
    컬럼1         VARCHAR2(40),
    컬럼2         VARCHAR2(8),
    컬럼3         NUMBER(22,0),
);

--중첩 테이블 형식의 컬렉션을 생성한다.
create or replace TYPE TYP_TBL IS TABLE OF TYP_OBJ;
```

### 2. 함수에서 테이블형식의 컬렉션 리턴

```sql
-- 테이블형식의 컬렉션을 리턴하는 함수
CREATE OR REPLACE FUNCTION  F_GET_TBL_STUDY1
(
    IN_PARAM1  VARCHAR2
)
RETURN TYP_TBL
IS
    --생성자를 이용해 초기화를
    TYP_TBL:=TYP_OBJECT( );  

    --데이터가 입력될 컬렉션 인덱스 변수를 초기화
    N NUMBER(22,0) := 0;

BEGIN

    FOR R IN ( SELECT COL1,COL2,COL3 FROM TABLE_NAME WHERE COND1 = IN_PARAM1)
    LOOP
        
        BEGIN

            --루프가 반복될 때 마다 컬렉션의 맨끝에 NULL인 요소를 하나 추가한다.
            TYP_TBL.EXTEND;        

            N := N + 1;
    
            TYP_TBL(N) := TYP_OBJECT(
                                        R.COL1,
                                        R.COL2,
                                        R.COL3
                            );
        END;
    
    END LOOP;

    --콜렉션을 리턴한다.
    RETURN TYP_TBL;

    --OPEN RC FOR SELECT * FROM TABLE(L_REMIT_TBL);
    --dbms_sql.return_result(RC);

END;
```

### 3. 리턴된 테이블유형의 SELECT 조회

```sql
SELECT 	COL1, COL2, COL3 
FROM TABLE( 
		CAST(F_GET_TBL_STUDY1(파라미터값1) AS TYP_TBL)
) 
```