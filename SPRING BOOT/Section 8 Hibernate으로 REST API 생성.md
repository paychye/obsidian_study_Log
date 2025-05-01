### Spring Boot로 REST API 생성하기

```java title=HelloWorldBean
package com.in28minutes.rest.webservices.restfulwebservices.helloworld;  
  
public class HelloWorldBean {  
    private String message;  
  
    public HelloWorldBean() {  
    }    public HelloWorldBean(String message) {  
        this.message = message;  
    }  
    public String getMessage() {  
        return message;  
    }  
    public void setMessage(String message) {  
        this.message = message;  
    }  
    @Override  
    public String toString() {  
        return "HelloWorldBean[ message=  " + message + " ]";  
    }  
}
```
**REST API에서 JSON을 반환하는 방법**
```java title=HelloController
@GetMapping("/hello-world-bean")  
public HelloWorldBean helloWorldBean() {  
    return new HelloWorldBean("Hello World");  
}
```
문자열을 반환하는 대신 인스턴스를 반환함.
##### 출력창으로 인스턴스를 반환하면 JSON으로 나오는걸 알수 있다.
![[Pasted image 20250414164622.png]]
##### 디버깅 로깅을 활성화하고 진행해보자
```title=applicaiton.properties
logging.level.org.springframework=debug
```

 **Spring MVC**에서 모든 요청은 **DispatcherServlet이** 처리함. ->  <span style="background:#ff4d4f">(FrontController Pattern)</span>
`DispatcherServlet은` 
-  사용하는 URL과 관계없이 가장 요청이 먼저 도달함
- Root URL에 매핑됨.
![[Pasted image 20250414165710.png]]
- 요청을 받으면 알맞은 Controller와 매핑함.
#### <span style="background:#d4b106">어디에서 `DispatcherServlet을` 설정하는가?</span>
- **Auto Configuration**에 의해 설정
- Spring Boot는  자동으로 웹 애플리케이션 혹은 REST API를 빌드한다는 걸 탐지하고 알아서 **DS**을 설정
#### <span style="background:#d4b106">왜 `HelloWorldBean은` JSON으로 변환되는가?</span>
 #### `@ResponseBody `+ `JacksonHttpMessageConverters`  이 2가지 설정이 JSON으로 변환시킨다.
 `@RestController` 에 들어보면 `@ResponseBody`가 포함되어있는걸 알수있다.
![[Pasted image 20250414170757.png]]
Bean을 있는 그대로 반환하라는 코드있으면 `Spring Boot Auto Configuration`이 기본 변환은 `JacksonHttpMessageConverters를` 사용함.
##### <span style="background:#d4b106">오류페이지는 어디에서 설정하는가?</span>
![[Pasted image 20250414171211.png]]
- 이것 역시 Auto Configuration(`ErrorMvcAutoConfiguration`)에 의해 구성됨.
- 아래 코드는 `ErrorMvcAutoConfiguration` 클래스 에서 나오는 View 코드 부분 ![[Pasted image 20250414171426.png]]

#### Path Parameter(경로 매개변수)란? -> `@PathVariable` 사용해서 매핑
- URL 경로의 일부로서 , 변수처럼 사용되는 값
- RESTful API에서 많이 사용.
```java title=HelloController
// /users/{id}/todos/{id} => /users/1/todos/10  
// /hello-world/path-variable/{name}  
// /hello-world/path-variable/Ranga  
@GetMapping("/hello-world/path-variable/{name}")  
public HelloWorldBean helloWorldPathVariable(@PathVariable String name) {  
    return new HelloWorldBean(String.format("Hello world, %s",name));  
}
```
![[Pasted image 20250414172320.png]]
`@PathVariable` -> String name에 매핑 되는 걸 알 수 있고 인스턴스에 포함해서 출력해보았다.

----
## SNS 애플리케이션용 REST API 설계하기

SNS 기본적으로 사용자와 게시물이고 조금 더 자세하게 들어가면 사용자의 아이디, 이름, 생년월일 그리고 게시물 id, 상세내용 등등
#### Request Methods for REST API
<font color="#ffc000">GET- > </font> 특정 리소스에서 상세정보를 검색할때 사용함.
<font color="#ffc000">POST-></font>  새로운 리소스를 생성하고싶을 때 , 새 사용자 , 새 게시물등등
<font color="#ffc000">PUT-></font> 기존 사용자의 상세정보를 업데이트할 때 
<font color="#ffc000">PATCH-></font>기존 리소스를 일부만 업데이트할때
 <font color="#ffc000">DELETE-></font>특정 사용자 , 게시물을 삭제할 때 
##### Users REST API
- 모든 유저 검색하기 -> GET/users
- 유저 생성하기 -> POST/users
- 특정 사용자를 검색하기(Path Parameter 사용)-> GET/users/{id}
- 특정 사용자 삭제하기(Path Parameter 사용)-> DELETE/users/{id}
- 게시물 생성 및 검색하기
	생성하기 -> GET/users/{id}/posts
    검색하기 -> POST/users/{id}/posts
    특정게시물검색하기-> GET/users/{id}/posts/{post_id}

**복수형을 선호하는 이유는->직관적이고 이해하기 쉬움**

