

#### `MyfirstwebappApplication`
애플리케이션을 실행하는 파일

#### `pom.xml`
애플리케이션에 관련된 모든 의존성을 정의하는곳
모든 Spring Boot 프로젝트는 starter-test에 대한 의존성을 갖게 됩니다.
단위 테스트를 작성하게 도와준다.

#### `application.properties`
애플리케이션의 많은 세부정보를 설정
포트설정도 이파일에서 설정할수있다.

---------
## Spring MVC 컨트롤러 `@ResponseBody `,`@Controller`

```java title="SayHelloController"
package com.in28minutes.springboot.myfirstwebapp.hello;  
  
import org.springframework.stereotype.Controller;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RestController;  
  
  
@Controller
public class SayHelloController {  
  
    //"say-hello"=> "Hello! What are you learning today?"  
  
    @RequestMapping("say-hello")  
    public String sayHello() {  
        return "Hello! What are you learning today?";  
    }  
}
```

 `@Controller`를 사용하고 `@RequestMapping`시 반환값은 URL을 리턴하면 뷰 이름으로 해석됨.
 ViewResolver가 "Hello! What  are you learning today?" 대한 뷰를 찾아서 렌더링하지만 없기 때문에 오류발생
 ![[Pasted image 20250325160530.png]]
 그래서 문자열 그대로 웹창에 보여주고 싶을때 `@ResponseBody`를 이용하면 리턴하것을 그대로 브라우저에 보여줌.



# HTML을 이용해서 Spring MVC Controller 개선하기

```java title="SayHelloController"
@Controller  
public class SayHelloController {  
    @RequestMapping("say-hello-html")  
    @ResponseBody    public String sayHelloHtml() {  
        StringBuffer sb = new StringBuffer();  
        sb.append("<html>");  
        sb.append("<head>");  
        sb.append("<title> My first HTML Page</title>");  
        sb.append("</head>");  
        sb.append("<body>");  
        sb.append("My first html page with body");  
        sb.append("</body>");  
        sb.append("</html>");  
        return sb.toString();  
    }  
}
```
![[Pasted image 20250325161213.png]]위처럼 하드코딩해서  아주 간단한 HTML 페이지를 만들어보았다.  하지만 아주 간단한 페이지도 이렇게 복잡한데 복잡한 HTML 구성시 컨트롤러에서는 작성을 하지 못한다.


# 뷰를 이용하여 JSP로 리다이렉션하기

### JSP(Java Server Page)
가장 오래된 뷰 기술이고, 아직도 널리 사용되는 기술 

##### 일반적으로 모든 JSP는 resources/META-iNF/resources/WEB-INF/jsp에 저장

```java title="SayHelloController"
package com.in28minutes.springboot.myfirstwebapp.hello;  
  
import org.springframework.stereotype.Controller;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.ResponseBody;  
  
@Controller  
public class SayHelloController {  
    //"say-hello-jsp" => sayHello.jsp    @RequestMapping("say-hello-jsp")  
    public String sayHelloJsp() {  
        return "say-hello-jsp";  
    }  
}
```

JSP로 해당 HTML 코드를 입력해두고 
``` jsp title="sayHello.jsp"
<!doctype html>  
<html lang="en">  
<head>  
    <title>My first HTML Page</title>  
</head>  
<body>  
 My first html page with body  
</body>  
</html>
```
#### ViewResovler 설정하기-> 
spring은` /src/main/resources/META-INF/resources` 이미 이 경로는 알고있다.
 그렇기때문에 생략하고 `/WEB-INF/jsp/`만 포함하면 된다.
``` title="application.properties"
spring.application.name=myfirstwebapp  
#server.port=8081  
spring.mvc.view.prefix=/WEB-INF/jsp/  
spring.mvc.view.suffix=.jsp
```

하지만 실행해도 오류가 발생하는데 여기서 `pom.xml `의존성하나 추가해주고 가자
``` xml title="pom.xml"
<dependency>  
    <groupId>org.apache.tomcat.embed</groupId>  
    <artifactId>tomcat-embed-jasper</artifactId>  
</dependency>
```

해당 의존성은 내장 Tomcat을 사용할때 JSP를 컴파일하고 실행할수 있도록 도와줌.
이유는? 기본적으로 SpringBoot는 JSP를 권장하지 않으며, spring-boot-starter-web만으로는 JSP를 사용할수 없음. `Tomcat이` JSP파일을 처리하려면 `Jasper`(JSP 엔진)가 필요함.

----
#### 웹의 작동방식: 요청과 응답

브라우저가 애플리케이션에거 요청을 전송하게 됩니다. 이 요청을 HTTP Request라고 합니다.
그 Request를 서버 또는 애플리케이션은 요청을 받으면 그 요청을 처리하게됩니다.
그리고 Response를 다시 브라우저에게 전달하는데 보통 HTML 형식이죠 이걸 HTTP Response라고 합니다.

-----
### RequestParam으로 QueryParameter 잡기 

```
localhost:8080/login?name=Ranga
```
물음표를 기점으로 name=Ranga는 `LoginController로` 넘어간다.
``` java title="LoginController"
package com.in28minutes.springboot.myfirstwebapp.login;  
  
import org.springframework.stereotype.Controller;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RequestParam;  
  
@Controller  
public class LoginController {  
    @RequestMapping("/login")  
    public String gotoLoginPage(@RequestParam String name) {  
        System.out.println("RequestParm is  " + name); //NOT RECOMMEND FOR CODE  
        return "login";  
    }  
}
```
URL에서 값을 받아서 Controller 코드에 전달 할수 있습니다. 이렇게 전달받으면 보통은 JSP에 전달을 해줍니다.
여기서 알아야되는 개념은 `Model이다` . `Controller에서` JSP로 전달하려고 할때, 무조건 `Model`을 통해서 전달해야한다.

 ```java
 package com.in28minutes.springboot.myfirstwebapp.login;  
  
import org.springframework.stereotype.Controller;  
import org.springframework.ui.ModelMap;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RequestParam;  
  
@Controller  
public class LoginController {  
    /// localhost:8080/login/  
    @RequestMapping("/login")  
    public String gotoLoginPage(@RequestParam String name, ModelMap model) {  
        model.put("name", name);  
        return "login";  
    }  
}
```

`model.put("name",name)-`> key, value값이다.

``` jsp
<html>  
<head>  
    <title>My first Login Page</title>  
</head>  
<body>  
Welcome Login Page! ${name}  
  
</body>  
</html>
```
model로 받은 값을 `${name}`으로 불러서 사용하면 해당 페이지에 쿼리파라미터를 노출시킬수있다

# Logging

``` title="application.properties"
logging.level.org.springframework=info
logging.level.org.springframework=debug
logging.level.com.in28minutes.springboot.myfirstwebapp=debug
```
3번줄 해당 패키지의 로깅레벨을 디버깅레벨로만 설정가능.

```java title="LoginController"
package com.in28minutes.springboot.myfirstwebapp.login;  
  
import org.slf4j.Logger;  
import org.slf4j.LoggerFactory;  
import org.springframework.stereotype.Controller;  
import org.springframework.ui.ModelMap;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RequestParam;  
  
@Controller  
public class LoginController {  
    private Logger logger = LoggerFactory.getLogger(getClass());  
    /// localhost:8080/login/  
    @RequestMapping("/login")  
    public String gotoLoginPage(@RequestParam String name, ModelMap model) {  
        model.put("name", name);  
        logger.debug("Request param is {}",name);  
        System.out.println("RequestParm is  " + name); //NOT RECOMMEND FOR CODE  
        return "login";  
    }  
}

```
```
2025-03-31T15:41:53.209+09:00 DEBUG 16120 --- [myfirstwebapp] [nio-8080-exec-1] c.i.s.m.login.LoginController            : Request param is RAvi
RequestParm is  RAvi
```

패키지를 DEBUG 수준으로 프린트되도록 설정했기때문에 이렇게 출력이된다.
여기서 INFO 레벨로 설정한다면 출력이 되지 않는다.
warn, info ,debug 설정한 레벨에 맞추어서 출력이 가능하다.
적절한 로깅레벨을 사용하시기 바람.


#### SpringBoot가 사용하는 기본 로깅 프레임워크는 SLF4j를 사용


-----
# DispatcherServlet

초창기 자바로 웹 애플리케이션을 개발하는 가장 초기에는 뷰(JSP)에 모든 코드를 담았습니다.
현재는 DispatcherServlet이 웹 요청을 받아서 어디로 보낼지 결정하고 응답을 만들어서 다시 돌려주는 역할을 합니다.

사용자가 웹 브라우저에서 URL을 입력하면 해당 요청이 DispatcherServlet으로 들어옴.
Handler(Controller)을 찾아 들어온 요청을 처리할 Controller를 찾아 요청을 전달함.
Controller는 요청을 받아 필요한 비즈니스 로직을 수행하고 결과를 반환함.
컨트롤러가 반환한 결과를 기반으로 어떤 화면을 보여줄지 결정하고 선택된 View에서 HTML을 생성한 후 , DispatcherServlet이 최종적으로 클라이언트에게 응답을 보냄.

```
[클라이언트 요청] → [DispatcherServlet] → [HandlerMapping] → [Controller]  
                 ← [ViewResolver] ← [View] ← [DispatcherServlet] ← [응답]
```


-----
# 로그인 양식 만들기 

```html
<html>  
<head>  
    <title>My first Login Page</title>  
</head>  
<body>  
Welcome Login Page!  
  
<form>  
    Name : <input type="text" name="name"/>  
    Password : <input type="password" name="password">  
    <input type="submit"/>  
</form>  
  
</body>  
</html>
```
![[Pasted image 20250402172023.png]]
지금 사용하는 요청메서드는 GET->  GET을 사용하면 모든 정보가 URL의 일부로서 전송됨 그렇기 떄문에 안전하지 않다.
FORM을 만들때마다 POST라는 걸 사용함.
##### Post로 만들어서 전송해보기
![[Pasted image 20250402172252.png]]
URL에서는 해당 정보들이 나오지 않는걸 확인했고 Payload를 들어가보면 해당 아이디와 비번을 볼수있다. post는 모든 정보가 Form Data의 일부로서 전송됨. 


