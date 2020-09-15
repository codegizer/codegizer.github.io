---
published: false
layout: post
title: 후처리를 위한 서비스레지스트리 구현
date:   2019-06-30 23:15:00 +0900
categories: 실험실
---
# 구현배경
결의서 후처리에 대한 개발자들의 니즈를 충족하기 위해서 고안해보았다.

# 구현요약
옵저버 패턴와 스프링 빈을 이용한다.

# 구현방법
MSA에서 서비스 레지스트리는 각 서비스 인스턴스의 네트워크 위치 정보를 보관한다.
이번 구현에서는 백엔드에서 결의서 처리에 따른 이벤트를 수신할 로컬 서비스들을 컨테이너 초기화시 레지스트리 등록하여 관리가 이루어지도록 하였다.

# 옵저버패턴
TODO::

# 스프링빈
TODO::

# 구현

### 옵저버 인터페이스

```java
package kr.co.apps.service;

public interface Observer {
	
	//컨테이너 초기화시 서비스레지스트리에 인스턴스를 등록한다.
	public void register();
	
	//이벤트를 수신한다.
	public void update(CbEventType evntType);
}

```

### 서비스레지스트리 인터페이스

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
### 서비스레지스트리 구현

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


### 옵저버 구현

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

### 후처리 이벤트 ENUM 정의

```java
package kr.co.apps.service;

public enum CbEventType{
	
	SAVE,
	DELETE,
	EDIT,	
	ROLLBACK,
	APPROVE
}
```

### 컨트롤러

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

### JSP

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



