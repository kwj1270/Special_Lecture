# Mysql로 전환하기  
      
1. config 폴더에 있는 ```database.properties``` 를 클릭해준다.         
2. 아래와 같은 코드로 수정을 해주자     
```   
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/lecture?serverTimezone=UTC
jdbc.username=root
jdbc.password=1234
```
3. 다시 서버를 구동시켜서 확인해보면 데이터베이스가 바뀐것을 알 수 있다.       
       
![mysql 전환](https://user-images.githubusercontent.com/50267433/84229450-88c4e980-ab24-11ea-930d-80a0fd7f4040.PNG)
![mysql 전환2](https://user-images.githubusercontent.com/50267433/84229479-a72ae500-ab24-11ea-92d2-e0f5a80abc2e.PNG)