---
#### 사용자 Bean과 UserDaoService 생성하기
```java title=User
package com.in28minutes.rest.webservices.restfulwebservices.user;  
  
import java.time.LocalDate;  
  
public class User {  
    private Integer id;  
    private String name;  
    private LocalDate birthday;  
  
    public User(Integer id, String name, LocalDate birthday) {  
        this.id = id;  
        this.name = name;  
        this.birthday = birthday;  
    }  
    public Integer getId() {  
        return id;  
    }  
    public void setId(Integer id) {  
        this.id = id;  
    }  
    public String getName() {  
        return name;  
    }  
    public void setName(String name) {  
        this.name = name;  
    }  
    public LocalDate getBirthday() {  
        return birthday;  
    }  
    public void setBirthday(LocalDate birthday) {  
        this.birthday = birthday;  
    }  
    @Override  
    public String toString() {  
        return "User{" +  
                "id=" + id +  
                ", name='" + name + '\'' +  
                ", birthday=" + birthday +  
                '}';  
    }  
}
```
```java title=UserDaoService
package com.in28minutes.rest.webservices.restfulwebservices.user;  
  
import org.springframework.stereotype.Component;  
  
import java.time.LocalDate;  
import java.util.ArrayList;  
import java.util.List;  
  
@Component  
public class UserDaoService {  
  
    // 모든 사용자 상세 정보를 DB 저장하고 JPA와 Hibernate를 활용해 DB와 통신함.
    //추후에 정적데이터가 아닌 DB와 통신하면 할 예정 .  
    private static List<User> users = new ArrayList<>();  
  
    static{  
        users.add(new User(1, "Adam", LocalDate.now().minusYears(30)));  
        users.add(new User(2, "Eve", LocalDate.now().minusYears(25)));  
        users.add(new User(3, "Jim", LocalDate.now().minusYears(20)));  
    }  
  
    public List<User> findAll() {  
        return users;  
    }  
}
```
###### DAO(Data Access Object) -> 데이터에 접근하는 객체
- DB랑 대화하는 역할을 전담하는 클래스 
- 데이터를 조회, 삽입, 수정, 삭제할때 마다 매번 복잡한 DB 코드를 여기저기 쓰면 너무 지저분하고 유지보수가 힘들기때문에 DAO 클래스에 한번에 모아서 사용
- 역할분리-> 비즈니스 로직과 DB 처리 코드를 분리
- 유지보수: DB 관련 코드는 DAO만 보면 됨.
- 재사용성: 여러 곳에서 같은 DAO 메서드 호출 가능 
---
### User Resource에서 GET메서드 구현하기 

```java title=UserResource
package com.in28minutes.rest.webservices.restfulwebservices.user;  
  
import org.springframework.web.bind.annotation.GetMapping;  
import org.springframework.web.bind.annotation.RestController;  
  
import java.util.List;  
  
@RestController  
public class UserResource {  
  
    private UserDaoService service;  
  
    public UserResource(UserDaoService service) {  
        this.service = service;  
    }  
    //GET /users  
    @GetMapping("/users")  
    public List<User> retrieveUsers() {  
        return service.findAll();  
    }  
}
```
![[Pasted image 20250415162146.png]]
##### 특정 사용자 조회하는 API

일단은 사용자아이디에 해당하는 유저만 출력되게 서비스 로직을 구성함.
```java title=UserDaoService
public User findById(@PathVariable int id) {  
    Predicate<? super User> predicate  = user -> user.getId() == id;  
   return users.stream().filter(predicate).findFirst().get();  
}
```

UserDaoService에서 만들어놓은 findById를 통해서 UserResource에서 사용해서 출력해주기
```java title=UserResource
//GET /users/{id}  
@GetMapping("/users/{id}")  
public User retrieveUser(@PathVariable int id) {  
    return service.findById(id);  
}
```
![[Pasted image 20250415163206.png]]
![[Pasted image 20250415163222.png]]
![[Pasted image 20250415163236.png]]

#### User Resource에서 Post 구현하기
`@RequestBody`란? 
- Client가 보낸 요청본문(Body)에 담긴 JSON 같은 데이터를 자바 객체로 바꿔주는 역할
- JSON 구조와 자바 클래스 필드 이름이 같아야 함
- 스프링이 Jackson 같은 라이브러리로 자동 변환해줌.

```java title=UserResource
@PostMapping("/users")  
public void createUser(@RequestBody User user) {  
    service.save(user);  
}
```



`private static int usersCount= 0;` -> 변수만들어서 아이디를 자동으로 증가하는 식

```java title=UserDaoService
package com.in28minutes.rest.webservices.restfulwebservices.user;  
  
import org.springframework.stereotype.Component;  
import org.springframework.web.bind.annotation.PathVariable;  
  
import java.time.LocalDate;  
import java.util.ArrayList;  
import java.util.List;  
import java.util.function.Predicate;  
  
@Component  
public class UserDaoService {  
  
    // 모든 사용자 상세 정보를 DB 저장하고 JPA와 Hibernate를 활용해 DB와 통신함.  
    private static List<User> users = new ArrayList<>();  
    private static int usersCount= 0;  
  
    static{  
        users.add(new User(++usersCount, "Adam", LocalDate.now().minusYears(30)));  
        users.add(new User(++usersCount, "Eve", LocalDate.now().minusYears(25)));  
        users.add(new User(++usersCount, "Jim", LocalDate.now().minusYears(20)));  
    }  
  
    public List<User> findAll() {  
        return users;  
    }  
  
    public User findById(@PathVariable int id) {  
        Predicate<? super User> predicate  = user -> user.getId() == id;  
       return users.stream().filter(predicate).findFirst().get();  
    }  
  
    public User save(User user) {  
        user.setId(++usersCount);  
        users.add(user);  
        return user;  
    }  
}
```
#### PostMan으로 Post 메서드를 보내주기
![[Pasted image 20250415170311.png]]![[Pasted image 20250415170357.png]]

### Post 메소드를  개선해 올바른 HTTP 상태 코드와 locat

정확한 응답 상태를 반환받는게 중요하기 때문에 status code를 알아보는 시간을 갖기

