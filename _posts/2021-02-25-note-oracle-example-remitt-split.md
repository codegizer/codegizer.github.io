---
layout: post
title: (오라클) 건당 이체한도에 따른 송금분할 예제
date: 2021-02-25 21:45:00 +0900
categories: 개발노트
tags: [오라클,예제,송금분할,테이블,콜렉션]
toc: true
---
이전에 테이블 유형의 컬렉션을 다루는 방법와 콤마로 이루어진 문자열을 행으로 변환하는 내용을 정리하여 공유했었다. 이번에는 그 최종 결정체인 건당 이체한도에 따라서 송금액을 분할하는 예제를 다루보겠다.


## 1. 우선 예제에 사용될 DB테이블와 데이터를 아래와 같이 생성해 보자.
```SQL
CREATE TABLE REMIT_REQUST (
	MGRT_NO		NUMBER(10,0),    
	DPSTR_NM 	VARCHAR2(50),
	BANK_NM		VARCHAR2(50),
	ACCNO		VARCHAR2(50),
	REQUST_AMT	NUMBER(10,0)
);

INSERT INTO REMIT_REQUST (MGRT_NO, DPSTR_NM, BANK_NM, ACCNO, REQUST_AMT) VALUES ( 1, '예금주1','한국은행1','111-111-111',5000000);
INSERT INTO REMIT_REQUST (MGRT_NO, DPSTR_NM, BANK_NM, ACCNO, REQUST_AMT) VALUES ( 2, '예금주2','한국은행2','222-222-222',2500000);
INSERT INTO REMIT_REQUST (MGRT_NO, DPSTR_NM, BANK_NM, ACCNO, REQUST_AMT) VALUES ( 3, '예금주3','한국은행3','333-333-333',2500000);
INSERT INTO REMIT_REQUST (MGRT_NO, DPSTR_NM, BANK_NM, ACCNO, REQUST_AMT) VALUES ( 4, '예금주4','한국은행4','444-444-444',15000000);
```

## 2. 객체유형와 테이블 컬렉션 생성

```SQL
--객체유형 생성
create or replace TYPE REMIT_OBJ IS OBJECT (
	MGRT_NO		NUMBER(10,0),    
	DPSTR_NM 	VARCHAR2(50),
	BANK_NM		VARCHAR2(50),
	ACCNO		VARCHAR2(50),
	REQUST_AMT	NUMBER(10,0)
);
```

```SQL
-- 테이블 컬렉션 생성
create or replace TYPE REMIT_TBL IS TABLE OF REMIT_OBJ;
```

## 3. 이체한도에 따라서 송금액을 분할하여 컬렉션을 리턴하는 함수 생성

