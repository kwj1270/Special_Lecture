
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
     
# 1. JPA 셋팅하기 
**InteliJ와 다르게 이클립스 EE 에서는 JPA를 사용하기 위한 설정이 필요하다**   

1. 프로젝트 폴더에 마우스 오른쪽 클릭을 하고 ```Properties```를 클릭한다.     
2. 옆 배너에서 ```Project Facets```를 클릭하고 JPA 체크박스를 체크해준다.      
3. 아래에 Further configuration required.. 에러 텍스트 링크가 나오는데 해당 링크를 클릭한다.   
4. 나오는 창에서 Type을 Uesr Library -> **Disable Library Configuration 으로 바꿔준다.**
5. 이후 Further configuration availavle.. 이 뜨는지 확인해보고 apply 시켜준다.     
6. src -> main -> resources -> META-INF -> pesristence.xml 이 생성된 것을 확인해주자      
해당 파일은 우리가 DB 연결및 JPA를 **어떻게 동작 시킬것(How to do)** 인지에 관한 정의를 해주는 곳이다.    

![JPA 셋팅하기1](https://user-images.githubusercontent.com/50267433/84454622-052b0a00-ac96-11ea-9d40-2bc3b089a92a.PNG)
![JPA 셋팅하기2](https://user-images.githubusercontent.com/50267433/84454629-0b20eb00-ac96-11ea-9f78-8acc2d85fee3.PNG)
![JPA 셋팅하기3](https://user-images.githubusercontent.com/50267433/84454632-0f4d0880-ac96-11ea-82bf-0e5deca16e18.PNG)
![JPA 셋팅하기4](https://user-images.githubusercontent.com/50267433/84454637-183dda00-ac96-11ea-9ac5-69491ec142d3.PNG)
![JPA 셋팅하기5](https://user-images.githubusercontent.com/50267433/84454649-20961500-ac96-11ea-97a0-d386671adf81.PNG)

