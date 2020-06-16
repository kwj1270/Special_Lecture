
09 MVC 패턴 사용하기
==================
# 0. MVC 구조
![mvc](https://user-images.githubusercontent.com/50267433/84458173-555a9a00-ac9f-11ea-9d4d-e8cfa60989a0.png)   

MVC는 Model-View-Controller의 약자입니다.        
개발할 때 3가지 형태로 구분하여 개발하는 소프트웨어 개발 방법론입니다.       
     
그 3가지 요소를 설명하면        
**Model :** 무엇을 할지 정의합니다. 비지니스 로직에서의 알고리즘, 데이터 등의 기능을 처리합니다.         
**Controller :** 어떻게 할지를 정의합니다. 화면의 처리기능과 Model과 View를 연결시켜주는 역할을 합니다.         
**View :** 화면을 보여주는 역할을 합니다. 웹이라면 웹페이지, 모바일이라면 어플의 화면의 보여지는 부분입니다.       

# 소스 코드 동작 구조   
## 가장 먼저 실행되는 web.xml    

**web.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee https://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

	<!-- The definition of the Root Spring Container shared by all Servlets and Filters -->
	<!-- 우선적으로 실행될 xml 지정 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/spring/root-context.xml</param-value>
	</context-param>
	
	<!-- Creates the Spring Container shared by all Servlets and Filters -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

	<!-- Processes application requests -->
	<servlet>
		<servlet-name>appServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
	
	<!-- 서블릿 실행 조건 url -->	
	<servlet-mapping>
		<servlet-name>appServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

</web-app>
```
___
**0.** 실행 우선순위 0 순위로 ```/WEB-INF/spring/root-context.xml``` 의 정보를 가진 contextConfigLocation을 만든다.    
   
**web.xml**
```xml
	<!-- 우선적으로 실행될 xml 지정 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/spring/root-context.xml</param-value>
	</context-param>
```
```/WEB-INF/spring/root-context.xml``` 데이터베이스 환경 설정에 대한 xml이다.       
**1.** 어플리케이션 실행시 ```webapp -> WEB-INF``` 의 web.xml 이 실행된다.            
**2.** 서블릿 생성에는 여러 방식이 있는데 ```ContextLoaderListener```방식을 채용했다.         
**3.** ```DispatcherServlet appServlet = new DispatcherServlet(ContextLoaderListener);```의 형태로 컨테이너에 만든다.        
**4.** ContextLoaderListener 는 ```servlet-context.xml```의 정보를 가지고 있다. (해당 컨테이너 이용 가능)              
**5.** 이렇게 여러 서블릿을 만들 수 있는데 해당 서블릿에 대해 실행 우선순위를 1로 준다.           
**6.** 그리고 해당 서블릿은 url의 맨 앞에 ```/```가 들어갈때 실행한다. (즉 모든 구간 사용)   
  
**web.xml의 일부분**
```xml
	<!-- Creates the Spring Container shared by all Servlets and Filters -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	<!-- 서블릿 생성 방식이 여러개인다 Cotext.LoaderListener 방식 채용 -->

	<servlet>
		<servlet-name>appServlet</servlet-name> <!-- 변수 이름은 appServlet -->
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class> <!-- DispatcherServlet 타입이다. -->
		<init-param>
			<param-name>contextConfigLocation</param-name> <!-- 파라미터 이름 -->
			<param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value> <!-- 아래 xml을 값-->
		</init-param>
		<load-on-startup>1</load-on-startup> <!-- 우선순위 1 -->
	</servlet>

	<!-- 서블릿 실행 조건 url -->	
	<servlet-mapping>
		<servlet-name>appServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
```
    
**스프링 컨테이너에서 동작된다면 아마 아래와 같이 동작 될 것**   
```java
ContextConfigLocation contextConfigLocation = new ContextConfigLocation("/WEB-INF/spring/appServlet/servlet-context.xml");
DispatcherServlet appServlet = new DispatcherServlet(contextConfigLocation); // <servlet-class> <servlet-name> = new <servlet-class>(param1); 
appServlet.setLoad-on-startup(1); 
```   

## Request 요청이 들어 왔다면  
url 요청이 들어오면 알맞는 서블릿을 실행한다.     
예를 들어 ```http://localhost:8080/myapp/``` 나 ```http://localhost:8080/myapp/board``` 요청이 들어왔다 가정하자

1. web.xml 에서 / 을 url 패턴으로 지정한 appServlet이 실행이 된다.  
2. appSerlvet 은 servlet-context.xml을 기반으로 동작하는 객체이기에 해당 정보를 읽을 수 있다.  
3. 즉 해당 servlet-context.xml로 만들어진 컨테이너로부터 의존성 주입을 받을 수 있다.   

**servlet-context.xml**
```xml
	<!-- Enables the Spring MVC @Controller programming model -->
	<annotation-driven /> <!-- 어노테이션 사용을 허락하는 것 -->
	<context:component-scan base-package="com.mycompany.myapp" /> <!-- 해당 패키지내의 모든 클래스를 훑어서  객체생성 어노테이션 있으면 컨테이너에 객체 생성-->    

	<!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the ${webappRoot}/resources directory -->
	<resources mapping="/resources/**" location="/resources/" />

	<!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
	<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"> <!-- Controller에서 리턴시에 붙여줌 -->
		<beans:property name="prefix" value="/WEB-INF/views/" />
		<beans:property name="suffix" value=".jsp" />
	</beans:bean>
```
이를 간단히 설명하자면 
1. ```<annotation-driven />``` 어노테이션 사용 가능하게함 -> appServlet에서 ```@Autowired``` 사용 가능
2. ```<context:component-scan base-package="com.mycompany.myapp" />``` 컨테이너에 생성할 객체를 해당 경로에서 찾는다     
3. 컨테이너에 객체 등록을 할 것인데 클래스는 ```org.springframework.web.servlet.view.InternalResourceViewResolver```     
4. 해당 객체의 prefix 변수에는 ```"/WEB-INF/views/"```를      
5. 해당 객체의 suffix 변수에는 ```".jsp"```를 넣어준다.  

이는 아래와 같다.

**servlet-context로 만들어진 컨테이너에서의 동작 표현**   
```java   
---------------------------------------------- 컨테이너 ----------------------------------------------
| com.mycompany.myapp 에서 생성된 모든 객체들.... 						     |
|												     |
| InternalResourceViewResolver internalResourceViewResolver = new InternalResourceViewResolver();    |
| internalResourceViewResolver.setPrefix("/WEB-INF/views/");					     |
| internalResourceViewResolver.setSuffix(".jsp");						     |
|----------------------------------------------------------------------------------------------------|
```   
**DispatcherServlet 예시**    
```java 
@Autowired
com.mycompany.myapp.Class class_1;

@Autowired
com.mycompany.myapp.Class_2 class_2;

@Autowired
com.mycompany.myapp.Class_3 class_3;
.
.
.
.
.

@Autowired
InternalResourceViewResolver internalResourceViewResolver;
```
**큰 그림()**
```java   
--------------------------------------------- 컨테이너 -----------------------------------------------
| com.mycompany.myapp.Class_1 class_1 = new Class_1(); 						     |
| .												     |
| .												     |
| .												     |
|												     |
| InternalResourceViewResolver internalResourceViewResolver = new InternalResourceViewResolver();    |
| internalResourceViewResolver.setPrefix("/WEB-INF/views/");					     |
| internalResourceViewResolver.setSuffix(".jsp");						     |
|----------------------------------------------------------------------------------------------------|	
						|
						|
						↓
---------------------------------------------- appServlet ----------------------------------------------
| @Autowired											     |
| com.mycompany.myapp.Class class_1;						     		     |
| .												     |
| .												     |
| .												     |
| @Autowired                                                                                         |
| InternalResourceViewResolver internalResourceViewResolver;                                         |
|----------------------------------------------------------------------------------------------------|
```

# 1. 기존 Controller로 동작 분석하기     
Controller는 
1. 모델에 관련된 명령을 보냄으로써 모델을 생성 또는 변경할 수 있다.                               
2. 뷰와 관련된 명령을 보냄으로써 뷰를 출력 할 수 도 있다.                            
          
```BoardController``` 의 코드를 살펴보면서 해석을해보자       

**BoardController**
```java
package com.mycompany.myapp.controller.board;

import java.text.DateFormat;
import java.util.Date;
import java.util.Locale;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.mycompany.myapp.HomeController;
import com.mycompany.myapp.service.board.BoardService;

////// 컨트롤러 //////
@Controller
public class BoardController {
	
	@Autowired
	BoardService service;
	
	private static final Logger logger = LoggerFactory.getLogger(HomeController.class);

	@RequestMapping(value = "/board", method = RequestMethod.GET)
	public String home(Model model) { // 모델 생성
		// 모델 부분 //  
		model.addAttribute("boards", service.getAll()); // 모델 호출 및 상태 변경
		////////////
		
		// view 부분//
		return "board"; // 뷰 출력
		/////////////
	}
	
}
///////////////////
```
우선 큰 동작순서를 말하자면 아래와 같습니다.  
   
1. 서블릿 컨테이너에서 알맞는 객체 생성후 DispatcherServlet 실행          
2. DispatcherServlet 에서 BoardController를 생성 및 메소드를 호출하고 객체를 메소드의 인자값으로 넘긴다.     
3. 실행된 메소드를 살펴보면 Model 부분이 있고 View 부분이 있다.      
         
모델은 데이터를 처리하는 역할을 합니다.             
대개 DAO를 기능을 통해서 얻은 결과값을 저장하는 역할로 사용이 됩니다.          
그리고 이렇게 얻은 결과값을 View에 전달해서 이를 가지고 게시판, 게시판 리스트등을 만들 수 있습니다.       
       
뷰는 말그대로 클라이언트에게 무언가를 보여주기 위한 코드입니다.          
그런데 컨트롤러에서는 board만 리턴하는데 어떻게 동작을 하는 것일까요?      

1. 우선 "board" 는 DispatcherServlet 객체인 appServlet으로 리턴이 됩니다.       
2. ```internalResourceViewResolver```의 메소드를 이용해서 리턴된 board와 알맞는 값들을 결합시켜준다.       
