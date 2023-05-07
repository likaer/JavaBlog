

# [Spring Boot(2.2.6)](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/)

## 什么是Spring Boot

Spring Boot 可帮助您创建可以运行的独立的、生产级的基于 Spring 的应用程序。我们对 Spring 平台和第三方库采取了统一的方式，以便您可以轻松上手。大多数 Spring Boot 应用程序只需要很少的 Spring 配置。

您可以使用 Spring Boot 创建可以使用`java -jar`或更传统的 war 部署启动的 Java 应用程序。我们还提供了一个运行“spring 脚本”的命令行工具。

| Build Tool | Version |
| :--------- | :------ |
| Maven      | 3.3+    |
| Java       | 1.8+    |

## 实战入门

### 1. pom依赖引用

**1.1 使用parent作为依赖**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.6.RELEASE</version>
    </parent>

    <description/>
    <developers>
        <developer/>
    </developers>
    <licenses>
        <license/>
    </licenses>
    <scm>
        <url/>
    </scm>
    <url/>

    <!-- Additional lines to be added here... -->

</project>
```

**1.2 不使用parent依赖**

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.2.6.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**1.3 打包插件（将项目打包为可执行的jar包）**

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

**1.4 Spring Boot Starter（启动器）**

注：spring-boot-starter-parent本身不提供依赖，但是通过``dependencyManagement``声明了默认版本

| Name                           | Description                                                  |
| ------------------------------ | ------------------------------------------------------------ |
| `spring-boot-starter`          | The core Spring Boot starter, including auto-configuration support, logging and YAML. |
| `spring-boot-starter-actuator` | Production ready features to help you monitor and manage your application. |
| `spring-boot-starter-redis`    | Support for the REDIS key-value data store, including `spring-redis`. |
| `spring-boot-starter-jdbc`     | Support for JDBC databases.                                  |
| `spring-boot-starter-security` | Support for `spring-security`.                               |
| `spring-boot-starter-test`     | Support for common test dependencies, including JUnit, Hamcrest and Mockito along with the `spring-test` module. |
| `spring-boot-starter-web`      | Support for full-stack web development, including Tomcat and `spring-webmvc`. |
| `spring-boot-starter-aop`      | Support for aspect-oriented programming including `spring-aop` and AspectJ. |

### 2. 关于配置

**2.1 Spring加载配置文件先后顺序**

`SpringApplication` 默认从以下位置加载 `application.properties` 文件，并把它们存放到 Spring环境变量里面

1. 项目根目录的 `/config`文件夹
2. 项目根目录
3. 资源文件夹下的 `/config` 文件夹
4. 资源文件夹路径

也可以通过``spring.config.location``修改配置文件的名字

```
$ java -jar myproject.jar --spring.config.name=myproject
```

```
$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties
```

**2.2 特定配置文件**

除了`application.properties`文件之外，还可以使用命名约定来定义特定于配置文件的属性`application-{profile}.properties`。

通过以下命令可以加载特定的配置文件

```bash
$ java -jar -Dspring.profiles.active=production demo-0.0.1-SNAPSHOT.jar
```

**2.3 Yaml**

```yaml
environments:
    dev:
        url: http://dev.bar.com
        name: Developer Setup
    prod:
        url: http://foo.bar.com
        name: My Cool App
```

```properties
environments.dev.url=http://dev.bar.com
environments.dev.name=Developer Setup
environments.prod.url=http://foo.bar.com
environments.prod.name=My Cool App
```



**2.4 不同环境下的不同配置支持**

```
server:
    address: 192.168.1.100
---
spring:
    profiles: development
server:
    address: 127.0.0.1
---
spring:
    profiles: production
server:
    address: 192.168.1.120
```

### 3. 主程序

**3.1 SpringBootApplication注解**

``@SpringBootApplication``等价于``@Configuration`` ``@EnableAutoConfiguration`` ``@ComponentScan``

```java
package com.example.myproject;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

**3.2 component注解**

spring有四种Component注解，标注了这4种注解的类，会被自动注册为Spring Bean

- ``@Controller ``标注Spring Web Controller
- ``@Service``标注Service
- ``@Repository``标注持久层
- ``@Component``不属于上面这3种，统一使用这个注解

**3.3 启动主程序**

方法1

```shell
maven clean package -DskipTests
java -jar /your/jar/path/myproject.jar
```

方法2

	mvn spring-boot:run

### 4. Spring的核心概念（IOC与AOP）