```SQL
CREATE OR REPLACE FUNCTION   F_REMIT_TBL_RET
(
	IN_MGRT_KEYS VARCHAR2
)
RETURN REMIT_TBL
IS
	L_REMIT_TBL REMIT_TBL:=REMIT_TBL();
	V_REMIT_DIVID_CNT NUMBER(22,0);
	V_REMIT_DIVID_AMT NUMBER(22,0);
	V_REMIT_SUMT_AMT NUMBER(22,0);
	V_REMIT_DIVID_MOD NUMBER(22,0);
	V_SET_DIVID_AMT NUMBER(22,0) := 5000000;	--이체한도
	V_N NUMBER(22,0) := 0;
	V_PRSS_CNT NUMBER(22,0) :=0;
	rc sys_refcursor;
BEGIN

--이체처리를 할 내역의 관리번호를 콤마로 결합된 문자열을 전달하면
--행으로 변환하여 IN 절을 이용하여 송금요청 테이블을 조회한다.
FOR R IN (
    SELECT  AA.MGRT_NO
			, AA.DPSTR_NM
            , AA.BANK_NM
            , AA.ACCNO
            , AA.REQUST_AMT
      FROM REMIT_REQUST AA
INNER JOIN (

            SELECT distinct regexp_substr(IN_MGRT_KEYS, '[^\\\\\\\\,]+', 1, LEVEL) MGRT_NO
                FROM DUAL
            CONNECT BY LEVEL <= length(regexp_replace(IN_MGRT_KEYS, '[^\\\\\\\\,]+',''))+1                         
      ) TBL_MGRT
      ON AA.MGRT_NO = TBL_MGRT.MGRT_NO	

)

--이체처리 요청내역을 커서를 이용해 루프를 돌린다.
LOOP

	BEGIN
	
	  --송금액이 이체한도를 초과 하는 경우
	  IF R.REQUST_AMT > V_SET_DIVID_AMT THEN
	
	    V_REMIT_DIVID_MOD := R.REQUST_AMT MOD V_SET_DIVID_AMT;

		--이체한도에 따른 분할되는 이체 건수
	    V_REMIT_DIVID_CNT := CEIL(R.REQUST_AMT / V_SET_DIVID_AMT);

		--처리건수 카운트 초기화
	    V_PRSS_CNT := 0;
	
	    LOOP
			--컬렉션을 확장한다.	    
	    	L_REMIT_TBL.EXTEND;

			--컬렉션 확장에 따른 인덱스 증가
	    	V_N := V_N+1;

			-- 처리건수 카운트 '1' 증가
	    	V_PRSS_CNT := V_PRSS_CNT + 1;				

			-- 마지막 분할 건
	    	IF V_PRSS_CNT = V_REMIT_DIVID_CNT THEN
				
				--이체한도로 나누었을 때 나머지가 0인경우(분할 금액이 모두 동일한경우)
				IF V_REMIT_DIVID_MOD = 0 THEN
					-- 마지막 불한 송금액은 이체한도액이 된다.
					V_REMIT_DIVID_AMT := V_SET_DIVID_AMT;
				ELSE
					--나머지가 0으로 나누어 떨어지지 않는 경우
					--나머지 금액을 마지막 송금액으로 한다.
			        V_REMIT_DIVID_AMT := V_REMIT_DIVID_MOD;
				END IF;

	    	ELSE

				--송금액을 이체한도 만큼으로 분할한다.
	    		V_REMIT_DIVID_AMT := V_SET_DIVID_AMT;

	    	END IF;

			-- 테이블 컬렉션에 분할된 금액을 담는다.
	      	L_REMIT_TBL(V_N) := REMIT_OBJ (
				R.MGRT_NO,
	        	R.DPSTR_NM,
	        	R.BANK_NM,
	        	R.ACCNO,
	        	V_REMIT_DIVID_AMT
	      	);
	
			--송금분할이 완료되면 루프를 종료한다.
	    	EXIT WHEN V_PRSS_CNT = V_REMIT_DIVID_CNT;
	
	    END LOOP;
	
	  ELSE
	    
		--송금액이 이체한도를 초과하지 않는 경우
		--해당 이체내역을 테이블 컬렉션에 그대로 저장한다.
	    L_REMIT_TBL.EXTEND;
	              
	    V_N := V_N+1;
	
	    L_REMIT_TBL(V_N) := REMIT_OBJ ( 
			R.MGRT_NO,
	      	R.DPSTR_NM,
	    	R.BANK_NM,
	    	R.ACCNO,
	    	R.REQUST_AMT
	    ); 
	
	  END IF;
	  
	END;

END LOOP;

RETURN L_REMIT_TBL;

END;
```

## 4. 테이블 컬렉션 조회

```sql
SELECT MGRT_NO
	 , DPSTR_NM	
	 , ACCNO
	 , BANK_NM
	 , REQUST_AMT
FROM (
		SELECT AA.MGRT_NO
			 , AA.DPSTR_NM      --예금주
			 , AA.BANK_NM       --은행명
			 , AA.ACCNO         --계좌번호
			 , AA.REQUST_AMT    --금액
		FROM TABLE( 
			CAST(F_REMIT_TBL_RET('1,2,3,4') AS REMIT_TBL)
		) AA
)
```

## 5. 조회결과
![이체한도에 송금분할 결과 조회](/assets/article_images/2021-02-25-note-oracle-example-remitt-split/oracle-example-remitt-split.png)


## 끝!