#### 모델로 JSP에 로그인 자격증명 표시하기 
```java
package com.in28minutes.springboot.myfirstwebapp.login;  
  
import org.springframework.stereotype.Controller;  
import org.springframework.ui.ModelMap;  
import org.springframework.web.bind.annotation.PostMapping;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RequestMethod;  
import org.springframework.web.bind.annotation.RequestParam;  
  
@Controller  
public class LoginController {  
    //login  
    @RequestMapping(value="/login", method = RequestMethod.GET)  
    public String gotoLoginPage() {  
        return "login";  
    }  
  
    @PostMapping("/login")  
    public String gotoWelcomePage() {  
        return "welcome";  
    }  
}
```
`@RequestMapping` 같은 경우 `method= RequestMethod.GET`을 명시해주지 않으면 `GET` , `POST`를 둘 다 처리하기 가능함.
`@PostMapping`을 이용해서 직관적이고 명시적으로 표현하는 방법도 있다.

#### `@RequestParam`의 동작 방식
주로 `쿼리 스트링(Query String )`이나 `x-www-form-urlencoded` 데이터를 처리하는 데 사용됨.
요청 데이터가 URL 또는 form 데이터로 전송될 때만 사용할수 있음.

### Query String이란?

URL의 일부로 포함되어 전송됨.
GET 요청에서 데이터를 전달하는 대표적인 방법
데이터튼 URL 끝에 `?`로 시작해서 `&`  로 구분
``` url
/search?name=John+Doe&age=25
```

#### application/x-www-form-urlencoded란?

- HTTP 요청의 본문(Body)에 포함되어 전송됨
- 주로 POST, PUT 요청에서 폼 데이터를 전송할 때 사용됨.
- Header에 Content-type : application/x-www-form-urlencoded 필요
- 인코딩 방식은 쿼리 스트링과 동일(key=value&key2=value2)
----
#### 하드코딩으로 사용자 ID 및 Password 검증 추가하기
애플리케이션을 만들 때는 단일 책임 원칙을 지키려고 함.
그래서 인증을 책임지는 별도의 클래스를 생성해보도록 합니다.

``` java title="AuthenticationService"
package com.in28minutes.springboot.myfirstwebapp.login;  
  
import org.springframework.stereotype.Component;  
import org.springframework.stereotype.Service;  
  
@Service  
public class AuthenticationService {  
  
    public boolean authenticate(String username, String password) {  
        boolean isValidUserName= username.equalsIgnoreCase("in28minutes");  
        boolean isValidPassword = password.equalsIgnoreCase("dummy");  
        return isValidPassword && isValidPassword;  
    }  
}
```

```java title="LoginController"
package com.in28minutes.springboot.myfirstwebapp.login;  
  
import org.springframework.stereotype.Controller;  
import org.springframework.ui.ModelMap;  
import org.springframework.web.bind.annotation.PostMapping;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RequestMethod;  
import org.springframework.web.bind.annotation.RequestParam;  
  
@Controller  
public class LoginController {  
  
    private AuthenticationService authenticationService;  
  
    public LoginController(AuthenticationService authenticationService) {  
        this.authenticationService = authenticationService;  
    }  
  
    @PostMapping("/login")  
    public String gotoWelcomePage(@RequestParam String name, @RequestParam String password, ModelMap model) {  
        if (authenticationService.authenticate(name, password)) {  
            model.put("name", name);  
            model.put("password", password);  
            return "welcome";  
        }  
        return "login";  
    }  
}
```

검증서비스가 맞으면 모델에 전달해서 화면에 표시하고 아니면 로그인 페이지를 계속 출력하는 코드.
아이디와 비번이 틀려도 해당 로그인 문제있다는걸 표시해주지않으니 표시해주는 작업을 하겠다.

```java title="LoginController"
@PostMapping("/login")  
public String gotoWelcomePage(@RequestParam String name, @RequestParam String password, ModelMap model) {  
    if (authenticationService.authenticate(name, password)) {  
        model.put("name", name);  
        model.put("password", password);  
        return "welcome";  
    }else {  
        model.put("errorMessage", "Invalid username or password");  
        return "login";  
    }  
}
```
오류메시지를 Model에 담아서 전달해주면 된다. 전달받은 메시지를 화면에 출력해주는 과정을 아래에 표시하겠다.

```jsp  title="Login.jsp"
<html>  
<head>  
    <title>My first Login Page</title>  
</head>  
<body>  
Welcome Login Page!  
<pre>${errorMessage}</pre>  
<form method="post">  
    Name : <input type="text" name="name"/>  
    Password : <input type="password" name="password">  
    <input type="submit"/>  
</form>  
  
</body>  
</html>

```
![[Pasted image 20250402232331.png]]

---

#  ToDo 기능만들기 

ToDo 클래스부터 만들어 보겠습니다.

```java title="Todo"
package com.in28minutes.springboot.myfirstwebapp.todo;  
  
import java.time.LocalDate;  
  
public class Todo {  
    private int id;  
    private String username;  
    private String description;  
    private LocalDate targetDate;  
    private boolean done;  
  
    public Todo(int id, String username, String description, LocalDate targetDate, boolean done) {  
        super();  
        this.id = id;  
        this.username = username;  
        this.description = description;  
        this.targetDate = targetDate;  
        this.done = done;  
    }  
    public int getId() {  
        return id;  
    }  
  
    public void setId(int id) {  
        this.id = id;  
    }  
  
    public String getUsername() {  
        return username;  
    }  
  
    public void setUsername(String username) {  
        this.username = username;  
    }  
  
    public String getDescription() {  
        return description;  
    }  
  
    public void setDescription(String description) {  
        this.description = description;  
    }  
  
    public LocalDate getTargetDate() {  
        return targetDate;  
    }  
  
    public void setTargetDate(LocalDate targetDate) {  
        this.targetDate = targetDate;  
    }  
  
    public boolean isDone() {  
        return done;  
    }  
  
    public void setDone(boolean done) {  
        this.done = done;  
    }  
    @Override  
    public String toString() {  
        return "Todo{" +  
                "id=" + id +  
                ", username='" + username + '\'' +  
                ", description='" + description + '\'' +  
                ", targetDate=" + targetDate +  
                ", done=" + done +  
                '}';  
    }  
}
```

```java title="TodoService"
package com.in28minutes.springboot.myfirstwebapp.todo;  
  
import org.springframework.stereotype.Service;  
  
import java.time.LocalDate;  
import java.util.List;  
  
@Service  
public class TodoService {  
   private static List<Todo> todos = new ArrayList<>(); 
    static {  
        todos.add(new Todo(1, "in28minutes", "Learn AWS", LocalDate.now().plusYears(1),false));  
        todos.add(new Todo(1, "in28minutes", "Learn DevOps", LocalDate.now().plusYears(1),false));  
        todos.add(new Todo(1, "in28minutes", "Learn Full Stack Development", LocalDate.now().plusYears(1),false));  
    }  
}
```

외부에서 이것들에 직접 액세스하길 원치 않고, 그래서 이걸 private으로 만들었음.
여기에 Todo리스트를 리턴하기 위한 메서드를 넣어줄겁니다.

----
#### Todo 리스트 페이지 만들기

##### TodosController 만들기 
```java title="TodosController"
package com.in28minutes.springboot.myfirstwebapp.todo;  
  
import org.springframework.stereotype.Controller;  
import org.springframework.ui.Model;  
import org.springframework.ui.ModelMap;  
import org.springframework.web.bind.annotation.GetMapping;  
import org.springframework.web.bind.annotation.RequestMapping;  
  
import java.util.List;  
  
@Controller  
public class TodoController {  
  
  
    private  TodoService todoService;  
  
    public TodoController(TodoService todoService) {  
        super();  
        this.todoService = todoService;  
    }  
  
    // /list-todos  
    @RequestMapping("list-todos")  
    public String listAllTodos(ModelMap model) {  
       List<Todo> todos= todoService.findByUsername("in28minutes");  
        model.addAttribute("todos", todos);  
        return "listTodos";  
    }  
}
```

``` jsp title="listTodos"
<html>  
<head>  
    <title>List Todos Page</title>  
</head>  
<body>  
<div>Welcome to in28minutes</div>  
<div>Your Todos are ${todos} </div>  
</body>  
</html>
```


----
# Session, Model, Request

### Request
Payload는 다음 페이지 -> Welcome, Todo페이지로 가면 이 값들을 사용할수 없게 될겁니다.
이렇게 요청에 있는 모든 값들은 오직 그 요청에 대해서만 유효함.
요청이 다시 전송되면 요청 속성은 메모리에서 삭제됨.
###  Model
기본값으로서 그 요청의 범위에서만 사용할수 있음.
### Session
위에 두개 보다 더 오래 사용할수 있는게 세션이고 여러 요청에 걸쳐 값을 유지할시 사용
세부정보가 다수의 요청에 걸쳐 저장됨.
추가로 메모리를 차지하고 모든 세부정보가 서버에 저장되기 때문입니다.
##### Session을 사용하는 법
`@SessionAttribute`를 사용할때 항상 그 값을 사용하려는 모든 컨트롤러에 그걸 넣어줘야 함.

