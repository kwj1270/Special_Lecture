# 1. JDBCUtil 작성 
DAO로 DB연결을 위해서 ```Connection, Statement, PreaparedStatement, ResultSet``` 객체를 생성해준다.     
하지만 데이터베이스 연결이 끝나면 즉각적으로 해당 객체들을 삭제해줘야 하는데          
예를들어 100개의 메소드에서 저 4개를 삭제하는 코드를 각각 작성하는 것은 매우 비효율적이므로        
이를 처리해주는 클래스를 만들어줄 것이다.         
    
1. 우선 myapp 밑에 dao 폴더를 만들어준다.       
2. 그 다음에 dao 밑에 util 폴더를 만들어준다.       
3. 그리고 JDBCUtil 이름을 가지는 클래스를 만들어준다.         
4. 아래 소스 코드를 복사 붙이기해주자 (나중에 불필요해지므로 빠르게 하자)          

![util 작성](https://user-images.githubusercontent.com/50267433/84100356-a9b70d00-aa46-11ea-9ce1-568fb3592101.PNG)

**JDBCUtil**
```java
package com.mycompany.myapp.dao.util;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;


public class JDBCUtil {
	public static Connection getConnection() {
		try {
			Class.forName("org.h2.Driver");
			return DriverManager.getConnection("jdbc:h2:tcp://localhost/~/test", "sa", "");
		} catch (Exception e) {
			e.printStackTrace();
		}
		return null;
	}

	public static void close(PreparedStatement pstmt, Connection conn) {
		if (pstmt != null) {
			try {
				if (!pstmt.isClosed())
					pstmt.close();
			} catch (Exception e) {
				e.printStackTrace();
			} finally {
				pstmt = null;
			}
		}
		if (conn != null) {
			try {
				if (!conn.isClosed())
					conn.close();
			} catch (Exception e) {
				e.printStackTrace();
			} finally {
				conn = null;
			}
		}

	}

	public static void close(ResultSet rs, PreparedStatement pstmt, Connection conn) {
		if (rs != null) {
			try {
				if (!rs.isClosed())
					rs.close();
			} catch (Exception e) {
				e.printStackTrace();
			} finally {
				rs = null;
			}
		}
		close(pstmt, conn);
	}
}
```
# 2. BoardDAOJDBC 클래스 작성 및 BoardDAO 인터페이스 작성  
이제 Board 객체를 이용한 데이터베이스 ```insert,delete,update, select``` 코드를 만들어 보자    
   
1. 우선 dao 밑에 board 폴더를 만들어준다.           
2. 그리고 BoardDAOJDBC 이름을 가지는 클래스를 만들어준다.             
3. 아래 소스 코드를 복사 붙이기해주자 (나중에 불필요해지므로 빠르게 하자)   

![BoardDAO 작성](https://user-images.githubusercontent.com/50267433/84102655-0832ba00-aa4c-11ea-8a29-99ddba615fda.PNG)
    
**BoardDAOJDBC**
```java
package com.mycompany.myapp.dao.board;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.ArrayList;
import java.util.List;

import org.springframework.stereotype.Repository;

import com.mycompany.myapp.dao.util.JDBCUtil;
import com.mycompany.myapp.dto.board.Board;

@Repository
public class BoardDAOJDBC implements BoardDAO {
	// JDBC 관련 변수
	private Connection conn = null;
	private PreparedStatement pstmt = null;
	private ResultSet rs = null;

	private final String BOARD_INSERT = "INSERT INTO board(seq, title, writer, content) "
			+ "VALUES((SELECT nvl(max(seq),0)+1 FROM board),?,?,?);"; // AUTOINCREMENT 기능 직접 추가
	// nvl(인자1, 인자2) 인자1이 null일 경우 인자2를 사용하겠다는 뜻
	private final String BOARD_UPDATE = "UPDATE BOARD SET title=?, content=? WHERE seq=?;";
	private final String BOARD_DELETE = "DELETE FROM BOARD WHERE seq=?;";
	private final String BOARD_GET = "SELECT * FROM board WHERE seq=?;";
	private final String BOARD_LIST = "SELECT * FROM board ORDER BY seq DESC;";


	@Override
	public int insert(Board vo) {
		System.out.println("===> JDBC로 insertBoard() 기능 처리");
		try {
			conn = JDBCUtil.getConnection();
			pstmt = conn.prepareStatement(BOARD_INSERT);
			pstmt.setString(1, vo.getTitle());
			pstmt.setString(2, vo.getWriter());
			pstmt.setString(3, vo.getContent());
			return pstmt.executeUpdate();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			JDBCUtil.close(pstmt, conn);
		}
		return -1;
	}

	@Override
	public int update(Board vo) {
		System.out.println("===> JDBC로 updateBoard() 기능 처리");
		try {
			conn = JDBCUtil.getConnection();
			pstmt = conn.prepareStatement(BOARD_UPDATE);
			pstmt.setString(1, vo.getTitle());
			pstmt.setString(2, vo.getContent());
			pstmt.setInt(3, vo.getSeq());
			return pstmt.executeUpdate();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			JDBCUtil.close(pstmt, conn);
		}
		return -1;
	}

	@Override
	public int delete(Board vo) {
		System.out.println("===> JDBC로 deleteBoard() 기능 처리");
		try {
			conn = JDBCUtil.getConnection();
			pstmt = conn.prepareStatement(BOARD_DELETE);
			pstmt.setInt(1, vo.getSeq());
			return pstmt.executeUpdate();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			JDBCUtil.close(pstmt, conn);
		}
		return -1;
	}

	@Override
	public Board getOne(Board vo) {
		System.out.println("===> JDBC로 getBoard() 기능 처리");
		Board board = null;

		try {
			conn = JDBCUtil.getConnection();
			pstmt = conn.prepareStatement(BOARD_GET);
			pstmt.setInt(1, vo.getSeq());
			rs = pstmt.executeQuery();
			if (rs.next()) {
				board = new Board();
				board.setSeq(rs.getInt("SEQ"));
				board.setTitle(rs.getString("TITLE"));
				board.setWriter(rs.getString("WRITER"));
				board.setContent(rs.getString("CONTENT"));
				board.setRegDate(rs.getDate("REGDATE"));
				board.setCnt(rs.getInt("CNT"));
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			JDBCUtil.close(rs, pstmt, conn);
		}
		return board;
	}

	@Override
	public List<Board> getAll() {
		System.out.println("===> JDBC로 getBoardList()기능 처리");
		List<Board> boardList = new ArrayList<Board>();
		try {
			conn = JDBCUtil.getConnection();
			pstmt = conn.prepareStatement(BOARD_LIST);
			rs = pstmt.executeQuery();
			while (rs.next()) {
				Board board = new Board();
				board.setSeq(rs.getInt("SEQ"));
				board.setTitle(rs.getString("TITLE"));
				board.setWriter(rs.getString("WRITER"));
				board.setContent(rs.getString("CONTENT"));
				board.setRegDate(rs.getDate("REGDATE"));
				board.setCnt(rs.getInt("CNT"));
				boardList.add(board);
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			JDBCUtil.close(rs, pstmt, conn);
		}
		return boardList;
	}
}
```

# 3. DAO 교체를 위한 인터페이스 작성 (유지보수 편리성)   
1. BoardDAOJDBC에서 ```Alt+Shift+T``` 를 눌러준다.   
2. 그후 Extract Interface를 클릭해준다.
3. 모든 우측의 SelectALL 버튼을 눌러 체크박스를 클릭하고 BoardDAO라는 이름을 지어주자 

![DAOInterface 작성](https://user-images.githubusercontent.com/50267433/84102729-37e1c200-aa4c-11ea-9b7f-5336cdffae9e.PNG)
![DAOInterface 작성2](https://user-images.githubusercontent.com/50267433/84102834-74152280-aa4c-11ea-9c22-1b61d6ff2a8c.PNG)
![DAOInterface 작성3](https://user-images.githubusercontent.com/50267433/84102818-6a8bba80-aa4c-11ea-86c9-025b0492aee8.PNG)
![boardDAO](https://user-images.githubusercontent.com/50267433/84109830-8435fd80-aa5e-11ea-91e4-8a811028e4ba.PNG)

**BoardDAO**
```java
package com.mycompany.myapp.dao.board;

import java.util.List;

import com.mycompany.myapp.dto.board.Board;

public interface BoardDAO {

	int insert(Board vo);

	int update(Board vo);

	int delete(Board vo);

	Board getOne(Board vo);

	List<Board> getAll();

}
```

# 4. Service 클래스 생성   

1. 우선 myapp 밑에 service 폴더를 만들어준다.       
2. 그 다음에 service 밑에 board 폴더를 만들어준다.       
3. 그리고 BoardService 이름을 가지는 클래스를 만들어준다.         
4. 아래 소스 코드를 복사 붙이기해주자   

![서비스](https://user-images.githubusercontent.com/50267433/84109855-9a43be00-aa5e-11ea-979a-47ca7f72717c.PNG)

```java
package com.mycompany.myapp.service.board;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.mycompany.myapp.dao.board.BoardDAO;
import com.mycompany.myapp.dto.board.Board;

@Service
public class BoardService{
	
	@Autowired
	BoardDAO dao;

	public int insert(Board vo) {
		return dao.insert(vo);
	}
	
	public int update(Board vo) {
		return dao.update(vo);
	}

	public int delete(Board vo) {
		return dao.delete(vo);
	}
	
	public Board getOne(Board vo) {
		return dao.getOne(vo);
	}
	
	public List<Board> getAll() {
		return dao.getAll();
	}
	
}
```
Service 클래스를 만들면서 한가지 의문점이 있다.       
Service 클래스는 단순히 DAO만 불러서 사용하는데 왜 쓰는 것일까?   
이유는 간단하다 예를 들어 Service 클래스 없이 DAO만 있다고 가정을 하면  
우리는 아래와 같이 코드를 작성할 것이다.
    
**예시**   
```java
class Controller1 {
	dao.insert(vo);
}
________________________
class Controller2 {
	dao.insert(vo);
}
________________________
class Controller3 {
	dao.insert(vo);
}
________________________
class Controller4 {
	dao.insert(vo);
}
________________________
.... 100 개의 파일이 있다.   
```
위와 같은 코드를 사용하는 클래스가 100개가 있다고 가정을 하자     
그런데 어느날 사장님이 오셔서 DAO 클래스를 바꾸라 하셨는데          
메소드 이름이 insert가 아니라 save였다면 어떻게 되겠는가 100개를 다 바꿔줘야 한다.       
   
하지만 Service 클래스를 만듬으로써    
우리는 Service 클래스의 메소드에 사용되는 insert만 save로 바꿔주면 된다.

```java
class Service {
	public void insert(Board vo) {
		dao.save(vo);
	}
}
________________________
class Controller1 {
	service.insert(vo);
}
________________________
class Controller2 {
	service.insert(vo);
}
________________________
class Controller3 {
	service.insert(vo);
}
________________________
class Controller4 {
	service.insert(vo);
}
________________________
.... 100 개의 파일이 있다.   
```

# 5.1. 일반 Controller 클래스 생성   
1. 우선 myapp 밑에 controller 폴더를 만들어준다.       
2. 그 다음에 controller 밑에 board 폴더를 만들어준다.       
3. 그리고 BoardController 이름을 가지는 클래스를 만들어준다.         
4. 아래 소스 코드를 복사 붙이기해주자   

![컨트롤러 만들기](https://user-images.githubusercontent.com/50267433/84110935-cc561f80-aa60-11ea-95a4-2dcd26917d94.PNG)

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
	
}
///////////////////
```

# 5.2. 일반 RestController 클래스 생성   

1. controller를 생성해줬던 폴더에 BoardAPIController 클래스를 생성해준다.    
2. 아래 코드와 같이 입력해준다.        
      
**BoardAPIController**
```java
package com.mycompany.myapp.controller.board;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

import com.mycompany.myapp.dto.board.Board;
import com.mycompany.myapp.dto.user.User;
import com.mycompany.myapp.service.board.BoardService;
import com.mycompany.myapp.service.user.UserService;

@RestController
public class BoardAPIController {

	@Autowired
	BoardService service;
	
	@PostMapping("/api/v1/board")
	public int insert(@RequestBody Board vo) {
		return service.insert(vo);
	}

	@PutMapping("/api/v1/board/{seq}")
	public int update(@PathVariable int seq) {
		Board vo = new Board();
		vo.setSeq(seq);
		return service.update(vo);
	}
	
	@GetMapping("/api/v1/board/{seq}")
	public Board getOne(@PathVariable int seq) {
		Board vo = new Board();
		vo.setSeq(seq);
		return service.getOne(vo);
	}
	
	@DeleteMapping("/api/v1/board/{seq}")
	public int delete(@PathVariable int seq) {
		Board vo = new Board();
		vo.setSeq(seq);
		return service.delete(vo);
	}
	
}
```
```@RestController```는 리턴을 페이지가 아닌 값 자체를 리턴한다.     
즉, ```@Controller```는 jsp 파일을 리턴, ```@RestController```는 DB처리로 인한 값을 리턴   
     
# 6. pom.xml 에 라이브러리 추가하기  
기존에 pom.xml 은 라이브러리를 추가하는 방법을 위해 h2 데이터베이스 라이브러리를 제외하고 제공드렸습니다.    
이제 아래 코드를 pom.xml에 추가해봅시다.        
위치는 ```<dependecies> </dependecies>```사이에 아무곳에나 넣어주시면 됩니다.      
단, 겹치지 않게 조심해주세요       
![폼 수정](https://user-images.githubusercontent.com/50267433/84111241-7a61c980-aa61-11ea-80d3-bff89bd623ca.PNG)

**pom.xml 추가 코드**   
```xml
		<!-- h2 데이터베이스 -->
		<dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>1.4.191</version>
		</dependency>
```

# 7. board 뷰 만들기

1. 기존에 우리가 작업했던곳을 벗어나서 **webapp**에서 작업을 할 것입니다.
2. webapp -> WEB-INF -> views 에서 board.jsp를 만들어 줍시다.
3. 만드는 방법은 views 에서 오른쪽 클릭 -> new -> other -> jsp 검색하면 jsp File이 나옵니다.

![board 홈피](https://user-images.githubusercontent.com/50267433/84111283-906f8a00-aa61-11ea-8831-465929e18eb4.PNG)
   
**board.jsp**
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ page session="false"%>

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	<c:forEach items="${boards}" var="board">
		<p>번호: ${board.seq}</p>
		<p>제목: ${board.title}</p>
		<p>작성자: ${board.writer}</p>
		<p>내용: ${board.content}</p>
		<p>조회수: ${board.cnt}</p>
		<hr>
	</c:forEach>
</body>
</html>
```

# 8. 테스트 해보기  
1. h2 콘솔 실행시키기  
![h2 콘솔](https://user-images.githubusercontent.com/50267433/84111771-94e87280-aa62-11ea-8208-82ffafd8cfbd.PNG)   
![h2로그인](https://user-images.githubusercontent.com/50267433/84111874-b6e1f500-aa62-11ea-95d8-72604ac7d2c1.PNG)
  
2. 프로젝트에 마우스 오른쪽 클릭하시고 ```Run as``` 클릭 후 서버 실행   
![서버실행](https://user-images.githubusercontent.com/50267433/84111815-9fa30780-aa62-11ea-9377-210a6b6621fa.PNG)    

3. 주소창에 board를 추가해주자 (``` http://localhost:8080/myapp/board ```)
![데이터베이스확인](https://user-images.githubusercontent.com/50267433/84112064-06282580-aa63-11ea-87db-e817c2211506.PNG)


