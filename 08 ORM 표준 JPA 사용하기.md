
ORM 표준 JPA 사용하기
==================
# 0. JPA 개념  
현대의 웹 애플리케이션에서 관계형 데이터베이스는 빠질 수 없는 요소이다.            
그러다 보니 객체를 관계형 데이터베이스에서 관리하는 것이 무엇보다 중요해졌다.         
이런 현상이 짙어지다 보니 모든 코드는 SQL 중심이 되기 시작했고          
현업 프로젝트는 애플리케이션 코드보다 SQL로 가득하게 되었다.       
     
개발자가 아무리 자바 클래스를 아름답게 설계해도 SQL을 통해야 데이터베이스를 사용할 수 있기에 피할 수 없다.         
하지만 **SQL 을 반복적으로 지속적으로 사용해야 하고 테이블이 수백개면 수백개의 SQL 코드를 작성해야한다.**      
그리고 **각각의 관계형 데이터베이스마다 쿼리문이 다르니 이는 기하 급수적으로 늘어나게 된다.**       

또 한가지 문제점이 있다. 바로 **패러다임 불일치**이다.    
* 관계형 데이터베이스 : **어떻게 데이터를 저장할지**
* 객체지향 프로그래밍 : **메시지를 기반으로 기능과 속성을 한 곳에서 관리하는 기술**

예를 들면 
```
User user = findUser();
Group group = user.getGroup();
```
위와 같이 User와 Group 부모 자식 관계로 User를 통해서 Group을 얻을 수 있지만
여기에 데이터베이스 코드가 들어가게 된다면  
```
User user = userDao.findUser();
Group group = GroupDao.findGroup(user.getGroupId());

// 상속 관계임에도 dao를 2개를 사용했고 
// 이러한 속성을 가진 값들이 많으면 많을 수록 dao를 더 만들어 사용해야 한다.   
```
상속, 1:N 등 다양한 객체 모델링을 데이터베이스로는 구현을 할 수 없어             
**각각에 DAO를 생성해주어 따로따로 조회를 해야하는 번거로움이 생긴다.**          
이렇다 보니 웹 애플리케이션 개발은 점점 데이터베이스 모델링에만 집중하게 되었다.   
     
**JPA**는 서로 지향하는 바가 다른 2개의 영역을 **중간에서 패러다임 일치**를 시켜주기 위한 기술이다.      
개발자는 객체지향적으로 프로그래밍을 하고, JPA가 이를 관계형 데이터베이스에 맞게 SQL을 대신 생성해서 실행한다.    
개발자는 객체지향 프로그래밍만 신경쓰면 되는 것으로 **SQL에 종속적인 개발을 하지 않아도 된다.**   

## 0.1. SpringData JPA 
JPA는 인터페이스로 자바 표준 명세서이다.  
즉, 인터페이스인 JPA를 사용하기 위해서는 구현체(실체)인 Hibernate, Eclipse Link등이 있다.  
  
Spring에서는 이러한 구현체를 직접 다루지 않고 이 위에 SpringData JPA 모듈을 이용하여 JPA 기술을 다룬다.   
```
JPA <- Hibernate <- SpringData JPA
```  
그럼 이렇게 사용하는 이유는 무엇이 있을까? 매번 그렇듯 유지보수를 편하기 하기 위해서이다.   
    
* 구현체 교체의 용이성  
* 저장소 교체의 용이성   

**구현체 교체의 용이성**
```
Hibernate외에 다른 구현체로 쉽게 교체하기 위함
```  
SpringData JPA 내부에서 구현체 매핑을 지원해주기 때문에    
Hibernate가 언젠가 수명을 다해서 새로운 JPA 구현체가 대세로 떠오를 경우 손쉽게 교체하기 위해서이다    .       
      
**저장소 교체의 용이성**    
```
관계형 데이터베이스 외에 다른 저장소로 쉽게 교체하기 위한 것이다.      
```   
서비스 초기에는 관계형 데이터베이스로 모든 기능을 처리했지만,     
점점 트래픽이 많아져 관계형 데이터베이스로는 도저히 감당이 안될 때 non-sql로 교체를 할 수도 있다.(Mongo DB)         
이때 개발자는 교체를 원한다면 SpringData JPA 에서 SpringData MongoDB로 의존성만 교체하면 된다.      
       
이는 SpringData의 하위 프로젝트들은 기본적으로 CRUD의 인터페이스가 같기 때문이다.     
그렇다보니 저장소가 교체되어도 기본적인 기능은 변경할 것이 없다.     
      