```java title="LoginController"
package com.in28minutes.springboot.myfirstwebapp.login;  
  
import org.springframework.stereotype.Controller;  
import org.springframework.ui.ModelMap;  
import org.springframework.web.bind.annotation.*;  
  
@Controller  
@SessionAttributes("name")  
public class LoginController {  
  
    private AuthenticationService authenticationService;  
  
    public LoginController(AuthenticationService authenticationService) {  
        this.authenticationService = authenticationService;  
    }  
  
    //login  
    @RequestMapping(value = "/login", method = RequestMethod.GET)  
    public String gotoLoginPage() {  
        return "login";  
    }  
  
    @PostMapping("/login")  
    public String gotoWelcomePage(@RequestParam String name, @RequestParam String password, ModelMap model) {  
        if (authenticationService.authenticate(name, password)) {  
            model.put("name", name);  
            return "welcome";  
        }else {  
            model.put("errorMessage", "Invalid username or password");  
            return "login";  
        }  
    }  
}
```
```java title="TodoController"
package com.in28minutes.springboot.myfirstwebapp.todo;  
  
import org.springframework.stereotype.Controller;  
import org.springframework.ui.Model;  
import org.springframework.ui.ModelMap;  
import org.springframework.web.bind.annotation.GetMapping;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.SessionAttributes;  
  
import java.util.List;  
  
@Controller  
@SessionAttributes("name")  
public class TodoController {  
  
  
    private  TodoService todoService;  
  
    public TodoController(TodoService todoService) {  
        super();  
        this.todoService = todoService;  
    }  
  
    // /list-todos  
    @RequestMapping("list-todos")  
    public String listAllTodos(ModelMap model) {  
       List<Todo> todos= todoService.findByUsername("in28minutes");  
        model.addAttribute("todos", todos);  
        return "listTodos";  
    }  
}
```
##### 위 두 Controller에 `@SessionAttributes` 함께 넣어주면 됨.


# JSTL을 추가하고 Todos를 테이블 표시하기

동적 콘텐츠를 중심으로 더욱 복잡한 걸 하고 싶은 경우에는 Todo는 컨트롤러에서 유입되는 값들의 동적 리스트.

```jsp
<html>  
<head>  
    <title>List Todos Page</title>  
</head>  
<body>  
<div>Welcome ${name}</div>  
<div>Your Todos are ${todos} </div>  
</body>  
</html>
```

`${todos}` -> 실행중에 만들어지거나 변경되는 코드(동적코드), 프로그램이 돌아가면서 새로 작성되거나 바뀐다. 더 복잡한 상황에 쓰기위해서 `JSTL 태그를 사용함.


#### JSTL을 쓰기위해서` pom.xml `의존성 추가하기 
``` xml

<dependency>  
    <groupId>jakarta.servlet.jsp.jstl</groupId>  
    <artifactId>jakarta.servlet.jsp.jstl-api</artifactId>  
</dependency>

<dependency>  
    <groupId>org.eclipse.jetty</groupId>  
    <artifactId>glassfish-jstl</artifactId>  
    <version>11.0.20</version>  
</dependency>
```

여기서 추가해줘야되는 부분은 JSTL 링크를 연결해줘야된다.
` <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>`
`taglib`-> 지시어이고 태그 라이브러리를 사용하겠다는 선언 부분
`prefix="c"` -> JSTL Core 태그를 사용할 때 쓸 접두사 (ex:`<c:forEach`)
`uri="..."->` 어떤 라이브러리인지를 알려주는 식별자

``` jsp
 <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
 <html>  
<head>  
    <title>List Todos Page</title>  
</head>  
<body>  
<div>Welcome ${name}</div>  
<div>Your Todos are ${todos} </div>  
</body>  
</html>
```
#### 조금 더 개선된 HTML 형식으로 바꾸면 
``` jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>  
<html>  
<head>  
    <title>List Todos Page</title>  
</head>  
<body>  
<div>Welcome ${name}</div>  
<hr>  
<h1>Your Todos</h1>  
  
<table>  
    <thead>  
        <tr>  
            <th>id</th>  
            <th>Description</th>  
            <th>Target Date</th>  
            <th>Is Done?</th>  
        </tr>  
    </thead>  
    <tbody>    <c:forEach items="${todos}" var="todo">  
        <tr>  
            <td>${todo.id}</td>  
            <td>${todo.description}</td>  
            <td>${todo.targetDate}</td>  
            <td>${todo.done}</td>  
        </tr>  
    </c:forEach>  
    </tbody>  
</table>  
</body>  
</html>
```
![[Pasted image 20250407154613.png]]

| 태그        | 설명                     |
| --------- | ---------------------- |
| `<table>` | 표 전체를 감싸는 태그           |
| `<thead>` | 표의 **머리글** 부분 (열 제목 등) |
| `<tbody>` | 표의 **본문 내용**을 담는 부분    |
| `<tr>`    | **행(Row)**을 나타냄 (한 줄)  |
| `<th>`    | **헤더 셀** (굵게, 가운데 정렬됨) |
| `<td>`    | **일반 셀** (데이터가 들어감)    |

### Bootstrap을 CSS 프레임워크를 Spring Boot 프로젝트에 추가하기 

##### webjars를 이용해서 간단하게 적용해보기 
Bootstrap같은 경우 version을 수동으로 세팅해줘야된다.
``` xml
<dependency>  
<groupId>org.webjars</groupId>  
    <artifactId>bootstrap</artifactId>  
    <version>5.1.3</version>  
</dependency>  
  
<dependency>  
    <groupId>org.webjars</groupId>  
    <artifactId>jquery</artifactId>  
    <version>3.6.0</version>  
</dependency>
```


### 링크 태그를 이용해서 부트스트랩을 연결해보기
```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>  
<html>  
<head>  
    <link href="webjars\bootstrap\5.1.3\css\bootstrap.min.css" rel="stylesheet">  
    <title>List Todos Page</title>  
</head>  
<body>  
<div>Welcome ${name}</div>  
<hr>  
<h1>Your Todos</h1>  
  
<table>  
    <thead>  
        <tr>  
            <th>id</th>  
            <th>Description</th>  
            <th>Target Date</th>  
            <th>Is Done?</th>  
        </tr>  
    </thead>  
    <tbody>    <c:forEach items="${todos}" var="todo">  
        <tr>  
            <td>${todo.id}</td>  
            <td>${todo.description}</td>  
            <td>${todo.targetDate}</td>  
            <td>${todo.done}</td>  
        </tr>  
    </c:forEach>  
    </tbody>  
    <script src="webjars\bootstrap\5.1.3\js\bootstrap.min.js"></script>  
    <script src="webjars\jquery\3.6.0\jquery.min.js"></script>  
</table>  
</body>  
</html>
```
##### 올바르게 다운로드된것을 알수 있다.

![[Pasted image 20250407164024.png]]
브라우저로 되돌아오는 첫번째 응답은 list-todos 에서 오는 것이고 , 브라우저는 list-todos의 콘텐츠에 bootstrap css파일에 대한 링크와 Javascript파일에 대한 링크가 있는걸 확인할수 있고 
브라우저는 이 모든 파일을 다운받습니다.



#### Boostrap CSS -> JSP 페이지 포맷만들기
```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>  
<html>  
<head>  
    <link href="webjars\bootstrap\5.1.3\css\bootstrap.min.css" rel="stylesheet">  
    <title>List Todos Page</title>  
</head>  
<body>  
<div class="container">  
    <h1>Your Todos</h1>  
    <table>        <thead>  
        <tr>  
            <th>id</th>  
            <th>Description</th>  
            <th>Target Date</th>  
            <th>Is Done?</th>  
        </tr>  
        </thead>  
        <tbody>        <c:forEach items="${todos}" var="todo">  
            <tr>  
                <td>${todo.id}</td>  
                <td>${todo.description}</td>  
                <td>${todo.targetDate}</td>  
                <td>${todo.done}</td>  
            </tr>  
        </c:forEach>  
        </tbody>  
    </table>  
</div>  
<script src="webjars\bootstrap\5.1.3\js\bootstrap.min.js"></script>  
<script src="webjars\jquery\3.6.0\jquery.min.js"></script>  
</body>  
</html>
```
container  클래스가 페이지의 본문 전체를 담고있는것.

![[Pasted image 20250407170227.png]]
container클래스를 사용하면 화면이 중앙으로 출력되는걸 볼수있다. container삭제시 화면은 다시 왼쪽으로 위치한다.
### `<table class="table"`>를 사용 시 
![[Pasted image 20250407170507.png]]

----
# TODO 추가하기 
```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>  
<html>  
<head>  
    <link href="webjars\bootstrap\5.1.3\css\bootstrap.min.css" rel="stylesheet">  
    <title>List Todos Page</title>  
</head>  
<body>  
<div class="container">  
    <h1>Your Todos</h1>  
    <table class="table">  
        <thead>  
        <tr>  
            <th>id</th>  
            <th>Description</th>  
            <th>Target Date</th>  
            <th>Is Done?</th>  
        </tr>  
        </thead>  
        <tbody>        <c:forEach items="${todos}" var="todo">  
            <tr>  
                <td>${todo.id}</td>  
                <td>${todo.description}</td>  
                <td>${todo.targetDate}</td>  
                <td>${todo.done}</td>  
            </tr>  
        </c:forEach>  
        </tbody>  
    </table>  
    <a href="add-todo" class="btn btn-success">Add Todo</a>  
</div>  
<script src="webjars\bootstrap\5.1.3\js\bootstrap.min.js"></script>  
<script src="webjars\jquery\3.6.0\jquery.min.js"></script>  
</body>  
</html>
```

  `<a href="add-todo" class="btn btn-success">Add Todo</a>` 를 추가해줘서 버튼을 이쁘게 출력 할수 있도록 도와준다.
  ![[Pasted image 20250407171122.png]]


##### 버튼을 클릭해서 들어가면 add-todo 주소가 TodoController로 매핑되서 Todo 뷰를 전달함.
``` java title="TodoController"
package com.in28minutes.springboot.myfirstwebapp.todo;  
  
import org.springframework.stereotype.Controller;  
import org.springframework.ui.Model;  
import org.springframework.ui.ModelMap;  
import org.springframework.web.bind.annotation.GetMapping;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.SessionAttributes;  
  
import java.util.List;  
  
@Controller  
@SessionAttributes("name")  
public class TodoController {  
  
