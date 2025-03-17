
<img src="https://capsule-render.vercel.app/api?type=waving&color=FFE146&height=300&section=header&text=Monitoring&fontSize=70&fontColor=FFFFFF&animation=fadeIn&width=1200" width="1200" />


<br>


<br>

## 📍 Contents
- [1️⃣ Contributors](#1%EF%B8%8F⃣-contributors)
- [2️⃣ Overview](#2%EF%B8%8F⃣-overview)
- [3️⃣ Business Model](#3%EF%B8%8F⃣-business-model)
- [4️⃣ Expectations](#4%EF%B8%8F⃣-expectations)
- [5️⃣ Skills & Architecture](#5%EF%B8%8F⃣-skills--architecture)
- [6️⃣ Trouble Shooting](#6%EF%B8%8F%E2%83%A3-trouble-shooting)
- [7️⃣ Retrospective](#7%EF%B8%8F%E2%83%A3-retrospective)

<br>
<br>

## 1️⃣ Contributors
<br>

|<img src="https://avatars.githubusercontent.com/u/80048007?v=4" width="220" height="200"/>|<img src="https://avatars.githubusercontent.com/u/60309978?v=4" width="220" height="200"/>|<img src="https://avatars.githubusercontent.com/u/193213283?v=4" width="220" height="200"/>|<img src="https://avatars.githubusercontent.com/u/115103394?v=4" width="220" height="200"/>|
|:-:|:-:|:-:|:-:|
|박영진<br/>[@DoomchitYJ](https://github.com/DoomchitYJ)|임하은<br/>[@imhaeunim](https://github.com/imhaeunim)|박진현<br/>[@jinhyunpark929](https://github.com/jinhyunpark929)|최윤정<br/>[@letmeloveyou82](https://github.com/letmeloveyou82)|

<br>

## 2️⃣ Overview

<br> 

### 💡 Background<br>

<br>


## 3️⃣ Business Model

<br>

## 4️⃣ Expectations

<br>

## 5️⃣ Skills & Architecture

### 🛠 Skills

**Tech Stack**

<div>

<img src="https://img.shields.io/badge/html-E34F26?style=for-the-badge&logo=html5&logoColor=white">
<img src="https://img.shields.io/badge/css-1572B6?style=for-the-badge&logo=css3&logoColor=white">
<img src="https://img.shields.io/badge/javascript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black"> 
<br>
<img src="https://img.shields.io/badge/oracle-F80000?style=for-the-badge&logo=oracle&logoColor=white">
<img src="https://img.shields.io/badge/java-007396?style=for-the-badge&logo=java&logoColor=white">
<img src="https://img.shields.io/badge/apache tomcat-F8DC75?style=for-the-badge&logo=apachetomcat&logoColor=white">
</div>

<br>

**Co-work Stack**

<div>
<img src="https://img.shields.io/badge/notion-000000?style=for-the-badge&logo=notion&logoColor=white"> 
<img src="https://img.shields.io/badge/github-303a50?style=for-the-badge&logo=github&logoColor=white">
<img src="https://img.shields.io/badge/slack-e01e5a?style=for-the-badge&logo=slack&logoColor=white">
</div>

<br>

### 🖥️ Architecture

**☁️ ERD**

![image](https://github.com/user-attachments/assets/922edc9b-e8a0-47fc-ad12-d149e3ac8a50)

<br>

**💻 System Architecture**

![image](https://github.com/user-attachments/assets/40f13681-7695-48f2-bb87-e5b650a0d767)


<br>


## 6️⃣ Trouble Shooting

### ⚔ 로그인을 해도 ‘나의 수업’, '로그아웃’ 헤더가 보이지 않는 문제

```java
@WebServlet("/login")
public class LoginController extends HttpServlet {
	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		
		System.out.println("***************");
		String inputId = request.getParameter("id");
		String inputPw = request.getParameter("pw");
		
		if (validateAccount(inputId, inputPw)) {
			Cookie userIdCookie = new Cookie("userId", inputId);
			Cookie userPwCookie = new Cookie("userPw", inputPw);
			userIdCookie.setMaxAge(60*60);
			userPwCookie.setMaxAge(60*60);
			userIdCookie.setPath("/");
			userPwCookie.setPath("/");
			
			response.addCookie(userIdCookie);
			response.addCookie(userPwCookie);
			
			request.getRequestDispatcher("view/main.jsp").forward(request, response);
		} else {
			System.out.println("test");
			request.setAttribute("errorMessage", "등록되지 않은 아이디 혹은 비밀번호입니다.");
            response.sendRedirect("view/login.html"); 
		}
	}
```

**🧨원인:**

forward 방식은 동일한 HTTP 요청을 유지하므로 로그인 정보(쿠키)가 이전의 상태(로그인 요청 전)를 유지하게 된다.

따라서, 새로운 쿠키(로그인 정보)를 가지고 새로운 HTTP 요청을 보내는 redirect 방식을 이용해야 한다.

<br>

> **forward vs redirect 차이점**
>
>| **구분** | **forward** | **redirect** |
>| --- | --- | --- |
>| **HTTP 요청 개수** | 1번 (동일한 요청 유지) | 2번 (새로운 요청 발생) |
>| **URL 변경 여부** | 변경되지 않음 (로그인 URL 유지됨) | 변경됨 (로그인 후 `main.jsp` URL로 바뀜) |
>| **서버 요청 방식** | 서버 내부에서 JSP로 제어 이동 | 클라이언트에게 새로운 URL 요청을 보냄 |
>| **새로고침 시 동작** | 로그인 정보가 유지됨 (재요청 가능) | 새로고침해도 로그인 요청이 다시 발생하지 않음 |
>
<br>

**👌 해결방안**

```response.sendRedirect("view/login.html");```

<br>

## 7️⃣ Retrospective

