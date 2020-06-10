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
      
	<insert id="insert">
		INSERT INTO board(seq, title, writer, content) VALUES((select nvl(max(seq), 0)+1 from board),#{title},#{writer},#{content})
	</insert>

	<update id="update">
		UPDATE board SET title=#{title}, content=#{content} WHERE seq=#{seq}
	</update>

	<delete id="delete">
		DELETE board WHERE seq = #{seq}
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
![board-mapper-1](https://user-images.githubusercontent.com/50267433/84231662-95980c00-ab29-11ea-853d-d5d1d5ddfd2c.PNG)

