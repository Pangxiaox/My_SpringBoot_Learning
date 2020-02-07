# Spring Boot学习笔记

### 1.Spring Boot项目中的parent

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
```

当我们创建一个Spring Boot工程时，可以继承自一个 `spring-boot-starter-parent`, 也可以不继承它。

①继承的情况

parent的基本功能有：

- 定义了Java编译版本为1.8
- 使用UTF-8格式编码
- 继承自 `spring-boot-dependencies`， 这个里面定义了依赖的版本。正是因为继承了这个依赖，我们在写依赖时才不需要写版本号
- 执行打包操作的配置
- 自动化的资源过滤
- 自动化的插件配置
- 针对 application.properties和application.yml的资源过滤，包括通过profile定义不同环境的配置文件，例如 application-dev.properties和application-dev.yml

②不继承的情况

如果公司有自己定义的parent，需要继承自公司内部的parent时，一个简单的办法就是自行定义dependencyManagement节点，然后在里边定义好版本号，这样接下来引用依赖时也不用写版本号，但关于打包的插件、编译的JDK版本、文件的编码格式等等配置都需要自己去配置。

```xml
<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.1.4.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```



### 2.Spring Boot数据持久化操作的几种方式

- 使用JDBCTemplate，是由Spring框架提供的
- 使用JPA（Java Persistence API)，在Springboot中JPA依靠Hibernate才得以实现
- 使用MyBatis，这是基于SqlSessionFactory构建的框架



### 3.Spring Boot整合MyBatis数据持久化操作

##### 3.1 注解方式

根据实体类中的属性在SQL中需要自己建库建表

实体类User

```java
package com.example.combine.bean;

public class User {
    private int id;
    private String name;
    private int age;
    private double money;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public double getMoney() {
        return money;
    }

    public void setMoney(double money) {
        this.money = money;
    }
}
```

服务层UserService

```java
package com.example.combine.service;

import com.example.combine.bean.User;
import com.example.combine.dao.UserDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserService {
    @Autowired
    private UserDao userDao;
    
    public List<User> selectUserByName(String name) {
        return userDao.findUserByName(name);
    }


    public List<User> selectAllUser() {
        return userDao.findAllUser();
    }
    
    public void insertService(String name,int age,double money) {
        /*
        User user = new User();
        user.setName(name);
        user.setAge(age);
        user.setMoney(money);
        userDao.insertUser(user);
         */
        userDao.insertUser(name,age,money);
    }

    public void deleteService(int id) {
        userDao.deleteUser(id);
    }
    
    public void changemoney(String name, int age, double money, int id) {
        userDao.updateUser(name,age,money,id);
    }
}
```

数据访问层UserDao

```java
package com.example.combine.dao;

import com.example.combine.bean.User;
import org.apache.ibatis.annotations.*;

import java.util.List;

@Mapper
public interface UserDao {
    @Select("SELECT * FROM user WHERE name = #{name}")
    List<User> findUserByName(@Param("name") String name);
    
    @Select("SELECT * FROM user")
    List<User> findAllUser();
    
    @Insert("INSERT INTO user(name, age,money) VALUES(#{name}, #{age}, #{money})")
    /*
    void insertUser(User user);
     */
    void insertUser(@Param("name")String name,@Param("age")int age,@Param("money")double money);
    
    @Update("UPDATE  user SET name = #{name},age = #{age},money= #{money} WHERE id = #{id}")
    void updateUser(@Param("name") String name, @Param("age") Integer age, @Param("money") Double money,
                    @Param("id") int id);
    
    @Delete("DELETE from user WHERE id = #{id}")
    void deleteUser(@Param("id") int id);
}
```

控制层UserController

```java
package com.example.combine.controller;

import com.example.combine.bean.User;
import com.example.combine.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/user")
public class UserController {
    @Autowired
    private UserService userService;

    @RequestMapping("/query")
    public List<User> testQuery(String name)
    {
        return userService.selectUserByName(name);
    }

    @RequestMapping("/findall")
    public List<User> findAll()
    {
        return userService.selectAllUser();
    }

    @RequestMapping("/insert")
    public List<User> testInsert(String name,int age,double money) {
        userService.insertService(name,age,money);
        return userService.selectAllUser();
    }
    