#### Status Code
1xx -> 처리 중 (거의 안 씀)
2xx ->  성공
3xx -> 리다이렉트(다른 곳으로 이동해)
4xx -> X 클라이언트가 잘못했어
5xx -> 서버(내가)가 잘못했어

`200(OK) ->` 요청 성공 : 잘 받았고 , 잘 처리함.
`201(Created)-> `새로 생성 완료 : 회원가입 완료했어~!
`204(No Content)->` 성공했지만 줄 건 없어 : 삭제했어 , 근데 응답은 없어

`404(Not Found)` -> 리소스를 찾지 못했다, 페이지 못찾겠다.
`400 (Bad Request)-`> 요청이 이상해! , 뭔가 잘못된 요청이야.
`401(Unauthorized)-> `인증 필요해!, 로그인해야 함.
`403(Forbidden)->` 접근 금지, 권한 없이, 못 들어감

`500(Internal Server Error)`-> 서버 터졌음

##### Status Code를 어떻게 변경할까?
```java title=UserResource
@PostMapping("/users")  
public ResponseEntity<User> createUser(@RequestBody User user) {  
    service.save(user);  
   return ResponseEntity.created(null).build();  
}
```
`ResponseEntity` -> 
- HTTP(응답)을 내 마음대로 조절해서 보내줄 수 있게 해주는 클래스
- 응답 상태 코드, 헤더 , Body 전부 다 설정해서 보낼 수 있음.

상태 코드 설정 -> `.status(HttpStatus.코드)`
Body 설정-> `.body(내용)`
Header 설정->` .headers(HttpHeaders 객체)`
자유롭게 조합 -> 원하는 응답을 내가 직접 구성 가능
###### 상태 코드 출력
![[Pasted image 20250415174927.png]]

###### 201 상태 코드는 보통 새로 생성된 리소스의 URI를 Location Header로 응답함.
```java title=UserResource
@PostMapping("/users")  
public ResponseEntity<User> createUser(@RequestBody User user) {  
    User saveUser = service.save(user);  
        URI location = ServletUriComponentsBuilder.fromCurrentRequestUri()  
            .path("/{id}")  
            .buildAndExpand(saveUser.getId())  
            .toUri();   
   return ResponseEntity.created(location).build();  
}
```

`ServletUriComponentsBuilder`
- 현재 요청을 기반으로 URI 를 동적으로 만들어주는 도구 
- users/{id} 같은 경우는  생성할 때마다 항상 바뀌기때문에 안전하게 URI를 만들수 있음.
`fromCurrentRequest()-` > 현재 요청 주소를 기반으로 시작
`path("/{id}") `-> 뒤에 경로를 덧붙임
`buildAndExpand(saveUser.getId())`-> saveUser.getId값으로 채움
`toUri()-`> 최종적으로 URI 객체로 변환
![[Pasted image 20250415175808.png]]

--- 
### 예외처리 구현하기 - 404 Resource Not Found

만약에 존재하지 않은 사용자를 조회하면 어떻게 될까?
![[Pasted image 20250415181049.png]]
Internal Server Error가 터지면서 예외 추적 로그도 찍혀 있기때문에 보안상 좋지 않음.
예외가 어디서 발생하는지 확인하고` Optional.get`

```java title=UserResource
public User findById(@PathVariable int id) {  
    Predicate<? super User> predicate  = user -> user.getId() == id;  
   return users.stream().filter(predicate).findFirst().orElse(null);  
}
```
조건에 맞는 항목이 없을때 NoSuchElementException발생, 그래서 첫번쨰에 항목도 없음.
조건항목이 없을때도 작동하는` orElse()`로 값이 있으면 있는 값 출력 없으면 Null 값 반환
###### 출력창
![[Pasted image 20250415231638.png]]
아무것도 뜨지않지만 status code는 200(성공)이다. 근데 이것 역시 좋지 못한 출력이다. 

```java title=UserResource
@GetMapping("/users/{id}")  
public User retrieveUser(@PathVariable int id) {  
    User user = service.findById(id);  
    if (user == null) {  
        throw new UserNotFoundException("id: " + id);  
    }  
    return service.findById(id);  
}
```

UserNotFoundException이라는 클래스는 없으니 만들어서 해당 존재하지 않는 id를 message로 반환
```java title=UserNotFoundException
package com.in28minutes.rest.webservices.restfulwebservices.user;  
  
public class UserNotFoundException extends RuntimeException {  
    public UserNotFoundException(String message) {  
        super(message);  
    }  
}
```
![[Pasted image 20250415232135.png]]하지만 오류 메시지는 잘 출력되지만 상태코드가 500으로 나오니 이부분 마저 수정해보자
```java title=UserNotFoundExceptionpackage com.in28minutes.rest.webservices.restfulwebservices.user;  
  
import org.springframework.http.HttpStatus;  
import org.springframework.web.bind.annotation.ResponseStatus;  
  
@ResponseStatus(code= HttpStatus.NOT_FOUND)  
public class UserNotFoundException extends RuntimeException {  
    public UserNotFoundException(String message) {  
        super(message);  
    }  
}
```
`@ResponseStatus(code= HttpStatus.NOT_FOUND)` -> 해당 클래스를 사용시 상태코드는 404(NOT FOUND)로 표시된다.
![[Pasted image 20250415232417.png]]


---
### 모든 리소스를 대상으로 예외 처리하기 

#### `ResponseEntityExceptionHandler.java`
Spring MVC에서 발생하는 모든 예외를 처리하는 `@ExceptionHandler` 메서드를 가진 클래스이며,
RFC 9457 형식의 오류 상세 정보를 `ResponseEntity`의 Body에 담아 반환함.
 전역 예외처리용 `@ControllerAdvice` 클래스의 기반(부모)클래스로 사용하기에 적합하다.
 하위 클래스는 개별 예외를 처리하는 메서드를 직접 재정의하거나 , 공통 처리 로직( `HandlerExceptionInternal`)이나 `ResponseEntity` 생성과정(createResponseEntity)도 재정의할수 있다.

