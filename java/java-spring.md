# Table of Contents
- [Java spring](#JavaSpring)
  - [Spring Architecture](#Architecture)
  - [Example](#Example)
  - [IoC Containers](#IoCcontainers)
  - [Bean](#Bean)
  - [Event Handling](#EventHandling)
  - [JDBC Framework](#JDBC)
  - [MVC Framework](#MVC)
- [Spring Boot](#Boot)
  - [Spring Boot Architecture](#BootArchitecture)


# Java spring <a id="JavaSpring"></a>

https://www.tutorialspoint.com/spring/index.htm

Spring framework is an open source Java platform that provides comprehensive infrastructure support for developing robust Java applications.

Features:
- POJO Based - Spring enables developers to develop enterprise-class applications using POJOs
- Modular - Spring is organized in a modular fashion.
- Integration with existing frameworks
- Testablity - Testing an application written with Spring is simple because environment-dependent code is moved into this framework.
- Web MVC - Spring's web framework is a well-designed web MVC framework, which provides a great alternative to web frameworks such as Struts
- Lightweight - Lightweight IoC containers tend to be lightweight, especially when compared to EJB containers
- Transaction management - Spring provides a consistent transaction management interface

The technology that Spring is most identified with is the __Dependency Injection__ (DI) flavor of Inversion of Control. 

When writing a complex Java application, application classes should be as __independent as possible__ of other Java classes to increase the possibility to reuse these classes and to test them independently of other classes while unit testing. __Dependency Injection helps in gluing these classes together and at the same time keeping them independent__

One of the key components of Spring is the __Aspect Oriented Programming__ (AOP) framework. The functions that span multiple points of an application are called cross-cutting concerns and these cross-cutting concerns are conceptually separate from the application's business logic. There are various common good examples of aspects including logging, declarative transactions, security, caching, etc.

## Architecture <a id="Architecture"></a>

Core container:
- The __Core module__ provides the fundamental parts of the framework, including the IoC and __Dependency Injection__ features.
- The __Bean__ module provides BeanFactory, which is a sophisticated implementation of the __factory pattern__.
- The __Context module__ builds on the solid base provided by the Core and Beans modules and it is a medium to access any objects defined and configured. The ApplicationContext interface is the focal point of the Context module.

Data Access/Integration:
- The __JDBC__ module provides a JDBC-abstraction layer that removes the need for tedious JDBC related coding.
- The __ORM__ module provides integration layers for popular object-relational mapping APIs, including __JPA__, JDO, __Hibernate__, and iBatis.
- The __OXM__ module provides an abstraction layer that supports Object/__XML__ mapping implementations for JAXB, Castor, XMLBeans, JiBX and XStream.
- The Java Messaging Service __JMS__ module contains features for __producing and consuming messages__.
- The __Transaction__ module supports programmatic and declarative transaction

Web modules:
- The __Web__ module provides basic __web-oriented integration features__ such as multipart file-upload functionality and the initialization of the IoC container using __servlet__ listeners and a __web-oriented application context__.
- The Web-MVC module contains Spring's __Model-View-Controller__ (MVC) implementation for web applications.
- The __Web-Socket__ module provides support for WebSocket-based, __two-way communication__ between the client and the server in web applications.
- The __Web-Portlet__ module provides

Other modules:
- The __AOP module__ provides an aspect-oriented programming implementation allowing you to define method-interceptors and pointcuts
- The __Aspects module__ provides integration with AspectJ, which is again a powerful and mature AOP framework.
- The __Test module__ supports the testing of Spring components with JUnit or TestNG frameworks.

## Example <a id="Example"></a>

Hello World: HelloWorld.java:
```
package com.tutorialspoint;

public class HelloWorld {
   private String message;

   public void setMessage(String message){
      this.message  = message;
   }
   public void getMessage(){
      System.out.println("Your Message : " + message);
   }
}
```
and MainApp.java:
```
package com.tutorialspoint;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MainApp {
   public static void main(String[] args) {
      ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
      obj.getMessage();
   }
}
```

## IoC Containers <a id="IoCcontainers"></a>

__The container will create the objects, wire them together, configure them, and manage their complete life cycle__ from creation till destruction.

The Spring container uses DI to manage the components that make up an application. __These objects are called Spring Beans__

The container gets its instructions on what objects to instantiate, configure, and assemble by reading the configuration metadata provided. The configuration metadata can be represented either by XML, Java annotations, or Java code.

## Bean <a id="Bean"></a>

The objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. A bean is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container. These beans are created with the configuration metadata that you supply to the container. 

Following are the three important methods to provide configuration metadata to the Spring Container −
- XML based configuration file.
- Annotation-based configuration
- Java-based configuration

When defining a <bean> you have the option of declaring a scope for that bean:
- to force Spring to produce a new bean instance each time one is needed, you should declare the bean's scope attribute to be __prototype__.
- if you want Spring to return the same bean instance each time one is needed, you should declare the bean's scope attribute to be __singleton__.
- __request__: This scopes a bean definition to an HTTP request. Only valid in the context of a web-aware Spring ApplicationContext.
- __session__: This scopes a bean definition to an HTTP session. Only valid in the context of a web-aware Spring ApplicationContext.

## Event Handling in Spring <a id="EventHandling"></a>

The ApplicationContext publishes certain types of events when loading the beans. For example, a ContextStartedEvent is published when the context is started and ContextStoppedEvent is published when the context is stopped.

Event handling in the ApplicationContext is provided through the ApplicationEvent class and ApplicationListener interface. Hence, if a bean implements the ApplicationListener, then every time an ApplicationEvent gets published to the ApplicationContext, that bean is notified.

### Listening to Context Events

To listen to a context event, a bean should implement the ApplicationListener interface which has just one method onApplicationEvent()

HelloWorld.java:
```
package com.tutorialspoint;

public class HelloWorld {
   private String message;

   public void setMessage(String message){
      this.message  = message;
   }
   public void getMessage(){
      System.out.println("Your Message : " + message);
   }
}
```
CStartEventHandler.java:
```
package com.tutorialspoint;

import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextStartedEvent;

public class CStartEventHandler 
   implements ApplicationListener<ContextStartedEvent>{

   public void onApplicationEvent(ContextStartedEvent event) {
      System.out.println("ContextStartedEvent Received");
   }
}
```
MainApp.java:
```
package com.tutorialspoint;

import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MainApp {
   public static void main(String[] args) {
      ConfigurableApplicationContext context = 
         new ClassPathXmlApplicationContext("Beans.xml");

      // Let us raise a start event.
      context.start();
	  
      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
      obj.getMessage();

      // Let us raise a stop event.
      context.stop();
   }
}
```

 Beans.xml:
 ```
 <?xml version = "1.0" encoding = "UTF-8"?>

<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id = "helloWorld" class = "com.tutorialspoint.HelloWorld">
      <property name = "message" value = "Hello World!"/>
   </bean>

   <bean id = "cStartEventHandler" class = "com.tutorialspoint.CStartEventHandler"/>
   <bean id = "cStopEventHandler" class = "com.tutorialspoint.CStopEventHandler"/>

</beans>
 ```

 ## Spring - JDBC Framework Overview <a id="JDBC"></a>

 The JDBC Template class executes SQL queries, updates statements, stores procedure calls, performs iteration over ResultSets, and extracts returned parameter values. It also catches JDBC exceptions and translates them to the generic, more informative, exception hierarchy defined in the org.springframework.dao package.

 A common practice when using the JDBC Template class is to configure a DataSource in your Spring configuration file, and then dependency-inject that shared DataSource bean into your DAO classes, and the JdbcTemplate is created in the setter for the DataSource.

 __DataSource__ to the JDBC Template so it can configure itself to get database access. You can configure the DataSource in the XML file with a piece of code as shown in the following code snippet:
 ```
 <bean id = "dataSource" 
   class = "org.springframework.jdbc.datasource.DriverManagerDataSource">
   <property name = "driverClassName" value = "com.mysql.jdbc.Driver"/>
   <property name = "url" value = "jdbc:mysql://localhost:3306/TEST"/>
   <property name = "username" value = "root"/>
   <property name = "password" value = "password"/>
</bean>
 ```

Query:
```
String SQL = "select count(*) from Student";
int rowCount = jdbcTemplateObject.queryForInt( SQL );
```

### Data Access Object (DAO)

Is used for database interaction. 
DAOs exist to provide a means to read and write data to the database and they should expose this functionality through an interface by which the rest of the application will access them.

StudentDAO.java:
```
package com.tutorialspoint;

import ...

public interface StudentDAO {
   public void setDataSource(DataSource ds);
   public void create(String name, Integer age);
   public Student getStudent(Integer id);
   public List<Student> listStudents();
   public void delete(Integer id);
   public void update(Integer id, Integer age);
}
```

Student.java:
```
package com.tutorialspoint;

public class Student {
   private Integer age;
   private String name;
   private Integer id;

   public void setAge(Integer age) {
      this.age = age;
   }
   public Integer getAge() {
      return age;
   }
   public void setName(String name) {
      this.name = name;
   }
   public String getName() {
      return name;
   }
   public void setId(Integer id) {
      this.id = id;
   }
   public Integer getId() {
      return id;
   }
}
```

StudentMapper.java:
```
package com.tutorialspoint;

import ...

public class StudentMapper implements RowMapper<Student> {
   public Student mapRow(ResultSet rs, int rowNum) throws SQLException {
      Student student = new Student();
      student.setId(rs.getInt("id"));
      student.setName(rs.getString("name"));
      student.setAge(rs.getInt("age"));
      
      return student;
   }
}
```

StudentJDBCTemplate.java:
```
public class StudentJDBCTemplate implements StudentDAO {
   private DataSource dataSource;
   private JdbcTemplate jdbcTemplateObject;
   
   public void setDataSource(DataSource dataSource) {
      this.dataSource = dataSource;
      this.jdbcTemplateObject = new JdbcTemplate(dataSource);
   }
   public void create(String name, Integer age) {
      String SQL = "insert into Student (name, age) values (?, ?)";
      jdbcTemplateObject.update( SQL, name, age);
      System.out.println("Created Record Name = " + name + " Age = " + age);
      return;
   }
   public Student getStudent(Integer id) {
      String SQL = "select * from Student where id = ?";
      Student student = jdbcTemplateObject.queryForObject(SQL, 
         new Object[]{id}, new StudentMapper());
      
      return student;
   }
   public List<Student> listStudents() {
      String SQL = "select * from Student";
      List <Student> students = jdbcTemplateObject.query(SQL, new StudentMapper());
      return students;
   }
   public void delete(Integer id) {
      String SQL = "delete from Student where id = ?";
      jdbcTemplateObject.update(SQL, id);
      System.out.println("Deleted Record with ID = " + id );
      return;
   }
   public void update(Integer id, Integer age){
      String SQL = "update Student set age = ? where id = ?";
      jdbcTemplateObject.update(SQL, age, id);
      System.out.println("Updated Record with ID = " + id );
      return;
   }
}
```
MainApp.java:
```
public class MainApp {
   public static void main(String[] args) {
      ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");

      StudentJDBCTemplate studentJDBCTemplate = 
         (StudentJDBCTemplate)context.getBean("studentJDBCTemplate");
      
      System.out.println("------Records Creation--------" );
      studentJDBCTemplate.create("Zara", 11);
      studentJDBCTemplate.create("Nuha", 2);
      studentJDBCTemplate.create("Ayan", 15);

      System.out.println("------Listing Multiple Records--------" );
      List<Student> students = studentJDBCTemplate.listStudents();
      
      for (Student record : students) {
         System.out.print("ID : " + record.getId() );
         System.out.print(", Name : " + record.getName() );
         System.out.println(", Age : " + record.getAge());
      }

      System.out.println("----Updating Record with ID = 2 -----" );
      studentJDBCTemplate.update(2, 20);

      System.out.println("----Listing Record with ID = 2 -----" );
      Student student = studentJDBCTemplate.getStudent(2);
      System.out.print("ID : " + student.getId() );
      System.out.print(", Name : " + student.getName() );
      System.out.println(", Age : " + student.getAge());
   }
}
```

Beans.xml:
```
<?xml version = "1.0" encoding = "UTF-8"?>
<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance" 
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd ">

   <!-- Initialization for data source -->
   <bean id="dataSource" 
      class = "org.springframework.jdbc.datasource.DriverManagerDataSource">
      <property name = "driverClassName" value = "com.mysql.jdbc.Driver"/>
      <property name = "url" value = "jdbc:mysql://localhost:3306/TEST"/>
      <property name = "username" value = "root"/>
      <property name = "password" value = "password"/>
   </bean>

   <!-- Definition for studentJDBCTemplate bean -->
   <bean id = "studentJDBCTemplate" 
      class = "com.tutorialspoint.StudentJDBCTemplate">
      <property name = "dataSource" ref = "dataSource" />    
   </bean>
      
</beans>
```

## Spring - MVC Framework <a id="MVC"></a>

The Spring Web MVC framework provides Model-View-Controller (MVC) architecture and ready components that can be used to develop flexible and loosely coupled web applications. The MVC pattern results in separating the different aspects of the application (input logic, business logic, and UI logic), while providing a loose coupling between these elements.

The __Model__ encapsulates the application data and in general they will consist of POJO.

The __View__ is responsible for rendering the model data and in general it generates HTML output that the client's browser can interpret.

The __Controller__ is responsible for processing user requests and building an appropriate model and passes it to the view for rendering.


### DispatcherServlet

The Spring Web model-view-controller (MVC) framework is designed around a DispatcherServlet that handles all the HTTP requests and responses.

- After receiving an HTTP request, DispatcherServlet consults the HandlerMapping to call the appropriate Controller.
- The Controller takes the request and calls the appropriate service methods based on used GET or POST method. The service method will set model data based on defined business logic and returns view name to the DispatcherServlet.
- The DispatcherServlet will take help from ViewResolver to pickup the defined view for the request.
- Once view is finalized, The DispatcherServlet passes the model data to the view which is finally rendered on the browser.

### Configuration

You need to map requests that you want the DispatcherServlet to handle, by using a URL mapping in the web.xml file:
```
<web-app id = "WebApp_ID" version = "2.4"
   xmlns = "http://java.sun.com/xml/ns/j2ee" 
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://java.sun.com/xml/ns/j2ee 
   http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
    
   <display-name>Spring MVC Application</display-name>
   
   <servlet>
      <servlet-name>HelloWeb</servlet-name>
      <servlet-class>
         org.springframework.web.servlet.DispatcherServlet
      </servlet-class>
      <load-on-startup>1</load-on-startup>
   </servlet>

   <servlet-mapping>
      <servlet-name>HelloWeb</servlet-name>
      <url-pattern>*.jsp</url-pattern>
   </servlet-mapping>

</web-app>
```

<servlet-mapping> tag indicates what URLs will be handled by which DispatcherServlet. Here all the HTTP requests ending with .jsp will be handled by the HelloWeb DispatcherServlet.

HelloWeb-servlet.xml:
```
<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:context = "http://www.springframework.org/schema/context"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans     
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
   http://www.springframework.org/schema/context 
   http://www.springframework.org/schema/context/spring-context-3.0.xsd">

   <context:component-scan base-package = "com.tutorialspoint" />

   <bean class = "org.springframework.web.servlet.view.InternalResourceViewResolver">
      <property name = "prefix" value = "/WEB-INF/jsp/" />
      <property name = "suffix" value = ".jsp" />
   </bean>

</beans>
```

The [servlet-name]-servlet.xml file will be used to create the beans defined, overriding the definitions of any beans defined with the same name in the global scope.

### Controller

The DispatcherServlet delegates the request to the controllers to execute the functionality specific to it. The @Controller annotation indicates that a particular class serves the role of a controller. The @RequestMapping annotation is used to map a URL to either an entire class or a particular handler method.

```
@Controller
@RequestMapping("/hello")
public class HelloController { 
   @RequestMapping(method = RequestMethod.GET)
   public String printHello(ModelMap model) {
      model.addAttribute("message", "Hello Spring MVC Framework!");
      return "hello";
   }
}
```

### JSP Views

Spring MVC supports many types of views for different presentation technologies. These include - JSPs, HTML, PDF etc.

simple hello view in /WEB-INF/hello/hello.jsp:
```
<html>
   <head>
      <title>Hello Spring MVC</title>
   </head>
   
   <body>
      <h2>${message}</h2>
   </body>
</html>
```

# Spring Boot <a id="Boot"></a>

Spring is widely used for creating scalable applications, but disadvantage of spring projects is that __configuration is really time-consuming__ and can be a bit overwhelming for the new developers.

Spring Boot is built on the top of the spring and contains all the features of spring. It enables the developers to directly focus on the logic instead of struggling with the configuration and set up. __Spring Boot is a microservice-based framework__ and making a production-ready application in it takes very less time. Prerequisite for Spring Boot is the basic knowledge Spring framework. 

Features:
- It allows to avoid heavy configuration of XML. In spring boot __everything is auto-configured.__
- It provides __easy maintenance and creation of REST end points__: Creating a REST API is very easy in Spring Boot. Just the annotation @RestController and @RequestMapping(/endPoint) over the controller class does the work.
- It includes __embedded Tomcat-server__: Unlike Spring MVC project where we have to manually add and install the tomcat server, Spring Boot comes with an embedded Tomcat server, so that the applications can be hosted on it. 
- Deployment is very easy, war and jar file can be easily deployed in the tomcat server:war or jar files can be directly deployed on the Tomcat Server
- Microservice Based Architecture: __Microservice__, as the name suggests is the name given to a module/service which focuses on a single type of feature, exposing an API

## Spring Boot Architecture <a id="BootArchitecture"></a>