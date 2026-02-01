# Leaarning Spring Boot

## Table of Contents
- [Spring Basics](#basics)
- [Spring Boot](#springboot)
- [REST APIs with Spring Boot](#springbootrestapi)
- [Database + Spring Data JPA](#jpa)
- [Exception Handling & Validation](#exceptions)
- [Spring Security](#security)
- [Configuration & Profiles](#configuration)
- [Testing Spring Boot Apps](#testing)
- [Advanced & Real-World Topics](#advanced)
- [Deploying sping boot app into kubernetes](#intokubernetes)
- [Real Projects](#projects)

## 🌱 1. Spring Basics  <a id="basics"></a>

### Core Spring Concepts

__Inversion of Control (IoC)__
- You do NOT create objects yourself: Control is inverted: You don’t control object creation → Spring does.
- The Spring container creates, manages, and injects them.
- Problems:
  - Tight coupling
  - Hard to test
  - Hard to change implementations

__Dependency Injection (DI)__
- Dependencies are injected, not created.
  - Constructor Injection (BEST PRACTICE)
    - Immutable
    - Easy testing
```
@Service
class OrderService {
    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```
  - Setter Injection: Used when dependency is optional.
  - Field Injection (Not recommended): hides dependencies.

__Spring Container__

Is responsible for:
- Creating objects (beans)
- Injecting dependencies
- Managing lifecycle
- Managing configurations

__Spring Beans__: An object created, configured, and managed by Spring

__Bean Scopes__

```
| Scope       | Meaning                              |
| ----------- | ------------------------------------ |
| singleton   | One instance per container (default) |
| prototype   | New instance each time               |
| request     | One per HTTP request                 |
| session     | One per HTTP session                 |
| application | One per app lifecycle                |
```

__Bean Lifecycle__
- Bean instantiation
- Dependency injection
- Initialization
- Bean ready to use
- Destruction (on shutdown)
- Lifecycle Hooks:
```
@PostConstruct
public void init() {
    System.out.println("Bean initialized");
}

@PreDestroy
public void destroy() {
    System.out.println("Bean destroyed");
}
```

__Spring Annotations__
- __@Component__: Generic bean
```
import org.springframework.stereotype.Component;
@Component
public class EmailSenderService {

    public void sendWelcomeEmail(String username, String email) {
        // pretend we're sending an email...
        System.out.println("Sending welcome email to " + username + " <" + email + ">");
        // real implementation would use JavaMailSender, SendGrid, etc.
    }
}
```
- __@Service__: Business logic layer
  - How @Service is different from @Component: 
    - Both work perfectly — but @Service communicates intent much better.
    - Use @Service → when the class:
        - Contains business rules / use-case logic
        - Orchestrates calls to repositories + other services
        - Usually has @Transactional
        - Represents a domain service or application service
    - Use @Component → when the class:
        - Is a utility (DateUtils, StringUtils, PdfGenerator…)
        - Is a mapper / converter / factory
        - Is an event listener without business logic (@EventListener)
- __@Repository:__ DAO layer + exception translation: Converts DB exceptions into Spring exceptions.
  - spring will generate "SELECT u FROM User u WHERE LOWER(u.email) = LOWER(:email)"
  - You must write @Query when:
    - The query logic cannot be expressed (or becomes ugly/unreadable) via method naming conventions
    - Complex joins
    - Subqueries
```
package com.example.users.repository;

import com.example.users.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, UUID> {   // UUID is very common in 2025+

    Optional<User> findByEmailIgnoreCase(String email);

    boolean existsByEmail(String email);

    // Soft-delete support (very popular pattern)
    @Modifying
    @Query("UPDATE User u SET u.deleted = true WHERE u.id = :id")
    void softDeleteById(@Param("id") UUID id);
}
```
- __@Controller / @RestController__: Web layer, handles HTTP requests
```
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@RestController
public class UserApiController {

    private final UserService userService;

    public UserApiController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/api/users/{id}")
    public UserDto getUser(@PathVariable Long id) {
        return userService.findUserById(id);   // ← returned object → auto → JSON
    }
}
```
- __@Autowired__: Dependency Resolution:
  - autowiring = automatically providing dependencies: find a bean of this type (or matching this qualifier/name) in the container and inject it here automatically
  - resolves dependencies:
    - By type
    - By name
    - Using @Qualifier
    - Using @Primary
  - Use @Autowired mainly when:
    - You have multiple constructors and want to mark one
    - You want optional dependencies (required = false)
- __@Configuration__: Marks configuration class.
  - process its @Bean-annotated methods and register the returned objects as Spring-managed beans.
  - It is the modern, Java-based replacement for the old XML <beans> configuration files.
```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

    @Bean
    public EmailSender emailSender() {
        return new SmtpEmailSender("smtp.example.com", 587);
    }
}
```
- __@Bean__: Creates beans manually.
  - Use when:
    - Third-party classes
    - Custom initialization
    - is a factory method — it says "here's exactly how to create this bean".

__Profiles__
- mechanism to segregate parts of your application configuration and make them active only in specific environments (dev, test, staging, production, etc.).
- Spring Boot automatically loads these files (in this order — last wins):
  - application.properties / application.yml
  - application-dev.properties / application-dev.yml (when dev is active)
  - application-prod.properties / application-prod.yml (when prod is active)
```
src/main/resources/
├── application.yml               # common settings
├── application-dev.yml           # developer workstation
├── application-test.yml          # integration / CI tests
├── application-staging.yml
└── application-prod.yml          # real production
```
__Spring project structure__
```
hello-spring-cli/
├── pom.xml
└── src/
    └── main/
        └── java/
            └── com/
                └── example/
                    └── hellocli/
                        └── HelloCliApplication.java
```

__application.properties__
- is Spring Boot's primary external configuration file.
- It is a key=value (or YAML) file placed in src/main/resources/ that lets you:
```
# Server / Tomcat
server.port=8085
server.servlet.context-path=/api

# Database (auto-configured DataSource)
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=appuser
spring.datasource.password=secret123
spring.jpa.hibernate.ddl-auto=update

# Logging
logging.level.org.springframework=DEBUG
logging.level.com.mycompany=INFO

# Custom properties (your own app)
app.api.key=xyz789
app.feature.new-payment-enabled=true
app.max-upload-size=10MB

# Actuator / management
management.endpoints.web.exposure.include=health,info,metrics
```

How spring does __Component Scanning__ ?
- automatic discovery mechanism that finds classes annotated with certain stereotypes (like @Component, @Service, @Repository, @Controller, @Configuration, etc.) and registers them as Spring beans — without you having to list them manually with @Bean methods




## ⚡ 2. Spring Boot <a id="springboot"></a>

Spring Boot is a framework built on top of the Spring Framework that simplifies the process of developing, deploying, and running Spring applications. It is an opinionated framework designed to reduce the complexity of configuration and setup by providing sensible defaults. The goal is to make Spring applications easier to develop with minimal effort and configuration.

Key Features of Spring Boot:
- Auto Configuration: Spring Boot can automatically configure your Spring application based on the libraries available in the classpath. This eliminates the need to manually configure many beans and components. For example, if you include Spring Data JPA, Spring Boot will configure the datasource, entity manager, and repositories automatically.
- Embedded Servers: Tomcat, Jetty, Undertow
- Starter Dependencies: set of "starter" dependencies that provide commonly used features. For example, spring-boot-starter-web includes everything you need to create a web app
- Production Ready Features
  - Health checks (check whether the app is running properly)
  - Metrics (monitor performance metrics)
  - Application info (e.g., app version, build info)
  - Externalized configuration (in properties or YAML files)
- Minimal Configuration: comes with sensible defaults for most applications, meaning you can start an application without needing to manually configure everything. For instance, database configuration can be set using a single line in application.properties.
```
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=root
```
- Microservices Ready: easier to create microservices by providing a framework that is lightweight, stand-alone, and production-ready. It integrates well with Spring Cloud, which provides tools for building microservices, including service discovery, configuration management, and resilience tools.


Some of the main reasons it was developed include:
- __Configuration Complexity__: Traditional Spring applications often required many XML files or Java configuration classes for things like database connections, transaction management, or application properties. Spring Boot eliminates the need for most of this configuration by providing sensible defaults.
- __Manual Setup & Deployment Challenges__: Normally, you would need to configure and deploy a web server (like Tomcat or Jetty) for a Spring application. With Spring Boot, you can simply run your application as a standalone Java application, with the embedded web server included. This simplifies development and testing.
- __Development Speed__: Spring Boot provides automatic configuration and pre-configured templates for various use cases (e.g., data access, messaging, security), allowing developers to focus on business logic instead of boilerplate code or configuration.
- __Microservices Architecture__: Spring Boot is ideal for building microservices — lightweight, independent, and self-contained services that communicate over HTTP/REST. It has built-in support for building REST APIs and integrates well with other microservice frameworks, such as Spring Cloud.
- __Production Readiness__: Spring Boot includes production-ready features such as health checks, metrics, logging, and monitoring out of the box, which makes it easy to deploy and maintain applications in production environments.

__Hello World__ in spring boot:
```
@SpringBootApplication
public class HelloWorldApplication {
    public static void main(String[] args) {
        SpringApplication.run(HelloWorldApplication.class, args);
    }

    @RestController
    public class HelloController {
        @GetMapping("/hello")
        public String hello() {
            return "Hello, Spring Boot!";
        }
    }
}
```

With Spring Boot, you can run this application directly:
```
java -jar hello-world-application.jar
```

### What is @SpringBootApplication

@SpringBootApplication is the main annotation in Spring Boot.
It marks the entry point of a Spring Boot application and enables key Spring Boot features in one line.

@SpringBootApplication = @Configuration + @EnableAutoConfiguration + @ComponentScan
- @Configuration: Marks the class as a configuration class. Allows you to define beans using @Bean
- @EnableAutoConfiguration: automatically configures beans based on: Classpath dependencies, Existing beans, Application properties
- @ComponentScan: Scans packages for Spring components: @Component, @Service, @Repository, @Controller, @RestController

### Starter dependencies

Spring Boot starters are predefined dependency bundles that:
- Group commonly used libraries
- Provide compatible versions
- Enable auto-configuration

#### Before Spring Boot (Painful)

```
<!-- You had to add everything manually -->
spring-webmvc
jackson-databind
tomcat
spring-context
spring-aop
commons-logging
```

Most Important Spring Boot Starters: spring-boot-starter-web, spring-boot-starter-data-jpa:

| Starter    | Purpose           |
| ---------- | ----------------- |
| web        | REST & web apps   |
| data-jpa   | ORM & DB          |
| security   | Authentication    |
| test       | Testing           |
| actuator   | Monitoring        |
| validation | Input validation  |
| jdbc       | JDBC              |
| aop        | Logging, security |
| cache      | Caching           |
| mail       | Email             |

#### Compare web app with and without spring boot

- ❌ More configuration
- ❌ Manual setup
- ❌ External server
- ❌ Harder to maintain

Project Structure Without Spring Boot:
```
my-spring-app/
├── src/main/java
│   └── com/example
│       ├── config
│       │   ├── WebConfig.java
│       │   └── AppInitializer.java
│       ├── controller
│       │   └── HelloController.java
├── src/main/resources
│   └── application.properties
├── webapp
│   └── WEB-INF
│       └── web.xml
└── pom.xml
```

Project Structure With Spring Boot:
```
my-spring-boot-app/
├── src/main/java
│   └── com/example
│       ├── HelloController.java
│       └── MyAppApplication.java
├── src/main/resources
│   └── application.properties
└── pom.xml
```

pom.xml Without Spring Boot:
```
<web-app>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

pom.xml With Spring Boot:
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Deployment Without Spring Boot:
- ❌ Build WAR
- ❌ Install Tomcat manually
- ❌ Deploy WAR to server

Deployment with Spring Boot:
- mvn spring-boot:run
- Embedded Tomcat
- ✅ No WAR
- ✅ No web.xml

### First App (Hello World)

```
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello Spring Boot";
    }
}
```


## 🌐 3. REST APIs with Spring Boot <a id="springbootrestapi"></a>

### @RestController 

It is a specialized Spring annotation used to create RESTful web services.

This class will handle HTTP requests and return data (JSON/XML) directly — not views (HTML).

@RestController = @Controller + @ResponseBody 
- very method automatically returns the response body
- No need to add @ResponseBody on each method

How it works:
- Client sends HTTP request
- Spring DispatcherServlet routes request
- Method executes
- Return value is:
  - Converted to JSON (via Jackson)
  - Written to HTTP response body

Example:
```
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello Spring Boot";
    }
}
```

@Controller vs @RestController:
| Feature               | @Controller | @RestController |
| --------------------- | ----------- | --------------- |
| Purpose               | MVC views   | REST APIs       |
| Returns               | View name   | JSON / XML      |
| @ResponseBody needed? | Yes         | ❌ No            |
| ViewResolver          | Used        | ❌ Not used      |

CRUD REST API example:
```
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping
    public List<User> getAllUsers() {
        return List.of(
            new User(1, "Alice"),
            new User(2, "Bob")
        );
    }

    @PostMapping
    public User createUser(@RequestBody User user) {
        return user;
    }

    @GetMapping("/{id}")
    public User getUserById(@PathVariable int id) {
        return new User(id, "Alice");
    }
}

```

### @GetMapping, @PostMapping, @PutMapping, @DeleteMapping

| Annotation       | HTTP Method | Purpose     |
| ---------------- | ----------- | ----------- |
| `@GetMapping`    | GET         | Read data   |
| `@PostMapping`   | POST        | Create data |
| `@PutMapping`    | PUT         | Update data |
| `@DeleteMapping` | DELETE      | Delete data |


- Client sends HTTP request
- DispatcherServlet receives it
- Spring finds matching URL + HTTP method
- Controller method is invoked
- Return value is written to response (JSON)

### @RequestParam

```
@GetMapping("/users")
public List<User> search(@RequestParam String name) {
    return userService.findByName(name);
}
```

### @PathVariable

```
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id);
}
```
- automatically returns http status code = 200
- Spring automatically adds default headers, such as: "Content-Type: application/json"
- If an Exception Is Thrown: 500 INTERNAL SERVER ERROR

Correct Way (Best Practice): Use ResponseEntity

### @RequestBody

```
@PutMapping("/users/{id}")
public User updateUser(
        @PathVariable Long id,
        @RequestBody User user) {
    return userService.update(id, user);
}
```


### ResponseEntity

It gives you full control over the HTTP response — not just the body.

ResponseEntity<T> represents the entire HTTP response, including:
- ✅ Response body
- ✅ HTTP status code
- ✅ HTTP headers

```
@PostMapping("/users")
public ResponseEntity<User> createUser(@RequestBody User user) {
    User savedUser = userService.save(user);
    return ResponseEntity.status(HttpStatus.CREATED)
                         .body(savedUser);
}
```

```
@PostMapping("/users")
public ResponseEntity<String> create(@RequestBody User user) {
    if (user.getEmail() == null) {
        return ResponseEntity
                .badRequest()
                .body("Email is required");
    }
    return ResponseEntity.ok("User created");
}
```

### HTTP status codes

```
@PostMapping("/users")
public ResponseEntity<User> create(@RequestBody User user) {
    return ResponseEntity.status(HttpStatus.CREATED)
                         .body(userService.save(user));
}
```

## 🗄️ 4. Database + Spring Data JPA <a id="jpa"></a>

Spring Data JPA builds on top of JPA and Hibernate to eliminate boilerplate code. It provides:
- Repository interfaces
- Auto-generated queries
- Pagination & sorting
- Transaction management

Spring Data JPA simplifies database access by providing repository abstractions that eliminate boilerplate CRUD code and automatically generate queries.

JPA (Java Persistence API) is a specification that defines:
- How Java objects are mapped to database tables
- How CRUD operations should be done
- How relationships are handled

Hibernate is a JPA implementation:
- JPA = Rules / Contract
- Hibernate = Actual engine that follows the rules

### Without JPA (Plain JDBC)

```
Connection con = dataSource.getConnection();
PreparedStatement ps =
    con.prepareStatement("SELECT * FROM users WHERE id=?");
ps.setLong(1, id);
ResultSet rs = ps.executeQuery();
```

Issues:
- Too much boilerplate
- Error-prone
- Manual mapping
- Hard to maintain

JPA (Java Persistence API) is a specification that defines:
- How Java objects map to database tables
- How CRUD operations should work

### @Entity, @Id, @GeneratedValue

```
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    // getters & setters
}
```

| Annotation      | Purpose          |
| --------------- | ---------------- |
| @Entity         | Marks JPA entity. Map class to table |
| @Table          | Maps to DB table |
| @Id             | Primary key      |
| @GeneratedValue | Auto PK          |

Common Strategies for @GeneratedValue:
| Strategy | Description            |
| -------- | ---------------------- |
| IDENTITY | Auto-increment (MySQL) |
| SEQUENCE | DB sequence            |
| AUTO     | Provider decides       |
| TABLE    | PK table. JPA creates (or expects) a small dedicated database table that acts like a sequence simulator              |

Example with TABLE generatror:
```
@Id
@GeneratedValue(strategy = GenerationType.TABLE, generator = "product_gen")
@TableGenerator(
    name = "product_gen",
    table = "id_generators",               // custom table name
    pkColumnName = "generator_name",        // name of the key column
    pkColumnValue = "product",              // value that identifies this sequence
    valueColumnName = "next_value",         // column that holds the counter
    allocationSize = 20                     // how many IDs to "pre-allocate"
)
private Long id;

// 
CREATE TABLE id_generators (
    generator_name VARCHAR(255) NOT NULL PRIMARY KEY,
    next_value     BIGINT
);

-- Example row inserted by Hibernate
INSERT INTO id_generators (generator_name, next_value) VALUES ('product', 1);
```

| Strategy | DB Support       | Performance | Notes              |
| -------- | ---------------- | ----------- | ------------------ |
| IDENTITY | Auto-increment   | 🔴 Slow     | Insert triggers ID |
| SEQUENCE | DB sequence      | 🟢 Fast     | Best option        |
| TABLE    | Any DB           | 🔴 Slowest  | Extra table        |
| AUTO     | Provider decides | ⚠️ Varies   | Default            |

### Entity Lifecycle States

```
new MyEntity() 
    → Transient 
        ↓ persist() / save()
    → Managed  ───────────────┐
        │   flush() / commit()  │   (automatic dirty checking → INSERT/UPDATE)
        │                       │
        ↓ close() / end @Transactional / clear() / evict()
    → Detached  ←──────────────┘
        │
        ↓ merge()   (most important method when working with detached entities!)
    → Managed again
        │
        ↓ remove() / delete()
    → Removed  → commit() → gone from database
```

- __Transient__
  - just a plain Java object
  - Create with new, set fields, add to collections, etc.
  - Nothing is persisted until you call save() or persist().
  - Most common mistake: forgetting to save → data silently lost.
```
User user = new User("alice", "alice@example.com");   // transient
```
- __Managed__ 
  - the "happy path" inside @Transactional
  - Almost everything you do in a @Service / @Transactional method happens with managed entities.
  - Spring Data JPA repositories usually return managed entities when called inside a transaction
```
@Service
@Transactional
public class UserService {

    public User updateUser(Long id, String newName) {
        User user = userRepository.findById(id).orElseThrow();   // → managed
        user.setName(newName);                                    // change tracked!
        // no need to call save() — dirty checking does it
        return user;
    }
}
```
- Detached → very common source of bugs
  - You explicitly call entityManager.detach() or clear()
```
// load fresh managed entity + copy changes (preferred in complex apps)
@Transactional
public User updateBetter(Long id, UserUpdateDto dto) {
    User managed = userRepository.getReferenceById(id);   // or findById()
    managed.setName(dto.name());
    managed.setEmail(dto.email());
    // ... copy only allowed fields
    return managed;
}
```


### JpaRepository

```
public interface UserRepository extends JpaRepository<User, Long> {
}
```

What You Get for Free:
- save()
- findById()
- findAll()
- deleteById()
- count()

Using Repository in Service Layer:
```
@Service
public class UserService {

    @Autowired
    private UserRepository userRepo;

    public User create(User user) {
        return userRepo.save(user);
    }

    public User get(Long id) {
        return userRepo.findById(id)
                       .orElse(null);
    }

    public List<User> getAll() {
        return userRepo.findAll();
    }
}
```

REST Controller:
```
@RestController
@RequestMapping("/users")
public class UserController {

    @Autowired
    private UserService service;

    @PostMapping
    public User create(@RequestBody User user) {
        return service.create(user);
    }

    @GetMapping("/{id}")
    public User get(@PathVariable Long id) {
        return service.get(id);
    }
}
```

### Pagination & sorting

```
PageRequest page = PageRequest.of(0, 5);
Page<User> users = userRepo.findAll(page);
```

### Relationships

- @ManyToOne
```
@Entity
class Order {
    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;
}
```
- @OneToMany
```
@Entity
class User {
    @OneToMany(mappedBy = "user")
    private List<Order> orders;
}
```
- @ManyToMany
```
@ManyToMany
@JoinTable(
  name = "user_roles",
  joinColumns = @JoinColumn(name="user_id"),
  inverseJoinColumns = @JoinColumn(name="role_id")
)
private Set<Role> roles;
```
- Fetch Types:
  - @ManyToOne and @OneToOne default to FetchType.EAGER.
  - Issues:
    - Loads the target entity every time you load the source — even if you only need 2 fields from the parent
    - Causes memory bloat (especially deep object graphs)
    - Makes serialization (Jackson → JSON) dangerous → can pull in entire database if not careful
| Type  | Behavior         |
| ----- | ---------------- |
| LAZY  | Load when needed |
| EAGER | Load immediately |
```
@ManyToOne(fetch = FetchType.LAZY)
```

### Custom Queries

JPQL:
```
@Query("SELECT u FROM User u WHERE u.email = :email")
User findByEmail(@Param("email") String email);

```

Native SQL:
```
@Query(
  value = "SELECT * FROM users WHERE email=?",
  nativeQuery = true)
User findByEmail(String email);
```

### Transactions

Spring Data JPA uses transactions automatically.
```
@Transactional
public void updateUser(...) {}
```
- ✔ Rollback on exception
- ✔ ACID compliant

### EntityManager

- Repositories = convenient & safe 95% of the time
- EntityManager = your escape hatch when you __need full JPA / Hibernate power__

Repositories are best when you need:
- ✔ Standard CRUD
- ✔ Simple queries
- ✔ Pagination & sorting
- ✔ Fast development
- ✔ Clean code

Use EntityManager when:
- Complex Dynamic Queries
- Bulk Updates & Delete
- Batch Inserts / Updates


```
Example:
@Service
@Transactional
class DocumentService {

    @PersistenceContext
    private EntityManager em;

    // 1. Refresh entity after external change (trigger, stored proc, another transaction)
    public void refreshAfterExternalUpdate(Long id) {
        Document doc = em.find(Document.class, id);
        // ... assume external system changed row
        em.refresh(doc);           // reloads fresh state from DB
    }

    // 2. Complex query with multiple JOIN FETCH to avoid N+1
    public List<Order> findOrdersWithDetails(String customerEmail) {
        return em.createQuery("""
            SELECT o FROM Order o
            JOIN FETCH o.customer c
            JOIN FETCH o.items i
            JOIN FETCH i.product
            WHERE c.email = :email
            """, Order.class)
            .setParameter("email", customerEmail)
            .getResultList();
    }

    // 3. Batch insert / update – important for performance
    @Transactional
    public void importLargeFile(List<Product> products) {
        int i = 0;
        for (Product p : products) {
            em.persist(p);
            i++;
            if (i % 100 == 0) {          // flush & clear every 100 entities
                em.flush();
                em.clear();               // ← prevents OutOfMemory
            }
        }
        em.flush();
        em.clear();
    }

    // 4. Get Hibernate Session when needed (e.g. natural-id lookup)
    public Product findByNaturalId(String sku) {
        Session session = em.unwrap(Session.class);
        return session.byNaturalId(Product.class)
                      .using("sku", sku)
                      .loadOptional()
                      .orElse(null);
    }
}
```


## 🔐 5. Exception Handling & Validation <a id="exceptions"></a>

Default Spring Boot Exception Handling (Out of the Box):
- Spring Boot returns JSON error response
- Status code depends on exception

Example error response:
```
{
  "timestamp": "2026-01-30T10:15:00",
  "status": 404,
  "error": "Not Found",
  "path": "/users/10"
}
```

Custom exception:
```
@ResponseStatus(HttpStatus.NOT_FOUND)
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String msg) {
        super(msg);
    }
}
```

### @ExceptionHandler

@ExceptionHandler is a Spring annotation that marks a method as responsible for handling specific exceptions thrown during request processing in controllers.

It allows you to centralize exception handling logic, return custom HTTP status codes, and provide meaningful error responses (especially important for REST APIs).

Use it:
- in one @Controller or @RestController
- recomended: Global (application-wide) @ExceptionHandler inside @ControllerAdvice or @RestControllerAdviceConsistent error responses across all controllers (recommended)Yes — almost always this way

```
@RestController
public class UserController {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handle(UserNotFoundException ex) {
        return ResponseEntity.status(404).body(ex.getMessage());
    }
}
```

Better:
```
import org.springframework.http.*;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.context.request.WebRequest;
import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;

import java.time.Instant;
import java.util.*;

@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    // Generic fallback for any unhandled exception (production: log + minimal info)
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleAllExceptions(
            Exception ex, WebRequest request) {

        ErrorResponse error = new ErrorResponse(
                HttpStatus.INTERNAL_SERVER_ERROR.value(),
                "Internal Server Error",
                ex.getMessage(),
                request.getDescription(false),
                Instant.now()
        );

        // In production → log.error("Unhandled exception", ex);
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }

    // Custom not-found
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
            ResourceNotFoundException ex, WebRequest request) {

        ErrorResponse error = new ErrorResponse(
                HttpStatus.NOT_FOUND.value(),
                "Resource Not Found",
                ex.getMessage(),
                request.getDescription(false),
                Instant.now()
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    // Validation errors (@Valid / @Validated)
    @Override
    protected ResponseEntity<Object> handleMethodArgumentNotValid(
            MethodArgumentNotValidException ex,
            HttpHeaders headers,
            HttpStatusCode status,
            WebRequest request) {

        List<String> errors = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(err -> err.getField() + ": " + err.getDefaultMessage())
                .toList();

        ErrorResponse error = new ErrorResponse(
                HttpStatus.BAD_REQUEST.value(),
                "Validation Failed",
                String.join("; ", errors),
                request.getDescription(false),
                Instant.now()
        );

        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }

    // You can add more: AccessDenied, Security exceptions, etc.
}
```

### @ControllerAdvice

@ControllerAdvice is a specialized Spring annotation used to define global behavior for controllers.

Most commonly, it’s used for:
- ✅ Global exception handling
- ✅ Global data binding
- ✅ Global model attributes

### Custom exceptions

Create a Custom Exception:
```
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }
}
```

Throw the Exception:
```
@Service
public class UserService {
    public User getUser(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() ->
                new UserNotFoundException("User not found with id " + id));
    }
}
```

Handle Exception Using @ControllerAdvice:
```
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ApiError> handleUserNotFound(
            UserNotFoundException ex) {

        ApiError error = new ApiError(
            404,
            ex.getMessage(),
            LocalDateTime.now()
        );

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(error);
    }
}