`handleException` -> 여기에서 표준 구조가 반환됨.


에러 클래스에 어떤 걸 나타내줄지 정해서 만들어보자.
```java title=ErrorDetails
package com.in28minutes.rest.webservices.restfulwebservices.exception;  
  
import java.time.LocalDate;  
  
public class ErrorDetails {  
  
    private LocalDate timeStamp;  
    private String message;  
    private String details;  
    public ErrorDetails(LocalDate timeStamp, String message, String details) {  
        this.timeStamp = timeStamp;  
        this.message = message;  
        this.details = details;  
    }  
    public LocalDate getTimeStamp() {  
        return timeStamp;  
    }  
    public String getMessage() {  
        return message;  
    }  
    public String getDetails() {  
        return details;  
    }  
}
```

ErrorDetails 클래스를 이용해서 에러 응답에 포함할 내용(에러 정보)를 정의하고 그 내용을 바탕으로 에러응답(ResponseEntity)를 구성해서  Client에게 반환화도록 설정한 것.
```java title=CustomizedResponseEntityExceptionHandler
package com.in28minutes.rest.webservices.restfulwebservices.exception;  
  
import org.springframework.http.HttpStatus;  
import org.springframework.http.ResponseEntity;  
import org.springframework.web.bind.annotation.ExceptionHandler;  
import org.springframework.web.context.request.WebRequest;  
import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;  
  
import java.time.LocalDate;  

@ControllerAdvice
public class CustomizedResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {  
    @ExceptionHandler(Exception.class)  
    public final ResponseEntity<Object> handlerAllException(Exception ex, WebRequest request) throws Exception {  
        ErrorDetails errorDetails = new ErrorDetails(LocalDate.now(),  
                ex.getMessage(),  
                request.getDescription(false));  
        return new ResponseEntity<>(errorDetails, HttpStatus.INTERNAL_SERVER_ERROR);  
    }  
}
```

`@ControllerAdvice`
- Spring에서 전역적으로 예외처리, 바인딩 설정, 모델 설정 등을 적용해주는 핵심 어노테이션
-  여러 컨트롤러에서 공통적으로 적용할 처리 로직을 한 곳에서 정의할수 있도록 해주는 전역 컨트롤러 보조 클래스를 만드는 어노테이션 
- 한 곳에서 처리해서 유지 보수가 쉽고 코드 중복도 줄일 수 있음.

`@ExceptionHandler` 
- 예외(에러)가 발생했을 때 해당 예외를 처리할 메서드를 지정해주는 역할
- `(Exception.class)` -> 이 예외가 발생했을 때 , 해당 메서드가  실행돼서 에러 응답을 반환함.
`ResponseEntity.class`
- Spring에서 제공하는 클래스, Client에거 HTTP 응답 전체를 직접 구성해서 반환할수 있게 하는 제네릭 클래스
- HTTP Body과 Header를 가질수 있음.
![[Pasted image 20250416171206.png]]
해당 예외처리 출력내용을 보면 ErrorDetails클래스에서 필드 기반으로 반환 된것을 알수 있다.

LocalDateTime으로 변환해주면 출력되는 결과
![[Pasted image 20250416171516.png]]
존재하지않은 사용자를 검색했을때 -> 404 NOT FOUND 에러로 출력하고싶을 때 설정
```java title=CustomizedResponseEntityExceptionHandler
@ExceptionHandler(UserNotFoundException.class)  
public final ResponseEntity<Object> handlerUserNotFoundException(Exception ex, WebRequest request) throws Exception {  
    ErrorDetails errorDetails = new ErrorDetails(LocalDateTime.now(),  
            ex.getMessage(),  
            request.getDescription(false));  
    return new ResponseEntity<>(errorDetails, HttpStatus.NOT_FOUND);  
}
```
![[Pasted image 20250416235333.png]]Error의 구조를 정확하게 정의하고 적합한 응답 상태를 반환하는 건 REST API를 구현할때 매우 중요함.


### DELETE 메소드로 사용자 리소스 삭제하기

사용자를 삭제하는 서비스 로직을 컨트롤러에서 사용하기 전에 서비스 로직을 완성해보자.
```java title=UserResource
@DeleteMapping("/users/{id}")  
public void deleteUser(@PathVariable int id) {  
     service.deleteById(id);  
}
```
아래 코드를 보면 사용자를 삭제하는 로직인데 
```java title=UserDaoService
public void deleteById(int id) {  
    Predicate<? super User> predicate  = user -> user.getId() == id;  
    users.removeIf(predicate);  
}
```

`predicate<타입> 변수명 = (매개변수) -> 조건식;`
`users.removeIf(predicate);`->
리스트 안에서 그 조건으로 만족하는 요소들이 삭제됨.
위 코드가 내부적으로 아래와 거의 동일하게 작동됨.
```java
for (Iterator<T> it = list.iterator(); it.hasNext();) {
    T element = it.next();
    if (predicate.test(element)) {
        it.remove();  // 조건을 만족하면 제거!
    }
}
```
![[Pasted image 20250417164843.png]]
해당 사용자가 삭제된걸 상태코드를 통해서 알수 있다.

---
### REST API에서 유효성 검증하기
![[Pasted image 20250417165229.png]]
![[Pasted image 20250417165256.png]]
이름이 비어있는 채로 사용자를 추가해도 추가가 되는 걸 알 수 있는데 이런 데이터가 문제가 발생한다.
이 부분을 해결할 의존성이 필요한데 `Validation` 의존성을 추가해주겠다.
```xml title=pom.xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-validation</artifactId>  
</dependency>
```

