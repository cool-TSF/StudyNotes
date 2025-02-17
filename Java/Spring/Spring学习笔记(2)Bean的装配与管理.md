# Spring学习笔记（二）Bean的装配与管理

---

[TOC]

---


## （一）、三种创建 Bean 对象的方式

### 1. 调用构造函数创建Bean

 调用构造方法创建Bean是最常用的一种情况，Spring容器通过new关键字调用构造器来创建Bean实例，通过class属性指定Bean实例的实现类，也就是说，如果使用构造器创建Bean方法，则<bean/>元素必须指定class属性，其实Spring容器也就是相当于通过实现类new了一个Bean实例。调用构造方法创建Bean实例，通过名字也可以看出，我们需要为该Bean类提供无参数的构造器。

* 1.Bean实现类Person.java

  ```java
  package ioc.pojo;
  public class Person{
      private String name;
      
      public String getName() {
          return name;
      }
      
      public void setName(String name) {
          this.name = name;
      }
  }
  ```

  因为是通过构造函数创建Bean，因此我们需要提供无参数构造函数，另外我们定义了一个name属性，并提供了setter方法，Spring容器通过该方法为name属性注入参数。

* 2.配置文件applicationContext.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
      <!-- 指定class属性，通过构造方法创建Bean实例 -->  
      <bean id="person" class="cn.edu.wtu.Person">  
        <!-- 通过构造方法赋值 -->  
       <constructor-arg name="name" value="魔术师"></constructor-arg>  
      </bean>  
  </beans>
  ```

  配置文件中，通过<bean>元素的id属性指定该bean的唯一名称，class属性指定该bean的实现类，可以理解成Spring容器就是通过该实现类new了一个Bean。通过<constructor-arg>标签的name属性和value属性指定了：构造方法赋值。

### 2. 调用静态工厂方法创建Bean

把创建Bean的任务交给了静态工厂，而不是构造函数，这个静态工厂就是一个Java类，那么使用静态工厂创建Bean咱们又需要添加哪些属性呢？我们同样需要在<bean/>元素内添加class属性，上面也说了，静态工厂是一个Java类，那么该class属性指定的就是该工厂的实现类，而不再是Bean的实现类，告诉Spring这个Bean应该由哪个静态工厂创建，另外我们还需要添加factory-method属性来指定由工厂的哪个方法来创建Bean实例，因此使用静态工厂方法创建Bean实例需要为<bean/>元素指定如下属性：

class：指定静态工厂的实现类，告诉Spring该Bean实例应该由哪个静态工厂创建（指定工厂地址）

factory-method:指定由静态工厂的哪个方法创建该Bean实例（指定由工厂的哪个车间创建Bean）

如果静态工厂方法需要参数，则使用<constructor-arg/>元素传入

~~~java
public class StaticFactory {
     public  static IAccountService getAccountService() {
                return new AccountServiceImpl();
     }
 }
~~~
配置方式如下：
~~~xml
<bean id = "accountService" class = "cn.edu.wtu.factory.StaticFactory" factory-method="getAccountService"></bean>        
~~~

### 3. 调用实例工厂方法创建Bean

使用某个类中的静态方法创建对象，并存入spring容器，如下

~~~java
/**
 *模拟一个工厂类，该类可能存在于jar包中，无法通过修改源码的方式来提供默认构造函数
 * 
 */
public class InstanceFactory {
    public IAccountService getAccountService() {
        return new AccountServiceImpl();
    }
}
~~~
配置方式如下：
~~~xml

<bean id = "instanceFactory" class = "cn.edu.wtu.factory.InstanceFactory"></bean>
<bean id = "accountService" factory-bean="instanceFactory" factory-method="getAccountService"></bean>
~~~

## （二）、bean 的作用范围

### 1. bean 标签的 scope 属性

作用：用于指定 bean 的作用范围

取值：常用的就是单例和多例

- singletond : 单例的（default）

  1）在容器启动完成之前就已经创建好对象，保存在容器中

  2）任何获取都是获取之前创建好的对象 

- prototype : 多例的

  1）容器启动默认不会去创建多实例bean

  2）获取时去创建这个bean

  3）每次获取都会创建一个对象

- request : 作用于 web 应用的请求范围

- session : 作用于 web  应用的会话范围

- global-session : 作用于集群的会话范围（全局会话范围），当不是集群范围时，它就是 session

  gloabl-session 示意图：

![global-session](https://krislin.oss-cn-beijing.aliyuncs.com/pictures/study-notes-images/global-session.jpg)

### 2. bean对象的生命周期

单例对象：

- 出生：当容器创建时发生
- 活着：只要容器还在对象就一直活着
- 死亡：容器销毁，对象消亡

(容器启动)构造器 -------> 初始化方法 -------> (容器关闭)销毁方法

总结：单例对象的声明周期和容器相同

多例对象：

- 出生：当我们使用对象时 Spring 框架为我们创建
- 活着：对象只要是在使用过程中就活着
- 死亡：当对象长时间不用，且没有别的对象引用时，由 Java 的GC回收

获取bean(构造器 ------> 初始化方法)  ---------> 容器关闭不会关闭销毁方法

后置处理器：

(容器启动)构造器 -------> 后置处理器before.... ---------->初始化方法 ------------>后置处理器after.... --------->bean初始化完成

无论bean是否有初始化方法，后置处理器都会默认其有，还会继续工作。