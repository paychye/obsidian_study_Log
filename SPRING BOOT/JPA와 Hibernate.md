#### H2를 사용하는 방법

`application.properties`를 들어가서 
```
spring.application.name=learn-jpa-and-hibernate  
spring.h2.console.enabled=true
```

localhost:8080/h2-console로 들어가면 해당 페이지가 보인다.
![[Pasted image 20250324172106.png]]
 ![[Pasted image 20250324172141.png]]
 H2 Database에 필요한 URL은 아직 구성하지 않았기때문에 동적 URL이 App을 실행하면서 뜬다. 그렇기 떄문에 해당 URL을 복사해서 jdbc:h2~/test 자리에 복사해서 붙여넣기 해주면 로그인이 된다.

이런 동적 URL은 항상 App을 시작할때마다 다르게 바뀌니깐 작업을 어럽게 만든다.
그렇기 때문에 정적 URL로 바꾸는 작업을 하겠음.

``` title="application.properties"
spring.application.name=learn-jpa-and-hibernate  
spring.h2.console.enabled=true  
Spring.datasource.url=jdbc:h2:mem:testdb
```

`Spring.datasource.url=jdbc:h2:mem:testdb`로 설정해서 정적 URL로 만들어준다.
위에 바꿔준 정적URL를 복사해서 아래 사진위치에 넣어주면 된다.
![[Pasted image 20250324172106.png]]

##### `src/resources `-> `schema.sql `파일 생성
``` sql title="schema.sql"
create table course  
(  
    id  bigint not null,  
    name  varchar(255) not null,  
    author varchar(255) not null,  
    primary key  (id)  
)
```
![[Pasted image 20250324173246.png]]
Spring Data JPA Starter를 활용할 때마다 자동으로 schema.sql파일을 가져와서 H2에 테이블을 생성해줍니다.

----- 
###  JDBC
- SQL를 엄청나게 작성하게 됨

```java
Connection conn = DriverManager.getConnection(url, user, password);
String sql = "SELECT * FROM users WHERE id = ?";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setInt(1, 1);
ResultSet rs = pstmt.executeQuery();
while (rs.next()) {
    System.out.println(rs.getString("name"));
}
rs.close();
pstmt.close();
conn.close();
```
### Spring JDBC
- 자바코드를 휠씬 더 적게 사용한다는것.
```java
@Autowired
private JdbcTemplate jdbcTemplate;

String sql = "SELECT name FROM users WHERE id = ?";
String name = jdbcTemplate.queryForObject(sql, String.class, 1);
System.out.println(name);
```


### Spring JDBC를 사용하여 하드코드로 작성된 데이터 삽입하기

`JdbcTemplate의` 역할-> Spring에서 JDBC를 쉽게 사용할수 있도록 도와주는 클래스
기존 JDBC코드에서 반복되는 보일러플레이트 코드(연결쿼리 실행, 예외처리 ,리소스 정리 등)를 자동으로 처리함.

```java title="CourseJdbcRepository"
package com.in28minutes.springboot.learnjpaandhibernate.course.jdbc;  
  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.jdbc.core.JdbcTemplate;  
import org.springframework.stereotype.Repository;  
  
@Repository  
public class CourseJdbcRepository {  
  
    @Autowired  
   private JdbcTemplate springJdbcTemplate;  
  
	private static String INSERT_QUERY=  
        """  
         insert into course(id,name,author)  
         values(1,'Learn AWS', 'in28minutes');  
                """;  
  
   public void insert() {  
       springJdbcTemplate.update(INSERT_QUERY);  
   }  
}
```