#### 4.1 [IOC](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans)

**Inversion of Control(IOC) -----dependency injection (DI)**

- Spring把类的实例化配置称为``Bean``

  ```java
  @Configuration
  public class FactoryMethodComponent {
  
      @Bean
      public TestBean prototypeInstance(InjectionPoint injectionPoint) {
          return new TestBean("prototypeInstance for " + injectionPoint.getMember());
      }
  }
  ```

  

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd">
  
      <bean id="..." class="...">
          <!-- collaborators and configuration for this bean go here -->
      </bean>
  
      <bean id="..." class="...">
          <!-- collaborators and configuration for this bean go here -->
      </bean>
  
      <!-- more bean definitions go here -->
  
  </beans>
  ```

- Spring使用``ApplicationContext``维护所有的Bean，所以``ApplicationContext``就是``Bean``的容器

  ```JAVA
  // create and configure beans
  ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
  
  // retrieve configured instance
  PetStoreService service = context.getBean("petStore", PetStoreService.class);
  
  // use configured instance
  List<String> userList = service.getUsernameList();
  ```

- Bean必须了解的概念

  | Property                 | Explained in…                                                |
  | :----------------------- | :----------------------------------------------------------- |
  | Class                    | [Instantiating Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class) |
  | Scope                    | [Bean Scopes](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes) |
  | Constructor arguments    | [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
  | Properties               | [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
  | Autowiring mode          | [Autowiring Collaborators](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire) |
  | Lazy initialization mode | [Lazy-initialized Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lazy-init) |
  | Initialization method    | [Initialization Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean) |
  | Destruction method       | [Destruction Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean) |



1. Bean的Scope，默认是单例模式;

2. @Autowired

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    private MovieCatalog movieCatalog;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

3. @Resource

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder") 
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

4. @Value

   ```java
   //@Component
   public class MovieRecommender {
   
       private final String catalog;
   
       public MovieRecommender(@Value("${catalog.name}") String catalog) {
           this.catalog = catalog;
       }
   }
   ```

5. @Component、@Repository、@Service、@Controller

6. 自动注册

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig {
    // ...
}
```



#### 4.2 [AOP](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop)

```
				<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
```

```Java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyTransaction {
}
```

```
public interface MyService {

    void method1();

    void method2();
    
}

```

```java
@Component
@Aspect
public class AspectMyTransaction {


    @Pointcut("@annotation(MyTransaction)")
    private void myPointCut(){

    }

    @Before("myPointCut()")
    public void before(){
        System.out.println("Start my transaction ");
    }

    @After("myPointCut()")
    public void after(){
        System.out.println("end my transaction");
    }
}

```

AOP是一种思想，这种思想在传递Spring框架一种核心理念非侵入性，使用Aspect只是其中的一种方式

### 5. [Spring Web](https://docs.spring.io/spring-framework/docs/5.3.13/reference/html/index.html)

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Spring Web本质上也是基于Servlet实现，Spring的所有请求都是从DispatcherServlet类开始

```
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/app-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>

</web-app>
```

**5.1 @Controller @RestController**

RestController=Controller+ResponseBody

**5.2 @RequestMapping @GetMapping @PostMapping**



**5.3 @RequestParam @RequestBody**



**5.4 Spring Web切面**

- HandlerInterceptor
  - `preHandle(..)`: Before the actual handler is run
  - `postHandle(..)`: After the handler is run
  - `afterCompletion(..)`: After the complete request has finished
- HandlerExceptionResolver



6. MyBatis

   https://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/

   ```
   <dependency>
       <groupId>org.mybatis.spring.boot</groupId>
       <artifactId>mybatis-spring-boot-starter</artifactId>
       <version>2.2.2</version>
   </dependency>
   ```

   

7. Spring Boot Test

   https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing

   https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications

8. @Transactional

   https://docs.spring.io/spring-framework/docs/4.2.x/spring-framework-reference/html/transaction.html

   ```
   propagation
   isolation
   rollbackFor
   noRollbackFor
   ```

   事务不生效的原因

   - 数据库不支持，MySQL-InnoDB
   - 注解对象没有交给Spring管理
   - 注解方法是private
   - 类内部调用


9. RocketMQ

   https://rocketmq.apache.org/docs/quick-start/

   https://help.aliyun.com/document_detail/114448.html

10. Nacos

    https://nacos.io/zh-cn/docs/what-is-nacos.html