    @RequestMapping("/changemoney")
    public List<User> testchangemoney(String name,int age,double money,int id) {
        userService.changemoney(name,age,money,id);
        return userService.selectAllUser();
    }

    @RequestMapping("/delete")
    public String testDelete(int id) {
        userService.deleteService(id);
        return "OK";
    }
}
```

resources文件夹下的配置文件application.properties

```properties
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/combine?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2B8
spring.datasource.username=root
spring.datasource.password=xxxxxx(Your-Pwd-Here)
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

CombineApplication

```java
package com.example.combine;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@MapperScan("com.example.combine.dao")
@SpringBootApplication
public class CombineApplication {
    public static void main(String[] args) {
        SpringApplication.run(CombineApplication.class, args);
    }
}

```

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>combine</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>combine</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.1</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

接下来使用Postman工具测试，所有测试均已通过

- 更新操作：http://localhost:8080/user/changemoney?name=pxx&age=36&money=520.0&id=9
- 查询所有用户：http://localhost:8080/user/findall
- 按名字查询用户：http://localhost:8080/user/query?name=test
- 删除用户：http://localhost:8080/user/delete?id=7
- 新增用户：http://localhost:8080/user/insert?name=abc&age=25&money=100.0



##### 3.2 XML方式

实体类User

```java
package com.example.combine.bean;

public class User {
    private int id;
    private String name;
    private int age;
    private double money;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public double getMoney() {
        return money;
    }

    public void setMoney(double money) {
        this.money = money;
    }
}
```

服务层UserService

```java
package com.example.combine.service;

import com.example.combine.bean.User;
import com.example.combine.dao.UserMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserService {
    @Autowired
    private UserMapper userDao;

    public List<User> selectUserByName(String name) {
        return userDao.findUserByName(name);
    }

    public List<User> selectAllUser() {
        return userDao.findAllUser();
    }

    public void insertService(String name,int age,double money) {
        /*
        User user = new User();
        user.setName(name);
        user.setAge(age);
        user.setMoney(money);
        userDao.insertUser(user);
         */
        userDao.insertUser(name,age,money);
    }
    
    public void deleteService(int id) {
        userDao.deleteUser(id);
    }

    public void changemoney(String name, int age, double money, int id) {
        userDao.updateUser(name,age,money,id);
    }
}
```

数据访问层Dao 接口映射文件UserMapper

```java
package com.example.combine.dao;

import com.example.combine.bean.User;
import org.apache.ibatis.annotations.Mapper;

import java.util.List;

@Mapper
public interface UserMapper {

    List<User> findUserByName(String name);

    List<User> findAllUser();

    void insertUser(String name,int age,double money);

    void updateUser(String name,int age,double money,int id);

    void deleteUser(int id);
}
```

控制器层Controller

```java
package com.example.combine.controller;

import com.example.combine.bean.User;
import com.example.combine.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/user")
public class UserController {
    @Autowired
    private UserService userService;

    @RequestMapping("/query")
    public List<User> testQuery(String name)
    {
        return userService.selectUserByName(name);
    }

    @RequestMapping("/findall")
    public List<User> findAll()
    {
        return userService.selectAllUser();
    }

    @RequestMapping("/insert")
    public List<User> testInsert(String name,int age,double money) {
        userService.insertService(name,age,money);
        return userService.selectAllUser();
    }

    @RequestMapping("/changemoney")
    public List<User> testchangemoney(String name,int age,double money,int id) {
        userService.changemoney(name,age,money,id);
        return userService.selectAllUser();
    }

    @RequestMapping("/delete")
    public String testDelete(int id) {
        userService.deleteService(id);
        return "OK";
    }
}
```

启动类CombineApplication

```java
package com.example.combine;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@MapperScan("com.example.combine.dao")
@SpringBootApplication
public class CombineApplication {
    public static void main(String[] args) {
        SpringApplication.run(CombineApplication.class, args);
    }
}
```

配置文件application.properties

```properties
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/combine?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2B8
spring.datasource.username=root
spring.datasource.password=xxxxxx(Your-Pwd-Here)
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
mybatis.mapper-locations=classpath:mapper/*.xml
```