```java title=UserResource
  @PostMapping("/users")  
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {  
        User saveUser = service.save(user);  
        URI location = ServletUriComponentsBuilder.fromCurrentRequestUri()  
                .path("/{id}")  
                .buildAndExpand(saveUser.getId())  
                .toUri();  
       return ResponseEntity.created(location).build();  
    }  
}
```
`@Valid`을 사용하면 바인딩 수행 될 때 객체에 정의된 유효성 검증이 자동으로 수행된다.

```java title=User
public class User {  
    private Integer id;  
      
    @Size(min=2)  
    private String name;  
      
    @Past  
    private LocalDate birthday;
    ....
```
`@Past`
- 과거 날짜/시간 데이터의 유효성을 보장하기 위함. 
- 현재 시점보다 이전인지 검증하여 예약 시스템, 출생일 입력 등에서 논리적 오류 방지
- 따로 `message=""`를 적지 않으면 기본 메시지를 내장하고 있어서 , 생략해도 자동으로 기본 메시지를 출력함.
- 이 메시지들은 
![[Pasted image 20250417174305.png]]
Validation으로 빈 이름을 사용하지 못하도록 했는데, 어떤 문제 때문에 상태코드가 Bad Request인지 설명을 제대로 해줘야겠죠? 항상 사용자의 입장을 고려해야합니다.


유효성 검사 실패를 처리할 때 호출되는 메서드
```java title=CustomizedResponseEntityException
@Override  
protected ResponseEntity<Object> handleMethodArgumentNotValid(  
        MethodArgumentNotValidException ex, HttpHeaders headers, HttpStatusCode status, WebRequest request) {  
    ErrorDetails errorDetails = new ErrorDetails(LocalDateTime.now(),  
            ex.getMessage(), request.getDescription(false));  
    return new ResponseEntity(errorDetails, HttpStatus.BAD_REQUEST);  
}
```

`MethodArgumentNotValidException`이 발생하면 기본적으로 400(Bad Request) 오류와 함께 오류 메시지를 보내줌. 기본 메시지는 우리가 원하는 포맷이나 내용이 아닐수 있기 때문에 오버라이드해서 에러 응답을 형식을 우리가 정한 `ErrorDetails` 객체로 바꾸고 , 에러 발생 시간, 메시지 요청 , 정보 등을 직접 구성해서 반환.

![[Pasted image 20250417181142.png]] 
설정한 해당 메시지가 출력되는 걸 확인할수있다.  하지만 출력되는 내용이 한눈에 보이지않아서 따로 설정해보도록 하겠다.

```java title=CustomizedResponseEntityExceptionHandler
@Override  
protected ResponseEntity<Object> handleMethodArgumentNotValid(  
        MethodArgumentNotValidException ex, HttpHeaders headers, HttpStatusCode status, WebRequest request) {  
    ErrorDetails errorDetails = new ErrorDetails(LocalDateTime.now(),  
            ex.getFieldError().getDefaultMessage(), request.getDescription(false));  
      
    return new ResponseEntity(errorDetails, HttpStatus.BAD_REQUEST);  
}
```
`ex.getFieldError()`
`MethodArguementNotValidException는` 유효성 검사 실패 시 발생하는 예외
그 중에서 첫 번째 오류 하나만 가져오는 메서드, 필드 단위의 검증 실패 정보를 담고 있는 객체


![[Pasted image 20250417181548.png]]
오류가 먼저 발생하는 부분만 반환하는거다. 자바 클래스 내부에서 필드 순서가 컴파일러에 따라 달라지거나 , 애노테이션 처리 순서도 보장되지 않기 때문에 모든 오류를 처리하는것이 좋은 방법이다.

```java 
@Override  
protected ResponseEntity<Object> handleMethodArgumentNotValid(  
        MethodArgumentNotValidException ex, HttpHeaders headers, HttpStatusCode status, WebRequest request) {  
    ErrorDetails errorDetails = new ErrorDetails(LocalDateTime.now(),  
        "Total Errors:"+  ex.getErrorCount()  +"First Error:"+ ex.getFieldError().getDefaultMessage(), request.getDescription(false));  
      
    return new ResponseEntity(errorDetails, HttpStatus.BAD_REQUEST);  
}
```
![[Pasted image 20250417182108.png]]
이렇게 에러의 개수를 나타내 줄 수 있는 메서드를 활용해봤다.


## Open API 사양 및 Swagger 파악하기

REST API를 잘 이해해야 하고 노출되고 있는 여러 리소스가  무엇인지 알아야 함.
또한 그에 맞는 요청 및 응답 구조 형식도 알아야 하는데 Cosumer가 기대하는 응답은 무엇인지 제약이나 검증은 있는지 따져봐야 합니다.

#### REST API 문서를 만들때 생기는 문제점
- 정확성: 최신 버전이고 정확한지 어떻게 확인 할 수 있을까?
- 일관성: 모든 API의 문서가 일관된 형식으로 이루어져 있는지 어떻게 보장할 수 있을까?

**해결방법**
	 1 .문서를 수동적으로 관리하기 -> 기본적으로 REST API의 관련 문서를 관리하는 문서나 HTML 파일을 갖고있음. 문서를 수동으로 관리하는 경우 : 코드와 동기화하는지 확인하기 위해 항상 노력을 기울어야 함.
	2 . 코드에서 문서를 생성하는 방법이 있음.

##### REST API 문서를 다룰때 이해해야할 중요한 2가지 용어 

