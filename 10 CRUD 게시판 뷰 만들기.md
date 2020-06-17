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
   
# 3. board-detail.jsp 생성하기   
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

# 4. board-write.jsp 생성하기   
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

# 5. board-update.jsp 생성하기   
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
