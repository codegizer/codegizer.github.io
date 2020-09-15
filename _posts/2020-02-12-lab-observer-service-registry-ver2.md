---
layout: post
title: 옵저버 서비스 레지스트리 구현
date:   2020-04-21 23:00:00 +0900
categories: 실험실
tags: [design-pattern,observer-pattern]
---

 ```연계처리 구현 적용사례에 대해 알아보기 전에 먼저 옵저버 패턴에 대해 간략하게 설명하자면 아래와 같다.```
 
 
## 옵저버 패턴이란..?

1. 한 객체의 상태가 바뀌면 그 객체에 의존하는 다른 객체들에게 그 정보를 알려주는 디자인 패턴으로서 발행/구독 모델이라고도 한다. 

2. 옵저버 패턴은 주제(Subject)라 불리는 이벤트를 발생시키는 관찰대상 객체 하나와 이벤트를 구독하는 N개 이상의 구독자(Observer)로 이루어진다.

![옵저버패턴 클래스 다이어그램]({{ site.baseurl }}/assets/article_images/2020-02-12-lab-observer-service-registry-ver2/design_pattern_observer_uml.png)
 
## 옵저버 패턴의 장점!!

1. 주제와 구독자의 관계를 느슨하게 유지할 수있다.
2. 주제는 구독자가 이벤트에 대해 어떤 처리를 하는지 알지 않아도 된다.
3. 구독자가 추가되어도 주제를 변경하지 않아도 된다.

> 이런한 장점은 다음의 구현사례에서도 드러났다.

<br/>
 

## 어쩌다 도입을 하게되었을까..?

1. 자동 결의서 생성 기능을 통해 여러 업무에서 결의서를 생성할 수 있도록 했다. 이때, 개발자들이 해당 결의서의 결재처리나 삭제시의 이벤트를 각 업무에서 부가적인 처리를 하고 싶어했다.

2. 이벤트를 처리하기 위해서 결의서 처리로직에 각 업무에 대한 비즈니스 로직을 끼어 넣고자 하는 지속적인 요구가 있었고, 그에 대응하여 나의 코드를 지켜내어야만 했다.

<br/>

## 개발자들의 요구를 그대로 수용했다면..?

1. 결의서 처리로직은 다른 업무로직와의 결합도가 기하급수적으로 높아지고 관리 범위가 넓어지게 된다. 
2. 자칫 개발자들의 부주의로 인한 간헐적인 실수가 코드에 결함을 일으키게 되어 신뢰할 수 없게 되고, 결의서와 연관된 연계업무들의 소스코드가 뒤엉켜 각 연계업무들의 기능을 보장할 수 없게된다.
3. 다수의 개발자가 하나의 결의서 소스코드에서 작업할 때 동기화가 쉽지 않고 작업간의 충돌이 발생할 수 있다.
4. 추후 유지보수가 어렵게 되고 리팩토링이 불가능할 수 있다.

<br/>

## 그래서!!!
연계업무의 결의서 후처리 로직와 결의서 로직을 분리하여 각자의 코드에 대해서만 책임을 가지고 개발을 할 수 있도록, 스프링 기반의 몇가지 제약사항을 고려하여 프로토타입을 구현하게 되었다.

1. 결의서의 저장/삭제/승인취소 등의 이벤트를 타연계업무에 전달할 수 있는 방법이 필요했다. Redis의 발행(Publisher))/구독(Subscribe) 개념와 MSA(마이크크로서비스아키텍처)에서 서비스의 위치를 보관하고 검색할 수 있는 서비스레지스트리 개념을 이용하고자 했다.

2. 그러다 문득 옵저버 패턴이 스쳐 지나가길래 검색을 해봤는데, 옵저버패턴이 이 두 개념을 모두 충족해주고 있었다.

3. 우선 서비스레지스트리에 옵저버들의 인스턴스를 등록할 수 있어야 했다. 처음에는 리플렉션을 기반으로 인스턴스를 동적 생성하여 서비스레지스트리에 등록하려했지만 이 경우 스프링 기반으로 작성된 비즈니스 로직들이 올바르게 동작하지 않았다.(예를 들어 DI와 트랜잭션 처리가 가장 큰 문제였다.) 

4. 스프링 기반에서 빈을 동적으로 호출할 방법을 찾아야했다. 이에 대해서는 두가지 방법이 있다. 빈 팩토리와 컨텍스트의 getBean을 이용해 빈의 인스턴스를 구하여 호출도 가능하다. 그렇지만 이 두 방법을 쓰지 않고 WAS기동시 옵저버를 구현하는 빈이 초기화될때 직접 서비스레지스트리에 등록이 되도록 옵저버 구현 클래스의 register 메소드에 @PostConstruct 애노테이션을 적용하도록 했다.

