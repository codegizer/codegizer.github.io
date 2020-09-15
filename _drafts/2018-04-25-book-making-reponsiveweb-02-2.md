---
published: false
layout: post
title: 반응형 웹 만들기 - 02-2. 가변 마진과 가변 패딩 이해하기
excerpt: 가변 그리드 이해하기
toc: true
tags: [책, 반응형웹]
categories: 책
---
### 가변 마진 설정하기
반응형 웹사이트에서는 모든 요소가 가변적이어야 합니다. 구조상의 간격 역시 마찬가지죠. 기존의 고정되어 있는 마진을 변할 수 있게 설정해줘야 합니다.

**가변 마진은 가변 그리드에서 사용했던 공식과 같습니다.**
 (가변 마진을 적용할 마진값 ÷ 적용할 박스를 감싸고 있는 박스의 가로너비) × 100 = 가변 마진 % 값

 ![enter image description here]({{ site.url }}/assets/article_images/2018-05-05-book-making-reponsiveweb/fluid_margin_formula.jpg)


### 가변 패딩을 적용하는 두가지 방법

**가변 패딩 적용방법1 - 기본방법**

첫 번째 방법은 가변 패딩을 적용하는 기본적인 방법입니다. 아래 예제 처럼 40px 크기의 패딩값을 가변 그리드 공식을 이용해서 가변 패딩을 적용할 수 있습니다.

(가변 패딩을 적용할 패딩값 ÷ 적용할 박스를 감싸고 있는 박스의 가로 너비) × 100 = 가변 패딩 % 값

다음 예제는 960px 크기의 박스 안에 또 다른 박스가 있고, 이 박스의 상하좌우에 40px 크기의 가변 패딩을 적용하는 예제입니다.

{% highlight html linenos %}
<head>
....
<style>
...
#wrap{
	width:90%;
	/* 960px */
	height:500px;
	margin:0 auto;
	border:4px solid #000;
	background:#f7e041;
}

#wrap p{
	padding:40px 4.16667%;
	/* 40px ÷ 960px */
}
</style>
</head>
<body>
	<div id="wrap">
		<p>...텍스트 생략 ...</p>
	</div>
</body>
{% endhighlight %}

**가변 패딩 적용방법2 - 제한적인 조건이 있을 때**

두 번째 방법은 박스에 패딩을 적용하더라도 박스의 정해진 너빗값 이상이 되지 말아야 하는 경우에 이용하는 방법입니다. 이때 이용하는 공식은 방법1과 같습니다.

다음 예제는 고정 크기일때 960px 크기의 박스 안에 하나의 박스가 있고, 이 박슨느 600px을 넘지 않음녀서 상하좌우 패딩 값을 50px로 적용하는 예제입니다.


{% highlight html linenos %}
<head>
....
<style>
...
#wrap{
	width:90%;
	/* 960px */
	height:500px;
	margin:0 auto;
	border:4px solid #000;
	background:#f7e041;
}

#wrap p{
	width:52.083333%;
	/* 500px ÷ 960px */
	padding:50px 5.20.8333%;
	/* 50px ÷ 960px */
	margin:0 auto;
	background:#f7e041;
}
</style>
</head>
<body>
	<div id="wrap">
		<p>...텍스트 생략 ...</p>
	</div>
</body>
{% endhighlight %}

**고정 크기의 마진과 패딩을 위해 calc 함수 이용하기**

가변 마진과 가변 패딩에는 소소한 문제가 있습니다. 바로 브라우저의 비율에 따라 마진과 패딩도 줄어든다는 것입니다. 박스는 가변적이되 마진이나 패딩은 고정되어 있도록 설정하고 싶다면 어떻게 해야할까요? 그럴 때는 CSS3의 calc 함수를 이용하면 됩니다.

다음 예제는 wrap이라는 아이디를 가진 박스 안에 자식 박스 한 개가 있고, 그 박스에 50px 크기의 고정 마진을 적용하기 위해 calc 함수를 이용해 총 너빗값에서 왼쪽, 오른쪽 패딩값을 더한 값인 100px을 뺀 값을 너빗값으로 설정하고, 마진값을 50px로 설정한 예제 입니다.

예제를 웹 브라우저에서 열어 브라우저 크기를 줄여가면서 확인해 보면 50px 크기의 마진은 그대로 적용되고 박스의 크기만 가변적으로 작동하는 것을 확인할 수 있습니다.

{% highlight html linenos %}
<head>
....
<style>
...
#wrap{
	width:90%;
	/* 960px */
	height:500px;
	margin:0 auto;
	border:4px solid #000;
}

#wrap div{
	/* 
	아래에서 calc를 이용해 wrap 아이디를 가진 부모 박스의 너비값(100%)에서 
	좌우 마진을 더한 값 100px을 뺸 값을 자식 박스의 너빗값으로 설정하고 있다.
	*/
	width:calc(100% - 100px);
	height:200px;
	margin:50px;
	background: #f7e041;
}
</style>
</head>
<body>
	<div id="wrap">
		<div></div>
	</div>
</body>
{% endhighlight %}