    @RequestMapping("add-todo")  
    public String showNewTodoPage() {  
        return "todo";  
    }  
}
```

#### Todo 뷰 만들기 

```jsp title="todo.jsp"
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>  
<html>  
<head>  
    <link href="webjars\bootstrap\5.1.3\css\bootstrap.min.css" rel="stylesheet">  
    <title>Add Todo Page</title>  
</head>  
<body>  
<div class="container">  
    <h1>Enter Todo Details</h1>  
    <form method="post">  
        Description: <input type="text" name="description"/>  
        <input type="submit" class="btn btn-success">  
    </form>  
</div>  
<script src="webjars\bootstrap\5.1.3\js\bootstrap.min.js"></script>  
<script src="webjars\jquery\3.6.0\jquery.min.js"></script>  
</body>  
</html>
```


## Todo를 추가하기 위해 TodoService 개선하기

```java title="TodoService"
package com.in28minutes.springboot.myfirstwebapp.todo;  
  
import org.springframework.stereotype.Service;  
  
import java.time.LocalDate;  
import java.util.ArrayList;  
import java.util.List;  
  
@Service  
public class TodoService {  
  
    private static List<Todo> todos = new ArrayList<>();  
    private static int todosCount= 0;  
  
    static {  
        todos.add(new Todo(++todosCount, "in28minutes", "Learn AWS", LocalDate.now().plusYears(1),false));  
        todos.add(new Todo(++todosCount, "in28minutes", "Learn DevOps", LocalDate.now().plusYears(1),false));  
        todos.add(new Todo(++todosCount, "in28minutes", "Learn Full Stack Development", LocalDate.now().plusYears(1),false));  
    }  
    public List<Todo> findByUsername(String username) {  
        return todos;  
    }  
  
    public void addTodo(String username, String description, LocalDate targetDate, boolean done) {  
        Todo todo = new Todo(++todosCount, username, description, targetDate, done);  
        todos.add(todo);  
    }  
}
```

10개의 Todo를 추가해도 서버를 재시작하면 추가한 모든 Todo는 사라지는 정적 리스트라는 점은 잊어선 안된다. 나중에 MYSQL 같은 영구적인 DB를 사용할 떄 이걸 수정할 예정 

```java title="TodoController"
package com.in28minutes.springboot.myfirstwebapp.todo;  
  
import org.springframework.stereotype.Controller;  
import org.springframework.ui.Model;  
import org.springframework.ui.ModelMap;  
import org.springframework.web.bind.annotation.*;  
  
import java.time.LocalDate;  
import java.util.List;  
  
@Controller  
@SessionAttributes("name")  
public class TodoController {  
  
  
    private  TodoService todoService;  
  
    public TodoController(TodoService todoService) {  
        super();  
        this.todoService = todoService;  
    }  
  
    // /list-todos  
    @RequestMapping("list-todos")  
    public String listAllTodos(ModelMap model) {  
       List<Todo> todos= todoService.findByUsername("in28minutes");  
        model.addAttribute("todos", todos);  
        return "listTodos";  
    }  
  
    //GET, POST  
    @GetMapping("add-todo")  
    public String showNewTodoPage() {  
        return "todo";  
    }  
  
    @PostMapping("add-todo")  
    public String addNewTodo(@RequestParam String description, ModelMap model) {  
        String username = (String) model.get("name");  
        todoService.addTodo(username,description, LocalDate.now().plusYears(1),false);  
        return "redirect:list-todos";  
    }  
}
```

# Spring Boot Starter Validation을 이용하여 검증하기 


HTML 코드를 조금 추가함으로써 프론트엔드 검증을 할수있음.
`required="required"`
```jsp title="todo.jsp"
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>  
<html>  
<head>  
    <link href="webjars\bootstrap\5.1.3\css\bootstrap.min.css" rel="stylesheet">  
    <title>Add Todo Page</title>  
</head>  
<body>  
<div class="container">  
    <h1>Enter Todo Details</h1>  
    <form method="post">  
        Description: <input type="text" name="description"  
      required="required"/>  
        <input type="submit" class="btn btn-success">  
  
    </form>  
</div>  
<script src="webjars\bootstrap\5.1.3\js\bootstrap.min.js"></script>  
<script src="webjars\jquery\3.6.0\jquery.min.js"></script>  
</body>  
</html>
```
![[Pasted image 20250408231909.png]]
기존에는 비어있는 Todo를 추가하면 추가되었는데 이제는 비어있는  Todo는 추가가 되지않는다.
프론트엔드에서 HTML이나 Javascript의 검증은 해커가 아주 쉽게 건너 뛸수 있다는 점이다.
최선의 방식은 서버측 검증인것이다.

#### Spring Boot Starter Validation 의존성 추가해주기 

```xml 
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-validation</artifactId>  
</dependency>
```
#### Command Bean(Form Backing Bean)
양방향 바인딩 개념을 구현 (todo.jsp & TodoController.java)
todo.jsp 폼에  Description이 입력되면 , 이걸 `addNewTodo()` 바인딩함.
하지만 추후에 생성할 완료했는지 여부, 기간을 추가하면 `addNewTodo()` 의`@RequestParam` 은 길어져 코드가 복잡해지기 때문에 `Todo.java`에 직접 변수에 바인딩해서 하는 방법을 추천한다.

먼저 `TodoController` 수정해보자
```java title="TodoController"
package com.in28minutes.springboot.myfirstwebapp.todo;  
  
import org.springframework.stereotype.Controller;  
import org.springframework.ui.Model;  
import org.springframework.ui.ModelMap;  
import org.springframework.web.bind.annotation.*;  
  
import java.time.LocalDate;  
import java.util.List;  
  
@Controller  
@SessionAttributes("name")  
public class TodoController {  
  
  
    private  TodoService todoService;  
  
    public TodoController(TodoService todoService) {  
        super();  
        this.todoService = todoService;  
    }  
  
    @PostMapping("add-todo")  
    public String addNewTodo(ModelMap model, Todo todo) {  
        String username = (String) model.get("name");  
        todoService.addTodo(username,todo.getDescription(), LocalDate.now().plusYears(1),false);  
        return "redirect:list-todos";  
    }  
}
```


###### Validation 태그 추가해주기
```jsp title="todo.jsp"
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
```
```jsp title="todo.jsp"
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>  
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>  
<div class="container">  
    <h1>Enter Todo Details</h1>  
    <form:form method="post" modelAttribute="todo">  
        Description: <form:input type="text" path="description"  
      required="required"/>  
        <input type="submit" class="btn btn-success">  
    </form:form>  
</div>  
```
form에서 사용할 데이터를 담을 객체의 이름이 ''todo"이고 이 값을 컨트롤러에 전달
`path="description"` 은 연결된 객체(todo)의 어떤 필드와 연결할지 지정(여기서는 description)


```java title="TodoController"
//GET, POST  
@GetMapping("add-todo")  
public String showNewTodoPage(ModelMap model) {  
    String username = (String) model.get("name");  
    Todo todo = new Todo(0, username, "", LocalDate.now().plusYears(1), false);  
    model.put("todo", todo);  
    return "todo";  
}
```
사용자가 /add-todo 주소로 접근해서 실행하고 ,현재 로그인한 유저이름을 가져오고, Todo객체를 하나만듬(Description은 비어있음), 그 객체를 Model에 넣음. View에서 modelAttribute="todo"로 쓸수 있게 됨.
```jsp title="todo.jsp"
    <form:form method="post" modelAttribute="todo">  
        Description: <form:input type="text" path="description"  
      required="required"/>  
        <form:input type="hidden" path="id"/>  
        <form:input type="hidden" path="done"/>  
        <input type="submit" class="btn btn-success">  
```
`modelAttribute="todo"` 모델에 있던 todo 객체를 폼에 연결함, `path="descritpion" `todo 객체안의 `description` 필드와 연결 사용자가 입력한 값이 나중에 자동으로 `todo.setDescription()`으로 들어감.

```java
@PostMapping("add-todo")  
public String addNewTodo(ModelMap model, Todo todo) {  
    String username = (String) model.get("name");  
    todoService.addTodo(username,todo.getDescription(), LocalDate.now().plusYears(1),false);  
    return "redirect:list-todos";  
}
```
`@ModelAttribute`는 컨트롤러쪽에서 생략가능,  뷰에서 쓰던 그 객체이름 `"todo"`그대로 받아줌./
Spring 폼 데이터를 자동으로 Todo 객체에 채워 넣어줌 (자동 바인딩)
`todo.getDescription()` 메서드로 꺼내 쓰면됨.  **<span style="background:rgba(255, 183, 139, 0.55)">이것이 양방향 바인딩</span>**

##### 양방향 바인딩 장점
검증을 구현할 때 요류를 표시하는게 아주 쉬워진다.


# Command Bean으로 새 Todo 페이지 검증 구현하기

Validation을 사용해보자 `@Size` 해당 폼은 최소 10글자, 10글자 입력하지 않을시 경고 메시지를 enter at least 10 chararcter가 출력될수 있도록 설정하였다.

```java title="Todo"
package com.in28minutes.springboot.myfirstwebapp.todo;  
  
import jakarta.validation.constraints.Size;  
import org.springframework.stereotype.Component;  
  
import java.time.LocalDate;  
  
  
//Database (MYSQL)  
//Static List of todos => Database(H2,MySQL)  
public class Todo {  
    private int id;  
    private String username;  
  
    @Size(min= 10, message = "Enter at least 10 character")  
    private String description;  
	  }
