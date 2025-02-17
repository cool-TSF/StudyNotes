# mybatis学习笔记(6)-Mybatis的输入和输出映射

标签： mybatis

---
**Contents**
[toc]
---

# Mybatis输入映射（掌握）

* 通过parameterType指定输入参数的类型，类型可以是简单类型、hashmap、pojo的包装类型
    * 传递pojo的包装对象
      * 需求：完成用户信息的综合查询，需要传入查询条件很复杂（可能包括用户信息、其它信息，比如商品、订单的）
      * 针对上边需求，建议使用自定义的包装类型的pojo。在包装类型的pojo中将复杂的查询条件包装进去。 

## 目录结构
![](../../images/Mybatis配置文件SqlMapConfig.xml13.jpg)

## UserCustom.java
~~~java
public class UserCustom extends User{
// 可扩展用户信息
}
~~~

## UserQueryVo.java
~~~java
public class UserQueryVo {
// 这里包装所需的查询条件

/**
* 用户查询
*/
private UserCustom userCustom;

public UserCustom getUserCustom() {
    return userCustom;
}

public void setUserCustom(UserCustom userCustom) {
    this.userCustom = userCustom;
}

//可包装其他的查询条件,订单,商品......
}
~~~

## UserMapper.java
~~~java
/**
* 用户综合查询
* @param userQueryVo
* @return UserCustom
* @throws Exception
*/
public UserCustom findUserList(UserQueryVo userQueryVo) throws Exception;
~~~

## UserMapper.xml中配置新的查询
~~~xml
<!--用户综合查询-->
<select id="findUserList" parameterType="cn.edu.wtu.po.UserQueryVo" resultType="cn.edu.wtu.po.UserCustom">
    select *from user where user.sex=#{userCustom.sex} and
    user.username like "%${userCustom.username}%";
</select>
~~~
注意不要将`#{userCustom.sex}`中的`userCustom`写成`UserCustom`,前者指属性名(由于使用IDE提示自动补全，所以只是把类型名首字母小写了)，后者指类型名，这里是`UserQueryVo`类中的`userCustom`属性，是**属性名**。写错会报如下异常：

```
org.apache.ibatis.exceptions.PersistenceException: 
### Error querying database.  Cause: org.apache.ibatis.reflection.ReflectionException: There is no getter for property named 'UserCustom' in 'class com.iot.mybatis.po.UserQueryVo'
### Cause: org.apache.ibatis.reflection.ReflectionException: There is no getter for property named 'UserCustom' in 'class com.iot.mybatis.po.UserQueryVo'
```

