
09 MVC 패턴 사용하기
==================
# 0. MVC 구조
![mvc](https://user-images.githubusercontent.com/50267433/84458173-555a9a00-ac9f-11ea-9d4d-e8cfa60989a0.png)   

MVC는 Model-View-Controller의 약자입니다.        
개발할 때 3가지 형태로 구분하여 개발하는 소프트웨어 개발 방법론입니다.       
     
그 3가지 요소를 설명하면
**Model :** 무엇을 할지 정의합니다. 비지니스 로직에서의 알고리즘, 데이터 등의 기능을 처합니다.
**Controller :** 어떻게 할지를 정의합니다. 화면의 처리기능과 Model과 View를 연결시켜주는 연활을 하지요. 
**View :** 화면을 보여주는 역할을 합니다. 웹이라면 웹페이지, 모바일이라면 어플의 화면의 보여지는 부분입니다.

# 소스 코드 동작 구조   
    
1. 클라이언트가 서버에게 Request를 전송하면 가장 먼저 web.xml 이 실행된다.        
2. 서블릿 생성에는 여러 방식이 있는데 ```ContextLoaderListener```방식을 채용했다.    
3. ```DispatcherServlet appServlet = new DispatcherServlet(ContextLoaderListener);```의 형태로 컨테이너에 만든다.    
4. ContextLoaderListener 는 ```servlet-context.xml```의 정보를 가지고 있다.       
5. 이렇게 여러 서블릿을 만들 수 있는데 해당 서블릿에 대해 실행 우선순위를 1로 준다.        
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
```
    
**스프링 컨테이너**
```java
ContextConfigLocation contextConfigLocation = new ContextConfigLocation("/WEB-INF/spring/appServlet/servlet-context.xml");
DispatcherServlet appServlet = new DispatcherServlet(contextConfigLocation);
appServlet.setLoad-on-startup(1);

<servlet-class> <servlet-name> = new <servlet-class>(param1, param2, load-on-startup);  
```   

3.
4.
5.
6.
7.
8.
9.


# 1. 기존 Controller 분석하기     
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

1. 우선 "board" 는 DispatcherServlet 객체로 리턴이 됩니다.       
**참고로 DispatcherServlet은 우리가 정의하지 않았습니다.**        
**스프링 컨테이너에서 알아서 만들어주고 동작시켜주는 Servlet 입니다.(물론 생성 명령은 내려야함)**

**web.xml에서 DispathcerServlet 생성을 위한 구문**
```xml
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
		
	<servlet-mapping>
		<servlet-name>appServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
```

2. DispatcherServlet 은 ```webapp -> WEB-INF -> spring -> appServlet -> servlet-context.xml```의       
```"org.springframework.web.servlet.view.InternalResourceViewResolver"``` 객체를 의존성 주입받는다.        
      
**DispatcherServlet 예시**    
```java 
@Autowired
InternalResourceViewResolver internalResourceViewResolver;
```
3. ```internalResourceViewResolver```의 메소드를 이용해서 리턴된 board와 알맞는 값들을 결합시켜준다.       
board 와 결합될 부분은 ```webapp -> WEB-INF -> spring -> appServlet -> servlet-context.xml```에 정의되어 있다.     
     
**servlet-context.xml 예시**  
```xml
~ 생략 ~ 
	<!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
	<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"> <!-- Controller에서 리턴시에 붙여줌 -->
		<beans:property name="prefix" value="/WEB-INF/views/" />
		<beans:property name="suffix" value=".jsp" />
```
