
#### <font color="#ebf1dd">Spring Boot를 사용하기 전 상황</font>
Spring Project를 설정하는 작업은 쉽지 않았음.
많은 것을 설정한 후에야 프로덕션 환경에 사용 가능한 애플리케이션을 얻을수 있었음.

#### <font color="#ffc000">아래에 나열한 문제점들은 개발자가 직접 수동으로 구현해야 했음.</font>
###### 문제점 1: 의존성 관리
- Spring Framework, Spring MVC, JSON 등등에 대한 버전관리및 호환성 문제 해결하는데 어려움
###### 문제점 2: web.xml
- 웹 애플리케이션의 많은 것을 설정이 필요함.
###### 문제점 3: Spring 설정
- 컴포넌트 스캔 , 웹 애플리케이션을 빌드한다면 View Resolver를 정의해야함.
###### 문제점 4: 비기능적 요구사항
- 애플리케이션에 로깅, 오류처리, 프로덕션 단계의 애플리케이션의 모니터링

-------------- 

# HelloWorld API 빌드해보기

```java
package com.in28minutes.springboot.learnspringboot;  
  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RestController;  
  
import java.util.Arrays;  
import java.util.List;  
  
@RestController  
public class CourseController {  
    //http:localhost:8080/courses  
    @RequestMapping("/courses")  
    public List<Course> retrieveAllCourses() {  
        return Arrays.asList(  
                new Course(1,"Learn AWS", "in28minutes"),  
                new Course(2,"Learn DevOps", "in28minutes")  
        );  
    }  
}
```
```java
package com.in28minutes.springboot.learnspringboot;  
  
public class Course {  
    private long id;  
    private String name;  
    private String author;  
  
    //Constructor  
    public Course(long id, String name, String author) {  
        this.id = id;  
        this.name = name;  
        this.author = author;  
    }  
    //Getters  
    public long getId() {  
        return id;  
    }  
    public String getName() {  
        return name;  
    }  
    public String getAuthor() {  
        return author;  
    }  
    //toString  
    @Override  
    public String toString() {  
        return "Course{" +  
                "id=" + id +  
                ", name='" + name + '\'' +  
                ", author='" + author + '\'' +  
                '}';  
    }  
}
```
### 출력창
```json title:"localhost:8080/courses"
[
  {
    "id": 1,
    "name": "Learn AWS",
    "author": "in28minutes"
  },
  {
    "id": 2,
    "name": "Learn DevOps",
    "author": "in28minutes"
  }
]
```

#### <span style="background:#b1ffff">Spring Boot의 가장 중요한 목표</span>
프로덕션 환경에서 사용 가능한 애플리케이션을 빠르게 빌드할수 있도록 돕는것.
#### <font color="#ffc000">Spring Boot Starter Project</font>
###### 빠르게 빌드하는것
- **Spring Initializr**
- **Spring Bott Starter Projects**
- **Spring Boot Auto Configuration**
- **Spring Boot DevTools**
###### 프로덕션을 올바르게 실행하기 위한 세팅
- Logging
- 여러 환경에 맞는 다양한 설정 제공
- Monitoring(Spring Boot Actuator)
#### <span style="background:#b1ffff">애플리케이션을 빌드할 때는 프레임워크가 많이 필요함.</span>
###### REST API를 빌드 할때
- Spring, Spring MVC, Tomcat, JSON conversion 등등
###### Unit test 할때
- SPring Test, Unit, Mockito

####  여러 프레임워크를 어떻게 그룹화를 해서 빌드를 할까?
바로 Starter Project으로 다양한 기능을 위한 편리한 의존성 디스크립터를 제공합니다.

#### `pom.xml`를 확인해보자.

```xml title:"Spring Boot Starter Web"
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter</artifactId>  
    <version>3.3.10</version>  
    <scope>compile</scope>  
  </dependency>  
  <dependency>    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-json</artifactId>  
    <version>3.3.10</version>  
    <scope>compile</scope>  
  </dependency>  
  <dependency>    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-tomcat</artifactId>  
    <version>3.3.10</version>  
    <scope>compile</scope>  
  </dependency>  
  <dependency>    <groupId>org.springframework</groupId>  
    <artifactId>spring-web</artifactId>  
    <version>6.1.18</version>  
    <scope>compile</scope>  
  </dependency>  
  <dependency>    <groupId>org.springframework</groupId>  
    <artifactId>spring-webmvc</artifactId>  
    <version>6.1.18</version>  
    <scope>compile</scope>  
  </dependency>  
</dependencies>
```

Spring Boot Starter는 필요한 의존성을 모두 정의합니다.
Web APP 이나 REST API를 정의할때 필요한 의존성을 모두 사전 정의되어있음.
특정한 프로젝트에 필요한 기능에 따라 다양한 Starter Project를 사용할수 있음.
#### <span style="background:#affad1">SpringBoot에서 제공하는 다양한 프로젝트들</span>
- **Web Application & REST API**
- **Unit Test**
- **Talk to DB using JPA**
- **Spring Security**
## <span style="background:#affad1">Spring Boot Auto Configuration</span>