resources文件夹下的mapper文件夹下的UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.combine.dao.UserMapper">
    <resultMap id="BaseResultMap" type="com.example.combine.bean.User">
        <id column="id" property="id" jdbcType="INTEGER" />
        <result column="name" property="name" jdbcType="VARCHAR" />
        <result column="age" property="age" jdbcType="INTEGER" />
        <result column="money" property="money" jdbcType="DOUBLE" />
    </resultMap>
    <select id="findUserByName" resultType="com.example.combine.bean.User">
        select * from user where name=#{name}
    </select>
    <select id="findAllUser" resultType="com.example.combine.bean.User">
        select * from user;
    </select>
    <insert id="insertUser">
        insert into user (name,age,money) values (#{name},#{age},#{money});
    </insert>
    <update id="updateUser">
        update user set name=#{name},age=#{age},money=#{money} where id=#{id}
    </update>
    <delete id="deleteUser">
        delete from user where id=#{id}
    </delete>
</mapper>
```

使用Postman工具测试接口的方法与3.1中的相同，均已测试通过



🔺注解方式相对比较简单，但SQL语句也耦合到java代码当中，适用于简单的查询。XML方式对于复杂的查询有较好的支持。



### 4. Spring Boot整合Thymeleaf（视图层）

Thymeleaf是新一代Java模板引擎，类似于Velocity、FreeMarker等传统Java模板引擎，它支持HTML原型。Thymeleaf除了展示基本的HTML进行页面渲染外也可以作为一个HTML片段进行渲染，例如在做邮件发送时，可使用Thymeleaf作为邮件发送模板。

com.example.combine.User

```java
package com.example.combine;

public class User {
    private Long id;
    private String name;
    private String address;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
```

com.example.combine.IndexController

```java
package com.example.combine;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

import java.util.ArrayList;
import java.util.List;

@Controller
public class IndexController {
    @GetMapping("/index")
    public String index(Model model) {
        List<User> users = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            User u = new User();
            u.setId((long) i);
            u.setName("javaboy:" + i);
            u.setAddress("深圳:" + i);
            users.add(u);
        }
        model.addAttribute("users", users);
        return "index";
    }
}
```

com.example.combine.CombineApplication

```java
package com.example.combine;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;

@SpringBootApplication(exclude= {DataSourceAutoConfiguration.class})
public class CombineApplication {
    public static void main(String[] args) {
        SpringApplication.run(CombineApplication.class, args);
    }
}
```

resources/templates/index.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<table border="1">
    <tr>
        <td>编号</td>
        <td>用户名</td>
        <td>地址</td>
    </tr>
    <tr th:each="user : ${users}">
        <td th:text="${user.id}"></td>
        <td th:text="${user.name}"></td>
        <td th:text="${user.address}"></td>
    </tr>
</table>
</body>
</html>
```

访问 127.0.0.1:8080/index 展示下图：

**放图Springboot-thymeleaf**



另外，Thymeleaf支持再js中直接获取Model中的变量：

```java
@Controller
public class IndexController {
    @GetMapping("/index")
    public String index(Model model) {
        model.addAttribute("username", "李四");
        return "index";
    }
}
```

在页面模板中，可以直接在js中获取到这个变量：

```js
<script th:inline="javascript">
    var username = [[${username}]];
    console.log(username)
</script>
```



### 6. Spring Boot整合FreeMarker（视图层）

FreeMarker是一个相当老牌的开源免费模板引擎。通过FreeMarker模板，可以将数据渲染成HTML网页、电子邮件、配置文件、源代码等。Freemarker模板后缀为.ftl（Freemarker Template Language)。

CombineApplication、IndexController、User与上面第5个分点一致

resources/templates/index.ftl

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<table border="1">
    <tr>
        <td>用户编号</td>
        <td>用户名称</td>
        <td>用户地址</td>
    </tr>
    <#list users as user>
        <tr>
            <td>${user.id}</td>
            <td>${user.name}</td>
            <td>${user.address}</td>
        </tr>
    </#list>