## 0.2. 실무에서 JPA     
실무에서 JPA를 사용하지 못하는 가장 큰 이유로 **높은 러닝 커브**를 이야기한다.      
JPA를 잘 쓰려면 객체지향 프로그래밍과 관계형 데이터베이스를 둘 다 이해해야 한다.     
(강의 사이트 : https://www.inflearn.com/instructors/74366/courses)       
      
하지만 JPA를 사용하게 되면 CRUD를 작성할 필요가 없어지고       
부모-자식 관계, 1:N 관계 표현, 상태와 행위를 한 곳에서 관리하는 등 객체지향 프로그래밍을 쉽게 할 수 있다.       
또한 속도 이슈도 없기에 많은 트래픽을 처리하는데도 사용해도 된다.       
     
# 1. 이클립스 JPA 셋팅하기 
**InteliJ와 다르게 이클립스 EE 에서는 JPA를 사용하기 위한 설정이 필요하다**   

1. 프로젝트 폴더에 마우스 오른쪽 클릭을 하고 ```Properties```를 클릭한다.     
2. 옆 배너에서 ```Project Facets```를 클릭하고 JPA 체크박스를 체크해준다.      
3. 아래에 ```Further configuration required..``` 에러 텍스트 링크가 나오는데 해당 링크를 클릭한다.   
4. 나오는 창에서 Type을 ```Uesr Library ->``` **Disable Library Configuration 으로 바꿔준다.**
5. 이후 ```Further configuration availavle..``` 이 뜨는지 확인해보고 apply 시켜준다.     
6. ```src``` -> ```main``` -> ```resources``` -> ```META-INF``` -> ```pesristence.xml``` 이 생성된 것을 확인해주자      
해당 파일은 우리가 DB 연결및 JPA를 **어떻게 동작 시킬것(How to do)** 인지에 관한 정의를 해주는 곳이다.    

![JPA 셋팅하기1](https://user-images.githubusercontent.com/50267433/84454622-052b0a00-ac96-11ea-9d40-2bc3b089a92a.PNG)
![JPA 셋팅하기2](https://user-images.githubusercontent.com/50267433/84454629-0b20eb00-ac96-11ea-9f78-8acc2d85fee3.PNG)
![JPA 셋팅하기3](https://user-images.githubusercontent.com/50267433/84454632-0f4d0880-ac96-11ea-82bf-0e5deca16e18.PNG)
![JPA 셋팅하기4](https://user-images.githubusercontent.com/50267433/84454637-183dda00-ac96-11ea-9ac5-69491ec142d3.PNG)
![JPA 셋팅하기5](https://user-images.githubusercontent.com/50267433/84454649-20961500-ac96-11ea-97a0-d386671adf81.PNG)

# 2. JPA 서비스 만들기  
기존 우리가 board를 만드는 것처럼 ```DTO, DAOInterface, DAO, Service, Controller```를 만들어 보도록 하겠습니다.     


## 2.1. User (DTO) 만들기 
1. ```myapp``` -> ```dto``` 에서 **user 폴더를 생성**해주고 **User 클래스를 만들어줍니다.**    
2. 아래와 같은 코드를 입력해줍니다.   
3. 에러가 뜨는데 User 클래스를 ```pesristence.xml```에서 사용하겠다는 선언을 안해서 그렇습니다.     
4. 선언을 위해 User가 속한 패키지 경로를 복사해놓도록 합시다. (```com.mycompany.myapp.dto.user```)     

**User**
```java
package com.mycompany.myapp.dto.user;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table
public class User {
	
	@Id
	private String id;
	
	@Column
	private String password;

	@Column
	private String name;

	@Column
	private String role;
	
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
	}
	public String getPassword() {
		return password;
	}
	public void setPassword(String password) {
		this.password = password;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getRole() {
		return role;
	}
	public void setRole(String role) {
		this.role = role;
	}
	
	@Override
	public String toString() {
		return "UserVO [id="+id+", password="+password+", name="+name+", role"+role+"]";
	} 
}
```
![JPA Service 만들기1](https://user-images.githubusercontent.com/50267433/84455522-8388ab80-ac98-11ea-9876-1e7024e0ecbe.PNG)

## 2.2. pesristence.xml 설정하기 
1. ```src``` -> ```main``` -> ```resources``` -> ```META-INF``` -> ```pesristence.xml``` 로 들어간다.
2. 아마 밑 ```General``` 탭으로 접속되어 코드가 안보일 것이니 ```Source``` 탭으로 바꿔준다.
3. 코드 내에서 ```<Persistence-unit></Persistence-unit>``` 태그 사이에 ```<class></class>``` 태그를 넣는다.
4. ```<class></class>``` 태그 사이에 복사한 User 클래스의 경로를 붙여 넣는다. (```com.mycompany.myapp.dto.user```)  

**persistence.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2" xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
	<persistence-unit name="special_lecture">
		<class>com.mycompany.myapp.dto.user.User</class>
	</persistence-unit>
</persistence>          
```
![JPA Service 만들기2](https://user-images.githubusercontent.com/50267433/84455803-3ce78100-ac99-11ea-973b-b0ccfa7e3353.PNG)

5. 그 다음에 ```<class></class>``` 태그 밑으로 ```<properties></properties>``` 태그를 만들어준다.   
6. ```<properties></properties>``` 안에는  ```<property></property>``` 태그를 이용하는데   
필수로 작성해야할  ```<properties>``` 태그가 있고 선택적으로 작성해야하는 ```<properties>```태그가 있다. 
7. 필수록 작성해야할 ```<properties>```태그 5개중에 4개는 DB연결에 관한 태그인데           
우리가 앞서 ```root-context.xml```에서 DB 연결에 관한 DataSource 객체를 등록 해주었기에 주석처리로 묶어도 된다.           
8. 아래와 같은 코드를 입력해준다.   

**persistence.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2" xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
	<persistence-unit name="special_lecture">
		<class>com.mycompany.myapp.dto.user.User</class>
		
		<properties>
			<!-- 필수 속성 -->
			
			<!-- root-context.xml에 이미 DataSource를 이용하므로 기술 안해줘도 된다. 
				<property name="javax.persistence.jdbc.driver" value="com.mysql.jdbc.Driver"/>
				<property name="javax.persistence.jdbc.user" value="root"/>
				<property name="javax.persistence.jdbc.password" value="1234"/>
				<property name="javax.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/lecture?serverTimezone=UTC"/>	
			-->
			<property name="hibernate.dialect" value="org.hibernate.dialect.MySQLDialect"/>			 
			
			<!-- 옵션 -->
			<property name="hibernate.show_sql" value="true"/>
			<property name="hibernate.format_sql" value="true"/>
			<property name="hibernate.use_sql_comments" value="false"/>
			<property name="hibernate.id.new_generator_mappings" value="true"/>
			<property name="hibernate.hbm2ddl.auto" value="create"/>
		</properties>
	
	</persistence-unit>
</persistence>
```
![JPA Service 만들기3](https://user-images.githubusercontent.com/50267433/84456043-f5152980-ac99-11ea-92df-5096475faded.PNG)
![JPA Service 만들기4](https://user-images.githubusercontent.com/50267433/84456051-fe9e9180-ac99-11ea-84b1-bee310f35c7d.PNG)

## 2.3. root-context.xml 에 JPA 관련 설정해주기          
JPA에 관련된 객체들을 ```root-context.xml```에 ```<bean>``` 등록해주자         
컨테이너에 해당 클래스를 기반으로 객체 생성(```Class class = new Class();```)명령을 하는것이다.          
   
1. ```webapp``` -> ```WEB-INF``` -> ```spring``` -> ```root-context.xml```      
2. 아래와 같은 코드들을 추가해주자.       

**추가 코드**
```xml
	<!-- Spring과 JPA 연동설정 -->
	<bean id="jpaVendorAdapter" class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter"></bean>
	
	<!-- 엔티티 매니저 팩토리 생성  -->
	<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
		<property name="dataSource" ref="dataSource"></property>
		<property name="jpaVendorAdapter" ref="jpaVendorAdapter"></property>
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
	
	<!-- Spring과 Mybatis 연동 설정 -->
	<bean id="sqlSession" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"/>
		<property name="configLocation" value="classpath:sql-map-config.xml" />
	</bean>
	
	<!-- SqlSessionTemplate 생성 -->
	<bean class="org.mybatis.spring.SqlSessionTemplate">
		<constructor-arg ref="sqlSession"></constructor-arg>
	</bean>
	
	<!-- Spring과 JPA 연동설정 -->
	<bean id="jpaVendorAdapter" class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter"></bean>
	
	<!-- 엔티티 매니저 팩토리 생성  -->
	<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
		<property name="dataSource" ref="dataSource"></property>
		<property name="jpaVendorAdapter" ref="jpaVendorAdapter"></property>
	</bean>
		
</beans>
```
![JPA Service 만들기5](https://user-images.githubusercontent.com/50267433/84456359-c055a200-ac9a-11ea-892a-b1fc719571e8.PNG)