일반적으로 Spring Boot를 사용하면 웹 어플을 빌드 할때 많은 설정이 필요함
Component Scan, DispathcherServlet, Data Sources, JSON Conversion 등등

#### 위에 여러 작업들을 간소화하려면 어떻게 해야될까?
바로 스프링 부트에  기본으로 제공하는 자동설정(Auto Configuration)를 사용하면 된다.
Auto Configuration은 클래스 경로에 있는 프레임워크에 따라 생성됩니다.

- 모든 Auto Configuration 로직은 특정 jar에서 정의가 되는데, 
 Maven dependencies에서  spring-boot-autoconfigure.jar를 들어가면 자동설정 클래스가 많은 것을 확인 할수 있다.

자동설정을 좀 더 자세히 알아보려면 `src/main/resources-`>`applicaiton.properties` 이동하면 로깅수준을 설정할수 있다.



## Logging에 대해서
#### Negative matches
- 자동 설정되지 않은 항목
#### Positive Matches
- 자동 설정된 항목 -> `DispatcherServlet` ,`ErrorMvc` ,`Tomcat`, `Json Conversion`

#### <span style="background:#affad1">`DispatcherServletAutoConfiguration`</span>
```java
@ConditionalOnWebApplication(type = Type.SERVLET)  
@ConditionalOnClass({DispatcherServlet.class})
```
여기서 이 특정 자동 설정이 사용 설정되는 시점에 관한 설정을 확인할수 있음
웹 애플리케이션이나 REST API를 위해 사용 설정됨 , 클래스 경로에 DispatcherServlet이 있을 때 사용 설정됨.
Spring Web Starter에 포함했기 떄문에 이미 클래스 경로에 있고, 따라 자동 설정됨.

#### <span style="background:#affad1">`ErrorMvcAutoConfiguration`</span>
Default 오류 페이지를 설정하는 내용
애플리케이션은 URL이 매핑되지 않았음을 파악하여 적절한 오류메시지를 사용자에게 표시
![[Pasted image 20250323225312.png]]
# <span style="background:#affad1">Spring Boot DevTools</span>

- 개발자 생산성을 높일수 있습니다.
-  코드를 변경할 때마다 수동으로 서버 재시작하지않고 자동으로 서버를 다시 시작하고 코드 변경사항을 적용함.

# Configuration using Profiles
- 프로필을 통해 환경 별 설정을 제공함
![[Pasted image 20250323231156.png]]
특정한 프로필을 사용할 때 하는 방법
``` title="application.properties"
spring.application.name=learn-spring-boot  
logging.level.org.springframework=debug  
spring.profiles.active=prod --> 여기를 dev OR prod로 프로필로 바꿀수있다.
```


# Spring에서 권장하는 방식

#### `@ConfigurationProperties(prefix="currency-service")`

``` title="application.propeties"
spring.application.name=learn-spring-boot  
logging.level.org.springframework=debug  
spring.profiles.active=dev  
  
currency-service.url=http://default.in28minutes.com  
currency-service.username=defaultusername  
currency-service.key=defaultkey
```

```java title="CurrencyServiceConfiguration"

package com.in28minutes.springboot.learnspringboot;  
  
  
import org.springframework.boot.context.properties.ConfigurationProperties;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.stereotype.Component;  
  
//currency-service.url=  
//currency-service.username=  
//currency-service.key=  
  
@ConfigurationProperties(prefix = "currency-service")  
@Component  
public class CurrencyServiceConfiguration {  
    private String url;  
    private String username;  
    private String key;  
  
    public String getUrl() {  
        return url;  
    }  
    public void setUrl(String url) {  
        this.url = url;  
    }  
    public String getUsername() {  
        return username;  
    }  
    public void setUsername(String username) {  
        this.username = username;  
    }  
    public String getKey() {  
        return key;  
    }  
    public void setKey(String key) {  
        this.key = key;  
    }  
}
```
###### <font color="#f79646">설정 파일(application.properties or application.yml)의 값을 Java 객체로 매핑</font>
-  `prefix= "currency-service"`를 사용하면 설정파일에서 `"currency-service"` 로 시작하는 값을 해당 객체의 필드에 자동으로 매핑해줘
###### <font color="#ffc000">Setter 기반 바인딩을 지원</font>
- 매핑되는 클래스는 기본 생성자가 필요하고, 필드는 private이어도 Setter 메서드가 존재해야 값이 주입됨.
###### <font color="#f79646">타입 안정성 보장</font>
-` @Value`를 이용해서 개별 값을 주입할 수 있지만, `@ConfigurationProperties`를 사용하면 전체 설정을 하나의 객체로 관리할 수 있어서 더 편리하고, 타입 안정성도 보장됨.



