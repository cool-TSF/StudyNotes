# springmvc学习笔记(2)-非注解的处理器映射器和适配器

---

[TOC]

---

![](../../images/SpringMVC/非注解的处理器映射器和适配器的项目结构.jpg)

# 1.导入的依赖
- pom.xml
```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <!-- spring版本号 -->
    <spring.version>5.1.5.RELEASE</spring.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>
    <!-- Spring依赖 -->
    <!-- 1.Spring核心依赖 -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-expression</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <!-- 2.Spring dao依赖 -->
    <!-- spring-jdbc包括了一些如jdbcTemplate的工具类 -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <!-- 3.Spring web依赖 -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <!-- 4.Spring test依赖：方便做单元测试和集成测试 -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>4.0.1</version>
      <scope>provided</scope>
    </dependency>

  </dependencies>
```

# 2.配置DispatcherServlet核心分发器(web.xml)
```xml
<!--配置DispatcherServlet核心分发器-->
  <servlet>
    <servlet-name>springmvc1</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--加载springmvc的配置文件-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>
  </servlet>
  <servlet-mapping>
    <servlet-name>springmvc1</servlet-name>
    <url-pattern>*.do</url-pattern>
  </servlet-mapping>
```

# 3.配置HandlerMapping映射器(springmvc.xml)
```xml
<!--处理器映射器-->
<!--根据bean的name进行查找 Handler将action的url配置在bean的name中-->
<!--这是一个默认的映射处理器,即使不配置,那么也是默认就是这个-->
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"></bean>
```

# 4.配置HandlerAdapter适配器(springmvc.xml)
```xml
<!--这个适配器不是必须配置的,这是默认的  它在servlet容器启动就加载-->
<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"></bean>
```

# 5.编写一个Controller类
- TestController.java
```java
public class TestController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
        /**
         * 1.收集参数、验证参数
         * 2.绑定参数到命令对象
         * 3.将命令对象传入业务对象进行处理
         * 4.选择视图
         */
        ModelAndView modelAndView = new ModelAndView();
        // 添加模型数据,那么这个数据可以是任意的POJO对象
        modelAndView.addObject("hello","hello world!!");
        // 设置逻辑视图名,视图解析器会根据该名字解析到具体的视图界面
        modelAndView.setViewName("hello");
        return modelAndView;
    }
}
```

# 6.配置自定义控制器(springmvc.xml)
```xml
<!--配置自定义controller,使用beanName:name="/hello.do"进行请求映射匹配-->
<bean name="/hello.do" class="club.krislin.controller.TestController"/>
```

# 7.定义一个相应页面
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>hello</title>
</head>
<body>
    <h1>返回成功</h1>
</body>
</html>
```

# 8.配置视图解析器(springmvc.xml)
```xml
<!--使用视图解析器解析逻辑视图,这样更方便,易于扩展-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <!--逻辑视图的前缀-->
    <property name="prefix" value="/WEB-INF/jsps/"/>
    <!--逻辑视图的后缀-->
    <property name="suffix" value=".jsp"/>
</bean>
```

# 9.分析程序执行流程
![](../../images/SpringMVC/非注解的处理器映射器和适配器的执行流程.jpg)

- 1. 首先用户发送请求http://localhost:9080/springmvc-01/hello——>web容器，web容器根据“/hello”路径映射到`DispatcherServlet`(url-pattern为/)进行处理；
- 2. `DispatcherServlet`——>`BeanNameUrlHandlerMapping`进行请求到处理的映射，`BeanNameUrlHandlerMapping`将“/hello”路径直接映射到名字为“/hello”的Bean进行处理，即`TestController`，`BeanNameUrlHandlerMapping`将其包装为`HandlerExecutionChain`(只包括`HelloWorldController`处理器，没有拦截器)
- 3. `DispatcherServlet`——> `SimpleControllerHandlerAdapter`，`SimpleControllerHandlerAdapter`将`HandlerExecutionChain`中的处理器(`TestController`)适配为`SimpleControllerHandlerAdapter`；
- 4. `SimpleControllerHandlerAdapter`——> `TestController`处理器功能处理方法的调用，`SimpleControllerHandlerAdapter`将会调用处理器的`handleRequest`方法进行功能处理，该处理方法返回一个`ModelAndView`给`DispatcherServlet`；
5、  `hello`(ModelAndView的逻辑视图名）——>`InternalResourceViewResolver`， `InternalResourceViewResolver`使用JstlView，具体视图页面在/WEB-INF/jsps/hello.jsp；
6、  JstlView（/WEB-INF/jsp/hello.jsp）——>渲染，将在处理器传入的模型数据(message=HelloWorld！)在视图中展示出来；
7、  返回控制权给`DispatcherServlet`，由`DispatcherServlet`返回响应给用户，到此一个流程结束。 

# 10.主要配置步骤
- 1. 前端控制器DispatcherServlet；
- 2. HandlerMapping
- 3. HandlerAdapter
- 4. ViewResolver
- 5. 处理器/页面控制器
- 6. 
视图