5. 지금 생각해보면 옵저버를 구현한 클래스의 Bean 정보를 Subject와 관련된 엔티티에 담을 수도 있었을 것 같다. 각 엔티티는 어떤 옵저버에 연계처리를 요청할지 알고 있기 때문에, 구지 서비스레지스트리의 콜렉션에 보관할 필요가 없고, Subject에서 엔티티에 담긴 Bean 이름을 getBean을 이용해 찾아 호출할 수도 있었겠다.. 무쪼록 나는 내가 했던 구상을 따라 구현을 하다보니 옵저버가 직접 서비스레지스트리에 등록하는 방법을 택하게 되었다.

<br/>
## 클래스 다이어그램

아래에는 프로토타입으로 구현해본 옵저버 서비스 레지스트리의 클래스 다이어그램이다.

![이벤트 후처리 클래스 다이어그램](/assets/article_images/2020-02-12-lab-observer-service-registry-ver2/observer_after_architecture.png)


1. 다이어그램에서 레지스트리 역할을 하는 옵저버관리서비스는 연계업무별 옵저버를 구현한 Bean의 인스턴스가 등록되어 관리되는 곳이다.
2. 결의서에서 저장/삭제 및 승인처리시 결의서에 저장된 **'태스크ID'**를 이용해 옵저버 인스턴스를 구현한 특정 옵저버구현 Bean을 서비스레지스트리에서 찾아 연계처리 요청을 전달하게 된다.


<br/>

## 프로토타입 구현

### 1. 옵저버 인터페이스

```java
package kr.co.apps.service;

public interface Observer {
	
	//컨테이너 초기화시 서비스레지스트리에 인스턴스를 등록한다.
	public void register();
	
	//이벤트를 수신한다.
	public void update(CbEventType evntType);
}

```

### 2. 서비스레지스트리 인터페이스

```java
package kr.co.apps.service;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;

public interface IServiceRegistry {
	
	//옵저버를 구현한 클래스들이 자신의 인스턴스(Bean)을 등록한다.
	public void register(String taskId, Observer observer);
	
	//작업ID에 따른 옵저버들에 이벤트 전달
	public void notify(String taskId, CbEventType eventType);
	
	//서비스레지스트리에 등록된 인스턴스 현황을 가져온다.
	public List<Map<String,String>> selectRegistryList();	
}
```

### 3. 옵저버관리서비스(서비스레지스트리) 구현

옵저버들은 공통된 인터페이스로 구현되어, 스프링 초기화시 서비스레지스트리에 내부에 옵저버들의 보관소로 쓰이는 콜렉션으로써 해쉬맵을 이용하며, 이 보관소에는 Key로 쓰이는  **'태스크ID'**를, 그에 따른 값으로 옵저버의 인스턴스가 쌍으로 저장되게하였다.

```java
package kr.co.apps.service;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Map;

import org.springframework.stereotype.Service;

@Service
public class ServiceRegistry implements IServiceRegistry {
	
	private Map<String,Observer> registyMap = new HashMap<String,Observer>();  
	
	@Override
	public void register(String taskId, Observer observer) {
		// TODO Auto-generated method stub
		registyMap.put(taskId,observer);
		System.out.println("새로운 옵저버가 등록되었습니다===>");
		System.out.println("태스크ID :" + taskId );
		System.out.println("이벤트처리 옵저버 :" + observer.getClass().getName() );
	}

	@Override
	public void notify(String taskId, CbEventType eventType) {
		// TODO Auto-generated method stub
		Observer observer = registyMap.get(taskId);
		
		if(observer!=null) observer.update(eventType);
		
	}

	@Override
	public List<Map<String,String>> selectRegistryList() {
		
		List<Map<String,String>> list = new ArrayList<Map<String,String>>();
		Map<String,String> map = null;
		// TODO Auto-generated method stub
		Iterator<String> keys = registyMap.keySet().iterator();
		
		while(keys.hasNext())
		{
			map = new HashMap<String,String>();
			map.put("TASKID", keys.next());
			map.put("CLASS_NAME", registyMap.get(map.get("TASKID").toString()).getClass().getName());
			list.add(map);
		}
		
		return list;
	}

}

```


### 4. 옵저버 구현

이제 결의서 관련 연계업무의 옵저버들은 결의서 이벤트 발생시 이를 전달받아, 그에 따른 연계업무의 부가적인 처리를 한다.

```java
package kr.co.apps.service;

import javax.annotation.PostConstruct;
import javax.swing.event.DocumentEvent.EventType;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class TestObserver implements Observer {

	@Autowired
	ServiceRegistry serviceRegistry;
	
	@PostConstruct
	@Override
	public void register() {
		serviceRegistry.register("ERP3401", this);
	}

	@Override
	public void update(CbEventType evntType) {
		System.out.println("ERP3401 업데이트==="+ evntType	);
		
	}

}
```

