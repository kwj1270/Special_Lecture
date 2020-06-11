
JDBC Template로 바꾸기
=======================
# 1. JDBCTemplate 클래스
JdbcTemplate은 GoF 디자인 패턴 중 템플릿 메소드 패턴이 적용된 클래스입니다.         
템플릿 메소드 패턴은 복잡하고 반복되는 알고리즘을 캡슐화해서 재사용하는 패턴으로 정의할 수 있습니다.       
    
JdbcTemplate은 JDBC의 반복적인 코드를 제거하기 위해 제공하는 클래스입니다.  
반복되는 DB 연동 로직은 JdbcTemplate 클래스의 템플릿 메소드가 제공하고,  
개발자는 달라지는 SQL 구문과 설정값만 신경쓰면 됩니다.  
    
```
간략히 말하면 기존에 우리가 사용하던 JDBC 방법에서 
Connection 연결과 같이 반복되는 부분을 클래스에서 알아서 처리해주고 
우리는 SQL문만 신경쓰면 됩니다.
```
***
# 2. 스프링 JDBC 설정
## 2.1. 라이브러리 추가    
스프링 JDBC를 이용하려면 BoardWeb 프로젝트에 있는 ```pom.xml``` 파일에 DBCP 관련 ```<depencency>``` 설정을 추가해야 한다.  
**기존에 제공했던 ```pom.xml```에 이미 추가해줬으니 있는지만 확인 부탁드립니다**   
     
**pom.xml**
```		<!-- jdbc -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<!-- DBCP -->
		<dependency>
			<groupId>commons-dbcp</groupId>
			<artifactId>commons-dbcp</artifactId>
			<version>1.4</version>
		</dependency>		
```  
## 2.2. DataSource       
DB 연동을 처리하려면 반드시 데이터베이스로부터 커넥션을 얻어야합니다.            
이러한 커넥션을 얻어주는 객체인 DataSource를 ```<bean>```등록하여 스프링 컨테이너가 생성하도록 합시다.           
DataSource 설정은 스프링뿐만 아니라 **트랜잭션 처리**나 **mybatis연동**, **JPA 연동**에서도 사용되므로      
스프링으로 프로젝트를 개발할 경우 거의 필수적으로 설정해주시는 것을 권유드립니다.         
    