###### SWAGGER 
- API 사용 설명서를 자동으로 만들어주는 도구
	 **주요 기능**: 실시간 문서화-> 코드 변경시 자동으로 설명서 업데이트 , 테스트  기능-> 설명서에서 직접 API 호출해 볼 수 있음 
##### Open API
- 누구나 사용할 수 있게 공개된 API(기상청 날씨 데이터, 카카오지도 등등)
- 특정 기능을 외부에서 쓸 수 있게 개방한 것


##### Springdoc-Openapi 설정하기 
```xml
<dependency>  
    <groupId>org.springdoc</groupId>  
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>  
    <version>2.5.0</version>  
</dependency>
```

```
http://localhost:8080/swagger-ui.html
```
![[Pasted image 20250428175548.png]]
![[Pasted image 20250428180021.png]]
# <span style="background:#d4b106">Content Negotiation</span>
##### 같은 URI를 사용할 때 -> localhost:8080/users
특정 리소스에 특정 URI가 있는 건데, 하지만 여러 표현이 가능함. 따라서 동일한 리소스에 대해 여러 표현을 보유하는 것이 가능함.
XML이나 JSON등 다른 콘텐츠 유형, 어떤 언어든지 표현 가능함.
어떻게 REST API 제공자에게 원하는 표현을 어떻게 알릴 수 있을까?
`Accept header` 를 통해서` application/xml` 값으로 생성하고 XML 응답을 원한다고 입력 할 수 있음.
`Accept-Language header` - 요청 헤더에는 Accept-Language를 헤더를 추가한  다음 영어나 네덜란드어 , 프랑스어를 원한다고 입력할수 있음.

이 작업에선 Request Header를 사용함.

###### <span style="background:#d4b106">일단 POM.XML 설정</span>
XML을 JSON처럼 다루게 해주는 확장 모듈 : Java < - > XML 간 변환을 가능하게 해줌
```xml
<dependency>  
    <groupId>com.fasterxml.jackson.dataformat</groupId>  
    <artifactId>jackson-dataformat-xml</artifactId>  
</dependency>
```
![[Pasted image 20250428181306.png]]
위 결과처럼 XML으로 나오는걸 알수있다.

XML 표현과 Swagger문서는 다른 고급 기능을 다룰 때 몇 가지 문제를 일으키기 때문에 주석 처리하겠다.

##### <font color="#ffc000">전 세계의 사용자들에게 REST API를 사용자 정의하려면 어떻게 하면 될까?</font>
 바로 국제화를 사용해야 함.
국제화를 처리할때 HTTP Request header를 사용함.(`Accept Language`)
예를 들어서 
'en' -> Good Morning(영어)
'nl' -> Goedemorgen(독일어)
'fr' -> Bonjour(프랑스어)
'de'-> Deutsch(네덜란드어)

```java title=HelloController
@GetMapping("/hello-world-internationalized")  
public String helloWorldInternationalized() {  
    return "Hello World V2";  
}
```

첫번째는 위에 인사말을 가져올수있도록 메시지를 어딘가에 저장해야 하는데 
두번쨰로는 이 값을 가져올수 있는 코드를 작성해야 함.

###### main/resources/messages.properties 파일을 하나 만들기
``` title= messages.propeties
good.morning.message=Good Morning
```


```java title=helloController
@RestController  
public class HelloController {  
  
    private MessageSource messageSource;
    
	public HelloController(MessageSource messageSource) {  
    this.messageSource = messageSource;  
	}
    
  @GetMapping("/hello-world-internationalized")  
public String helloWorldInternationalized() {  
    Locale locale = LocaleContextHolder.getLocale();  
    return messageSource.getMessage("good.morning.message", null, "Default Message", locale);  
	} 
}
```

`MessageSource` 는  Spring Framework에서 다국어 지원을 위한 인터페이스
주로 국제화된 메시지를 관리하고 , 코드에 따라 메시지를 가져오는 데 사용됨.

- `Locale locale= LocaleContextHolder.getLocale();` 
   현재 스레드에 바인딩된 Locale(언어 및 지역 정보)를 반환함.
   Spring에서는 보통 사용자의 요청에 따라 Locale를 자동으로 설정하고 , 이 메서드를 통해 이를 가져옴.
- `messageSource.getMessage("good.morning.message", null , "Default Message", locale)`
	`key`-> messages.properties에 정의된 메시지 키
	`args`-> 메시지에 동적으로 넣을 파라미터
	`default Message`->  해당 키가 없을 때 대신 반환할 기본 메시지
	`locale`-> 언어 설정

![[Pasted image 20250429165101.png]]
사용자의 지역이 네덜란드면 저렇게 네덜란드 말이 나오는걸 알수있다. 

#### <font color="#ffc000">REST API 버전관리 - URI 버전 관리</font>
지금처럼 이름 전체를 반환하던 API를 성과 이름을 나눠서 반환하도록 갑자기 바꾸면, 기존 사용자에게 문제가 생깁니다.  
API를 바꿀 땐 사용자도 바로 수정해야 하므로 부담이 큽니다.  
그래서 API 구조를 바꿀 땐 **즉시 적용**하지 말고, **버전 관리**를 통해 사용자가 원할 때 새 버전으로 넘어갈 수 있게 해야 합니다.
###### <font color="#ffc000">해결 방안</font>
- <font color="#ffc000">URL</font> 
- Request Parameter
- Header
- Media Type

먼저 URL을  기준으로 REST API 버전관리를 해보자
```java title=PersonV1
package com.in28minutes.rest.webservices.restfulwebservices.versioning;  
  
public class PersonV1 {  
    private String name;  
    public PersonV1(String name) {  
        this.name = name;  
    }  
    public String getName() {  
        return name;  
    }  
    @Override  
    public String toString() {  
        return "PersonV1{" +  
                "name='" + name + '\'' +  
                '}';  
    }  
}
```


