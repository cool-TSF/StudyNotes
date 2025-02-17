# mybatis学习笔记(3)-入门程序

标签： mybatis
---
**Contents**
[toc]
---

# 目录结构
![](../../images/mybatis入门程序目录结构.jpg)

# 数据库表的设计
~~~sql
/*
 Navicat Premium Data Transfer

 Source Server         : Mysql
 Source Server Type    : MySQL
 Source Server Version : 50726
 Source Host           : localhost:3306
 Source Schema         : mybatis

 Target Server Type    : MySQL
 Target Server Version : 50726
 File Encoding         : 65001

 Date: 03/11/2019 12:01:59
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user`  (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `birthday` date NULL DEFAULT NULL,
  `sex` char(1) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `address` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 36 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

SET FOREIGN_KEY_CHECKS = 1;

~~~

# 配置文件
## pom.xml
~~~xml
<dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.6</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.47</version>
    </dependency>
    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>1.2.17</version>
    </dependency>
  </dependencies>
~~~

## log4j.properties

```properties
# Global logging configuration
# 开发环境下，日志级别要设置成DEBUG或者ERROR
log4j.rootLogger=DEBUG, stdout
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

## SqlMapConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 和spring整合后 environments配置将废除-->
    <environments default="development">
        <environment id="development">
            <!-- 使用jdbc事务管理-->
            <transactionManager type="JDBC" />
            <!-- 数据库连接池-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8" />
                <property name="username" value="root" />
                <property name="password" value="" />
            </dataSource>
        </environment>
    </environments>
    <!--加载映射文件-->
    <mappers>
        <mapper resource="sqlmap/User.xml"/>
    </mappers>
</configuration>
```

# User.java
~~~java
import java.util.Date;

public class User {
    //用户po
    //属性名和数据库字段名对应
    private int id;
    private String username;// 用户姓名
    private String sex;// 性别
    private Date birthday;// 生日
    private String address;// 地址

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
~~~


# User.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
    命名空间，作用为，对sql进行分类化管理，理解sql隔离，
    注意：使用mapper代理方法开发，namespace有特殊作用
-->
<mapper namespace="test">
    <!--在映射文件中配置sql-->
    <!--
        findUserById
        通过select执行数据库查询
        id：标识映射文件的sql
        将sql语句封装到mappedStatement对象中，所以将id称为Statement的id
        #{}:表示一个占位符
        parameterType:指定输入参数的类型
        #{id}：其中的id表示接收输入的参数，参数的名称就是id，如果输入参数类型为简单类型，那么#{}中的参数可以任意，可以是value或其他
        resultType：指定sql输出结果的所映射的Java对象类型，select指定的resultType表示将单条记录映射成的Java对象
    -->
    <select id="findUserById" parameterType="int" resultType="cn.edu.wtu.po.User">
        select * from user where id=#{VALUE }
    </select>




    <!--
        findUserByName
        ${}:表示拼接字符串，将接收到的sql不加任何修饰拼接在sql语句里
        使用${}拼接sql，可能会引起sql注入,一般不建议使用
        ${value}：接收参数的内容，如果传入的的是简单类型，${}中只能使用value
    -->
    <select id="findUserByName" parameterType="java.lang.String" resultType="cn.edu.wtu.po.User">
        select * from user WHERE username LIKE '%${value}%'
    </select>



    <!--
        添加用户
        parameterType:指定参数类型为pojo类型
        #{}中指定pojo的属性名，接收到的pojo对象的属性值，mybatis通过OGNL获取对象的值
        SELECT LAST_INSERT_ID():得到刚刚insert进去的记录的主键值，只适用于主键自增
        非主键自的则需要使用uuid()来实现,表的id类型也得设置为tring(详见下面的注释)
        keyProperty：将查询到的主键值设置到SparameterType指定的对象的哪个属性
        order:SELECT LAST_INSERT_ID()执行顺序，相当于insert语句来说它的实现顺序

    -->
    <insert id="insertUser" parameterType="cn.edu.wtu.po.User">
        <!--uuid()-->
        <!--
            <selectKey keyProperty="id" order="AFTER" resultType="java.lang.String">
              SELECT uuid()
            </selectKey>
            insert into user (id,username,birthday,sex,address) value(#{id},#{username},#{birthday},#{sex},#{address})
        -->
        <selectKey keyProperty="id" order="AFTER" resultType="java.lang.Integer">
            SELECT LAST_INSERT_ID()
        </selectKey>
        insert into user (username,birthday,sex,address) value(#{username},#{birthday},#{sex},#{address})
    </insert>


    <delete id="deleteUser" parameterType="java.lang.Integer">
        delete from user where id=#{id}
    </delete>

    <update id="updateUser" parameterType="cn.edu.wtu.po.User">
        UPDATE user set username=#{username},birthday=#{birthday},sex=#{sex},address=#{address} where id=#{id}
    </update>
</mapper>
```


# MybatisFirst.java
~~~java
public class MybatisFirst {
    /**
     * 根据id查询用户信息，得到一条记录
     * @throws IOException
     */
    @Test
    public void findUserByIdTest() throws IOException {
        // mybatis配置文件
        String resource = "SqlMapConfig.xml";
        InputStream is = Resources.getResourceAsStream(resource);
        // 创建会话工厂
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
        // 通过工厂得到SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //通过SqlSession操作数据库
        //第一个参数：映射文件中的statement的id，等于namespace+"."+statement的id
        //第二个参数：指定和映射文件中所匹配的所有parameterType的类型
        //sqlSession.selectOne()的结果是映射文件中所匹配的resultType类型的对象
        User user = sqlSession.selectOne("test.findUserById",1);
        System.out.println(user);
        // 释放资源
        sqlSession.close();
    }

    @Test
    public void findUserByName() throws IOException {
        // mybatis配置文件
        String resource = "SqlMapConfig.xml";
        InputStream is = Resources.getResourceAsStream(resource);
        // 创建会话工厂
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
        // 通过工厂得到SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        List<User> list= sqlSession.selectList("test.findUserByName","小明");
        System.out.println(list);
        sqlSession.close();
    }

    /*
        小结：
        selectOne和selectList：

        selectOne表示查询出一条记录进行映射。如果使用selectOne可以实现使用selectList也可以实现（list中只有一个对象）。
        selectList表示查询出一个列表（多条记录）进行映射。如果使用selectList查询多条记录，不能使用selectOne。
        如果使用selectOne报错：
        org.apache.ibatis.exceptions.TooManyResultsException: Expected one result (or null) to be returned by selectOne(), but found: 4

     */

    @Test
    public void insertUserTest() throws IOException {
        //mybatis配置文件
        String resource = "SqlMapConfig.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        //创建会话工厂
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        //通过工厂得到SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        User user = new User();
        user.setUsername("宋江涛");
        user.setBirthday(new Date());
        user.setSex("男");
        user.setAddress("山西");
        //list中的user和映射文件User.xml中的resultType的类型一直
        sqlSession.insert("test.insertUser",user);
        //提交事务
        sqlSession.commit();
        //获取主键
        System.out.println(user.getId());
        sqlSession.close();
    }

    /**
     * 删除用户
     * @throws IOException
     */
    @Test
    public void deleteUserTest() throws IOException {
        //mybatis配置文件
        String resource = "SqlMapConfig.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        //创建会话工厂
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        //通过工厂得到SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //list中的user和映射文件User.xml中的resultType的类型一直
        sqlSession.delete("test.deleteUser",30);
        //提交事务
        sqlSession.commit();
        sqlSession.close();
    }

    /**
     * 更新用户
     * @throws IOException
     */
    @Test
    public void updateUserTest() throws IOException {
        //mybatis配置文件
        String resource = "SqlMapConfig.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        //创建会话工厂
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        //通过工厂得到SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        User user = new User();
        user.setId(27);
        user.setUsername("宋江涛new2");
        user.setBirthday(new Date());
        user.setSex("男");
        user.setAddress("山西太原new");
        //list中的user和映射文件User.xml中的resultType的类型一直
        sqlSession.update("test.updateUser",user);
        //提交事务
        sqlSession.commit();
        sqlSession.close();
    }
}
~~~

# 总结

- `parameterType`

在映射文件中通过parameterType指定输入参数的类型


- `resultType`

在映射文件中通过resultType指定输出结果的类型


- `#{}`和`${}`

`#{}`表示一个占位符号;

`${}`表示一个拼接符号，会引起sql注入，所以不建议使用


- `selectOne`和`selectList`

`selectOne`表示查询一条记录进行映射，使用`selectList`也可以使用，只不过只有一个对象

`selectList`表示查询出一个列表(参数记录)进行映射，不能够使用`selectOne`查，不然会报下面的错:

```
org.apache.ibatis.exceptions.TooManyResultsException: Expected one result (or null) to be returned by selectOne(), but found: 3
```