public class ApiError {
    private int status;
    private String message;
    private LocalDateTime timestamp;

    // constructors, getters
}
```

### Bean Validation

Common Validation Annotations
- @NotNull
- @Size
- @Email

```
public class UserRequest {

    @NotNull
    @Size(min = 3, max = 20)
    private String name;

    @Email
    private String email;

    @Min(18)
    private int age;
}
```
| Annotation  | Purpose              |
| ----------- | -------------------- |
| @NotNull    | Not null             |
| @NotBlank   | Not null + not empty |
| @Size       | String length        |
| @Email      | Email format         |
| @Min / @Max | Numeric range        |
| @Pattern    | Regex                |


Triggering Validation in Controller:
```
@PostMapping("/users")
public ResponseEntity<User> createUser(
        @Valid @RequestBody UserRequest request) {
    return ResponseEntity.ok(service.create(request));
}
```

If validation fails Spring triggers MethodArgumentNotValidException



## 🔒 6. Spring Security <a id="security"></a>

Spring Security is a powerful framework that provides:
- ✅ Authentication (Who are you?)
- ✅ Authorization (What are you allowed to do?)
- ✅ Protection against common attacks (CSRF, XSS, etc.)

If authentication or authorization fails → request never reaches controller:
```
HTTP Request
   ↓