### 5. 후처리 이벤트 ENUM 정의

```java
package kr.co.apps.service;

public enum CbEventType{
	
    EDIT,       //편집요청
    DELETE,     //삭제요청
    CANCEL,     //승인취소 요청
    APPROVE     //결재처리요청
    
}

```

### 6. 컨트롤러

```
package kr.co.apps;

import java.text.DateFormat;
import java.util.Date;
import java.util.Locale;

import javax.servlet.http.HttpServletRequest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import kr.co.apps.service.CbEventType;
import kr.co.apps.service.ServiceRegistry;

Handles requests for the application home page.
 */
@Controller
@RequestMapping("/reg")
public class HomeController {
	
	private static final Logger logger = LoggerFactory.getLogger(HomeController.class);
	
	@Autowired
	ServiceRegistry serviceRegistry;
	
	// 서비스 레지스트리 현황 목록
	@RequestMapping(value = "/get", method = RequestMethod.GET)
	public String home(Locale locale, Model model) {
		logger.info("Welcome home! The client locale is {}.", locale);
		
		Date date = new Date();
		DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG, locale);
		
		String formattedDate = dateFormat.format(date);
		
		model.addAttribute("serverTime", formattedDate );
		
		//서비레지스트리 목록을 리턴합니다.
		model.addAttribute("registryMap", serviceRegistry.selectRegistryList());
		
		return "home";
	}
	
	// 옵저버들에 이벤트 전달.
	@RequestMapping(value = "/post", method = RequestMethod.POST)
	public String home(HttpServletRequest request, Model model) {
		//작업ID에 해댕하는 옵저버 인스턴스에 이벤트를 전달
		serviceRegistry.notify(request.getParameter("taskid"), CbEventType.APPROVE);
		//서비스레지스트리 목록
		model.addAttribute("registryMap", serviceRegistry.selectRegistryList());
		return "home";
	}
	
}

```

### 7. JSP

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib uri = "http://java.sun.com/jsp/jstl/core" prefix = "c" %>
<html>
<head> 
	<title>Home</title>
</head>
<body>
<h1>
	Hello world!  
</h1>

<P><b>등록된 서비스 레지스트리 현황</b> </P>

<c:forEach items="${registryMap}" var="registry">
<li>
	${registry.TASKID} ===> ${registry.CLASS_NAME}
</li>
</c:forEach>
<form action="./post" method="post">
	<!--작업ID가 ERP3401인 옵저버에 이벤트를 전달-->
	<input type="text" name="taskid" value="ERP3401">
	<input type="submit" value="전송">
</form>
</body>
</html>

```

<br/>

## 이를 구현함으로써 다음의 처리가 가능해졌다.

1. 연계업무로직에서 직접 결의서를 생성할때는 연계업무로직에 주도권이 있어 생성 호출시 파라미터를 제어하고 결과를 이용해 연계업무에 필요한 부가적인 처리를 할 수 있다.

2. 그렇지만 결의서 자체에서 수정, 삭제, 결제처리 등의 이벤트를 발생할 때는 결의서 처리 프로세스로만 끝나게된다. 이때 옵저버패턴을 이용해 평소에는 연결이 없는 느슨한 상태의 연계업무 관련 옵저버에 이벤트와 파라미터를 전달하여 연계업무의 부가적인 처리가 가능하다.

3. 예를 들어, 결의서 삭제 전 후에 대한 이벤트를 옵저버에 전달하여하여 삭제가 가능한 결의서인지 연계업무에 물어보고 삭제 시에는 연계업무의 해당 데이터를 함께 지우도록 할 수 있다.

<br/>

## 데모는 아래에서 확인 가능합니다.

어디에나 쉽게 응용할 수 있는 형태의 오픈소스로 재작성된 데모 애플리케이션 영상와 소스코드를 아래에 공개합니다.

#### 1. 데모 애플리케이션 영상

[![Video Label](http://img.youtube.com/vi/LWjt0Uf1DZQ/0.jpg)](https://youtu.be/LWjt0Uf1DZQ?t=0s)

<p></p>
#### 2. 깃허브주소 공개 

[https://github.com/codegizer/lab-observer-registry](https://github.com/codegizer/lab-observer-registry)

## 감사합니다!!

이번 프로젝트를 함께한 참여사 및 동료들에게 감사의 뜻을 전한다. 혼자 보다는 여럿이가 더 의미있고 진중한 시간을 보낼 수 있음을 다시한번 확인할 수 있었다.



