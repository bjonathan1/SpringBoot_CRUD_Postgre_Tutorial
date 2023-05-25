# Spring-Boot CRUD Tutorial for beginners
> This project provides basic skills to implement a simple RESTful API. 

Author : Sungwoo (Jonathan) Kim

*The reference of this tutorial is from [Amigoscode youtube channel](https://youtu.be/9SGDpanrc8U). 

## What is Spring? Why do you use it?

Spring is a Java based web framework that provides various of configurations, security, and utilities to
build an application. Spring-Boot, especially, makes easier and faster to connect and service a powerful web application. 
Also, it is one of the most popular framework currently used all over the world. 

This project will show how to build a fast simple RESTful Web Service just to give you an idea of how the framework
works. It is an hour course tutorial and here is some steps for the tutorial :  
- Run the Server
- Display a simple phrase on the server
- Construct a business architecture of Spring Framework
- Create and Connect to Database
- Implement CRUD using Database

Some fundamental concepts will be included in this tutorial, so wish you follow along through the end of this tutorial.

Now, let's get started! 

## Running the server

In order to start the Spring project, the easiest way is to initialize with [spring initializer](https://start.spring.io)
which is officially provided by spring. 

![spring initializr web page](./img/spring_initializr.png)

Select just as the screenshot above : Maven Project - Java Language - 
3.0.6 ver (any version is fine) - default Metadatas - Jar packaging -
Java version 17 - Dependencies(Spring Web, Spring Data JPA, PosegreSQL Driver)

To give a little description of some options above : 
- Maven : Maven and Gradle are both tool that makes the build process easy. The main feature of Maven is that it uses
  pom.xml to manage a project's build. It is a classic type management tool used throughout the years, however, the 
  trend is moving on to Gradle which is considered to be more flexible and faster than Maven. 
- Jar : Jar is a deploy file format that is acquired after building the project. You will basically use Jar when dealing
  with Spring-Boot projects.
- Spring Data JPA : The feature of the dependency helps to easily manipulate the database. Basic CRUD queries are provided
  in libraries. 

After generating the project, open with Spring IDE (Eclipse, VScode, Intellij). I used Intellij IDEA. 

Once you open the project, find **pom.xml**. You will see all the dependencies needed 
for this project. In order to run the server we are going to _comment_ the jpa dependency. 
```xml
  <!--		<dependency>-->
  <!--			<groupId>org.springframework.boot</groupId>-->
  <!--			<artifactId>spring-boot-starter-data-jpa</artifactId>-->
  <!--		</dependency>-->
```


Then apply it to maven by : 
> Right click on demo Project file > Maven > Reload Project

And after all Run the application. And type https://localhost:8080 on your browser. 
If you see a _Whitelabel Error Page_, it means the server is running correctly. 

## Display a simple phrase on the server

Now we are going to make a Controller that handles the request of a user. Let's create a 
**'student'** package and controller java file in the package. 
> src > main > java > com.example.demo > student > StudentController.java

Now that we have a Controller component, we will let the Controller to display 
_Hello World_ on the web page instead of _Whiltelabel Error page_.

```java
package com.example.demo.student;

@RestController
public class StudentController {
  @GetMapping("/")
  public String hello() {
    return "Hello World";
  }
}
```
Re-run the application, and refresh your web browser. You will see a nice little _Hello 
world_ displayed on your web. 

or a lot of times, we use List to interact with web. So let's try this example also.

```
@GetMapping("/")
public List<String> hello() {
  return List.of("Hello"+ "World");
}
```

## Construct a business architecture of Spring Framework

This is the project overview that we will implement for this project.
We will make database named `student`, and table named `Student` 
and implement CRUD(Create, Read, Update, Delete) by RESTful API. Here is an example
of basic idea that will be done. 

![Project Overview](./img/project_overview.png)

Before we start to learn the business architecture, we need took understand some 
fundamental concepts of Spring in order to understand fully.

### MVC Pattern
MVC is a designing pattern consists of Model, View, and Controller. By splitting into 
3 roles of composing element of an application, it is much easier to implement in such
manner of business logics.  

- Model : Role that manages the interaction between Databases. All the manipulations 
  pertinent to the Entity of Database is the role of Model. 
- View : Role of controlling the views of user interface. It displays the data that 
  is acquired from Model. 
- Controller : Role of managing multiple `Models` and `Views`. It controls general 
  processes of the Spring as a Control Tower. 

### Student Class
Now that we know MVC patterns, we have to keep in mind which logic the service belongs 
while working. First we will make a `Student Entity` which belongs to `Model` logic.

> Entity class is an Object that maps with Database Table. It should contain exactly
  same fields with the columns of the table. 

The features of `Student` includes : 
- Id
- Name
- Email
- Date of Birth
- Age

Now let's go to the project and make a Student class. 
> src > main > java > com.example.demo > student > Student.java

```java
package com.example.demo.student;

public class Student {
    private Long id;
    private String name;
    private String email;
    private LocalDate dob;
    private Integer age;

    public Student() {
    }

    public Student(Long id, String name, String email, LocalDate dob, Integer age) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.dob = dob;
        this.age = age;
    }

    public Student(String name, String email, LocalDate dob, Integer age) {
        this.name = name;
        this.email = email;
        this.dob = dob;
        this.age = age;
    }

    //Getters and Setters
    public Long getId() {
        return id;
    }

    //...
}
```
As we have our Student Entity ready, let's check this Object works fine by changing
the return value of the Controller. 
```java
@RestController
public class StudentController {
    @GetMapping("/")
    public List<Student> hello() {
        return List.of(new Student(
                1L,
                "Mariam",
                "mariam.jamal@gmail.com",
                LocalDate.of(2000, Month.JANUARY, 5),
                21
        ));
    }
}
```
Run the Application, and refresh your browser. If you follewed the steps right 
you will see an output like this on your browser. 
```json
[
  {
    "id": 1,
    "name": "Mariam",
    "email": "mariam.jamal@gmail.com",
    "dob": "2000-01-05",
    "age": 21
  }
]
```
Lastly, in order to build a Business logic we need make a service Layer. We will allocate
the role to get student logic to service layer. In order to use service instance in
`Controller`, Spring-Boot uses a method called Dependency Injection. 
> DI(Dependency Injection) : Spring providing utility of Injecting function of a 
  dependent Object. It helps in flexibility on modification of an Object.  

So here are some codes how `Service` Object is injected to the `Controller`. Note
that it uses `@Autowired` annotation to implement Dependency Injection. 

*Note that `RectController`'s URL has changed. 
```java
package com.example.demo.student;

@RestController
@RequestMapping(path = "api/v1/student")
public class StudentController {
    private final StudentService studentService;

    @Autowired
    public StudentController(StudentService studentService) {
        this.studentService = studentService;
    }
    @GetMapping
    public List<Student> getStudent() {
        return studentService.getStudents();
    }
}
```
```java
package com.example.demo.student;

@Service
public class StudentService {

    public List<Student> getStudents() {
        return List.of(new Student(
                1L,
                "Mariam",
                "mariam.jamal@gmail.com",
                LocalDate.of(2000, Month.JANUARY, 5),
                21
        ));
    }
}
```
After modifying and running the application, we can see the exact same result as before
but with **business logic** included.

## Create and Connect to Database

First we need to provide some basic data to our project. No matter what database you
use, you will need to fill the `application properties` in order to connect with the
database. 
> src > main > resources > application.properties
```
spring.datasource.url=jdbc:postgresql://localhost:5432/student
spring.datasource.username=
spring.datasource.password=
# Entity 정보를 이용해 자동으로 테이블 생성 / 삭제
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.format_sql=true

server.error.include-message=always
```
The data above shows some connection data via database, and some features related to 
databases used in this project. For example, hibernate `ddl-auto` helps to 
create/delete table of the declared Entity. 

Next, download [Postgres](https://postgresapp.com) database from web. 
Execute the shell for Postgres and we will create _student_ databse and give 
_privilage_ to the database. 

```postgresql
CREATE DATABASE student;
\du -- to find all users in postgre
GRANT ALL PRIVILEGES ON DATABASE "student" TO jonmac;
GRANT ALL PRIVILEGES ON DATABASE "student" TO postgres;
\c student -- connect to database student
```
```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
```

Then we will go to pom.xml to un-comment the jpa dependency, and reload the project again,
and run the application. You will find that JPA is running and database has been connected
by running logs of the application .

Now, let's map our Entity to the database simply by adding annotation to our
`Student` class. 

```java
package com.example.demo.student;

import jakarta.persistence.*;

@Entity
@Table
public class Student {
  @Id
  @SequenceGenerator(
          name = "student_sequence",
          sequenceName = "student_sequence",
          allocationSize = 1
  )
  @GeneratedValue(
          strategy = GenerationType.SEQUENCE,
          generator = "student_sequence"
  )
  private Long id;
  private String name;
  private String email;
  private LocalDate dob;
  private Integer age;

  public Student() {
  }

  public Student(Long id, String name, String email, LocalDate dob, Integer age) {
    this.id = id;
    this.name = name;
    this.email = email;
    this.dob = dob;
    this.age = age;
  }

  public Student(String name, String email, LocalDate dob, Integer age) {
    this.name = name;
    this.email = email;
    this.dob = dob;
    this.age = age;
  }
  
  //...
}
```
Now that we have modified `Student` Entity. By running the application, it will automatically **create the Student table**
in the database. And by stopping to application, it will automatically **delete the Student table**. Here are my 
examples of created tables in student database. 
```
student=# \d
               List of relations
 Schema |       Name       |   Type   | Owner
--------+------------------+----------+--------
 public | student          | table    | jonmac
 public | student_sequence | sequence | jonmac
(2 rows)

student=# \d student
                      Table "public.student"
 Column |          Type          | Collation | Nullable | Default
--------+------------------------+-----------+----------+---------
 id     | bigint                 |           | not null |
 dob    | date                   |           |          |
 email  | character varying(255) |           |          |
 name   | character varying(255) |           |          |
Indexes:
    "student_pkey" PRIMARY KEY, btree (id)
```
So finally, we are ready with our _Postgres database connected to our application_. It was a long way until here, 
but we have one last but not the least section to go! Let's keep it up!  

## Implement CRUD using Database

As I mentioned of above, Spring Data JPA is a feature of the dependency helps to easily 
manipulate the database. We will directly implement this by using `JPA Repository`, and find 
out how power the JPA is. 

Create an **Interface** file in directory below: 
> src > main > java > com.example.demo > student > StudentRepository.java
```java
package com.example.demo.student;

@Repository
public interface StudentRepository extends JpaRepository<Student, Long> {

}
```
The `Generic` specified in `JpaRepository` is <Type of date, ID of the Table> thus,in this 
case, `<Student, Long>`. We are going to utilize this Repository in `Service Layer`, so let's
inject dependency (DI) to Service logic, and use JPA Repository to **find all students** from the 
`Student Table`. 
```java
package com.example.demo.student;

@Service
public class StudentService {
    private final StudentRepository studentRepository;

  //Repository Dependecny  Injection
    @Autowired
    public StudentService(StudentRepository studentRepository) { 
        this.studentRepository = studentRepository;
    }

    public List<Student> getStudents() {
        // find All students in Table
        return studentRepository.findAll();
    }
}
```
As you can see above, you just need a simple `findAll()` method to find all students in the 
table. In order to test whether the service works fine, let's insert some _students_ to the 
table in advance by using a `configuration` utility from Spring. 
> src > main > java > com.example.demo > student > StudentConfig.java
```java
package com.example.demo.student;

import static java.time.Month.*;

@Configuration
public class StudentConfig {

    @Bean
    CommandLineRunner commandLineRunner(StudentRepository repository){
        return args -> {
            Student mariam = new Student(
                    "Mariam",
                    "mariam.jamal@gmail.com",
                    LocalDate.of(2000, JANUARY, 5),
                    21
            );
            Student alex = new Student(
                    "Alex",
                    "alex@gmail.com",
                    LocalDate.of(2004, JANUARY, 5),
                    21
            );

            repository.saveAll(
                    List.of(mariam, alex)
            );
        };
    }
}
```
We can find two annotations in the java code above : `Bean`, `Configuration`. What are they?
> @Bean : Essential composition of Spring. It is a POJO(Plain Old Java Object) which is literally
  designed to share one instance throughout the whole Spring Application. 

> @Configuration : Place where you need to add Bean manually. 

The `StudentConfig` file will execute the bean as soon as the application starts running. Thus, 
two students, _Mariam and Alex_, is added to the `Student Table` by calling `saveAll()` method
of `StudentRepository`. 

If we run the Application, two rows of database has been added, and these two data has successfully
displayed on the browser. 
```
student=# SELECT * FROM student;
 id | age |    dob     |         email          |  name
----+-----+------------+------------------------+--------
  1 |  21 | 2000-01-05 | mariam.jamal@gmail.com | Mariam
  2 |  21 | 2004-01-05 | alex@gmail.com         | Alex
(2 rows)
```
```json
[
  {
    "id": 1,
    "name": "Mariam",
    "email": "mariam.jamal@gmail.com",
    "dob": "2000-01-05",
    "age": 21
  },
  {
    "id": 2,
    "name": "Alex",
    "email": "alex@gmail.com",
    "dob": "2004-01-05",
    "age": 21
  }
]
```
Next, we actually do not need the **age** column since we can calculate the age by Data of Birth.
We will use `@Transient` annotation to implement this. 
> @Transient : Specify the field does not belong to the Database.

By applying `@Transient` to _age field_ in Student Entity, we will be able to manufacture the
age field by using the data from database. 
```java
package com.example.demo.student;

import jakarta.persistence.*;

@Entity
@Table
public class Student {
    //...
  
    @Transient
    private Integer age;

    public Student() {
    }

    public Student(Long id, String name, String email, LocalDate dob) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.dob = dob;
    }

    public Student(String name, String email, LocalDate dob) {
        this.name = name;
        this.email = email;
        this.dob = dob;
    }

    //...

    public Integer getAge() {
        // Age calculation
        return Period.between(this.dob, LocalDate.now()).getYears();
    }

    //...
}
```
Thus now, we do not need to insert age value to add student. Now we are done with **R**(Read) from 
**CRUD**.
---
Now, let's make a logic for **C**(Create) the data. json format of _Name, Email, and Date of 
Birth_ will be given as input of Post Request, and it will save to Database **If the email not
exist in the Database**. 
```java
package com.example.demo.student;

@RestController
@RequestMapping(path = "api/v1/student")
public class StudentController {
  private final StudentService studentService;

  //...

  @PostMapping
  public void registerStudent(@RequestBody Student student) {
    studentService.addNewStudent(student);
  }
}
```
Add PostMapping logic in the Controller. 
```java
package com.example.demo.student;

public interface StudentRepository extends JpaRepository<Student, Long> {
    @Query("SELECT s FROM Student s WHERE s.email=?1")
    Optional<Student> findStudentByEmail(String email);
}
```
Add a new **find Student By Email logic** into `Student Repository`. 
```java
package com.example.demo.student;

@Service
public class StudentService {
    private final StudentRepository studentRepository;
    
    //...

    public void addNewStudent(Student student) {
        Optional<Student> studentOptional = studentRepository.findStudentByEmail(student.getEmail());
        if (studentOptional.isPresent()) {
            throw new IllegalStateException("email exist");
        }
        studentRepository.save(student);
    }
}
```
And add the business logic into `Student Service`. By using the utility of `generated-request.http`
from Intellij, we can test the data is being added to the database. 
```
###
POST http://localhost:8080/api/v1/student
Content-Type: application/json

{
  "name" : "Bilal",
  "email" : "bilal.ahmed@gmail.com",
  "dob" : "1995-12-17"
}
```
Result : 
```json
[
  {
    "id": 1,
    "name": "Mariam",
    "email": "mariam.jamal@gmail.com",
    "dob": "2000-01-05",
    "age": 23
  },
  {
    "id": 2,
    "name": "Alex",
    "email": "alex@gmail.com",
    "dob": "2004-01-05",
    "age": 19
  },
  {
    "id": 3,
    "name": "Bilal",
    "email": "bilal.ahmed@gmail.com",
    "dob": "1995-12-17",
    "age": 27
  }
]
```
We successfully implemented **C**(Create) of CRUD. 

---
Now we will make a logic to **D**(Delete) the data. This logic will be quite straight forward.
```java
package com.example.demo.student;

@RestController
@RequestMapping(path = "api/v1/student")
public class StudentController {
    private final StudentService studentService;

    //...

    @DeleteMapping(path = "{studentId}")
    public void deleteStudent(@PathVariable("studentId") Long studentId) {
        studentService.deleteStudent(studentId);
    }
}
```
Add `DeleteMapping` to the `Controller` with the student variable to delete. 
```java
package com.example.demo.student;

@Service
public class StudentService {
    private final StudentRepository studentRepository;
    
    //...
  
    public void deleteStudent(Long studentId) {
        boolean exist = studentRepository.existsById(studentId);
        if (!exist) {
            throw new IllegalStateException("Student with ID " + studentId + " does not exist");
        }
        studentRepository.deleteById(studentId);
    }
}
```
Add business logic to `Service`. Check if ID exists in database, and delete it if exists. 
And here are the results. 
```
###
POST http://localhost:8080/api/v1/student
Content-Type: application/json

{
  "name" : "Bilal",
  "email" : "bilal.ahmed@gmail.com",
  "dob" : "1995-12-17"
}

###
DELETE http://localhost:8080/api/v1/student/1
```
Result : 
```json
[
  {
    "id": 2,
    "name": "Alex",
    "email": "alex@gmail.com",
    "dob": "2004-01-05",
    "age": 19
  },
  {
    "id": 3,
    "name": "Bilal",
    "email": "bilal.ahmed@gmail.com",
    "dob": "1995-12-17",
    "age": 27
  }
]
```
We can know that **ID : 1** has successfully deleted! 

---
Lastly, we will implement **U**(Update) of CRUD. We will `PutMapping` to update name or email
(or both) by _student ID_. 

```java
package com.example.demo.student;

@RestController
@RequestMapping(path = "api/v1/student")
public class StudentController {
    private final StudentService studentService;

    //...
  
    @PutMapping(path = "{studentId}")
    public void updateStudent(@PathVariable("studentId") Long studentId,
                              @RequestParam(required = false) String name,
                              @RequestParam(required = false) String email){
        studentService.updateStudent(studentId, name, email);
    }
}
```
Three possible cases were verified during Service logic. 
1. Targeted ID must be included in Database. 
2. If name exists or has changed.
3. If email exists or has changed. 

```java
package com.example.demo.student;

@Service
public class StudentService {
    private final StudentRepository studentRepository;
    
    //...

    @Transactional
    public void updateStudent(Long studentId, String name, String email) {
        Student student = studentRepository.findById(studentId)
                .orElseThrow(() -> new IllegalStateException("Student with ID " + studentId + " does not exist"));

        if (name != null && name.length() > 0 && !Objects.equals(name, student.getName())) {
            student.setName(name);
        }

        if (email != null && email.length() > 0 && !Objects.equals(email, student.getEmail())) {
            Optional<Student> studentOptional = studentRepository.findStudentByEmail(email);
            if (studentOptional.isPresent()) {
                throw new IllegalStateException("email exist");
            }
            student.setEmail(email);
        }
        
        studentRepository.save(student);
    }
}
```
> @Transactional : It guarantees 'All or Nothing' policy. If a little error is found within 
  the Transaction, it will roll back to the point as nothing had happened. On the other hand, 
  it will save the result if and only if it has fully satisfied the safety regulation of saving
  the  data.

And here are the results.

HTTP request : 
```
###
PUT http://localhost:8080/api/v1/student/1?name=Maria&email=maria@gmail.com
Content-Type: application/json
```
Result : 
```json
[
  {
    "id": 2,
    "name": "Alex",
    "email": "alex@gmail.com",
    "dob": "2004-01-05",
    "age": 19
  },
  {
    "id": 1,
    "name": "Maria",
    "email": "maria@gmail.com",
    "dob": "2000-01-05",
    "age": 23
  }
]
```
All in all, we finished our journey of implementing CRUD. It took so long to explore all the
concepts pertinent to Spring-Boot, however I bet it will worth it. 


Keep in mind the tutorial source of this practice was not made by me but was provided 
by [Amigoscode youtube channel](https://youtu.be/9SGDpanrc8U). I added some basic Spring concepts
to make the guide a bit more clear. 