</table>
</body>
</html>
```

resources/application.properties

```properties
spring.freemarker.tempalte-loader-path=classpath:/templates
spring.freemarker.cache=false
spring.freemarker.charset=UTF-8
spring.freemarker.check-template-location=true
spring.freemarker.content-type=text/html
spring.freemarker.expose-request-attributes=true
spring.freemarker.expose-session-attributes=true
spring.freemarker.request-context-attribute=request
spring.freemarker.suffix=.ftl
```

访问 localhost:8080/index 展示如图：

**放图Springboot-freemarker**



### 7. Spring Boot静态资源配置

**SSM中的配置**

使用Spring MVC框架时，静态资源会被拦截，需要添加额外配置

```xml
<mvc:resources mapping="/js/**" location="/js/"/>
<mvc:resources mapping="/css/**" location="/css/"/>
<mvc:resources mapping="/html/**" location="/html/"/>
```

**Spring Boot中的配置**

在Spring Boot中，有5个位置可以存放静态资源：

- `classpath:/META-INF/resources/`

- `classpath:/resources/`
- `classpath:/static/`
- `classpath:/public/`
- `/`

🔺在Spring Boot项目中默认没有webapp这个目录，我们可以自己添加（例如需要使用JSP），第5个 `/`表示webapp目录中的静态资源也不会被拦截。

⭐如果同一个文件分别出现在五个目录下，优先级也是按照上面列出顺序

一般情况下系统默认创建了 `classpath:/static/`，我们只需把静态资源放到这个目录下就可以了，不需要额外去创建其他静态资源目录。

例子：在 `classpath/static/`目录下放了一张名为1.png的图片，那么访问路径是：

http://localhost:8080/1.jpg

无需要加上static

**自定义配置**

可以自定义静态资源位置和映射，可以通过application.properties或Java代码实现

- application.properties

```properties
spring.resources.static-locations=classpath:/
spring.mvc.static-path-pattern=/**
```

第一行配置：定义资源位置。第二行配置：定义请求URL规则

如上表示可以将静态资源放在resources目录下的任意地方，这时候如果想要访问该目录下的静态资源1.png，则访问路径变为：

http://localhost:8080/static/1.png

static不能忽略

- Java代码

```java
@Configuration
public class WebMVCConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**").addResourceLocations("classpath:/aaa/");
    }
}
```



### 8. @ControllerAdvice三种使用场景

这是Spring MVC提供的功能，在Spring Boot中可直接使用

#### 全局异常处理

```java
@ControllerAdvice
public class MyGlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    public ModelAndView customException(Exception e) {
        ModelAndView mv = new ModelAndView();
        mv.addObject("message", e.getMessage());
        mv.setViewName("myerror");
        return mv;
    }
}
```

可定义多个方法，不同方法处理不同异常

也可以像上面一样在一个方法里处理所有异常信息

@ExceptionHandler注解指明异常的处理类型，如果这里指定为NullPointerException，则数组越界异常不会进入这个方法。

#### 全局数据绑定

数据初始化，将一些公共数据定义在添加了@ControllerAdvice注解的类中，这样在每一个Controller接口中都可以访问到这些数据。

定义全局数据：

```java
@ControllerAdvice
public class MyGlobalExceptionHandler {
    @ModelAttribute(name = "md")
    public Map<String,Object> mydata() {
        HashMap<String, Object> map = new HashMap<>();
        map.put("age", 99);
        map.put("gender", "男");
        return map;
    }
}
```

@ModelAttribute注解标记该方法返回数据是全局数据，默认情况下这个全局数据的key就是返回的变量名，value就是方法返回值，也可以通过注解的name属性重新指定key

在任何一格Controller接口中都可以获取到全局数据：

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello(Model model) {
        Map<String, Object> map = model.asMap();
        System.out.println(map);
        int i = 1 / 0;
        return "hello controller advice";
    }
}
```

效果如图：

**放图Springboot-8th**



#### 全局数据预处理

两个实体类：Book和Author

```java
public class Book {
    private String name;
    private Long price;
    //getter/setter
}
public class Author {
    private String name;
    private Integer age;
    //getter/setter
}
```

现在来定义一个添加数据接口：

```java
@PostMapping("/book")
public void addBook(Book book, Author author) {
    System.out.println(book);
    System.out.println(author);
}
```

因为两个实体类都有一个name属性，从前端传递时无法区分，通过@ControllerAdvice注解全局预处理功能解决：

①给接口中变量取别名：

```java
@PostMapping("/book")
public void addBook(@ModelAttribute("b") Book book, @ModelAttribute("a") Author author) {
    System.out.println(book);
    System.out.println(author);
}
```

②请求数据预处理。在@ControllerAdvice标记的类中添加如下代码：

```java
@InitBinder("b")
public void b(WebDataBinder binder) {
    binder.setFieldDefaultPrefix("b.");
}
@InitBinder("a")
public void a(WebDataBinder binder) {
    binder.setFieldDefaultPrefix("a.");
}
```

@InitBinder("b")注解表示该方法用于处理和Book相关参数，表明请求参数要有b前缀。

③发送请求

给不同对象添加不同前缀实现参数区分

http://127.0.0.1:8080/book?b.name=书本&b.price=98&a.name=作者&a.age=60



### 9. Spring Boot全局异常处理

默认情况下异常页面如图：

**放图index**

出现这个页面原因是开发者没有明确提供一格/error路径。Spring Boot处理异常时，是当所有条件都不满足时才去找/error路径。

下面展示如何自定义error页面：

#### 静态异常页面

使用HTTP响应码命名页面：如404.html、405.html、500.html等

或者直接定义一个4xx.html，表示400-499状态都显示这个异常页面；5xx.html表示500-599状态显示这个异常页面。

如果项目抛出500请求错误，就会展示500.html这个页面，其他同理。

🔺如果既有5xx.html和500.html，则发生500异常时优先展示500.html页面

相关html页面默认存放在 `classpath:/static/error/`

下图展示抛出405异常时，使用了4xx.html页面的情况：

**放图index-405**



#### 动态异常页面

可采用的页面模板有jsp、freemarker、thymeleaf。也支持404.html或4xx.html。

**动态页面模板，不需要开发者自己去定义控制器，直接定义异常页面**

相关html页面默认存放在 `classpath:/templates/error/`

5xx.html页面：(thymeleaf模板)

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>5xx</h1>
<table border="1">
    <tr>
        <td>path</td>
        <td th:text="${path}"></td>
    </tr>
    <tr>
        <td>error</td>
        <td th:text="${error}"></td>
    </tr>
    <tr>
        <td>message</td>
        <td th:text="${message}"></td>
    </tr>
    <tr>
        <td>timestamp</td>
        <td th:text="${timestamp}"></td>
    </tr>
    <tr>
        <td>status</td>
        <td th:text="${status}"></td>
    </tr>
</table>
</body>
</html>
```

抛出500异常时展示如图：

**放图500-error**

🔺如果动态页面和静态页面同时定义了异常处理页面，则默认使用动态页面。

⭐完整的错误页面查找方式：发生了500错误👉查找动态500.html页面👉查找静态500.html页面👉查找动态5xx.html👉查找静态5xx.html

#### 自定义异常数据

如果开发者没有自己提供一格ErrorAttributes实例，Spring Boot自动提供一个，就是DefaultErrorAttributes。

开发者自定义ErrorAttributes两种方式：

- 直接实现ErrorAttributes接口
- 继承DefaultErrorAttributes（推荐）

```java
@Component
public class MyErrorAttributes  extends DefaultErrorAttributes {
    @Override
    public Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
        Map<String, Object> map = super.getErrorAttributes(webRequest, includeStackTrace);
        if ((Integer)map.get("status") == 500) {
            map.put("message", "服务器内部错误!");
        }
        return map;
    }
}
```

⭐定义好的ErrorAttributes一定要注册成一格Bean，这样就不会使用默认的DefaultErrorAttributes了。

效果如图：

**放图500-error。。。。**



#### 自定义异常视图

由于DefaultErrorViewResolver是在ErrorMvcAutoConfiguration类中提供的实例。当开发者提供了自己的ErrorViewResolver实例后，默认配置的DefaultErrorViewResolver失效。

下面提供一格ErrorViewResolver实例：

```java
@Component
public class MyErrorViewResolver extends DefaultErrorViewResolver {
    public MyErrorViewResolver(ApplicationContext applicationContext, ResourceProperties resourceProperties) {
        super(applicationContext, resourceProperties);
    }
    @Override
    public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) {
        return new ModelAndView("/aaa/1", model);
    }
}
```

开发者也可以在这里定义异常数据（直接在resolveErrorView方法重新定义一个model。将参数中的model数据拷贝过去修改，参数中的model类型为UnmodifiableMap，即不可以直接修改）而不需要自定义MyErrorAttributes。

相关html页面在 `resources/templates/aaa/1.html`

效果如下图：

**放图NewErrorPage**



### 10. CORS解决跨域问题

同源策略：浏览器最核心最基本的安全功能，指协议、域名以及端口要相同。

传统的跨域方案：JSONP，局限性在于只支持GET请求，不支持其他类型请求。

CORS（Cross-origin resource sharing，跨域资源共享）是一个W3C标准，提供了Web服务从不同网域传来沙盒脚本的方法，以避开浏览器同源策略。

创建两个Spring Boot项目：第一个命名为provider提供服务，配置端口8080，第二个命名为consumer消费服务，配置端口为8081.

在provider接口提供两个hello接口，一个get、一个post：

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
    @PostMapping("/hello")
    public String hello2() {
        return "post hello";
    }
}
```

在consumer的resources/static目录下创建一格html文件，发生一格简单ajax请求：

```html
<div id="app"></div>
<input type="button" onclick="btnClick()" value="get_button">
<input type="button" onclick="btnClick2()" value="post_button">
<script>
    function btnClick() {
        $.get('http://localhost:8080/hello', function (msg) {
            $("#app").html(msg);
        });
    }

    function btnClick2() {
        $.post('http://localhost:8080/hello', function (msg) {
            $("#app").html(msg);
        });
    }
</script>
```

分别启动两项目，发送请求按钮，浏览器控制台如下：

`Access to XMLHttpRequest at 'http://localhost:8080/hello' from origin 'http://localhost:8081' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.`

受到了同源策略限制

**使用CORS：通过@CrossOrigin注解配置某一个方法接受某一个域的请求**

```java
@RestController
public class HelloController {
    @CrossOrigin(value = "http://localhost:8081")
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

    @CrossOrigin(value = "http://localhost:8081")
    @PostMapping("/hello")
    public String hello2() {
        return "post hello";
    }
}
```

这样就可以接受请求了。

🍉每一个方法上都去加@CrossOrigin注解比较麻烦。通过全局配置一次性解决问题：

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
        .allowedOrigins("http://localhost:8081")
        .allowedMethods("*")
        .allowedHeaders("*");
    }
}
```

/** 表示本应用所有方法都会处理跨域请求

allowedMethods表示允许通过的请求数

allowedHeaders表示允许的请求头

🔺使用了CORS之后也有潜在威胁：CSRF（Cross-site request forgery，跨站请求伪造），即挟制用户在当前已登录得Web应用程序上执行非本意得操作得攻击方法。

例子：

一家银行用来运行转账操作得URL：http://icbc.com/aa?bb=cc，一个攻击者可以在另一个网站上放置代码：`<img src="http://icbc.com/aa?bb=cc">`，如果用户访问了恶意站点，而他之前刚访问过银行不久，登录信息尚未过期，那么就有危险！

🔺避免CSRF攻击：浏览器对请求进行分类：简单请求，预先请求，带凭证的请求等，预先请求首先发送一个options请求，和浏览器进行协商是否接受请求。默认情况下跨域请求不需要凭证，但服务端可配置要求客户端提供凭证。



### 11. 定义系统启动任务的两种方式

涉及到系统任务，例如项目启动阶段数据初始化操作，这些操作只在项目启动时进行，以后不再执行。

#### CommandLineRunner

```java
@Component
@Order(100)
public class MyCommandLineRunner1 implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
    }
}
```

@Component注解将MyCommandLineRunner1注册为Spring容器中一个Bean

添加@Order注解表示这个启动任务的执行优先级。@Order注解中数字越小优先级越大

在run方法中，写启动任务核心逻辑，项目启动时run方法自动执行

run方法参数来自于项目启动参数，即项目入口类中main方法参数会被传到这里

**参数有两种方式传递：**

- IDEA：在Run/Debug Configurations找到对应Application，在Program arguments中填入
- 项目打包：`java -jar devtools-0.0.1-SNAPSHOT.jar 三国演义 西游记`

🔺参数传递没有key，直接写value

#### ApplicationRunner

```java
@Component
@Order(98)
public class MyApplicationRunner1 implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        List<String> nonOptionArgs = args.getNonOptionArgs();
        System.out.println("MyApplicationRunner1>>>"+nonOptionArgs);
        Set<String> optionNames = args.getOptionNames();
        for (String key : optionNames) {
            System.out.println("MyApplicationRunner1>>>"+key + ":" + args.getOptionValues(key));
        }
        String[] sourceArgs = args.getSourceArgs();
        System.out.println("MyApplicationRunner1>>>"+Arrays.toString(sourceArgs));
    }
}
```

项目启动时run方法就会被自动执行

- args.getNonOptionArgs()：获取命令行中的无key参数（和CommandLineRunner一样）
- args.getOptionNames()：获取所有key/value形式参数的key
- args.getOptionValues(key)：根据key获取key/value形式参数的value
- args.getSourceArgs()：获取命令行中所有参数

**参数有两种方式传递：**

- IDEA：在Run/Debug Configurations找到对应Application，在Program arguments中填入
- 项目打包：--key=value形式：`java -jar devtools-0.0.1-SNAPSHOT.jar 三国演义 西游记 --age=99`



### 12. 定时任务两种实现方式

#### @Scheduled

@EnableScheduling注解：

```java
@SpringBootApplication
@EnableScheduling
public class ScheduledApplication {
    public static void main(String[] args) {
        SpringApplication.run(ScheduledApplication.class, args);
    }
}
```

配置定时任务：

```java
    @Scheduled(fixedRate = 2000)
    public void fixedRate() {
        System.out.println("fixedRate>>>"+new Date());    
    }
    @Scheduled(fixedDelay = 2000)
    public void fixedDelay() {
        System.out.println("fixedDelay>>>"+new Date());
    }
    @Scheduled(initialDelay = 2000,fixedDelay = 2000)
    public void initialDelay() {
        System.out.println("initialDelay>>>"+new Date());
    }
```

- @Scheduled注解开启一个定时任务
- fixedRate：任务执行之间时间间隔，两次任务开始时间间隔，即第二次任务开始时，第一次任务可能还没结束
- fixedDelay：任务执行之间时间间隔，本次任务结束到下次任务开始之间的时间间隔
- initialDelay：首次任务启动的延迟时间
- 所有时间单位都是毫秒

展示效果如图：

**放图Scheduled**

🔺@Scheduled注解也支持cron表达式：

cron表达式格式： `[秒] [分] [小时] [日] [月] [周] [年]`

例子：每隔5秒触发一次

```java
@Scheduled(cron = "0/5 * * * * *")
public void cron() {
    System.out.println(new Date());
}
```

#### Quartz

开启定时任务注解：

```java
@SpringBootApplication
@EnableScheduling
public class QuartzApplication {
    public static void main(String[] args) {
        SpringApplication.run(QuartzApplication.class, args);
    }
}
```

Quartz使用过程中两个关键概念：JobDetail（要做的事情）和触发器（什么时候做）。

要定义JobDetail，先定义Job：

**Job的定义有两种方式：**

①直接定义一个Bean：

```java
@Component
public class MyJob1 {
    public void sayHello() {
        System.out.println("MyJob1>>>"+new Date());
    }
}
```

首先将这个Job注册到Spring容器中，这种方式无法传参

②继承QuartzJobBean并实现默认方法：

```java
public class MyJob2 extends QuartzJobBean {
    HelloService helloService;
    public HelloService getHelloService() {
        return helloService;
    }
    public void setHelloService(HelloService helloService) {
        this.helloService = helloService;
    }
    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        helloService.sayHello();
    }
}
public class HelloService {
    public void sayHello() {
        System.out.println("hello service >>>"+new Date());
    }
}
```

这种方式支持传参，任务启动时executeInternal方法被执行

接下来配置JobDetail和Trigger触发器

```java
@Configuration
public class QuartzConfig {
    @Bean
    MethodInvokingJobDetailFactoryBean methodInvokingJobDetailFactoryBean() {
        MethodInvokingJobDetailFactoryBean bean = new MethodInvokingJobDetailFactoryBean();
        bean.setTargetBeanName("myJob1");
        bean.setTargetMethod("sayHello");
        return bean;
    }
    @Bean
    JobDetailFactoryBean jobDetailFactoryBean() {
        JobDetailFactoryBean bean = new JobDetailFactoryBean();
        bean.setJobClass(MyJob2.class);
        JobDataMap map = new JobDataMap();
        map.put("helloService", helloService());
        bean.setJobDataMap(map);
        return bean;
    }
    @Bean
    SimpleTriggerFactoryBean simpleTriggerFactoryBean() {
        SimpleTriggerFactoryBean bean = new SimpleTriggerFactoryBean();
        bean.setStartTime(new Date());
        bean.setRepeatCount(5);
        bean.setJobDetail(methodInvokingJobDetailFactoryBean().getObject());
        bean.setRepeatInterval(3000);
        return bean;
    }
    @Bean
    CronTriggerFactoryBean cronTrigger() {
        CronTriggerFactoryBean bean = new CronTriggerFactoryBean();
        bean.setCronExpression("0/10 * * * * ?");
        bean.setJobDetail(jobDetailFactoryBean().getObject());
        return bean;
    }
    @Bean
    SchedulerFactoryBean schedulerFactoryBean() {
        SchedulerFactoryBean bean = new SchedulerFactoryBean();
        bean.setTriggers(cronTrigger().getObject(), simpleTriggerFactoryBean().getObject());
        return bean;
    }
    @Bean
    HelloService helloService() {
        return new HelloService();
    }
}
```

- JobDetail配置两种方式：MethodInvokingJobDetailFactoryBean和JobDetailFactoryBean
- 使用MethodInvokingJobDetailFactoryBean，可以配置目标Bean的名字和目标方法的名字，不支持传参
- 使用JobDetailFactoryBean，可以配置JobDetail，任务类继承自QuartzJobBean，支持传参，将参数封装在JobDataMap进行传递
- Trigger触发器，Quartz定义了多个触发器，如SimpleTrigger和CronTrigger
- SimpleTrigger类似于@Scheduled基本用法
- CronTrigger类似于@Scheduled中cron表达式用法



### 13. Spring Boot中自定义SpringMVC 配置

#### WebMvcConfigurerAdapter

在Spring Boot 1.x中自定义SpringMVC时继承的一个抽象类。抽象类里全是空方法。

```java
public abstract class WebMvcConfigurerAdapter implements WebMvcConfigurer {
    //各种 SpringMVC 配置的方法
}
```

这个类的注释：

```java
/**
 * An implementation of {@link WebMvcConfigurer} with empty methods allowing
 * subclasses to override only the methods they're interested in.
 * @deprecated as of 5.0 {@link WebMvcConfigurer} has default methods (made
 * possible by a Java 8 baseline) and can be implemented directly without the
 * need for this adapter
 */
