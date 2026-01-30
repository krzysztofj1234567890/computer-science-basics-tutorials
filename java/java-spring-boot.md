# Leaarning Spring Boot


## 🌱 1. Understand Spring Basics 

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




## ⚡ 2. Spring Boot

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

### @SpringBootApplication

### Auto-configuration

### Starter dependencies

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


## 🌐 3. Learn REST APIs with Spring Boot

### @RestController

### @GetMapping, @PostMapping, @PutMapping, @DeleteMapping

### @RequestParam

### @PathVariable

### @RequestBody

### ResponseEntity

### HTTP status codes

### Build a CRUD REST API



## 🗄️ 4. Database + Spring Data JPA

### JPA & Hibernate basics

### @Entity, @Id, @GeneratedValue

### JpaRepository

### CRUD operations

### Pagination & sorting

### Relationships:

#### @OneToMany

#### @ManyToOne

### Example Repository

```
public interface UserRepository extends JpaRepository<User, Long> {
    User findByEmail(String email);
}
```


## 🔐 5. Exception Handling & Validation

### @ExceptionHandler

### @ControllerAdvice

### Custom exceptions

### Bean Validation:

#### @NotNull

#### @Size

#### @Email



## 🔒 6. Spring Security (Very Important for Jobs)

### Basic authentication

### Password encryption (BCrypt)

### Role-based authorization

### JWT authentication (for REST APIs)

### Securing endpoints



## ⚙️ 7. Configuration & Profiles

### application.properties

### Profiles (dev, test, prod)

### @Value

### @ConfigurationProperties

### External configs


## 🧪 8. Testing Spring Boot Apps

### @SpringBootTest

### @WebMvcTest

### @MockBean

### Unit vs integration testing


## ☁️ 9. Advanced & Real-World Topics

### Spring Actuator

### Logging (Logback, SLF4J)

### Caching (Redis)

### Dockerizing Spring Boot

### Spring Cloud (Microservices)

### Kafka / RabbitMQ (optional)


## 🛠️ 10. Build Real Projects

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