해당 쿼리를 실행하기 위한 필요한 클래스를 만들기 
```java title="CourseJdbcCommandLineRunner"
package com.in28minutes.springboot.learnjpaandhibernate.course.jdbc;  
  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.boot.CommandLineRunner;  
import org.springframework.stereotype.Component;  
  
@Component  
public class CourseJdbcCommandLineRunner implements CommandLineRunner {  
  
    @Autowired  
    private CourseJdbcRepository repository;  
  
    @Override  
    public void run(String... args) throws Exception {  
        repository.insert();  
    }  
}
```
쿼리를 실행하기위한 `CommandLineRunner` 역할
<span style="background:rgba(255, 183, 139, 0.55)">**애플리케이션 초기 데이터 작업</span>:** DB에 기본 데이터를 넣는 용도로 사용
<span style="background:rgba(255, 183, 139, 0.55)">**초기설정 로직 실행:** </span>설정값을 로드하거나 캐시를 미리 채우는 작업
<span style="background:rgba(255, 183, 139, 0.55)">**디버깅용 출력:** </span>개발중 특정 데이터를 확인하는 용도로 사용
**<span style="background:rgba(255, 183, 139, 0.55)">배치 작업 실행:</span>** 애플리케이션 실행 직후 배치프로세스 실행
![[Pasted image 20250324210221.png]]

### Spring JDBC를 사용하여 데이터 삽입 및 삭제하기

```java title="Course"
package com.in28minutes.springboot.learnjpaandhibernate.course;  
  
public class Course {  
    private long id;  
    private String name;  
    private String author;  
    //constructor  
    public Course() {  
    }    public Course(long id, String name, String author) {  
        this.id = id;  
        this.name = name;  
        this.author = author;  
    }  
    //getters  
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

```java title="CourseJdbcRepository"
package com.in28minutes.springboot.learnjpaandhibernate.course.jdbc;  
  
import com.in28minutes.springboot.learnjpaandhibernate.course.Course;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.jdbc.core.JdbcTemplate;  
import org.springframework.stereotype.Repository;  
  
@Repository  
public class CourseJdbcRepository {  
  
    @Autowired  
   private JdbcTemplate springJdbcTemplate;  
  
private static String INSERT_QUERY=  
        """  
         insert into course(id,name,author)  
         values(?,?,?);  
                """;  
  
   public void insert(Course course) {  
       springJdbcTemplate.update(INSERT_QUERY,  
               course.getId(),course.getName(),course.getAuthor());  
   }  
}
```
`insert() ` 메서드에서 전달한 인자 값들이  쿼리 물음표(placeholder)해당 위치에 순서대로 들어감 
```java title="CourseJdbcCommandLineRunner"
package com.in28minutes.springboot.learnjpaandhibernate.course.jdbc;  
  
import com.in28minutes.springboot.learnjpaandhibernate.course.Course;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.boot.CommandLineRunner;  
import org.springframework.stereotype.Component;  
  
@Component  
public class CourseJdbcCommandLineRunner implements CommandLineRunner {  
  
    @Autowired  
    private CourseJdbcRepository repository;  
  
    @Override  
    public void run(String... args) throws Exception {  
        repository.insert(new Course(1,"Learn AWS NOW!","in28minutes"));  
    }  
}
```

![[Pasted image 20250324211619.png]]


## 이제 SELECT문을 사용해보자
```java title="CourseJdbcRepository"
package com.in28minutes.springboot.learnjpaandhibernate.course.jdbc;  
  
import com.in28minutes.springboot.learnjpaandhibernate.course.Course;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.jdbc.core.BeanPropertyRowMapper;  
import org.springframework.jdbc.core.JdbcTemplate;  
import org.springframework.stereotype.Repository;  
  
@Repository  
public class CourseJdbcRepository {  
  
    @Autowired  
   private JdbcTemplate springJdbcTemplate;  
  
private static String SELECT_QUERY=  
        """  
        select * from course where id = ?;                """;  
    public Course findById(long id) {  
        //ResultSet -> Bean => Row Mapper  
        //id        
        return springJdbcTemplate.queryForObject(SELECT_QUERY,  
                new BeanPropertyRowMapper<>(Course.class), id);  
    }  
}
```

``` title="력창"
Course{id=2, name='Learn Azure NOW!', author='in28minutes'}
Course{id=3, name='Learn DevOps NOW!', author='in28minutes'}
```

SELECT문는 찾는거니깐 리턴값이 있어야되고 그 리턴값은 Course클래스형태로 받을꺼니깐 Course로 설정 
`Course` 클래스에 Setter 메소드를 추가해줘야지 DB결과를 자바객체에 넣을수 있다.(안넣어주면 다 null값으로 표시가 된다.)

`ResultSet`을 객체로 자동 변환해주는 클래스, 즉 테이블 조회 결과를 Java(Bean)로 쉽게 매핑할수 있도록 도와줌.
쿼리결과-> Java객체로(POJO)변환
컬럼명과 객체 필드명으로 자동으로 매핑됨 (snake)_case(DB)<-> camelcase(Java)자동변환 가능 )


-----
#  JPA & Entity Manager 시작하기

`@Entity`를 추가해줘서 Java Bean과 테이블 사이에 매핑을 생성하고 그 매핑을 이용해서 값을 삽입하고 값을 검색하고 테이블에서 작업을 수행함
```java title="Course"
package com.in28minutes.springboot.learnjpaandhibernate.course;  
  