```


```java title="TodoController"
@PostMapping("add-todo")  
public String addNewTodo(ModelMap model, @Valid Todo todo) {  
    String username = (String) model.get("name");  
    todoService.addTodo(username,todo.getDescription(), LocalDate.now().plusYears(1),false);  
    return "redirect:list-todos";  
}
```

`TodoController`의 Bean에 바인딩 하는 곳에 `@Valid`를 추가해줘서 바인딩이 이루어지기 전에 Todo Bean을 검증하게 됨.

### 오류 출력창에 Validation 적용한 부분이 나타난다.
![[Pasted image 20250409191340.png]]

하지만 이렇게 오류창을 보여주는거보단 사용자가 바로 알아볼수있는 정상 출력창에서 오류메시지가 표시되는게 훨씬 낫기때문에 조금 더 개선해보자

```java title="TodoController"
@PostMapping("add-todo")  
public String addNewTodo(ModelMap model, @Valid Todo todo, BindingResult bindResult) {  
  
    if (bindResult.hasErrors()) {  
        return "todo";  
    }  
    String username = (String) model.get("name");  
    todoService.addTodo(username,todo.getDescription(), LocalDate.now().plusYears(1),false);  
    return "redirect:list-todos";  
}
```
`@BindResult`-> 폼 데이터 바인딩 및 검증 결과를 담는 객체이고 ,폼에서 넘어온 데이터가 객체(`@ModelAttribute`)에 바인딩할때 생기는 에러메시지, 유효성 실패 등을 저장해줌.

**<span style="background:rgba(255, 183, 139, 0.55)">항상 @ModelAttribute 나 @Valid 바로 뒤에 위치해야한다.</span>**

코드에서 에러가 발생시 todo 페이지를 다시 돌아갈수있도록 성공시 리다이렉트 되도록 설정함.
하지만 이것 역시 사용자가 오류가 발생했는지 쉽게 알 수 없기 때문에 좋지 못한 방법이다.

###### View에서 오류메시지를 사용자에게 알기 쉽게 보여주기
```jsp title="todo.jsp"
<div class="container">  
    <h1>Enter Todo Details</h1>  
    <form:form method="post" modelAttribute="todo">  
        Description: <form:input type="text" path="description"  
                                 required="required"/>  
       <form:errors type="text" path="description"/>  
        <form:input type="hidden" path="id"/>  
        <form:input type="hidden" path="done"/>  
        <input type="submit" class="btn btn-success">  
  
    </form:form>  
</div>
```
![[Pasted image 20250409192328.png]]
위처럼 설정해놓은 메시지가 오류메시지로 나오는걸 알수있다. 조금 더 색깔을 넣어 보면 이렇게 보인다.

```jsp title="todo.jsp"
<div class="container">  
    <h1>Enter Todo Details</h1>  
    <form:form method="post" modelAttribute="todo">  
        Description: <form:input type="text" path="description"  
                                 required="required"/>  
       <form:errors type="text" path="description" cssClass="text-warning"/>  
        <form:input type="hidden" path="id"/>  
        <form:input type="hidden" path="done"/>  
        <input type="submit" class="btn btn-success">  
  
    </form:form>  
</div>
```
`cssClass="text-warning"`-> Bootstarp에서 제공하는 클래스입니다.
![[Pasted image 20250409192632.png]]

----
## Todo 삭제 기능 구현하기

##### 삭제하기 위해선 삭제버튼이 필요하니깐 삭제버튼을 만들어보자

```jsp title="listTodos.jsp"
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>  
<html>  
<head>  
    <link href="webjars\bootstrap\5.1.3\css\bootstrap.min.css" rel="stylesheet">  
    <title>List Todos Page</title>  
</head>  
<body>  
<div class="container">  
    <h1>Your Todos</h1>  
    <table class="table">  
        <thead>  
        <tr>  
            <th>id</th>  
            <th>Description</th>  
            <th>Target Date</th>  
            <th></th>        </tr>  
        </thead>  
        <tbody>        <c:forEach items="${todos}" var="todo">  
            <tr>  
                <td>${todo.id}</td>  
                <td>${todo.description}</td>  
                <td>${todo.targetDate}</td>  
                <td>${todo.done}</td>  
                <td><a href="delete-todo?id=${todo.id}" class="btn btn-warning">DELETE ${todo.id}</a></td>  
            </tr>  
        </c:forEach>  
        </tbody>  
    </table>  
    <a href="add-todo" class="btn btn-success">Add Todo</a>  
</div>  
<script src="webjars\bootstrap\5.1.3\js\bootstrap.min.js"></script>  
<script src="webjars\jquery\3.6.0\jquery.min.js"></script>  
</body>  
</html>
```
![[Pasted image 20250409220905.png]]

``` jsp
    <td><a href="delete-todo?id=${todo.id}" class="btn btn-warning">DELETE ${todo.id}</a></td>  
```
앵커 태그로 되어있어서 클릭해서 들어가면 `delete-todo`로 들어 가진다. 근데 어떤 글을 삭제할지는 구체적인 요소가 필요한데 이때  `id`를 이용한다.

#### Predicate가 뭔가?
```java
Predicate<? super Todo> predicate;
```
함수형 인터페이스로, T 타입의 값을 받아서 true 또는 false를 리턴하는 함수 .
`Predicate<Todo>`-> Todo 객체를 받아서 true/false를 판단하는 조건 

`? super Todo`-> Todo 또는 Todo의 부모 클래스를 받아도 된다.라는 의미 
더 범용적으로 만들기 위해 쓰는 와일드 카드 구문 

```java title="TodoService"
public void deleteById(int id) {  
    Predicate<? super Todo> predicate  
            = todo -> todo.getId() == id;  
    todos.removeIf(predicate);  
}
```
Todo에 매칭되는 Id가 있는지 묻는 predicate, 조건에 매칭되면 그 Todo를 삭제할 것.
#### TodoController에서 TodoService의 deleteById 사용하기

```java title="TodoController"
@GetMapping("delete-todo")  
public String deleteTodo(@RequestParam int id) {  
    //Delete todo  
   todoService.deleteById(id);  
    return "redirect:list-todos";  
}
```
`delete-todo?id=`-> 전달되면 `@RequestParam`을 통해서 id값을 받아서 `TodoService.deleteById() ` -> Id를 넘기면 찾아서 삭제한다. 그리고 리다이렉트로 `list-todos`

----
# Todo Update 기능 구현하기 

##### View로 Update 버튼 만들기 
```jsp title=""
<c:forEach items="${todos}" var="todo">  
    <tr>  
        <td>${todo.id}</td>  
        <td>${todo.description}</td>  
        <td>${todo.targetDate}</td>  
        <td>${todo.done}</td>  
        <td><a href="delete-todo?id=${todo.id}" class="btn btn-warning">Delete</a></td>  
        <td><a href="update-todo?id=${todo.id}" class="btn btn-success">Update</a></td>  
    </tr>  
</c:forEach>
```
![[Pasted image 20250409222925.png]]

##### TodoService에서 findById 추가하고 TodoController에서 적용하기 

```java title="TodoService"
public Todo findById(int id) {  
    Predicate<? super Todo> predicate  
            = todo -> todo.getId() == id;  
    Todo todo= todos.stream().filter(predicate).findFirst().get();  
    return todo;  
}  
```

```java title="TodoController"
@GetMapping("update-todo")  
public String showUpdateTodoPage(@RequestParam int id, ModelMap model) {  
    Todo todo = todoService.findById(id);  
    model.addAttribute("todo", todo);  
    return "todo";  
}
```

`update-todo `->  `todoService.findById`로 해당 Todo객체를 찾아서  Todo객체를 `model.addAttribute`에 담아서 화면에 출력하기 
![[Pasted image 20250409230311.png]]

## 2번째  업데이트 기능 추가

##### TodoService 코드 추가하기 
```java title="TodoService"
public void updateTodo(@Valid Todo todo) {  
    deleteById(todo.getId());  
    todos.add(todo);  
}
```
간단하게 진행해보자면 해당 아이디를 찾은 후 삭제하고 다시 생성하는 것도 하나의 방법이다.

```java title="TodoController"
@PostMapping("update-todo")  
public String updateTodo(ModelMap model, @Valid Todo todo, BindingResult bindResult) {  
    if (bindResult.hasErrors()) {  
        return "todo";  
    }  
  String username =(String) model.get("name");  
    todo.setUsername(username);  
    todoService.updateTodo(todo);  
    return "redirect:list-todos";  
}
```
![[Pasted image 20250409231230.png]]
	`username`은 로그인시 해당 글을 삭제와 수정 및 추가를 해당 사용자가 했다는걸 보여주기위해서 표시했음. 하지만 `Date`를  보면 비어있는걸 알수있다.  다음 단계에서 수정해보도록 하겠음.

----
## Todo에 목표날짜 추가하기 

```jsp title="todo.jsp"
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>  
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>  
  
<html>  
<head>  
    <link href="webjars\bootstrap\5.1.3\css\bootstrap.min.css" rel="stylesheet">  
    <title>Add Todo Page</title>  
</head>  
<body>  
<div class="container">  
    <h1>Enter Todo Details</h1>  
    <form:form method="post" modelAttribute="todo">  
        <fieldset class="mb-3">  
            <form:label path="description">Description</form:label>  
            <form:input type="text" path="description" required="required"/>  
            <form:errors type="text" path="description" cssClass="text-warning"/>  
        </fieldset>  
        <fieldset class="mb-3">  
            <form:label path="targetDate">Target Date</form:label>  
            <form:input type="text" path="targetDate" required="required"/>  
            <form:errors type="text" path="targetDate" cssClass="text-warning"/>  
        </fieldset>  
        <form:input type="hidden" path="id"/>  
        <form:input type="hidden" path="done"/>  
        <input type="submit" class="btn btn-success">  
    </form:form>  
