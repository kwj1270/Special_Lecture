# 1. 게시판 디렉토리 및 게시판 DTO 생성
myapp 밑으로 dto -> board -> Board.class 를 만든다.  
![게시판 만들기1](https://user-images.githubusercontent.com/50267433/84029768-e4c92a00-a9cd-11ea-89d0-9d42865312c2.PNG)

# 1.1. Getter/Setter 및 생성자를 자동으로 만들어주는 제네레이터 사용하기 
**Board**
```
package com.mycompany.myapp.dto.board;

import java.sql.Date;

public class Board {
	private int seq;
	private String title;
	private String writer;
	private String content;
	private Date regDate;
	private int cnt;

}
```
우선 위와 같은 코드를 작성한 뒤에 ```Alt+Shift+S```를 눌러준다.   
![제네러이터](https://user-images.githubusercontent.com/50267433/84098539-76727f00-aa42-11ea-95db-bfb913f2fe73.PNG)

1. 우리가 만들어낸 변수를 기반으로 Getter/Setter 메소드를 만들어 준다.     
2. 우리가 만들어낸 변수를 기반으로 생성자를 만들어 준다.      
      
우선 Getter/Setter 메소드를 만들어주자   
![제네러이터2](https://user-images.githubusercontent.com/50267433/84099465-a3279600-aa44-11ea-8714-adc34609185f.PNG)   
![제네러이터3](https://user-images.githubusercontent.com/50267433/84099473-aae73a80-aa44-11ea-8b09-3751744d1e7b.PNG)     
   
그리고 객체에 들어간 값을 확인할 수 있게 아래 코드를 붙여넣어주자  
```java
	@Override
	public String toString() {
		return "BoardVO [seq="+seq+", title="+title+", writer="+writer+
			   ", content="+content+", regDate="+regDate+", cnt="+cnt+"]";
	}
```
**전체 코드**
```java
package com.mycompany.myapp.dto.board;

import java.sql.Date;

public class Board {
	private int seq;
	private String title;
	private String writer;
	private String content;
	private Date regDate;
	private int cnt;
	
	public int getSeq() {
		return seq;
	}
	public void setSeq(int seq) {
		this.seq = seq;
	}
	public String getTitle() {
		return title;
	}
	public void setTitle(String title) {
		this.title = title;
	}
	public String getWriter() {
		return writer;
	}
	public void setWriter(String writer) {
		this.writer = writer;
	}
	public String getContent() {
		return content;
	}
	public void setContent(String content) {
		this.content = content;
	}
	public Date getRegDate() {
		return regDate;
	}
	public void setRegDate(Date regDate) {
		this.regDate = regDate;
	}
	public int getCnt() {
		return cnt;
	}
	public void setCnt(int cnt) {
		this.cnt = cnt;
	}
	
	@Override
	public String toString() {
		return "BoardVO [seq="+seq+", title="+title+", writer="+writer+
			   ", content="+content+", regDate="+regDate+", cnt="+cnt+"]";
	}
	
}
```

Getter/Setter 사용이유는?      
우리는 변수를 private로 선언을 했다.    
private 선언은 해당 클래스내에서만 접근이 가능하기에   
```java
Board board = new board();   
board.title = "제목";    <- 여기가 문제  
```
위와 같은 코드를 사용 못하게 막는 것이고       
```java
Board board = new board();   
board.setTitle("제목");  
```   
set변수명 이므로 이렇게 사용함으로써 **내가 set 설정하겠다 Title을** 이라는 뜻으로      
메소드 이름만 봐도 어떻게 동작할것인지 예상이 가능하고 내가 넣으려는 데이터가 명확해진다.       
무엇보다 외부에서 변수를 접근 못하게 한다 이와 같은 기법을 **정보 은닉**이라고 한다.          
          
또한 나중에 스프링 컨테이너에서 이름이 같을 경우 자동으로 데이터를 넣어주는데 기준이 이 Setter메소드이다.    
(나중에 설명할 것이니 일단 이름이 같아야 된다는 것만 알자)     
```html
<input type="text" name="title">
<input type="text" name="writer">
<input type="text" name="content">
```
```java
public class Board {
	~ 생략 ~
	private String title;
	private String writer;
	private String content;
	~ 생략 ~
}
```      