1. webapp -> WEB-INF -> spring -> root-context.xml
2. 아래와 같은 코드를 입력해 줍니다.  
**root-context.xml**
```xml
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="org.h2.Driver"/>
		<property name="url" value="jdbc:h2:tcp://localhost/~/test" />
		<property name="username" value="sa"/>
		<property name="password" value=""/>
	</bean>
```
![jdbcTemplet 사용1](https://user-images.githubusercontent.com/50267433/84333511-ed3a8400-abca-11ea-82a4-d0505def131c.PNG)    
소스 코드를 해석하자면 기존에 우리가 JDBC에서 사용했던 방법을 XML로 풀어쓴거라 생각하면 된다.           
    
## 2.3. DataSource 를 쉽게 관리하는 database.properties 
database.properties는 데이터베이스 연결에 필요한 값들을 미리 저장해놓는 파일입니다.      
우리가 기존 JDBC에서 Connnection을 얻기 위한 값들을 저장해 놓은 파일이라 보시면 됩니다.           
           
1. main -> resources 에서 config 폴더를 생성
2. config 폴더에서 마우스 오른쪽 클릭 -> new -> file
3. 파일명을 database.properties 로 입력 
4. 아래와 같은 코드를 입력  
**database.properties**
```
jdbc.driver=org.h2.Driver
jdbc.url=jdbc:h2:tcp://localhost/~/test
jdbc.username=sa
jdbc.password=
```
![jdbcTemplet 사용2](https://user-images.githubusercontent.com/50267433/84333901-f8da7a80-abcb-11ea-9590-1cd9a4e1b8f6.PNG)   

5. 그리고 다시 webapp -> WEB-INF -> spring -> root-context.xml 의 값을 아래와 같이 바꿔줍시다.  

**root-context.xml**
```xml
  <context:property-placeholder location="classpath:config/database.properties"/>
  
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="${jdbc.driver}"/>
		<property name="url" value="${jdbc.url}" />
		<property name="username" value="${jdbc.username}"/>
		<property name="password" value="${jdbc.password}"/>
	</bean>
```   
   
![jdbcTemplet 사용3](https://user-images.githubusercontent.com/50267433/84334068-5ec70200-abcc-11ea-997c-184326e0360c.PNG)   
```<context:property-placeholder location="classpath:config/database.properties"/>```는 properties를 사용하겠다는 뜻이다.    

## 2.4. JDBCTemplate 객체 생성하기  
JDBCTemplate 에 사용되는 DataSource 객체를 만들어 주었으니 JDBCTemplate 객체를 만들어 줍시다.  

1.  webapp -> WEB-INF -> spring -> root-context.xml 에 아래와 같은 코드를 추가해줍니다.        
**추가할 코드**     
```xml
	<!-- Spring JDBC 설정 -->
	<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
		<property name="dataSource" ref="dataSource"/>
	</bean>	
```

**root-context.xml 전체 코드**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">
	
	<!-- Root Context: defines shared resources visible to all other web components -->
	
  <context:property-placeholder location="classpath:config/database.properties"/>
  
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="${jdbc.driver}"/>
		<property name="url" value="${jdbc.url}" />
		<property name="username" value="${jdbc.username}"/>
		<property name="password" value="${jdbc.password}"/>
	</bean>
	
	<!-- Spring JDBC 설정 -->
	<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
		<property name="dataSource" ref="dataSource"/>
	</bean>	
		
</beans>
```
![jdbcTemplet 사용4](https://user-images.githubusercontent.com/50267433/84334276-fc223600-abcc-11ea-9dae-9291ea062281.PNG)    
```
이제 우리는 JDBC대신에 이를 간략화 시켜줄 수 있는 스프링 JDBCTemplate을 이용할 준비가 되었습니다!!!
```
    
***
# 3. JDBCTemplate 을 이용한 BoardDAOTemplate 생성   

1. myapp -> dao -> board 에서 BoardDAOTemplate 클래스를 생성해준다.         
2. 아래 코드를 입력해줍니다.          
3. 이제 기존 BoardDAOJDBC 에서 ```@Repository("boardDAOJDBC")```를 주석처리를 해줍니다.        

**BoardDAOTemplate**
```java
package com.mycompany.myapp.dao.board;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.stereotype.Repository;

import com.mycompany.myapp.dto.board.Board;

@Repository
public class BoardDAOTemplate implements BoardDAO{
	
	@Autowired
	private JdbcTemplate jdbcTemplate;
	
	// SQL 명령어들
	private final String BOARD_INSERT = "insert into board(seq, title, writer, content) values((select nvl(max(seq), 0)+1 from board),?,?,?)";
	private final String BOARD_UPDATE = "update board set title=?, content=? where seq=?";
	private final String BOARD_DELETE = "delete board where seq=?";
	private final String BOARD_GET = "select * from board where seq=?";
	private final String BOARD_LIST = "select * from board order by seq desc";

	// CRUD 기능의 메소드 구현
	// 글 등록
	@Override
	public void insert(Board vo) {
		System.out.println("===> Spring JDBC로 insertBoard() 기능 처리");
		jdbcTemplate.update(BOARD_INSERT, vo.getTitle(), vo.getWriter(), vo.getContent());
	}

	// 글 수정
	@Override
	public void update(Board vo) {
		System.out.println("===> Spring JDBC로 updateBoard() 기능 처리");
		jdbcTemplate.update(BOARD_UPDATE, vo.getTitle(), vo.getContent(), vo.getSeq());
	}

	// 글 삭제
	@Override
	public void delete(Board vo) {
		System.out.println("===> Spring JDBC로 deleteBoard() 기능 처리");
		jdbcTemplate.update(BOARD_DELETE, vo.getSeq());
	}

	// 글 상세 조회
	@Override
	public Board getOne(Board vo) {
		System.out.println("===> Spring JDBC로 getBoard() 기능 처리");
		Object[] args = { vo.getSeq() };
		return jdbcTemplate.queryForObject(BOARD_GET, args, new BoardRowmapper());
	}

	// 글 목록 조회
	@Override
	public List<Board> getAll() {
		System.out.println("===> Spring JDBC로 getBoardList() 기능 처리");
		return jdbcTemplate.query(BOARD_LIST, new BoardRowmapper());
	}
}

class BoardRowmapper implements RowMapper<Board> {
	public Board mapRow(ResultSet rs, int rowNum) throws SQLException {
		Board board = new Board();
		board.setSeq(rs.getInt("SEQ"));
		board.setTitle(rs.getString("TITLE"));
		board.setWriter(rs.getString("WRITER"));
		board.setContent(rs.getString("CONTENT"));
		board.setRegDate(rs.getDate("REGDATE"));
		board.setCnt(rs.getInt("CNT"));
		return board;
	}
}
```
![jdbcTemplet 사용5](https://user-images.githubusercontent.com/50267433/84334495-75ba2400-abcd-11ea-9d65-65d3baeb2ce3.PNG)     
![jdbcTemplet 사용6](https://user-images.githubusercontent.com/50267433/84334644-ce89bc80-abcd-11ea-98f7-a1e9ea8dc7bc.PNG)