```java title=PersonV2
package com.in28minutes.rest.webservices.restfulwebservices.versioning;  
  
public class PersonV2 {  
    private Name name;  
    public PersonV2(Name name) {  
        this.name = name;  
    }  
    public Name getName() {  
        return name;  
    }  
  
    @Override  
    public String toString() {  
        return "PersonV2{" +  
                "name=" + name +  
                '}';  
    }  
}
```

```java title=Name
package com.in28minutes.rest.webservices.restfulwebservices.versioning;  
  
public class Name {  
    private String lastName;  
    private String firstName;  
  
    public Name(String lastName, String firstName) {  
        this.lastName = lastName;  
        this.firstName = firstName;  
    }  
    public String getLastName() {  
        return lastName;  
    }  
  
    public String getFirstName() {  
        return firstName;  
    }  
    @Override  
    public String toString() {  
        return "Name{" +  
                "lastName='" + lastName + '\'' +  
                ", firstName='" + firstName + '\'' +  
                '}';  
    }  
}
```

```java title=VersioningPersonController
package com.in28minutes.rest.webservices.restfulwebservices.versioning;  
  
import org.springframework.web.bind.annotation.GetMapping;  
import org.springframework.web.bind.annotation.RestController;  
  
@RestController  
public class VersioningPersonController {  
  
    @GetMapping("/v1/person")  
    public PersonV1 getFirstVersionOfPerson() {  
        return new PersonV1("Bob Charlie");  
    }  
}
```

```json
  
{"name": "Bob Charlie"}

```

```java
@GetMapping("/v2/person")  
public PersonV2 getSecondVersionOfPerson() {  
    return new PersonV2(new Name("Bob", "Charlie"));  
}
```

```json
  
  
{"name": {"lastName": "Bob","firstName": "Charlie"}}

```

#### Request Parameter 
```java
@GetMapping(path="/person", params = "version=1")  
public PersonV1 getFirstVersionOfPersonRequestParameter() {  
    return new PersonV1("Bob Charlie");
}
@GetMapping(path="/person", params = "version=2")  
public PersonV2 getSecondVersionOfPersonRequestParameter() {  
    return new PersonV2(new Name("Bob", "Charlie"));  
}
```
#### Header로 버전관리
```java
@GetMapping(path = "/person/header",headers ="X-API-VERSION=1")  
public PersonV1 getFirstVersionOfPersonRequestHeader() {  
    return new PersonV1("Bob Charlie");  
}  
  
@GetMapping(path = "/person/header", headers = "X-API-VERSION=2")  
public PersonV2 getSecondVersionOfPersonRequestHeader() {  
    return new PersonV2(new Name("Bob", "Charlie"));  
}
```
#### Media으로 버전관리
```java
@GetMapping(path = "/person/accept",produces ="application/vnd.company.app-v1+json")  
public PersonV1 getFirstVersionOfPersonAcceptHeader() {  
    return new PersonV1("Bob Charlie");  
}  
@GetMapping(path = "/person/accept",produces ="application/vnd.company.app-v2+json")  
public PersonV2 getSecondVersionOfPersonAcceptHeader() {  
    return new PersonV2(new Name("Bob", "Charlie"));  
}
```

##### REST API 버전 관리 방법을 결정할때 고려사항
- **URI Pollution** -> URI 버전 관리와 요청 매개 변수 버전 관리의 경우, URL Pollution이 상당히 많이 발생함. header나 Media를 이용한 버전관리는 URI Pollution이 적음

- **HTTP 헤더의 오용 ->** 버전 관리용도로 사용해서는 안됨. 따라서 헤더 버전, 미디어 유형 버전 관리는 HTTP 헤더를 오용하고 있는것.

- **Caching->** 일반적으로  URL을 기반으로 수행되는데 , 헤더 버전와 미디어 유형 버전 관리은 URL을 기반으로 캐싱을 할수  없기 때문에  캐싱을 수행하기 전에 헤더를 살펴봐야함.

- **브라우저에서 요청을 실행할수 있는지?** -> URI , Request경우 URL로 간편하게 브라우저에서 사용할 수 있음 ,하지만 헤더 , 미디어 버전 관리에서는 헤더있기 때문에 일반적으로 명령줄 유틸리티를 갖고 있거나 REST API 클라이언트를 사용해서 헤드를 기준으로 구분할 수 있어야 함.

- **API 문서 ->** URI, RequestParam은 API문서를 생성하기는 쉬움 , 하지만 헤더 기준으로 구분하는 문서의 생성을 지원하지 않을 수 있음.

<span style="background:#d4b106">REST API를 구축하기 시작할 때 버전 관리에 대해 생각해야 함.</span>
<span style="background:#d4b106"> 일관된 버전 관리 방식을 사용해야 함.</span>


##### <font color="#ffc000">HATEOAS (Hypermedia as the engine of Application State)</font>
서버가 클라이언트에게  지금 이 데이터를 보고 난 뒤엔 이런 행동을 할수 있어 라고 알려주는 시스템
모든 API 응답에 다음 가능한 행동을 하이퍼링크로 제공하는 규칙 
클라이언트가 서버의  API문서 없이도 모든 기능을 자동 발견할 수 있게 함.
##### <span style="background:#d4b106"> 구현방식</span>
 
- 사용자 정의 형식과 구현 하는 것은 유지 관리하기엔 까다롭다.
- 표준 구현 방식: HAL을 이용하기
	API의 리소스간에 하이퍼링크를 생성하는 일관적이고 쉬운 방법을 제공하는 간단한 방법 
	_links라는 요소를 생성하면 그안에 여러 링크를 가질 수 있음.
	표준을 갖고 있으면 좋은 점은 모든 애플리케이션이 이 표준을 따른다는 것