Spring Security Filters
   ↓
Authentication
   ↓
Authorization
   ↓
Controller
```

Default Spring Security Behavior (Out of the Box)
- Username: user
- Password: printed in console
- All endpoints require login
- Form-based login enabled

### Basic authentication

__Simple REST Controller (to protect)__
```
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/public")
    public String publicEndpoint() {
        return "This is public — no authentication needed";
    }

    @GetMapping("/api/hello")
    public String hello() {
        return "Hello! You are authenticated with Basic Auth.";
    }

    @GetMapping("/api/admin")
    public String admin() {
        return "Admin area — you need ADMIN role";
    }
}
```

__Security Configuration — Basic Auth + In-Memory Users__
```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;
import static org.springframework.security.config.Customizer.withDefaults;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // → Disable CSRF (common for pure REST APIs with Basic Auth)
            .csrf(csrf -> csrf.disable())

            // → Authorization rules
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public").permitAll()           // open to everyone
                .requestMatchers("/api/admin").hasRole("ADMIN")   // only ADMIN role
                .anyRequest().authenticated()                     // everything else needs login
            )

            // → Enable HTTP Basic Authentication
            .httpBasic(withDefaults());   // ← this is the key line for Basic Auth

        // Optional: disable default form login if you don't want it
        // .formLogin(form -> form.disable());

        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService(PasswordEncoder passwordEncoder) {
        UserDetails user = User.builder()
                .username("user")
                .password(passwordEncoder.encode("user123"))
                .roles("USER")
                .build();

        UserDetails admin = User.builder()
                .username("admin")
                .password(passwordEncoder.encode("admin456"))
                .roles("ADMIN")
                .build();

        return new InMemoryUserDetailsManager(user, admin);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### Authentication with Database (JPA)

```
@Entity
public class AppUser {

    @Id
    @GeneratedValue
    private Long id;

    private String username;
    private String password;
    private String role;
}

@Service
public class CustomUserDetailsService
        implements UserDetailsService {

    @Autowired
    private UserRepository repo;

    @Override
    public UserDetails loadUserByUsername(String username)
            throws UsernameNotFoundException {

        AppUser user = repo.findByUsername(username)
            .orElseThrow(() ->
                new UsernameNotFoundException("User not found"));

        return User.withUsername(user.getUsername())
                .password(user.getPassword())
                .roles(user.getRole())
                .build();
    }
}

```

### JWT authentication (for REST APIs)

JWT-based security is currently (in January 2026) the most popular and recommended stateless authentication mechanism

It replaces session cookies or Basic Auth in microservices, SPAs (React/Vue/Angular), mobile apps, and API-first architectures.

### Password encryption (BCrypt)

### Role-based authorization

```
...
@Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
        ...
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .requestMatchers("/user/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
)
```


### Securing endpoints

Enable method-level security:
- @EnableMethodSecurity → Enables annotations like @PreAuthorize so methods can be secured based on roles.
```
@SpringBootApplication
@EnableMethodSecurity
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

Controller with @PreAuthorize:
- @PreAuthorize("hasAnyRole('USER','ADMIN')") → Checks before method runs if authenticated user has either USER or ADMIN role.
```
@RestController
public class SecureController {

    @GetMapping("/public")
    public String publicApi() {
        return "Public API";
    }

    @PreAuthorize("hasAnyRole('USER','ADMIN')")
    @GetMapping("/secure/user")
    public String userAccess() {
        return "User access";
    }

    @PreAuthorize("hasRole('ADMIN')")
    @GetMapping("/secure/admin")
    public String adminAccess() {
        return "Admin access";
    }
}
```

### how jwt token authentication works

JWT (JSON Web Token) is a stateless authentication mechanism.

The server gives the client a signed token after login, and the __client sends that token with every request__ to prove who they are.

A JWT has 3 parts, separated by dots:
- Header: Algorithm used to sign the token
```
{ "alg": "HS256", "typ": "JWT" }     
```
- Payload: actual claims / data
```
{ "sub": "user-1234", "name": "Anna Smith", "roles": ["user", "order-manager"], "iat": 1738390400, "exp": 1738391000, "iss": "api.example.com" }
```
- Signature: Cryptographic proof — created using secret key + header + payload
```
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

The token when encoded looks like this:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyLTEyMzQiLCJuYW1lIjoiQW5uYSBTbWl0aCIsInJvbGVzIjpbInVzZXIiLCJvcmRlci1tYW5hZ2VyIl0sImlhdCI6MTczODM5MDQwMCwiZXhwIjoxNzM4MzkxMDAwLCJpc3MiOiJhcGkuZXhhbXBsZS5jb20ifQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

JWT Authentication Flow:
```
1. Client sends login request (username/password)
2. Server validates credentials
3. Server generates JWT
4. Client stores JWT
5. Client sends JWT in every request (JWT is sent in the HTTP request header)
```
GET /api/users
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```
6. Server validates JWT
7. Request allowed or rejected
```

How Server Validates JWT:
```
- Extract token from Authorization header
- Verify signature using secret key
- Check expiration
- Extract user details
- Set authentication in SecurityContext
- Continue request
```

Example of token verification in spring:
```
public class JwtAuthFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain) throws ServletException, IOException {

        String header = request.getHeader("Authorization");

        if (header != null && header.startsWith("Bearer ")) {
            String token = header.substring(7);
            String username = jwtService.extractUsername(token);

            UsernamePasswordAuthenticationToken auth =
                new UsernamePasswordAuthenticationToken(
                    username, null, jwtService.extractRoles(token));

            SecurityContextHolder.getContext().setAuthentication(auth);
        }

        filterChain.doFilter(request, response);
    }
}
```

### how oAuth2  authentication works with example

It allows an application to access a user’s data on another system without knowing the user’s password.

Key Roles in OAuth2:
```
| Role                     | Meaning                                |
| ------------------------ | -------------------------------------- |
| **Resource Owner**       | The user (you)                         |
| **Client**               | The application (your app)             |
| **Authorization Server** | Issues tokens (Google, Keycloak, Okta) |
| **Resource Server**      | Hosts protected APIs                   |
```

OAuth2 Flow (Authorization Code Grant – MOST COMMON):
1. User clicks Login
2. App redirects user to Authorization Server
  - Your app redirects the browser:
```
GET https://accounts.google.com/o/oauth2/v2/auth?
    client_id=abc123
    &redirect_uri=http://localhost:8080/login/oauth2/code/google
    &response_type=code
    &scope=profile email
```  
3. User logs in & grants consent
4. Auth Server sends Authorization Code to Client
5. Client exchanges code for Access Token
  - request to Authorization server (google):
```
POST https://oauth2.googleapis.com/token
Content-Type: application/x-www-form-urlencoded

client_id=abc123
client_secret=secret
code=4/xyz123
grant_type=authorization_code
redirect_uri=http://localhost:8080/login/oauth2/code/google
```
  - google returns:
```
{
  "access_token": "ya29.a0AfH6...",
  "expires_in": 3600,
  "refresh_token": "1//0g...",
  "token_type": "Bearer"
}
```
6. Client calls ResourceServer using Bearer token
7. Resource Server validates the JWT token: validates the token’s cryptographic signature using a secret key or public key that only the Authorization Server controls.

```
User (Browser)        Client App          Authorization Server       Resource Server
     |                    |                        |                     |
     |--- Click Login --->|                        |                     |
     |                    |--- Redirect /authorize --->|                 |
     |                    |                        |--- Prompt Login & Consent --->|
     |                    |                        |<-- User submits credentials --|
     |                    |<-- Authorization Code --|                     |
     |                    |--- Exchange code for token --->|             |
     |                    |<-- Access Token (JWT) ---|                     |
     |                    |                        |                     |
     |                    |--- API Request ----------------------------->|
     |                    |      Authorization: Bearer <AccessToken>     |
     |                    |                        |    Validate JWT --->|          
     |                    |                        |      (signature, exp, iss, aud)
     |                    |<--------------------- Protected Resource ----|
     |<------------------ Response ----------------|                     |
```

the Resource Server does NOT need to contact the Authorization Server for every request when validating a JWT access token.
JWT (JSON Web Token) is self-contained and can verify all of this locally using cryptography

### how SAML works

SAML is a protocol for Single Sign-On (SSO).

- It allows a user to log in once with an Identity Provider (IdP) and access multiple Service Providers (SP) without entering credentials again.
- It’s XML-based (the messages are XML documents).
- Common in enterprise apps (Okta, ADFS, Keycloak, Salesforce).

Core Concepts:
| Term                        | Meaning                                              |
| --------------------------- | ---------------------------------------------------- |
| **Identity Provider (IdP)** | Authenticates users (e.g., Okta, Keycloak, ADFS)     |
| **Service Provider (SP)**   | Application that consumes SAML token (your app)      |
| **SAML Assertion**          | XML document containing user identity and attributes |

```
User (Browser)        Service Provider (SP)       Identity Provider (IdP)
     |                        |                           |
     |--- Access /dashboard -->|                           |
     |                        |--- Redirect /sso/login -->|
     |                        |                           |--- Prompt login / consent -->|
     |                        |                           |<-- User submits credentials --|
     |                        |<-- SAML Response (Base64) --|
     |<-- Browser POST SAML --|                           |
     |                        |--- Validate signature --->|
     |                        |--- Extract user info ---->|
     |                        |--- Create session -------->|
     |<-- Session cookie ------|                           |
     |--- Access /dashboard -->|                           |
     |<-- Protected resource ---|                           |
```

- User → SP: Tries to access a protected resource (/dashboard).
- SP → IdP: Redirects user to IdP with SAMLRequest (XML).
- IdP → User: Prompts login/consent if needed.                  // SP never authenticates user directly — it relies on the IdP.
- User → IdP: Submits credentials.
- IdP → SP (via Browser): Sends SAMLResponse (signed assertion).
- SP validates the assertion (signature, expiration, issuer).   // SAMLResponse is signed by IdP, SP verifies signature.
- SP creates session for the user.
- User → SP: Requests resource again, SP serves it using the session.

Spring example:
```
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .saml2Login();
        return http.build();
    }
}
```

## ⚙️ 7. Configuration & Profiles <a id="configuration"></a>

Spring Boot uses externalized configuration to make apps flexible without changing code.
This means you can define properties outside your Java code:
- application.properties or application.yml (default)
- Environment variables
- Command-line arguments
- Config Server (Spring Cloud)

### application.properties

```
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=root
```

### Profiles (dev, test, prod)

Profiles allow you to run the same application with different configurations based on the environment (dev, test, prod).

Default profile: default

application-dev.properties:
```
server.port=8081
spring.datasource.url=jdbc:mysql://localhost:3306/devdb
spring.datasource.username=devuser
spring.datasource.password=devpass
```

application-prod.properties:
```
server.port=80
spring.datasource.url=jdbc:mysql://prod-db-server:3306/proddb
spring.datasource.username=produser
spring.datasource.password=prodpass
```

application.yml can include multiple profiles:
```
spring:
  profiles:
    active: dev

---
spring:
  profiles: dev
server:
  port: 8081
datasource:
  url: jdbc:mysql://localhost:3306/devdb
  username: devuser
  password: devpass

---
spring:
  profiles: prod
server:
  port: 80
datasource:
  url: jdbc:mysql://prod-db-server:3306/proddb
  username: produser
  password: prodpass

```

How to Activate a Profile:
- Command-line argument
```
java -jar myapp.jar --spring.profiles.active=dev
```
- Environment variable
```
export SPRING_PROFILES_ACTIVE=prod
java -jar myapp.jar
```

### @Value

@Value is a way to inject external configuration values into your beans

application.properties:
```
app.name=MySpringApp
app.version=1.0.0
```

You can inject these values into a Spring bean:
```
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class AppInfo {

    @Value("${app.name}")
    private String name;

    @Value("${app.version}")
    private String version;

    public void printInfo() {
        System.out.println("App: " + name + ", Version: " + version);
    }
}
```

Injecting Environment Variables:
```
@Value("${HOME}")
private String homeDirectory;
```

### @ConfigurationProperties

@ConfigurationProperties is a Spring Boot annotation that __binds a whole group of properties__ (from application.properties or application.yml) to a Java class.

It’s ideal for structured configuration instead of injecting individual values one by one.

application.yml:
```
app:
  name: MySpringBootApp
  version: 1.0.0
  features:
    login: true
    payments: false
```

Create a class to bind all these properties:
```
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {

    private String name;
    private String version;
    private Features features;

    public static class Features {
        private boolean login;
        private boolean payments;

        // getters and setters
        public boolean isLogin() { return login; }
        public void setLogin(boolean login) { this.login = login; }
        public boolean isPayments() { return payments; }
        public void setPayments(boolean payments) { this.payments = payments; }
    }

    // getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getVersion() { return version; }
    public void setVersion(String version) { this.version = version; }
    public Features getFeatures() { return features; }
    public void setFeatures(Features features) { this.features = features; }
}
```

Differences Between @Value and @ConfigurationProperties:
```
| Feature             | @Value                    | @ConfigurationProperties                    |
| ------------------- | ------------------------- | ------------------------------------------- |
| Usage               | Single property injection | Bind multiple related properties to a class |
| Type Safety         | No compile-time checking  | Strongly typed POJO                         |
| Nested Properties   | Hard to manage            | Natural with nested classes                 |
| Ease for many props | Tedious for many props    | Very convenient                             |
| SpEL support        | Yes                       | No (focus on binding only)                  |
```

### External configs

Spring Boot allows you to specify an external property file outside your packaged .jar file (useful for sensitive data and environment-specific configurations).

```
java -jar myapp.jar --spring.config.location=file:/path/to/external-config/
```

### Spring Cloud Config

For large-scale systems with multiple microservices, it’s common to externalize configuration using Spring Cloud Config. This solution stores configuration in a centralized repository (Git, filesystem, etc.) and allows all your microservices to pull the same configuration dynamically.



## 🧪 8. Testing Spring Boot Apps <a id="testing"></a>

- Unit Tests – test a single class or method, no Spring context required.
- Integration Tests – test the Spring context, database, and beans together.
- Web/Controller Tests – test REST endpoints, including HTTP requests and responses.

Unit test:
```
import static org.junit.jupiter.api.Assertions.assertEquals;
import org.junit.jupiter.api.Test;

public class CalculatorServiceTest {

    private final CalculatorService calculatorService = new CalculatorService();

    @Test
    void testAdd() {
        int result = calculatorService.add(2, 3);
        assertEquals(5, result);
    }
}
```

Testing with Spring Context (Integration Test):
```
@Service
public class UserService {
    private final UserRepository userRepository;
    public UserService(UserRepository userRepository) { this.userRepository = userRepository; }

    public User findByUsername(String username) {
        return userRepository.findByUsername(username).orElse(null);
    }
}


import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.beans.factory.annotation.Autowired;
import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest
public class UserServiceIntegrationTest {

    @Autowired
    private UserService userService;

    @Test
    void testFindByUsername() {
        User user = userService.findByUsername("john");
        assertThat(user).isNotNull();
        assertThat(user.getUsername()).isEqualTo("john");
    }
}
```


Mocking Dependencies with Mockito: For faster unit tests, we can mock repositories:

### Mockito

It allows you to create fake (mock) objects that replace real dependencies (services, repositories, external clients, etc.) during tests.

Without mocking:
- tests become slow (real DB, real HTTP calls)
- tests become flaky (external services down, data changes)
- tests are hard to set up (need test database, credentials, etc.)

```
@ExtendWith(MockitoExtension.class)           // ← enables Mockito annotations
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepo;

    @Mock
    private PaymentClient paymentClient;

    @InjectMocks                               // ← auto-injects mocks into constructor
    private OrderService orderService;

    @Test
    void shouldCreateOrderAndMarkAsPaidWhenPaymentSucceeds() {
        // given (BDD style – very readable)
        CreateOrderRequest request = new CreateOrderRequest(...);
        Order savedOrder = new Order(...);
        given(orderRepo.save(any(Order.class))).willReturn(savedOrder);         
        given(paymentClient.charge(anyBigDecimal(), anyString())).willReturn(true);

        // when
        Order result = orderService.createOrder(request);

        // then
        assertThat(result.isPaid()).isTrue();
        then(orderRepo).should(times(2)).save(any(Order.class));   // called twice
        then(paymentClient).should().charge(eq(savedOrder.getTotalAmount()), anyString());
    }

    @Test
    void shouldThrowExceptionWhenPaymentFails() {
        // given
        CreateOrderRequest request = ...;
        given(paymentClient.charge(any(), any())).willReturn(false);

        // when + then
        assertThatThrownBy(() -> orderService.createOrder(request))
                .isInstanceOf(PaymentFailedException.class);

        then(orderRepo).should(times(1)).save(any());   // saved only once (before payment)
    }
}
```

Web Layer Test with MockMvc:
- @WebMvcTest loads only controller layer.
- @MockBean injects a mock of the service.
```
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.web.servlet.MockMvc;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(UserController.class)
public class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void testGetUser() throws Exception {
        User user = new User();
        user.setUsername("john");
        when(userService.findByUsername("john")).thenReturn(user);

        mockMvc.perform(get("/api/users/john"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.username").value("john"));
    }
}
```


## ☁️ 9. Advanced & Real-World Topics <a id="advanced"></a>

### Spring Actuator

Spring Boot Actuator is a sub-project of Spring Boot that adds production-ready features to your application.
It provides monitoring, metrics, health checks, and management endpoints out of the box.

```
| Endpoint               | Purpose                                      |
| ---------------------- | -------------------------------------------- |
| `/actuator/health`     | Shows app health (UP/DOWN)                   |
| `/actuator/info`       | Shows custom application info                |
| `/actuator/metrics`    | Shows metrics (CPU, memory, JVM, HTTP, etc.) |
| `/actuator/env`        | Shows environment properties                 |
| `/actuator/loggers`    | View or modify log levels at runtime         |
| `/actuator/threaddump` | Shows JVM thread dump                        |
| `/actuator/httptrace`  | Recent HTTP request traces                   |
```



### Logging (Logback, SLF4J)

- SLF4J (Simple Logging Facade for Java) is just a facade, not a logging framework.
- It allows you to write logging code once and plug in any logging framework (Logback, Log4j2, etc.) at runtime.
- In Spring Boot, SLF4J is used by default, and Logback is the default logging implementation.


### Caching (Redis)

Spring Boot provides Spring Cache abstraction that supports multiple caching providers:
- In-memory: ConcurrentHashMap (default), EhCache
- Distributed: Redis, Hazelcast, Memcached

Enable Caching in Spring Boot:
- Add @EnableCaching in your main application class
```
@SpringBootApplication
@EnableCaching
public class CachingDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(CachingDemoApplication.class, args);
    }
}
```

Using @Cacheable:
- @Cacheable tells Spring: “Check the cache before executing the method. If the value is present, return it; otherwise, execute and cache the result.”
```
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @Cacheable("users")
    public String getUserById(Long id) {
        simulateSlowService();
        return "User" + id;
    }

    private void simulateSlowService() {
        try {
            Thread.sleep(3000); // simulate delay
        } catch (InterruptedException e) {
            throw new IllegalStateException(e);
        }
    }
}
```

| Annotation       | Purpose                                                            |
| ---------------- | ------------------------------------------------------------------ |
| `@Cacheable`     | Reads from cache if exists; else executes method and caches result |
| `@CachePut`      | Always executes method and updates cache                           |
| `@CacheEvict`    | Removes cache entries                                              |
| `@EnableCaching` | Enables caching in Spring Boot                                     |



### Dockerizing Spring Boot

Dockerizing means packaging your Spring Boot application into a Docker container, which includes:
- Your application JAR
- JVM/runtime
- Dependencies
- Configuration


### Spring Cloud (Microservices)

Spring Cloud is a set of tools for building distributed, cloud-native applications.

It provides features for:
- Service Discovery (Eureka, Consul)
  - When this microservice starts, it registers with Eureka Server automatically
```
spring.application.name=user-service
server.port=8081
eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
```
- Load Balancing (Ribbon, Spring Cloud LoadBalancer)
  - @LoadBalanced → allows service name resolution via Eureka instead of hardcoding URLs.
- API Gateway (Spring Cloud Gateway)
- Circuit Breakers (Resilience4J, Hystrix)
- Configuration Management (Spring Cloud Config)
```
server.port=8888
spring.cloud.config.server.git.uri=https://github.com/your-repo/config-repo
```
- Distributed Tracing & Messaging (Sleuth, Stream)

Spring Cloud builds on top of Spring Boot, so it’s easy to integrate

#### should i use spring cloud and service discovery (Consul), load balancing (ribbon) when deploying spring boot app to kubernetes?

not usually – Kubernetes already provides these features natively

kubernetes provides:
- Service Discovery: Kubernetes Service objects give DNS names for pods


Use Kubernetes native features for:
- Service discovery → Service DNS
- Load balancing → Service or Ingress
- Health checks → liveness/readiness probes

✅ Use Spring Cloud components only for:
- Configuration server (Spring Cloud Config)
- Circuit breakers (Resilience4J)
- API Gateway / routing (Spring Cloud Gateway)
- Distributed tracing (Sleuth + Zipkin)

#### how spring boot helps with event driven applications?



## 10. Deploy spring boot into kubernetes - best practices <a id="intokubernetes"></a>

- Containerize the Spring Boot App
  - Use slim/base images for smaller images.
  - Multi-stage build to exclude build tools from final image.
  - Avoid running as root user inside the container
- Externalize Configuration
  - Use Kubernetes ConfigMap and Secrets for sensitive info (DB passwords, API keys).
  - Environment variables injected into the container.
- Define Kubernetes Deployment
  - Replicas > 1 for high availability.
  - Use readiness and liveness probes to manage pod health:
- Expose the Service
  - Use ClusterIP, NodePort, or LoadBalancer:
- Use Spring Boot Actuator
  - Expose health, metrics, and info endpoints for Kubernetes probes and monitoring
- Logging & Monitoring
  - Spring Boot logs → stdout/stderr, Kubernetes will capture logs via kubectl logs.
  - Integrate Prometheus + Grafana using Spring Boot Actuator metrics.
  - Use ELK stack (Elasticsearch, Logstash, Kibana) or Loki for centralized logging.
- Scaling & Resource Management
  - Define resources and limits:
  - Horizontal scaling: kubectl scale deployment springboot-app --replicas=5
  - Kubernetes Horizontal Pod Autoscaler (HPA) can scale pods automatically based on CPU or custom metrics.
- Use CI/CD for Deployment
  - Build Docker images using GitHub Actions, Jenkins, GitLab CI.
  - Push images to Docker Registry.
  - Automate kubectl apply or Helm charts for deployment.
  - Version control Kubernetes manifests.



## 🛠️ 11. Real Projects <a id="projects"></a>

### Beginner Projects

#### Student Management System

#### Todo REST API

#### User Registration & Login


### Intermediate Projects

#### E-commerce backend

#### Blog REST API with JWT

#### Expense Tracker


### Advanced Projects

#### Microservices system

#### Payment gateway integration

#### Distributed system with Spring Cloud




