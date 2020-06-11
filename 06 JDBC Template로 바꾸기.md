
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
```<context:property-placeholder location="classpath:config/database.properties"/>```는 properties를 사용하겠다는 뜻이다.    
![jdbcTemplet 사용3](https://user-images.githubusercontent.com/50267433/84334068-5ec70200-abcc-11ea-997c-184326e0360c.PNG)

   

