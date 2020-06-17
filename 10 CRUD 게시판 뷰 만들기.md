# 1. BoardController 변경하기  
* BoardController를 아래 코드와 같이 변경한다.  
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
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.mycompany.myapp.HomeController;
import com.mycompany.myapp.dto.board.Board;
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
	
	@RequestMapping(value = "/board/{seq}", method = RequestMethod.GET)
	public String detail(@PathVariable int seq ,Model model) { // 모델 생성
		Board vo = new Board();
		vo.setSeq(seq);
		// 모델 부분 //  
		model.addAttribute("board", service.getOne(vo)); // 모델 호출 및 상태 변경
		////////////
		
		// view 부분//
		return "board-detail"; // 뷰 출력
		/////////////
	}
	
	@RequestMapping(value = "/board/{seq}/update", method = RequestMethod.GET)
	public String update(@PathVariable int seq ,Model model) { // 모델 생성
		Board vo = new Board();
		vo.setSeq(seq);
		// 모델 부분 //  
		model.addAttribute("board", service.getOne(vo)); // 모델 호출 및 상태 변경
		////////////
		
		// view 부분//
		return "board-update"; // 뷰 출력
		/////////////
	}
	
	@RequestMapping(value = "/board/write", method = RequestMethod.GET)
	public String detail(Model model) { // 모델 생성
		return "board-write"; // 뷰 출력
	}
	
}
///////////////////
```
# 2. home.jsp 변경하기 
board 게시판으로 쉽게 이동할 수 있게 아래와 같이 ```home.jsp``` 를 수정해주자
     
**home.jsp**
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ page session="false"%>

<html>
<head>
	<title>Home</title>
</head>
<body>
<h1>
	Hello world!  
</h1>

<P>  The time on the server is ${serverTime}</P>
<a href="board">게시판으로 이동</a>
</body>
</html>
```   
![홈](https://user-images.githubusercontent.com/50267433/84847182-7e0dd580-b08b-11ea-96a1-3cbbc28822b8.PNG)    
     
# 3. board CRUD View 만들기
## 3.1. board-detail.jsp 생성하기   
* ```webapp``` -> ```WEB-INF``` -> ```views``` 에 board-detail.jsp를 만들어 준다.  
* 아래와 같은 코드를 입력해준다.  

**board-detail.jsp**
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ page session="false"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport"
	content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="ie=edge">
<title>게시판 상세</title>
<link rel="stylesheet"
	href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
</head>
<body>
	<h2>게시판 상세</h2>
	<div class="col-md-12">
		<div class="col-md-4">
			<form>
				<div class="form-group">
					<label for="seq">게시글 번호</label>
					<input type="text" class="form-control" id="seq" readonly="readonly" value="${board.seq}">
				</div>
				<div class="form-group">
					<label for="title">제목</label>
				<input type="text" class="form-control" id="title" readonly="readonly" value="${board.title}">
				</div>
				<div class="form-group">
					<label for="writer">글쓴이</label>
				<input type="text" class="form-control" id="writer" readonly="readonly" value="${board.writer}"> 
				</div>
				<div class="form-group">
					<label for="content">내용</label>
				<textarea class="form-control" readonly="readonly" id="content">${board.content}</textarea>
				</div>
				<div class="form-group">
					<label for="cnt">조회수</label>
				<input type="text" class="form-control" id="cnt" readonly="readonly" value="${board.cnt}">
				</div>
			</form>
			<button type="button" class="btn btn-primary" id="btn-update">수정</button>
			<button type="button" class="btn btn-danger" id="btn-delete">삭제</button>
			<button type="button" class="btn btn-secondary" id="btn-list">목록</button>
		</div>
	</div>
	<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
	<script
		src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>
	<script src="/myapp/board/js/board-detail.js"></script>
</body>
</html>
```

## 3.2. board-write.jsp 생성하기   
* ```webapp``` -> ```WEB-INF``` -> ```views``` 에 board-write.jsp를 만들어 준다.  
* 아래와 같은 코드를 입력해준다.  

**board-write.jsp**
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ page session="false"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport"
	content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="ie=edge">
<title>게시판 상세</title>
<link rel="stylesheet"
	href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
</head>
<body>
	<h2>게시판 상세</h2>
	<div class="col-md-12">
		<div class="col-md-4">
			<form>
				<div class="form-group">
					<label for="title">제목</label>
				<input type="text" class="form-control" id="title">
				</div>
				<div class="form-group">
					<label for="writer">글쓴이</label>
				<input type="text" class="form-control" id="writer"> 
				</div>
				<div class="form-group">
					<label for="content">내용</label>
				<textarea class="form-control" id="content"></textarea>
				</div>
			</form>
			<button type="button" class="btn btn-primary" id="btn-write">저장</button>
			<button type="button" class="btn btn-secondary" id="btn-list">목록</button>
		</div>
	</div>
	<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
	<script
		src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>
	<script src="/myapp/board/js/board-write.js"></script>
</body>
</html>
```

## 3.3. board-update.jsp 생성하기   
* ```webapp``` -> ```WEB-INF``` -> ```views``` 에 board-update.jsp를 만들어 준다.  
* 아래와 같은 코드를 입력해준다.   
     
**board-update.jsp**
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ page session="false"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport"
	content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="ie=edge">
<title>게시판 상세</title>
<link rel="stylesheet"
	href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
</head>
<body>
	<h2>게시판 상세</h2>
	<div class="col-md-12">
		<div class="col-md-4">
			<form>
				<div class="form-group">
					<label for="seq">게시글 번호</label>
					<input type="text" class="form-control" id="seq" readonly="readonly" value="${board.seq}">
				</div>
				<div class="form-group">
					<label for="title">제목</label>
				<input type="text" class="form-control" id="title" value="${board.title}">
				</div>
				<div class="form-group">
					<label for="writer">글쓴이</label>
				<input type="text" class="form-control" id="writer" readonly="readonly" value="${board.writer}"> 
				</div>
				<div class="form-group">
					<label for="content">내용</label>
				<textarea class="form-control"  id="content">${board.content}</textarea>
				</div>
				<div class="form-group">
					<label for="cnt">조회수</label>
				<input type="text" class="form-control" id="cnt" readonly="readonly" value="${board.cnt}">
				</div>
			</form>
			<button type="button" class="btn btn-primary" id="btn-update">수정</button>
			<button type="button" class="btn btn-secondary" id="btn-list">목록</button>
		</div>
	</div>
	<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
	<script
		src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>
	<script src="/myapp/board/js/board-update.js"></script>  
	
</body>
</html>
``` 
## 3.4. JS파일 접근을 위한 매핑 설정
각 파일들의 맨 아래쪽을 보면 아래와 같은 코드들들을 찾을 수 있을 것이다.    
```
<script src="/myapp/board/js/board.js"></script>
<script src="/myapp/board/js/board-detail.js"></script>
<script src="/myapp/board/js/board-write.js"></script>
<script src="/myapp/board/js/board-update.js"></script>  
```
위 코드는 JS 파일을 불러와서 이벤트 동작을 처리할 수 있게끔해주는 코드이다.      
하지만 위 코드만으로는 에러가 발생한다.         
   
우리가 url에 값을 주어 동작시킬때 알 수 있는 점은 프로젝트의 webapp안에서 찾는다. (뷰 찾을때)            
즉, 위 코드는 ```/myapp/**webapp**/board/js/board-detail.js```를 찾는 동작이 된다.          
그래서 이를 방지하고자 mapping 설정을 해줘야 하는데 이런 동작은         
```webapp``` -> ```WEB-INF``` -> ```spring``` -> ```appServlet``` -> ```servlet-context.xml``` 에서 처리해준다.    
이후 아래와 같은 코드를 추가시켜주자  
```xml	
<resources location="/resources/board/" mapping="/board/**"/>
```
**web.xml 전체 코드**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/mvc"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:beans="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd
		http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

	<!-- DispatcherServlet Context: defines this servlet's request-processing infrastructure -->
	
	<!-- Enables the Spring MVC @Controller programming model -->
	<annotation-driven /> <!-- 어노테이션 사용을 허락하는 것 -->
	<context:component-scan base-package="com.mycompany.myapp" /> <!-- 해당 패키지내의 모든 클래스를 훑어서  객체생성 어노테이션 있으면 컨테이너에 객체 생성-->    

	<!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the ${webappRoot}/resources directory -->
	<resources location="/resources/" mapping="/resources/**"/>
	<resources location="/resources/board/" mapping="/board/**"/>

	<!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
	<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"> <!-- Controller에서 리턴시에 붙여줌 -->
		<beans:property name="prefix" value="/WEB-INF/views/" />
		<beans:property name="suffix" value=".jsp" />
	</beans:bean>
</beans:beans>
```
![리소스 추가](https://user-images.githubusercontent.com/50267433/84848653-87e50800-b08e-11ea-96b0-618760360496.PNG)    
   
# 4. board CRUD 를 위한 JS파일 만들기         
**Java 와 Javascript는 엄연히 다른 언어입니다.**          
과거 네스케이프라는 회사는 HTML페이지에 경량의 프로그램 언어를 통하여 인터렉티브한 **동작**을 추가 하고 싶었다.                   
그래서 자바를 만든 SUN 회사와 손을 잡아 언어를 만들었고 SUN사의 JAVA의 이름을 빌려와 Javascript가 되었다.             
(이름의 역사 : 모카 -> 라이브스크립트 -> 자바스크립트)            
         
이후 MicroSoft사는 IE 3.0에서 동작하는 ‘JSrcipt’라는 자바스크립트와 똑같은 언어를 만들어 냈다.                
즉, JS가 2개가 존재해지고 비슷하지만 다른 부분이 많기에 이를 표준화하고자 ECMAScript를 만들었습니다.            
         
현재 저희가 사용하는 자바스크립트는 표준화된 ECMAScript인데,            
자바스크립트라는 이름이 이미 자주 사용되어서 그냥 자바 스크립트라고 사용하고 있습니다.       
     
자바스크립트는 이벤트에 대한 처리를 해줄 수 있습니다.         
즉, 마우스클릭시, 드래그시, 키보드 입력시 -> '배경화면을 바꿔라' 와 같이 원하는 동작을 할 수 있습니다.        
자바는 자바 프로그램 자체에 대해서는 이러한 이벤트 처리를 할 수 있지만 웹 브라우저에 대해서는 처리하기 힘듭니다.         
그렇기에 우리는 자바 스크립트를 이용할 것이고 **버튼을 누르면 REQUEST 요청을 보내는 동작을 할 것입니다.**    

## 4.1. board CRUD 를 위한 디렉토리 만들기
JS, CSS, Image 같은 파일들은 주로 **resources 폴더에 넣는다.**      
**그러나 우리 프로젝트의 구조를 보면 reources 폴더가 2개가 존재한다.**   
      
1. ```main``` -> ```resources```      
2. ```main``` -> ```webapp``` -> ```resources```  
   
개발자가 알아서 맞춰 사용해도 되지만 우리는 ```main -> webapp -> resources``` 의 폴더를 사용할 것이다.    
  
1. ```main``` -> ```webapp``` -> ```resources``` 폴더에 board 폴더를 만든다.    
2. 생성된 board 폴더 밑에 js 폴더를 만든다.       
3. 이제 우리는 JS 파일을 만들 준비가 다 되었다.      
     
![리소스파일](https://user-images.githubusercontent.com/50267433/84847227-9a117700-b08b-11ea-83e0-957296f6c2a3.PNG)    

## 4.2. board.js 를 만든다.  
1.  ```main``` -> ```webapp``` -> ```resources``` -> ```board``` -> ```js``` 에 board.js를 만든다.    
2. 아래와 같은 코드를 입력해주자       
           
**board.js**
```javascript
var board = {
	init : function() {
		var _this = this;
		$('#btn-board-write').on('click', function() {
			_this.go_write();
		})
	},
	go_write : function() {
		window.location.href = "http://localhost:8080/myapp/board/write"
	}
}
board.init();
```
    
## 4.3. board-deatail.js 를 만든다.  
1.  ```main``` -> ```webapp``` -> ```resources``` -> ```board``` -> ```js``` 에 board-deatail.js를 만든다.    
2. 아래와 같은 코드를 입력해주자      
    
**board-deatail.js**
```javascript
var board_detail = {
	init : function() {
		var _this = this;
		
		$('#btn-list').on('click', function() {
			_this.list();
		})
		
		$('#btn-update').on('click', function() {
			_this.update();
		})
		
		$('#btn-delete').on('click', function() {
			_this.delete();
		})
	},
	
	list : function() {
		window.location.href = "http://localhost:8080/myapp/board"
	},

	update : function() {
		var stringArr = document.location.href.split("http://localhost:8080/myapp/board/");
		var seq = stringArr[1];
		window.location.href = "http://localhost:8080/myapp/board/" + seq + "/update"
	},

	delete : function() {
		var stringArr =  document.location.href.split("http://localhost:8080/myapp/board/");
		var seq = stringArr[1];
			$.ajax({
				type:'DELETE',
				url:'/myapp/api/v1/board/' + seq,
				dataType:'json',
				contentType: "application/json; charset=utf-8",
			}).done(function() {
				alert('글이 삭제되었습니다');
				window.location.href = "/myapp/board";
			}).fail(function(error) {
				alert(JSON.stringify(error));
			});
	}
};

board_detail.init();
```
## 4.3. board-write.js 를 만든다.  
1.  ```main``` -> ```webapp``` -> ```resources``` -> ```board``` -> ```js``` 에 board-write.js를 만든다.
2. 아래와 같은 코드를 입력해주자  

**board-write.js**
```javascript
var board_write = {
	init : function() {
		var _this = this;
		
		$('#btn-list').on('click', function() {
			_this.list();
		})
		
		$('#btn-write').on('click', function() {
			_this.write();
		})
	},
	
	list : function() {
		window.location.href = "http://localhost:8080/myapp/board"
	},

	write : function() {
		var _this =  this;
		var data = {
			title: $('#title').val(),
			writer: $('#writer').val(),
			content: $('#content').val(),
		};
		
			$.ajax({
				type:'POST',
				url:'/myapp/api/v1/board/',
				dataType:'json',
				contentType: "application/json; charset=utf-8",
				data: JSON.stringify(data)
			}).done(function() {
				alert('글이 생성되었습니다.');
				window.location.href = "/myapp/board";
			}).fail(function(error) {
				alert(JSON.stringify(error));
			});
	}
};
     
board_write.init();
```   
   
## 4.4. board-update.js 를 만든다.  
1.  ```main``` -> ```webapp``` -> ```resources``` -> ```board``` -> ```js``` 에 board-update.js 를 만든다.      
2. 아래와 같은 코드를 입력해주자       
           
**board-update.js**    
```javascript
var board_update = {
	init : function() {
		var _this = this;
		
		$('#btn-list').on('click', function() {
			_this.list();
		})
		
		$('#btn-update').on('click', function() {
			_this.update();
		})
	},
	
	list : function() {
		window.location.href = "http://localhost:8080/myapp/board"
	},

	update : function() {
		var _this =  this;
		var stringArr = document.location.href.split("http://localhost:8080/myapp/board/");
		var stringArr2 = stringArr[1].split("/");
		var seq = stringArr2[0];
		var data = {
			seq : $('#seq').val(),
			title: $('#title').val(),
			content: $('#content').val(),
		};
			$.ajax({
				type:'PUT',
				url:'/myapp/api/v1/board/'+seq,
				dataType:'json',
				contentType: "application/json; charset=utf-8",
				data: JSON.stringify(data)
			}).done(function() {
				alert('글이 수정되었습니다.');
				window.location.href = "/myapp/board";
			}).fail(function(error) {
				alert(JSON.stringify(error));
			});
	}
};
      
board_update.init();  
```   
   

  