</div>  
<script src="webjars\bootstrap\5.1.3\js\bootstrap.min.js"></script>  
<script src="webjars\jquery\3.6.0\jquery.min.js"></script>  
</body>  
</html>
```

#### 날짜 형식이 일정하지 않기 때문에 application.properties에서 설정하기 
![[Pasted image 20250409232520.png]]
![[Pasted image 20250409232501.png]]

```
spring.mvc.format.date=yyyy-MM-dd
```
![[Pasted image 20250409232548.png]]

#### TodoController에도 적용해주기 
```java title="TodoController"
@PostMapping("add-todo")  
public String addNewTodo(ModelMap model, @Valid Todo todo, BindingResult bindResult) {  
    if (bindResult.hasErrors()) {  
        return "todo";  
    }  
    String username = (String) model.get("name");  
    todoService.addTodo(username, todo.getDescription(), todo.getTargetDate(), false);  
    return "redirect:list-todos";  
}
```
이제는 자동으로 날짜가 입력되는게 아닌 사용자가 직접 입력해서 날짜쓰는 칸이 있어서 날짜 설정이 가능하다.
![[Pasted image 20250409233131.png]]

하지만 텍스트 입력하고 있기 때문에 불편함이 있다. 이걸 간단하게 해결할수 있는 BootStrap Datepicker를 사용 할 것이다.

#### Pom.xml에 BootStrapDatepicker 의존성 추가하기
``` xml
<dependency>  
    <groupId>org.webjars</groupId>  
    <artifactId>bootstrap-datepicker</artifactId>  
    <version>1.9.0</version>  
</dependency>
```

#### Todo.jsp에 코드 추가해줘서 BootStrap DatePicker 설정하기 
CSS와 JS코드 추가해주고 
 `targetDate` id클래스에 설정해줘서 datePicker에 적용될수 있도록 설정하기.
```jsp
<link href="webjars\bootstrap-datepicker\1.9.0\css\bootstrap-datepicker.standalone.min.css" rel="stylesheet">


<script src="webjars\bootstrap-datepicker\1.9.0\js\bootstrap-datepicker.min.js"></script>  
<script type="text/javascript">  
    $('#targetDate').datepicker({  
        format: 'yyyy-mm-dd'  
    });  
</script>
```
![[Pasted image 20250409235051.png]]

----- 
# 내비게이션 바를 추가하고 JSP 프래그먼트 구현하기 

##### Common 폴더를 만들고 여기에 JSPF 파일을 추가한다.
**<span style="background:rgba(255, 183, 139, 0.55)">JSPF란?</span>-**> Java Server Page Fragment 약자로
조각 형식으로 사용되는 파일, 완전한 JSP 페이지가 아닌 다른 JSP 파일에 포함되기 위해 작성된 부분적인 코드 
	
- 단독 실행 불가
- `<%@ include file=""%>`-> 통해서 다른 JSP에 포함시킴
![[Pasted image 20250411181244.png]]

```jsp title="footer.jspf"
<script src="webjars\bootstrap\5.1.3\js\bootstrap.min.js"></script>  
<script src="webjars\jquery\3.6.0\jquery.min.js"></script>  
<script src="webjars\bootstrap-datepicker\1.9.0\js\bootstrap-datepicker.min.js"></script>  
</body>  
</html>
```

```jsp title="navigation.jspf"
<nav class="navbar navbar-expand-md navbar-light bg-light p-1">  
    <div class="collapse navbar-collapse">  
        <ul class="navbar-nav">  
            <li class="nav-item"><a class="nav-link" href="/">Home</a></li>  
            <li class="nav-item"><a class="nav-link" href="/list-todos">Todos</a></li>  
        </ul>  
    </div>  
    <ul class="navbar-nav">  
        <li class="nav-item"><a class="nav-link" href="/logout">Logout</a></li>  
    </ul>  
</nav>
```

```jsp title="header.jspf"
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>  
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>  
  
<html>  
<head>  
    <link href="webjars\bootstrap\5.1.3\css\bootstrap.min.css" rel="stylesheet">  
    <link href="webjars\bootstrap-datepicker\1.9.0\css\bootstrap-datepicker.standalone.min.css" rel="stylesheet">  
    <title>Manage Your Todos</title>  
</head>  
<body>
```

##### 순서대로 넣어주기만 하면 올바르게 실행되는 걸 확인할수있다.
``` jsp title="listTodos"
<%@ include file="common/header.jspf"%>  
<%@ include file="common/navigation.jspf"%>  
<div class="container">  
    <h1>Your Todos</h1>  
    <table class="table">  
        <thead>  
        <tr>  
            <th>Description</th>  
            <th>Target Date</th>  
            <th></th>            <th></th>        </tr>  
        </thead>  
        <tbody>        <c:forEach items="${todos}" var="todo">  
            <tr>  
                <td>${todo.description}</td>  
                <td>${todo.targetDate}</td>  
                <td>${todo.done}</td>  
                <td><a href="delete-todo?id=${todo.id}" class="btn btn-warning">Delete</a></td>  
                <td><a href="update-todo?id=${todo.id}" class="btn btn-success">Update</a></td>  
            </tr>  
        </c:forEach>  
        </tbody>  
    </table>  
    <a href="add-todo" class="btn btn-success">Add Todo</a>  
</div>  
<%@ include file="common/footer.jspf"%>
```


``` jsp title="todo.jsp"
<%@ include file="common/header.jspf" %>  
<%@ include file="common/navigation.jspf" %>  
  
<div class="container">  
  
    <h1>Enter Todo Details</h1>  
  
    <form:form method="post" modelAttribute="todo">  
  
        <fieldset class="mb-3">  
            <form:label path="description">Description</form:label>  
            <form:input type="text" path="description" required="required"/>  
            <form:errors type="text" path="description" cssClass="text-warning"/>  
        </fieldset>  
        <fieldset class="mb-3">  
            <form:label path="targetDate">Target Date</form:label>  
            <form:input type="text" path="targetDate" required="required"/>  
            <form:errors type="text" path="targetDate" cssClass="text-warning"/>  
        </fieldset>  
        <form:input type="hidden" path="id"/>  
        <form:input type="hidden" path="done"/>  
        <input type="submit" class="btn btn-success">  
    </form:form>  
</div>  
<%@ include file="common/footer.jspf" %>  
<script type="text/javascript">  
    $('#targetDate').datepicker({  
        format: 'yyyy-mm-dd'  
    });  
</script>
```

------
## Spring Boot Starter Security로 Spring Security 설정하기

``` xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-security</artifactId>  
</dependency>
```
### 의존성 추가한 후 실행해보기 
![[Pasted image 20250412140729.png]]
전에 잘 열렸던 페이지들은 로그인페이지에서 통과해야지 이동하는걸 알수있다. Spring Security가 자동으로 로그인 페이지를 띄운다.
하지만 우리가 설정한 것이 아닌 Spring Security가 설정한것이라 아이디와 비번을 모르기 때문에 로그인 못한다고 생각하지만? Spring Security가 아이디와 비밀번호를 제공해준다. 기본 아이디는 `"user"` 이고 비번은 매번 실행할때 마다 달라진다. 실행창을 참고하면 된다.
![[Pasted image 20250412141037.png]]

#### Spring Security아이디와 비번 변경하기 

##### main에서 Securtiy 패키지를 만들고 SpringSecurityConfiguration 클래스 만들기
일반적으로 사용자 아이디와 비번을 저장하려 할때는 <font color="#ff0000">LDAP</font> 혹은 최소한 <font color="#c0504d">DB</font>를 사용함
하지만 간단히 인메모리로 사용할것
아래 코드는 확인 용도로만 사용할것이고 실제 서비서에선 다른 방식을 사용함.
```java title="SpringSecurityConfiguration"
package com.in28minutes.springboot.myfirstwebapp.security;  
  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.security.core.userdetails.User;  
import org.springframework.security.core.userdetails.UserDetails;  
import org.springframework.security.provisioning.InMemoryUserDetailsManager;  
  
@Configuration  
public class SpringSecurityConfiguration {  
    @Bean  
    public InMemoryUserDetailsManager createUserDetailsManager() {  
        UserDetails userDetails =User.withDefaultPasswordEncoder()  
                .username("in28")  
                .password("dummy")  
                .roles("USER", "ADMIN")  
                .build();  
        return new InMemoryUserDetailsManager(userDetails);  
    }  
}
```
`@Configuration` -> 설정 클래스 인식
`InMemoryDetailsManager` -> 메모리안에 사용자 정보를 저장하는 관리자
#####   `Builder` 방식을 이용해 처리함.
`User.withDefaultPasswordEncoder()`-> 간단하게 테스트할 수 있도록 비밀번호 암호화를 기본 방식으로 처리함.
`.username()`-> 로그인할 때 사용할 아이디
`.password()`-> 로그인할때 사용할 비번
`.roles` -> 사용자 "USER"와 "ADMIN" 두 역할을 가지고 있고 (나중에 접근 권한 체크할 때 이 역할을 기반으로 판단함)
return은 Spring Security가 로그인할 때 여기 저장된 유저 정보를 사용하게 됨.

###### `UserDetails란`?
사용자 정보를 담는 인터페이스, 한 명의 유저를 표현하는 객체

|속성|설명|
|---|---|
|username|로그인 아이디|
|password|로그인 비밀번호|
|roles/authorities|권한 또는 역할 (예: USER, ADMIN)|
|isAccountNonExpired|계정이 만료됐는지 여부|
|isAccountNonLocked|계정이 잠겼는지 여부|
|isCredentialsNonExpired|비밀번호가 만료됐는지 여부|
|isEnabled|계정이 활성화됐는지 여부|

##### 현재 사용하는 방식(`BCryptPasswordEncoder` ) 
**BCrypt**알고리즘을 사용한 비밀번호 방식 
**BCypt의** 특징
- 매번 다른 암호가 나옴
- 복호화(암호화를 평문으로 변환하는 과정) 불가능 -> 일방향 암호화
- 시간이 걸림(일부러 암호화하는 데 시간이 걸리게 만들어서 해커가 수십억개의 비밀번호를 빠르게 시도하지 못하게 막아줌.)

```java title="SpringSecurityConfiguration"
package com.in28minutes.springboot.myfirstwebapp.security;  
  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.security.core.userdetails.User;  
import org.springframework.security.core.userdetails.UserDetails;  
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;  
import org.springframework.security.crypto.password.PasswordEncoder;  
import org.springframework.security.provisioning.InMemoryUserDetailsManager;  
  