```java title="CurrencyController"
package com.in28minutes.springboot.learnspringboot;  
  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RestController;  
  
import java.util.Arrays;  
import java.util.List;  
  
@RestController  
public class CurrencyController {  
    @Autowired  //
    private CurrencyServiceConfiguration configuration;  
  
    @RequestMapping("/currency-configuration")  
    public CurrencyServiceConfiguration retrieveAllCourses() {  
        return configuration;  
    }  
}
```

`@Autowired`를 해주면서 해당 `CurrencyServiceConfiguration` 빈이  `CurrencyController에` 의존성 주입
``` json title="localhost:8080/currency-configuration"
{
  "url": "http://dev.in28minutes.com",
  "username": "devusername",
  "key": "devkey"
}
```

프로필을 dev로 바꾸면 dev properties에 대한 결과가 나온다.
이를 통해 애플리케이션에 필요한 모든 설정을 외부화할수 있음.

------------
# SpringBoot (Embedded Server)

### WAR Approach 배포방식(복잡함.)
1. 자바 설치
2. Web/Application 서버 설치(Tomcat,WEblogic, WebSphere)
3. WAR 파일로 배포를 함.
### Embedded Server(간단함) -> 추천
1. 자바설치
2. 자바만 설치하면 JAR파일을 실행할수 있음.

웹서버를 따로 설치해주지 않아도 된다 , 왜냐면 웹 서버가 JAR파일의 일부이기 때문.
전체 배포 프로세스가 간소화됩니다.
기본 웹 서버는 Tomcat

-----
# Spring Boot Acutuator
- 관리 및 모니터링
- 여러 개의 엔드 포인트를 제공 
	beans: SpringBoot에서 사용하는 빈확인
	helath: 애플리케이션이 제대로 작동하는지 확인
	metrics- 애플리케이션 매트릭스
	mappings- 설정된 모든 요청 매핑 관련 세부 사항을 확인 

## 어떻게 사용할까?
``` title="dependecies에 추가하기"
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-test</artifactId>  
    <scope>test</scope>  
</dependency>
```

### http://localhost:8080/actuator 들어가기
```json title="http://localhost:8080/actuator"
{"_links": {"self": {"href": "[http://localhost:8080/actuator](http://localhost:8080/actuator)","templated": false},"health": {"href": "[http://localhost:8080/actuator/health](http://localhost:8080/actuator/health)","templated": false},"health-path": {"href": "[http://localhost:8080/actuator/health/{*path}](http://localhost:8080/actuator/health/%7B*path%7D)","templated": true}}}
```

##### 더 많은 기능을 사용하려면 properties를 통해서 설정해야된다.
``` title="application.properties"
management.endpoints.web.exposure.include=*
```
이렇게 하면 Acutatuor에서 제공하는 모든 엔드포인트가 노출됨.

#### 주요 기능만 살펴보자면?
``` json title="http://localhost:8080/actuator/beans"
"beans": {"href": "[http://localhost:8080/actuator/beans](http://localhost:8080/actuator/beans)","templated": false},
```
자동설정된 모든것에 대해서는 여기 표시된 bean을 확인하면 됨. 
특정 항목이 자동설정되었는지 아닌지를 확인하려면 이 방법을 사용해도 됨

##### <font color="#f79646">`Configprops`</font>
application.properties에서 설정할수 있는 모든 항목이 표시됨
#### <font color="#f79646">`env`</font>
환경에 관한 세부 사항을 모두 표시함
8080 포트를 사용하고 자바 17을 사용한다는 걸 표시해준다.
##### <font color="#f79646">`metrics`</font>
애플리케이션의 운영 상태, 성능, 리소스 사용량등의 다양한 매트릭(지표)을 제공하는 기능 
애플케이션의 모니터링, HTTP 요청 및 응답 메트리 수정
DB 모니터링, 사용자 정의 매트릭 제공 가능 

##### 여러 엔드포인트를 사용설정시 
여러 정보를 수집하기때문에 CPU와 메모리가 많이 사용된다는 점 그러기 떄문에 엔드포인트를 포함하려면 명시적으로 설정해야 함.
```
management.endpoints.web.exposure.include=health,metrics
```
----
# <span style="background:#affad1">SpringBoot vs Spring MVC vs Spring</span>
#### <font color="#f79646">Spring Framework</font>
 -  의존성이 식별하고 자동 연결 하는것
 - 의존성 주입만으로 강력한 애플케이션을 빌드 할수 없음.
  Spring Modules 과 Spring Projects는 Spring 생태계를 확장하여 다른 프레임워크와 쉽게 통합할 수 있도록 지원함.
#### <font color="#f79646">Spring  MVC</font>
- Spring Module이고 웹 애플리케이션과 REST API의 빌드 과정을 간소화
#### <font color="#f79646">Spring Boot</font>
- Spring Project이고 프로덕션 환경에 사용 가능한 애플리케이션을 빠르게 빌드하도록 지원하는 것
- 주요 기능은 Starter Projects와 Auto Configuration
- 프로젝트에 필요한 의존성을 다 세팅해주기 때문에 따로 세팅할 필요가 없다.
- 비기능 요구사항도 사용 설정함.
- 디폴트 로깅과 오류처리 제공