## UserMapperTest.java中新增测试
~~~java
/**
* 用户综合查询
* @throws Exception
*/
@Test
public void testFindUserList() throws Exception{
    // 创建会话
    SqlSession sqlSession = sqlSessionFactory.openSession();
    // 创建代理对象
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

    // 创建包对象,设置查询条件
    UserQueryVo userQueryVo = new UserQueryVo();
    UserCustom userCustom = new UserCustom();

    userCustom.setSex("1");
    userCustom.setUsername("张三");

    userQueryVo.setUserCustom(userCustom);

    UserCustom userCustom1 = userMapper.findUserList(userQueryVo);
    List<UserCustom> list = new ArrayList<>();
    list.add(userCustom1);
    System.out.println(list);
~~~

# Mybatis输出映射（掌握）

## 一、resultType
  * 使用resultType进行输出映射，只有查询出来的列名和pojo中的属性名一致，该列才可以映射成功
  * 如果查询出来的列名和pojo中的属性名全部不一致，没有创建pojo对象。
  * 只要查询出来的列名和pojo中的属性有一个一致，就会创建pojo对象

### 1.resultType的输出简单类型

- UserMapper.java
```java
  /**
   * 用户综合查询个数
   * @param userQueryVo
   * @return 用户个数
   * @throws Exception
   */
  public int findUserCount(UserQueryVo userQueryVo) throws Exception;
```

- UserMapper.xml

```xml
 <!-- 用户信息综合查询总数
        parameterType：指定输入类型和findUserList一样
        resultType：输出结果类型
    -->
    <select id="findUserCount" parameterType="com.iot.mybatis.po.UserQueryVo" resultType="int">
        SELECT count(*) FROM user WHERE user.sex=#{userCustom.sex} AND user.username LIKE '%${userCustom.username}%'
    </select>
```

- UserMapperTest.java
```java
/**
     * 用户综合查询 用户个数
     * @throws Exception
     */
    @Test
    public void testFindUserCount() throws Exception{
        // 创建会话
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建代理对象
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

        // 创建包装对象,设置查询条件
        UserQueryVo userQueryVo = new UserQueryVo();
        UserCustom userCustom = new UserCustom();

        userCustom.setUsername("1");
        userCustom.setUsername("陈晓明");
        userQueryVo.setUserCustom(userCustom);

        int count = userMapper.findUserCount(userQueryVo);

        System.out.println(count);
    }
```
- 小结

查询出来的结果集只有一行且一列，可以使用简单类型进行输出映射。


### 2.resultType的输出pojo对象和pojo列表

**不管是输出的pojo单个对象还是一个列表（list中包括pojo），在UserMapper.xml中`resultType`指定的类型是一样的。**

在mapper.java指定的方法返回值类型不一样：

- 输出单个pojo对象，方法返回值是单个对象类型

```java
//根据id查询用户信息
public User findUserById(int id) throws Exception;
```

- 输出pojo对象list，方法返回值是List\<Pojo>

```java
//根据用户名列查询用户列表
public List<User> findUserByName(String name) throws Exception;
```


**生成的动态代理对象中是根据mapper方法的返回值类型确定是调用`selectOne`(返回单个对象调用)还是`selectList` （返回集合对象调用 ）.**

## 二、resultMap

如果查询出来的列名和pojo的属性名不一致，通过定义一个resultMap对列名和pojo属性名之间作一个映射关系。

1.定义resultMap

2.使用resultMap作为statement的输出映射类型

- 定义reusltMap

```xml
    <!-- 定义resultMap
	将id_,username_ 和User类中的属性作映射

	type：resultMap最终映射的java对象类型,可以使用别名
	id：对resultMap的唯一标识
	 -->
	 <resultMap  id="userResultMap" type="cn.edu.wtu.po.Userr">
	 	<!-- id表示查询结果集中唯一标识 
	 	column：查询出来的列名
	 	property：type指定的pojo类型中的属性名
	 	最终resultMap对column和property作一个映射关系 （对应关系）
	 	-->
	 	<id column="id_" property="id"/>
	 	<!-- 
	 	result：对普通名映射定义
	 	column：查询出来的列名
	 	property：type指定的pojo类型中的属性名
	 	最终resultMap对column和property作一个映射关系 （对应关系）
	 	 -->
	 	<result column="username_" property="username"/>
	 
	 </resultMap>
```

- 使用resultMap作为statement的输出映射类型

```xml
<!-- 使用resultMap进行输出映射
        resultMap：指定定义的resultMap的id，如果这个resultMap在其它的mapper文件，前边需要加namespace
        -->
    <select id="findUserByIdResultMap" parameterType="int" resultMap="userResultMap">
        SELECT id id_,username username_ FROM USER WHERE id=#{value}
    </select>

```

- UserMapper.java

```java
//根据id查询用户信息，使用resultMap输出
public User findUserByIdResultMap(int id) throws Exception;
```

- 测试代码

```java
@Test
public void testFindUserByIdResultMap() throws Exception {

	SqlSession sqlSession = sqlSessionFactory.openSession();

	//创建UserMapper对象，mybatis自动生成mapper代理对象
	UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

	//调用userMapper的方法

	User user = userMapper.findUserByIdResultMap(1);

	System.out.println(user);


}
```


## 总结
* 使用resultType进行输出映射，只有查询出来的列名和pojo中的属性名一致，该列才可以映射成功。
* 如果查询出来的列名和pojo的属性名不一致，通过定义一个resultMap对列名和pojo属性名之间作一个映射关系。
