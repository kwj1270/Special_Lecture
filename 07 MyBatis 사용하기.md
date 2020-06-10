# Mysql로 전환하기  
      
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

# MyBatis 시작하기    
![mybatis 생성](https://user-images.githubusercontent.com/50267433/84231084-52896900-ab28-11ea-9d47-f11fd6c1b307.png)
![mybatis 생성2](https://user-images.githubusercontent.com/50267433/84231094-587f4a00-ab28-11ea-9055-f56b3e7b3b92.PNG)
![mybatis 생성3](https://user-images.githubusercontent.com/50267433/84231117-6208b200-ab28-11ea-8c46-ec3ec6393511.PNG)
