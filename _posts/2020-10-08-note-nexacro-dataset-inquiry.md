---
layout: post
title: (넥사크로) 효율적인 데이터셋 탐색을 위한  리팩토링
date: 2020-10-08 21:00:00 +0900
categories: 개발노트
tags: [넥사크로,데이터셋,탐색,알고리즘,콜백,리팩토링]
toc: true
---

일반적인 경험으로는 넥사크로 환경에서 특정 체크박스로 선택된 내역만을 별도의 데이터셋에 담아 저장 이나 삭제 등의 트랜잭션 요청을 하는 일이 빈번했다. 그리고 이 체크박스가 선택된 것만을 골라내기 위해서 FOR문으로 반복하고 체크박스가 체크되었는지의 대부분의 로직에 공통적으로 적용되고 있다는 것을 깨달았다.

그래서 특정 조건에 대한 탐색을 수행하고 해당 ROW의 처리를 콜백함수를 요청하도록 하여 코드를 효율적으로 리팩토리할 수 있었다.

아래는 그 예시를 담은 코드이다.

```JS
//체크박스가 선택된 조건의 목록을 찾아 콜백에 처리를 위임한다.
function loopByChk(Dataset dataset, Function callback)
{
    var nRow = -1;

    while( nRow = ds.findRowExpr("CHK==1", nRow) >= 0)
    {
            calback(dataset, nRow);
    }
}
```
위의 코드에서는 데이터셋의 findRowExpr 함수를 이용해 특정 위치부터 조건에 해당하는 ROW 정보를 사용자가 지정한 콜백함수에 처리를 위임한다.

![findRowExpr](/assets/article_images/2020-10-08-note-nexacro-dataset-inquiry/nexacro-reference-dataset-findRowExpr.png)



```JS

//목록에서 체크박스가 선택 된 것의 순번을 배열에 담는다.
function fnSave()
{
	var arrTarget = [];

	loopByChk(dataset, function(dataset, nRow) {
		arrTarget.push(nRow);
	});
        
    trace(arrTarget);
}
```