@Configuration  
public class SpringSecurityConfiguration {  
    //LDAP or DataBase  
    //In Memory  
    @Bean  
    public InMemoryUserDetailsManager createUserDetailsManager() {  
        UserDetails userDetails =User.withDefaultPasswordEncoder()  
                .username("in28")  
                .password("dummy")  
                .roles("USER", "ADMIN")  
                .build();  
  
        return new InMemoryUserDetailsManager(userDetails);  
    }  
  
    @Bean  
    public PasswordEncoder passwordEncoder() {  
        return new BCryptPasswordEncoder();  
    }  
}
```
![[Pasted image 20250412144035.png]]
`User.withDefaultPasswordEncoder()`는 그냥 User 만들 때만 쓰는 유틸 메서드에 불과함.-> 테스트 용도
암호화하는 방식-> `"{noop}"dummy` 으로  암호화

##### Spring한테 비밀번호 인코더를 등록한다는 뜻
```java
   @Bean  
    public PasswordEncoder passwordEncoder() {  
        return new BCryptPasswordEncoder();  
    }  
```
**Spring Security**는 `BCryptPasswordEncoder로` 비교하려고 하니까 비밀번호가 매칭이 되지 않아서 자격증명이 실패한것이다.

```java 
Function<String, String> passwordEncoder =  
        input -> passwordEncoder().encode(input);
        
        UserDetails userDetails =User.builder()  
                        .passwordEncoder(passwordEncoder)  
                        .username("in28")  
                        .password("dummy")  
                        .roles("USER", "ADMIN")  
                        .build();
```
`Function<String, String> `-> Java의 함수형 인터페이스 , String을 받아서 String을 반환하는 함수
`input -> passwordEncoder().encode(input)-> `
- input은 사용자가 입력한 비밀번호
- `passwordEncoder().encode()`에 넣어서 암호화된 비밀번호를 만들어냄.
----
#### 하드코딩된 유저ID 삭제하고 리팩토링하기

```java title="WelcomeController"
package com.in28minutes.springboot.myfirstwebapp.login;  
  
import org.springframework.security.core.Authentication;  
import org.springframework.security.core.context.SecurityContextHolder;  
import org.springframework.stereotype.Controller;  
import org.springframework.ui.ModelMap;  
import org.springframework.web.bind.annotation.*;  
  
@Controller  
@SessionAttributes("name")  
public class WelcomeController {  
  
    private String getLoggedInUser() {  
        Authentication authentication =  
                SecurityContextHolder.getContext().getAuthentication();  
        return authentication.getName();  
    }  
    //login  
    @RequestMapping(value = "/", method = RequestMethod.GET)  
    public String gotoWelcomePage(ModelMap model) {  
        model.put("name", getLoggedInUser());  
        return "welcome";  
    }  
}
```

`SecurityContextHolder` 
- Spring Security에서 현재 인증된 사용자 정보를 저장하고 관리하는 클래스
- 내부적으로 ThreadLocal을 사용하여 각 요청(쓰레드)마다 독립적인 인증 정보를 보관함.

`getContext()` 
- `SecurityContextHolder`에서 현재 요청과 관련된 보안 컨텍스트(SecurityContext)를 가져옴.
- 이 컨텍스트는 인증(Authentication) 정보를 포함함.

`getAuthentication()`
- `SecurityContext`에서 현재 사용자의 인증 정보를 반환함.
- 이 정보는 `Authentication` 객체로 표현되며, 로그인한 사용자에 대한 세부 정보를 담고 있음.

`authentication.getName()
- `Authentication` 객체에서 로그인한 사용자의 이름을 반환함.
- 일반적으로 이는 사용자가 로그인할 때 입력한 아이디입니다.

Spring Security는 로그인한 사용자의 정보를 SecurityContextHolder에 저장함.
애플리케이션에서 현재 로그인한 사용자의 정보를 자주 참조해야하는 경우가 많기때문에 필요함.
간단히 사용자의 정보를 가져올수있도록 설계됨.

#### TodoController로  리팩토링하기 
<font color="#00b050">코드 리팩토링 전</font> 
```java title="TodoController"
@RequestMapping("list-todos")  
public String listAllTodos(ModelMap model) {  
    List<Todo> todos = todoService.findByUsername("in28");  
    model.addAttribute("todos", todos);  
    return "listTodos";  
}
```
<font color="#00b050">코드리팩토링 후</font>
```java title="TodoController"
@RequestMapping("list-todos")  
public String listAllTodos(ModelMap model) {  
    String username = (String) model.get("name");  
    List<Todo> todos = todoService.findByUsername(username);  
    model.addAttribute("todos", todos);  
    return "listTodos";  
}
```

<font color="#00b050">코드 리팩토링 전 </font>
```java title="TodoService"
public List<Todo> findByUsername(String username) {  
	 return todos;
}
```
<font color="#00b050">코드 리팩토링 후 </font>
```java title="TodoService"
public List<Todo> findByUsername(String username) {  
    Predicate<? super Todo> predicate  
            = todo -> todo.getUsername() .equalsIgnoreCase(username);  
    return todos.stream().filter(predicate).toList();  
}
```

`todos.stream().filter(predicate).toList();`
 이건 `todos` 리스트(전체 할일 목록)중에서 위에서 만든 조건을 만족하는 것만 골라서 리스트로 만든 것
 
`.stream()`-> 리스트를 하나 씩 처리 가능한 흐름으로 바꿈
`.fliter(predicate)`-> 조건을 만족하는 요소만 남김
`.toList()` -> 다시 리스트로 모음

 ```java title=WelcomeController
 @Controller  
@SessionAttributes("name")  
public class WelcomeController {  
  
    private String getLoggedInUser() {  
        Authentication authentication =  
                SecurityContextHolder.getContext().getAuthentication();  
        return authentication.getName();  
    }  
    //login  
    @RequestMapping(value = "/", method = RequestMethod.GET)  
    public String gotoWelcomePage(ModelMap model) {  
        model.put("name", getLoggedInUser());  
        return "welcome";  
    }  
}
 ```
 `WelcomeController`에선 **Spring Security**로부터 name을 받을 수 있음. `getLoggedInUser()` 통해서 
 현재 인증된 사용자의정보를 받았기 때문이다.
 `@SessionAttributes`-> 모델(Model)에 있는 `"name"` 값을 세션에 등록해서 다음 요청까지도 기억되게 해주는 어노테이션
###### <font color="#ff0000">주의할 점</font>
- `@SessionAttribute` 는 Controller 단위로 적용됨
- 모델에 `addAttribute("name",..)` 해야 세션에도 들어감
- 세션에서 값 제거하려면 `SessionStatus.setComplete()`로 정리 필요함.


#### 문제 발생 -> listTodos에 아무것도 출력되지않음.
**<span style="background:rgba(255, 183, 139, 0.55)">발생원인</span>** -> `WelcomeController`에서 `getLoggedInUser()`로 인증된 사용자를 "name"으로 등록해주는데 `WelcomeController를` 거치지않고 바로 `TodoController` `"list-todos"`로 들어오면 세션에서 값을 사용할수 없다.

**<span style="background:#ff4d4f">해결방법</span>
- Spring Security로 부터 값을 직접 받기.
![[Pasted image 20250412163316.png]]

##### 세션에 있는 "name"값을  직접받는걸 Spring Security를 통해서 받는걸 리팩토링
```java 
private static String getLoggedInUsername(ModelMap model) {  
    return (String) model.get("name");  
}
```
로그인된 사용자를 model로부터 받는 대신에 Spring Security로부터 받기 
```java 
private static String getLoggedInUsername(ModelMap model) {  
    Authentication authentication =  
            SecurityContextHolder.getContext().getAuthentication();  
    return authentication.getName();  
}
```

---
#### Todo 애플리케이션에 새로운 사용자 설정하기

<span style="background:#ff4d4f">리팩토링 전 </span>
```java
@Configuration  
public class SpringSecurityConfiguration {  
    //LDAP or DataBase  
    //In Memory  

    @Bean  
    public InMemoryUserDetailsManager createUserDetailsManager() {  
    Function<String, String> passwordEncoder =  
        input -> passwordEncoder().encode(input);
        
        UserDetails userDetails =User.builder()  
                        .passwordEncoder(passwordEncoder)  
                        .username("in28")  
                        .password("dummy")  
                        .roles("USER", "ADMIN")  
                        .build();
  
        return new InMemoryUserDetailsManager(userDetails);  
    }  
  
    @Bean  
    public PasswordEncoder passwordEncoder() {  
        return new BCryptPasswordEncoder();  
    }  
```

```java title=SpringSecurityConfiguration
package com.in28minutes.springboot.myfirstwebapp.security;  
  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.security.core.userdetails.User;  
import org.springframework.security.core.userdetails.UserDetails;  
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;  
import org.springframework.security.crypto.password.PasswordEncoder;  
import org.springframework.security.provisioning.InMemoryUserDetailsManager;  
  
import java.util.function.Function;  
  
@Configuration  
public class SpringSecurityConfiguration {  
    //LDAP or DataBase  
    //In Memory  
    @Bean  
    public InMemoryUserDetailsManager createUserDetailsManager() {  
        UserDetails userDetails1 = createNewUser("in28", "dummy");  
        UserDetails userDetails2 = createNewUser("ye", "1234");  
        return new InMemoryUserDetailsManager(userDetails1,userDetails2);  
    }  
  
    private UserDetails createNewUser(String username, String password) {  
        Function<String, String> passwordEncoder =  
                input -> passwordEncoder().encode(input);  
  
        UserDetails userDetails =User.builder()  
                                .passwordEncoder(passwordEncoder)  
                                .username(username)  
                                .password(password)  
                                .roles("USER", "ADMIN")  
                                .build();  
        return userDetails;  
    }  
  
    @Bean  
    public PasswordEncoder passwordEncoder() {  
        return new BCryptPasswordEncoder();  
    }  
}
```