##### <span style="background:#d4b106">스프링 HATEOAS 의존성 추가하기 </span>
```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-hateoas</artifactId>  
</dependency>
```

HATEOAS를 사용하여 링크를 추가하려면
1. `EntityModel`로 객체 감싸기
- 반환할 도메인 객체(`User`등)를 `EntityModel.of(user)`로 감쌈
2. 링크 추가하기
 - `WebMvcLinkBuilder`를 사용해 링크 생성 
- `linkTo(methodOn(this.getClass()).retrieveAllUsers());`
3. 링크를 `EntityModel`에 추가
- `add(link) `메서드 사용

`EntityModel`은 데이터 + 링크를 함께 전달하는 래퍼
`WebMvcLinkBuilder`로 컨트롤러 메서드에 대한 링크 생성 
링크는 `withSelfRel()` , `withRel("다른이름")`등으로 설정 가능 

```java title=UserResource
import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.linkTo;  
import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.methodOn;
import org.springframework.hateoas.server.mvc.WebMvcLinkBuilder;

@GetMapping("/users/{id}")  
public EntityModel<User> retrieveUser(@PathVariable int id) {  
    User user = service.findById(id);  
    if (user == null) {  
        throw new UserNotFoundException("id: " + id);  
    }  
    EntityModel<User> entityModel = EntityModel.of(user);  
    WebMvcLinkBuilder link = linkTo(methodOn(this.getClass()).retrieveAllUsers());  
    entityModel.add(link.withRel("all-users"));  
    return entityModel;  
}
```

```json
  
{"id": 1,
	"name": "Adam",
	"birthday": "1995-04-29",
		"_links": {
				"all-users": {
						"href": "[http://localhost:8080/users](http://localhost:8080/users)"
						}
			}
	}
```

## REST API 정적 필터링 구현하기

###### REST API 응답의 커스터마이징
<font color="#ffc000">Serialization(직렬화)</font> -> 
- 객체를 바이트(Byte)형태로 변하는 과정
- 컴퓨터 프로그램에서 사용하는 객체는 메모리 안에서만 쓸수 있음, 이 객체를 파일로 저장하거나, 다른 컴퓨터로 보내려면 "메모리 안의 복잡한 모습" 을 바깥에서도 쓸 수 있게 일렬로 펼쳐진 데이터로 바꿔야 함.

커스터마이징 하는 방법 
1.  `@JsonProperty`->
	 JSON 데이터와 자바 객체의 필드 이름이 다를 때, 서로 연결해주는 역할을 하는 어노테이션
	 **"JSON에서 오는 데이터의 이름과 내 자바 변수 이름이 다를 때 , 둘을 이어주는 라벨"**

```java title=user
public class User {  
    private Integer id;  
  
    @Size(min=2, message = "Name should have atleast 2 characters")  
    @JsonProperty("user_name")  
    private String name;  
 .... 
}
```
```json 
  
[{"id": 1,"name": "Adam","birthday": "1995-05-01"},{"id": 2,"name": "Eve","birthday": "2000-05-01"},{"id": 3,"name": "Jim","birthday": "2005-05-01"}]

//@JsonProperty 적용 후
[{"id": 1,"birthday": "1995-05-01","user_name": "Adam"},{"id": 2,"birthday": "2000-05-01","user_name": "Eve"},{"id": 3,"birthday": "2005-05-01","user_name": "Jim"}]

```
2. Flitering - >
	  정적 필터링 : `@JsonIgnoreProperties,` `@JsonIgnore`
	  동적 필터링: `@JsonFliter` with FilterProvider

 ```java title=SomeBean
 package com.in28minutes.rest.webservices.restfulwebservices.flitering;  
  
import com.fasterxml.jackson.annotation.JsonIgnore;  
  
public class SomeBean {  
  
    private  String field1;  
  
    @JsonIgnore  
    private String field2;  
  
    private String field3;  
  
    public SomeBean(String field1, String field2, String field3) {  
        this.field1 = field1;  
        this.field2 = field2;  
        this.field3 = field3;  
    }  
    public String getfield1() {  
        return field1;  
    }  
    public String getfield2() {  
        return field2;  
    }  
    public String getfield3() {  
        return field3;  
    }  
    @Override  
    public String toString() {  
        return "SomeBean{" +  
                "field1='" + field1 + '\'' +  
                ", field2='" + field2 + '\'' +  
                ", field3='" + field3 + '\'' +  
                '}';  
    }  
}
```
```json
//적용 하기 전 
{
	"field1": "value1",
	"field2": "value2",
	"field3": "value3"
}
// @JsonIgnore 적용 후
{
	"field1": "value1",
	"field3": "value3"
}

[
	{"field1": "value1","field3": "value3"},
	{"field1": "value4","field3": "value6"}
]
```
`SomeBean`의 리스트가 반환 되더라도 field2는 전혀 표시되지 않을 겁니다.
`/fliterinmg-list` 를 적용하면 이처럼 리스트의 모든 요소에 대해서도 field2에 대해서 표시가 되지 않는다.

```java
  @JsonIgnoreProperties("field1")
public class SomeBean {  
    private  String field1;  
    @JsonIgnore  
    private String field2;  
    private String field3;  
```
```json
[
	{"field3": "value3"},
	{"field3": "value6"}
]
```

`@JsonIgnoreProperties`를 사용하는 대신 특정 필드에 `@JsonIgnore`를 추가하는게 낫습니다.
추후 필드 이름을 변경하더라도 실제로 다른 곳에서 변경할 필요가 없음.