import jakarta.persistence.Column;  
import jakarta.persistence.Entity;  
import jakarta.persistence.Id;  
  
@Entity  
public class Course {  
    @Id  
    private long id;  
    private String name;  
    private String author;  
    //constructor  
    public Course() {  
    }    public Course(long id, String name, String author) {  
        this.id = id;  
        this.name = name;  
        this.author = author;  
    }  
    //getters  
    public long getId() {  
        return id;  
    }  
    public String getName() {  
        return name;  
    }  
    public String getAuthor() {  
        return author;  
    }  
    public void setId(long id) {  
        this.id = id;  
    }  
    public void setName(String name) {  
        this.name = name;  
    }  
    public void setAuthor(String author) {  
        this.author = author;  
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

`@Id`로 기본키로 설정을 하고 `@Column`으로  DB 컬럼명 설정하지만 테이블컬럼명이랑 일치하기때문에 따로 설정할필요가 없다.
`@Entity`도 name을 이용해서 원하는 테이블명으로 할수 있다.

``` java title="CourseJpaRepository"
import jakarta.persistence.EntityManager;  
import jakarta.persistence.PersistenceContext;  
import jakarta.transaction.Transactional;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.stereotype.Repository;  
  
@Repository  
@Transactional  
public class CourseJpaRepository  {  
    @PersistenceContext  
    private EntityManager entityManager;  
  
    public void insert(Course course) {  
        entityManager.merge(course);  
    }  
    public Course findById(long id) {  
        return entityManager.find(Course.class, id);  
    }  
    public void deleteById(long id) {  
         Course course =entityManager.find(Course.class, id);  
        entityManager.remove(course);  
    }  
}
```

`@Transactional`-> DB 트랜잭션을 관리하는 역할, 즉 여러개의 SQL 실행을 하나의 작업 단위로 묶어서 실행하고, 오류가 발생하면 자동으로 롤백되게끔 해주는 역할.
데이터의 일관성을 유지하기 위해 트랜잭션이 필요함.

`@Repository`-> DB와 연결을 담당하는 계층, DAO(Data Access Object)역할을 수행, SQL을 실행하여 데이터 CRUD(생성,조회,수정,삭제)를 처리
비즈니스 로직과 (DB)사이의 중간 역할

`EntityManager`- > 엔티티 관리, DB와 상호작용하는 핵심 객체, JPA에서 SQL을 실행하는 역할


# Spring Data JPA


``` java title="CourseSpringDataJpaRepository"
package com.in28minutes.springboot.learnjpaandhibernate.course.springdatajpa;  
  
import com.in28minutes.springboot.learnjpaandhibernate.course.Course;  
import org.springframework.data.jpa.repository.JpaRepository;  
  
public interface CourseSpringDataJpaRepository extends JpaRepository<Course,Long> {  
}
```

```java title="CourseCommandLineRunner"
package com.in28minutes.springboot.learnjpaandhibernate.course;  
  
import com.in28minutes.springboot.learnjpaandhibernate.course.springdatajpa.CourseSpringDataJpaRepository;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.boot.CommandLineRunner;  
import org.springframework.stereotype.Component;  
  
@Component  
public class CourseCommandLineRunner implements CommandLineRunner {  
  
    @Autowired  
    private CourseSpringDataJpaRepository repository;  
  
    @Override  
    public void run(String... args) throws Exception {  
        repository.save(new Course(1,"Learn AWS JPA!","in28minutes"));  
        repository.save(new Course(2,"Learn Azure JPA!","in28minutes"));  
        repository.save(new Course(3,"Learn DevOps JPA!","in28minutes"));  
  
        repository.deleteById(1l);  
  
        System.out.println(repository.findById(2l));  
        System.out.println(repository.findById(3l));  
    }  
}
```
EntityManager도 백그라운드에서 이루어지니까 JpaRepository를 extends로 상속받아 인터페이스 를 생성하기만 하면 되고요. 좋은 메서드가 아주 많이 제공되고 있음.

#### Spring Data Jpa 기능 살펴보기
커스텀 메서드를 추가할수있다는 점.
```java
package com.in28minutes.springboot.learnjpaandhibernate.course.springdatajpa;  
  
import com.in28minutes.springboot.learnjpaandhibernate.course.Course;  
import org.springframework.data.jpa.repository.JpaRepository;  
  
import java.util.List;  
  
public interface CourseSpringDataJpaRepository extends JpaRepository<Course,Long> {  
    List<Course> findByAuthor(String author);  
}
```

```java
package com.in28minutes.springboot.learnjpaandhibernate.course;  
  
import com.in28minutes.springboot.learnjpaandhibernate.course.springdatajpa.CourseSpringDataJpaRepository;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.boot.CommandLineRunner;  
import org.springframework.stereotype.Component;  
  
@Component  
public class CourseCommandLineRunner implements CommandLineRunner {  
  
    @Autowired  
    private CourseSpringDataJpaRepository repository;  
  
    @Override  
    public void run(String... args) throws Exception {  
        repository.save(new Course(1,"Learn AWS JPA!","in28minutes"));  
        repository.save(new Course(2,"Learn Azure JPA!","in28minutes"));  
        repository.save(new Course(3,"Learn DevOps JPA!","in28minutes"));  
  
        repository.deleteById(1l);  
  
        System.out.println(repository.findById(2l));  
        System.out.println(repository.findById(3l));  
        System.out.println(repository.findAll());  
        System.out.println(repository.count());  
        System.out.println(repository.findByAuthor("in28minutes"));  
        System.out.println(repository.findByAuthor(""));  
    }  
}
```
```
bernate: select c1_0.id,c1_0.author,c1_0.name from course c1_0 where c1_0.id=?
Optional[Course{id=2, name='Learn Azure JPA!', author='in28minutes'}]
Hibernate: select c1_0.id,c1_0.author,c1_0.name from course c1_0 where c1_0.id=?
Optional[Course{id=3, name='Learn DevOps JPA!', author='in28minutes'}]
Hibernate: select c1_0.id,c1_0.author,c1_0.name from course c1_0
[Course{id=2, name='Learn Azure JPA!', author='in28minutes'}, Course{id=3, name='Learn DevOps JPA!', author='in28minutes'}]
Hibernate: select count(*) from course c1_0
2
Hibernate: select c1_0.id,c1_0.author,c1_0.name from course c1_0 where c1_0.author=?
[Course{id=2, name='Learn Azure JPA!', author='in28minutes'}, Course{id=3, name='Learn DevOps JPA!', author='in28minutes'}]
Hibernate: select c1_0.id,c1_0.author,c1_0.name from course c1_0 where c1_0.author=?
[]
```

# Hibernate vs JPA
### JPA ->API
- 기술 명세서 , 인터페이스 , 엔티티가 무엇인지 정의하는 방식
- 객체를 테이블로 매핑하는 방식을 정의하고 Hibernate는 그걸 구현함.
### Hibernate
 - JPA에서 매우 인기 있는 구현체

Hibernate를 직접적으로 쓸수있는 `@Entity`를 ->  org.hibernate.annotations 패키지로 설정하면 사용가능  하지만 직접 사용하는게 아닌 JPA를 사용하고 있음.
Hibernate JAR를 클래스 경로에 추가해서 Hibernate를 JPA구현체로 사용하고 있음 
여러가지 JPA 구현체를 사용할수 있어서 Hibernate를 한정해서 쓰지 않는다.