리팩토링 후 `createNewUser()` 통해서 새로운 사용자를 추가적으로 등록할수 있도록 바꾸어주었다.

----
## Spring Boot Starter Data JPA를 추가하고 H2 DB 준비하기

지금까지 정적 멤버 변수를 이용해서 모든 Todo를 저장했음. 이제는 인메모리 DB를 사용할것이다.
```java title=TodoService
@Service  
public class TodoService {  
  
    private static List<Todo> todos = new ArrayList<>();  
    private static int todosCount= 0;  
  
    static {  
        todos.add(new Todo(++todosCount, "in28minutes", "Learn AWS great job", LocalDate.now().plusYears(1),false));  
        todos.add(new Todo(++todosCount, "in28minutes", "Learn DevOps+Docker", LocalDate.now().plusYears(1),false));  
        todos.add(new Todo(++todosCount, "in28minutes", "Learn Full Stack Development", LocalDate.now().plusYears(1),false));  
    }
```

#### 의존성 추가하기 
```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-data-jpa</artifactId>  
</dependency>  
  
<dependency>  
    <groupId>com.h2database</groupId>  
    <artifactId>h2</artifactId>  
    <scope>runtime</scope>  
</dependency>
```

`<scope>` -> 어느 범위에서 쓰일지를 지정 
`runtime`-> 실행시에만 필요


#### H2 URL을 상수로 만들기
``` title=application.properties
spring.datasource.url=jdbc:h2:mem:testdb
```
![[Pasted image 20250412175847.png]]

##### 등록해놓은 URL로 들어가기
![[Pasted image 20250412175957.png]]
######  Spring Security설정으로 액세스 거부로 생기는 오류 발생
![[Pasted image 20250412180035.png]]
기본값으로서, 아무것도 설정하지 않으면 2가지로 설정이 됌.
- 모든 URL은 보호된다.-> 해당 사이트 페이지들을 이용하려면 로그인을 해야 한다.
- 인증되지 않은 요청은 로그인 폼이 보여진다.-> 자동으로 로그인 화면을 띄어줌.
- CSRF Disable : 테스트나 학습용 앱은 보안보다 편의가 더 중요하기 때문에 해제
- Frames  iframe 사용을 허용하거나 제한하는 설정 

<span style="background:#ff4d4f">Frames:</span> HTML`<iframe>`  태그를 써서 웹페이지 안에 다른 페이지를 끼워 넣는것
<span style="background:#ff4d4f">CSRF:</span>사이트 간 요청 위조, 사용자가 로그인 상태일 때, 해커가 몰래 요청을 보내서, 사용자의 의도와 다르게 무언가 실행되게 만드는 공격

##### SecurityFilterChain
요청이 들어오면 그 요청에 맞는 필터들의 묶음(FilterChain)을 적용할지 말지를 판단해주는 설정
Spring Security의 FliterChainProxy에 설정됨.

- 기본적으로 모든 URL 보호 및 인증되지않은 요청은 로그인폼을 보여주는 기능은 제공함.
- 여기에 나머지 2가지 기능을 설정한다.

#### HttpSecurity
- 예전 스프링 보안 설정 방식에서 쓰던 xml방식에서 자바 버전임.
- 특정 웹 요청에 대한 보안 설정을 할 수 있음.
- 모든 요청에 보안이 적용되지만, 원하면 특정 URL만 적용되게 설정 할 수 있음.

##### `Customizer.withDefaults()`
Spring Security의 기본 설정대로 사용한다 라는 의미를 가진 헬퍼 메서드 

```java title=SpringSecurityConfiguration
@Bean  
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {  
    http.authorizeHttpRequests(  
            auth -> auth.anyRequest().authenticated()  
    );  
    http.formLogin(withDefaults());  
    http.csrf().disable();  
    http.headers().frameOptions().disable();  
 return http.build();  
}
```
3번줄-> 모든 요청은 로그인한 사용자만 접근 가능
6번줄-> 로그인폼은 Spring Security 기본 로그인 폼으로 설정
7번줄->csrf 해제
8번줄->브라우저에서 이 웹사이트를 `<iframe>`안에 띄우는걸 허용함.

Spring Security 설정이 끝나서 올바르게 H2에 들어갈수있다.

---
#### Todo Entity를 만들고 Todo 데이터를 H2에 채워넣기 

```java title=Todo
@Entity  
public class Todo {  
  
    @Id    @GeneratedValue    private int id;  
    private String username;  
  
    @Size(min= 10, message = "Enter at least 10 character")  
    private String description;  
  
    private LocalDate targetDate;  
    private boolean done;  
  
    public Todo() {  
    }
    ///생략 

```

` @Entity -> ` Todo 클래스를 DB 테이블에 매핑함.

`@Id` -> PK(기본)키로 설정

`@GeneratedValue` ->지정한 기본키 값을 어떻게 자동으로 생성하는지 설정
- 설정을 하지않으면 기본으로` AUTO`이고 DB에 맞게 자동 선택됨.
- `IDENTITY`: MYSQL처럼 DB가 자동 증가하는 컬럼 사용
- `SEQUENCE`: 시퀸스 객체 사용(Oracle, PostgreSQL등)
- `TABLE`: 테이블로 시퀸스 흉내냄(잘 안 씀)


##### 시작 데이터를 채우기
`src/main/resources/data.sql`
```sql title=data.sql
insert into todo (ID, USERNAME, DESCRIPTION, TARGET_DATE, DONE)  
    values (10001,'in28','GET AWS Certified',CURRENT_DATE(),false);  
  
insert into todo (ID, USERNAME, DESCRIPTION, TARGET_DATE, DONE)  
    values (10002,'in28','GET Azure Certified',CURRENT_DATE(),false);  
  
insert into todo (ID, USERNAME, DESCRIPTION, TARGET_DATE, DONE)  
    values (10003,'in28','GET GCP Certified',CURRENT_DATE(),false);  
  
insert into todo (ID, USERNAME, DESCRIPTION, TARGET_DATE, DONE)  
    values (10004,'in28','Learn DevOps',CURRENT_DATE(),false);
 ```
먼저 테이블이 생성되고 위 데이터들이 삽입이 되어야 되는데 그전에 삽입이 되어서 생기는 오류 발생
아래처럼 오류 해결은 이렇게 하면 된다.

```properties title=application.properties
spring.jpa.defer-datasource-initialization=true
```
![[Pasted image 20250412234948.png]]
서버 재시작 시에는 인메모리 DB이기 때문에 데이터가 날라간다.

---
#### TodoRepository를 만들고 H2 DB와 list-todos 페이지 연결하기

```java title=TodoRepository
package com.in28minutes.springboot.myfirstwebapp.todo;  
  
import org.springframework.data.jpa.repository.JpaRepository;  
  
public interface TodoRepository extends JpaRepository<Todo,Integer> {  
}
```

정적 데이터는 이제 더 이상 사용하지 않고 DB에서 사용하려고 한다.

##### findByUsername()
해당 메서드는 JPA가 제공하지 않기때문에 메서드를 직접 생성해줬다.
```java title=TodoRepository
package com.in28minutes.springboot.myfirstwebapp.todo;  
  
import org.springframework.data.jpa.repository.JpaRepository;  
  
import java.util.List;  
  
public interface TodoRepository extends JpaRepository<Todo,Integer> {  
    public List<Todo> findByUsername(String username);  
}
```
Spring Data JPA는 username으로 검색하기 위한 메서드를 자동으로 제공함.
```java title=TodoControllerJpa
// /list-todos  
@RequestMapping("list-todos")  
public String listAllTodos(ModelMap model) {  
    String username = getLoggedInUsername(model);  
    List<Todo> todos = todoRepository.findByUsername(username);  
    model.addAttribute("todos", todos);  
    return "listTodos";  
}
```
`TodoRepository.findByUsername(username)` 이용해서 화면을 출력했는데 잘 출력되는걸 알수있다.

![[Pasted image 20250413003032.png]]

## 모든 Todo 기능을 H2 DB와 연결하기

변경한 Controller 기능들 (삭제, 변경 , 삽입)
```java title=TodoControllerJpa
  @PostMapping("add-todo")  
    public String addNewTodo(ModelMap model, @Valid Todo todo, BindingResult bindResult) {  
        if (bindResult.hasErrors()) {  
            return "todo";  
        }  
        String username = getLoggedInUsername(model);  
        todo.setUsername(username);  
        todoRepository.save(todo);  
//        todoService.addTodo(username, todo.getDescription(), todo.getTargetDate(), todo.isDone());  
        return "redirect:list-todos";  
    }
	@GetMapping("delete-todo")  
	public String deleteTodo(@RequestParam int id) {  
	    //Delete todo  
	    todoRepository.deleteById(id);  
	    return "redirect:list-todos";  
	}
	@PostMapping("update-todo")  
	public String updateTodo(ModelMap model, @Valid Todo todo, BindingResult bindResult) {  
	    if (bindResult.hasErrors()) {  
	        return "todo";  
	    }  
	  String username = getLoggedInUsername(model);  
	    todo.setUsername(username);  
	    todoRepository.save(todo);  
	    return "redirect:list-todos";  
	}

```
Todo 클래스에 다 값을 받아오지만 `username`만 인증된 사용자 정보로 받아오기 때문에 따로 `todo.setUsername()`로 설정을 해준다. `TodoRepository는` 모든 속성을 받는 메서드가 없기 때문에
`save(`)메서드로 todo를 입력값을 받는다.


--- 
#### Spring Boot Starter JPA와 JpaRepository의 세부 작동방식 이해하기

![[Pasted image 20250413005414.png]]

<font color="#ff0000">Spring Boot Auto Configuration은 </font>
- JPA와 Spring Data JPA 프레임워크를 초기화 
- 인메모리 DB H2도 시작됨.  애플리케이션 <-> DB 연결이 설정
- data.sql 같은 스크립트를 실행하려 한다면 시작할때 실행할수 있도록 설정함.