```

在Spring Boot 1.x时代，自定义SpringMVC配置，直接继承WebMvcConfigurerAdapter类即可。

#### WebMvcConfigurer

是一个接口，接口中方法和WebMvcConfigurerAdapter中定义的空方法一样。因此，从Spring Boot 1.x切换到Spring Boot 2.x，只需把继承类改成实现接口即可。

#### WebMvcConfigurationSupport

在WebMvcConfigurationSupport类中，提供了用Java配置SpringMVC所需的所有方法。自定义SpringMVC配置可通过继承WebMvcConfigurationSupport来实现，但一般只在Java配置的SSM项目中使用。

因为：Spring Boot中，Spring MVC相关的自动化配置是在WebMvcAutoConfiguration配置类中实现：

```java
@Configuration
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
        ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
}
```

它的生效条件里有一条：当不存在WebMvcConfigurationSupport实例

**Spring Boot给我们提供了很多自动化配置，很多时候我们修改这些配置时不是全盘否定这些自动化配置，我们可能指数修改某一个配置，继承WebMvcConfigurationSupport来实现对SpringMVC配置会导致所有SpringMVC自动化配置失效，因此一般不选用这种方案。**

#### @EnableWebMvc

启用WebMvcConfigurationSupport

```java
/**
 * Adding this annotation to an {@code @Configuration} class imports the Spring MVC
 * configuration from {@link WebMvcConfigurationSupport}, e.g.:
```

加了这个注解就会自动导入WebMvcConfigurationSupport，因此在Spring Boot中也不建议使用@EnableMvc注解。



