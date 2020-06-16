MyBatis 사용하기
==================
# 1. Mysql로 전환하기  
      
1. config 폴더에 있는 ```database.properties``` 를 클릭해준다.         
2. 아래와 같은 코드로 수정을 해주자     
       
**database.properties**    
```   
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/lecture?serverTimezone=UTC
jdbc.username=root
jdbc.password=1234
```
3. 다시 서버를 구동시켜서 확인해보면 데이터베이스가 바뀐것을 알 수 있다.       
       
![mysql 전환](https://user-images.githubusercontent.com/50267433/84229450-88c4e980-ab24-11ea-930d-80a0fd7f4040.PNG)
![mysql 전환2](https://user-images.githubusercontent.com/50267433/84229479-a72ae500-ab24-11ea-92d2-e0f5a80abc2e.PNG)

# 2. MyBatis 시작하기    
## 2.1. Mapper xml 정의하기  
1. 프로젝트에 마우스를 대고 오른쪽 클릭후 new -> other 클릭  
2. mybatis를 검색하고 mybatis mapper xml 클릭 
3. board-mapping.xml 이라 입력을 해주자 (내가 이름 지정한 것이다)   
4. board-mapping.xml 생성되었다.   
5. 바깥쪽에 생성되었기에 ```src -> main -> resources``` 에 ```mappings``` 폴더를 만든다
6. mappings 폴더에 board-mapping.xml를 드래그해서 넣어주고 아래 코드로 입력해주자    
    
**board-mapping**
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="BoardDAO">
	<resultMap id="boardResult" type="board">
		<id property="seq" column="SEQ" />
		<result property="title" column="TITLE" />
		<result property="writer" column="WRITER" />
		<result property="content" column="CONTENT" />
		<result property="regDate" column="REGDATE" />
		<result property="cnt" column="CNT" />
	</resultMap>
	
	<insert id="insert">
		INSERT INTO board(seq, title, writer, content) VALUES((select nvl(max(seq), 0)+1 from board),#{title},#{writer},#{content})
	</insert>

	<update id="update">
		UPDATE board SET title=#{title}, content=#{content} WHERE seq=#{seq}
	</update>

	<delete id="delete">
		DELETE FROM board WHERE seq=#{seq}
	</delete>

	<select id="getOne" resultType="board">
		SELECT * FROM board WHERE seq=#{seq}
	</select>
		
	<select id="getAll" resultType="board">
		SELECT * FROM board
	</select>
</mapper>
```
![mybatis 생성](https://user-images.githubusercontent.com/50267433/84231084-52896900-ab28-11ea-9d47-f11fd6c1b307.png)
![mybatis 생성2](https://user-images.githubusercontent.com/50267433/84231094-587f4a00-ab28-11ea-9055-f56b3e7b3b92.PNG)
![mybatis 생성3](https://user-images.githubusercontent.com/50267433/84231117-6208b200-ab28-11ea-8c46-ec3ec6393511.PNG)
![mybatis 생성4](https://user-images.githubusercontent.com/50267433/84231649-8e70fe00-ab29-11ea-8edc-1e614d77aee5.PNG)
![board-mapper-1](https://user-images.githubusercontent.com/50267433/84234308-f249f580-ab2e-11ea-8c76-a43ba9ec3f0d.PNG)

## 2.2. Configuration xml 정의하기    

1. 프로젝트에 마우스를 대고 오른쪽 클릭후 new -> other 클릭  
2. mybatis를 검색하고 mybatis configuration xml 클릭 
3. sql-map-config.xml 이라 입력을 해주자 (실무에서 주로 사용하는 이름)   
4. sql-map-config.xml 와 db.properties가 생성되었다.
5. db.properties는 우리가 기존에 config 폴더에 database.properties를 생성해주었으니 삭제하자  
6. 바깥쪽에 생성되었기에 ```src -> main -> resources``` 로 sql-map-config.xml를 넣어주자  
7. 아래 코드로 입력해주자    
    
```xml   
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<properties resource="config/database.properties" />
	<typeAliases>
		<typeAlias type="com.mycompany.myapp.dto.board.Board" alias="board"></typeAlias>
	</typeAliases>
	<mappers>
		<mapper resource="mappings/board-mapping.xml" />
	</mappers>
</configuration>
```
![mybatis 생성](https://user-images.githubusercontent.com/50267433/84231084-52896900-ab28-11ea-9d47-f11fd6c1b307.png)
![configuration 생성1](https://user-images.githubusercontent.com/50267433/84232578-8b770d00-ab2b-11ea-90a9-5965613c8400.PNG)
![configuration 생성2](https://user-images.githubusercontent.com/50267433/84232586-8fa32a80-ab2b-11ea-8f5c-ada2a87e6c23.PNG)
![configuration 생성3](https://user-images.githubusercontent.com/50267433/84232597-9467de80-ab2b-11ea-9ccd-977c17516e18.PNG)
![configuration 생성4](https://user-images.githubusercontent.com/50267433/84232604-992c9280-ab2b-11ea-8cc5-fee2b92c1f64.PNG)
![configuration 생성5](https://user-images.githubusercontent.com/50267433/84232610-9d58b000-ab2b-11ea-93a1-998453312bb6.PNG)

## 2.3. SqlSession 객체 생성하기 
MyBatis를 이용해서 DAO를 구현하려면 SqlSession 객체가 필요하다.        
SqlSession 객체는 스프링 설정 파일에 SqlSessionFactoryBean 클래스를 Bean 등록해야한다.       
그래야 SqlSessionFactoryBean 객체로부터 DB 연동 구현에 사용한 SqlSession 객체를 얻을 수 있다.      
      
1. webapp -> WEB-INF -> spring -> root-context.xml 에 들어간다.
2. 아래와 같은 코드를 입력해주자  

```xml
	<!-- Spring과 Mybatis 연동 설정 -->
	<bean id="sqlSession" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"/>
		<property name="configLocation" value="classpath:sql-map-config.xml" />
	</bean>
	
	<!-- SqlSessionTemplate 생성 -->
	<bean class="org.mybatis.spring.SqlSessionTemplate">
		<constructor-arg ref="sqlSession"></constructor-arg>
	</bean>
```

**전체 root-context.xml**
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
	
	<!-- Spring과 Mybatis 연동 설정 -->
	<bean id="sqlSession" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"/>
		<property name="configLocation" value="classpath:sql-map-config.xml" />
	</bean>
	
	<!-- SqlSessionTemplate 생성 -->
	<bean class="org.mybatis.spring.SqlSessionTemplate">
		<constructor-arg ref="sqlSession"></constructor-arg>
	</bean>
		
</beans>
```
![root xml 설정](https://user-images.githubusercontent.com/50267433/84234384-20c7d080-ab2f-11ea-9755-7acb5c2e3d3f.PNG)

## 2.4. BoardDAOMyBatis 클래스 만들기   
바로 앞에서 만든 SqlSession 객체는 mapper를 의미한다.      
다시 설명하자면 SqlSession -> sql-mpa-config.xml의 정보를 읽어서 board-mapper.xml의 정보를 가진 객체이다.     
즉, SqlSession 은 board-mapper.xml의 객체라고 보면된다.     
그렇기에 이를 의존성 주입으로 입력 받은뒤 해당 객체를 실행해주는 것이 MyBatis를 사용하는 방법이다.   
   
1. myapp -> dao -> board 에 BoardDAOMyBatis 클래스를 생성하자  
2. 아래와 같은 코드를 입력해주자  
3. **이전에 BoardDAOTemplates에 있던 @Repository를 주석을 달아주자**  

**BoardDAOMyBatis**   
```java
package com.mycompany.myapp.dao.board;

import java.util.List;

import org.mybatis.spring.SqlSessionTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import com.mycompany.myapp.dto.board.Board;

@Repository
public class BoardDAOMyBatis implements BoardDAO{
	
	@Autowired
	private SqlSessionTemplate mybatis;

	public int insert(Board vo) {
		return mybatis.insert("BoardDAO.insert", vo);
	}

	public int update(Board vo) {
		return mybatis.update("BoardDAO.update", vo);
	}

	public int delete(Board vo) {
		return mybatis.delete("BoardDAO.delete", vo);
	}

	public Board getOne(Board vo) {
		return (Board)mybatis.selectOne("BoardDAO.getOne", vo);
	}

	public List<Board> getAll() {
		return mybatis.selectList("BoardDAO.getAll");
	}
}
```
![mybatis dao](https://user-images.githubusercontent.com/50267433/84234869-e90d5880-ab2f-11ea-9190-975957d0eddc.PNG)     
![mybatis dao2](https://user-images.githubusercontent.com/50267433/84234896-f591b100-ab2f-11ea-8815-cfa9983175a2.PNG)
![마이배티스결과](https://user-images.githubusercontent.com/50267433/84235184-5f11bf80-ab30-11ea-8d11-51b49c911a94.PNG)
        



