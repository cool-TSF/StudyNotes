
---
[toc]
---

什么是动态sql？
mybatis核心 对sql语句进行灵活操作，通过表达式进行判断，对sql进行灵活拼接、组装。

# if判断
## UserMapper.xml
```java
<!--用户综合查询-->
    <select id="findUserList" parameterType="cn.edu.wtu.po.UserQueryVo" resultType="cn.edu.wtu.po.UserCustom">
        select * from user
        <where>
            <if test="userCustom!=null">
            <if test="userCustom.sex!=null and userCustom.sex!=''">
                and user.sex = #{userCustom.sex}
            </if>
            <if test="userCustom.username!=null and userCustom.username!=''">
                and user.username LIKE '%${userCustom.username}%'
            </if>
            </if>
        </where>
    </select>
    <!--用户综合查询总数-->
    <select id="findUserCount" parameterType="cn.edu.wtu.po.UserQueryVo" resultType="java.lang.Integer">
                SELECT count(*) FROM user
                <where>
                    <if test="userCustom!=null">
                        <if test="userCustom.sex!=null and userCustom.sex!=''">
                            and user.sex = #{userCustom.sex}
                        </if>
                        <if test="userCustom.username!=null and userCustom.username!=''">
                            and user.username LIKE '%${userCustom.username}%'
                        </if>
                    </if>
                </where>
    </select>
```
## 测试结果

### 1.注释掉`testFindUserList()`方法中的`userCustom.setUsername("张三");`

UserMapperTest.java
```java
//由于这里使用动态sql，如果不设置某个值，条件不会拼接在sql中
userCustom.setSex("1");
// userCustom.setUsername("晓明");
userQueryVo.setUserCustom(userCustom);
```

输出

```
DEBUG [main] - Created connection 1293618474.
DEBUG [main] - Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@4d1b0d2a]
DEBUG [main] - ==>  Preparing: select * from user WHERE user.sex = ? 
DEBUG [main] - ==> Parameters: 1(String)
DEBUG [main] - <==      Total: 9
[id:10
username:张三
sex:1
birthday:Wed Nov 06 00:00:00 CST 2019
address:北京市, id:22
username:张晓明
sex:1
birthday:Tue Nov 26 00:00:00 CST 2019
address:天津, id:24
username:陈晓明
sex:1
birthday:Tue Apr 04 00:00:00 CST 2017
address:重庆, id:26
username:小明
sex:1
birthday:Tue Jul 24 00:00:00 CST 2018
address:武汉, id:28
username:李梅梅
sex:1
birthday:Wed Jul 26 00:00:00 CST 2000
address:西安, id:32
username:江江
sex:1
birthday:Wed Jul 05 00:00:00 CST 2017
address:宁夏, id:33
username:红红
sex:1
birthday:Tue Jun 20 00:00:00 CST 2017
address:广州, id:34
username:木木
sex:1
birthday:Tue Apr 26 00:00:00 CST 2016
address:河北, id:37
username:陈晓明
sex:1
birthday:Wed Nov 27 00:00:00 CST 2019
address:null]
```
可以看到sql语句为`reparing: SELECT * FROM user WHERE user.sex=? `，没有username的部分


### 2.`userQueryVo`设为null,则`userCustom`为null

```java
//List<UserCustom> list = userMapper.findUserList(userQueryVo);
List<UserCustom> list = userMapper.findUserList(null);
```

输出

```
DEBUG [main] - Created connection 1859039536.
DEBUG [main] - Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@6eceb130]
DEBUG [main] - ==>  Preparing: select * from user 
DEBUG [main] - ==> Parameters: 
DEBUG [main] - <==      Total: 15
[id:1
username:王五
sex:2
birthday:Tue Nov 05 00:00:00 CST 2019
address:null, id:10
username:张三
sex:1
birthday:Wed Nov 06 00:00:00 CST 2019
address:北京市, id:22
username:张晓明
sex:1
birthday:Tue Nov 26 00:00:00 CST 2019
address:天津, id:24
username:陈晓明
sex:1
birthday:Tue Apr 04 00:00:00 CST 2017
address:重庆, id:26
username:小明
sex:1
birthday:Tue Jul 24 00:00:00 CST 2018
address:武汉, id:27
username:宋江涛new2
sex:男
birthday:Sun Nov 03 00:00:00 CST 2019
address:山西太原new, id:28
username:李梅梅
sex:1
birthday:Wed Jul 26 00:00:00 CST 2000
address:西安, id:29
username:丽丽
sex:2
birthday:Tue Oct 29 00:00:00 CST 2019
address:新疆, id:31
username:涵涵
sex:2
birthday:Tue Jun 12 00:00:00 CST 2018
address:西藏, id:32
username:江江
sex:1
birthday:Wed Jul 05 00:00:00 CST 2017
address:宁夏, id:33
username:红红
sex:1
birthday:Tue Jun 20 00:00:00 CST 2017
address:广州, id:34
username:木木
sex:1
birthday:Tue Apr 26 00:00:00 CST 2016
address:河北, id:35
username:宋江涛
sex:男
birthday:Sun Nov 03 00:00:00 CST 2019
address:山西, id:36
username:张宇
sex:2
birthday:Sun Nov 03 00:00:00 CST 2019
address:襄阳, id:37
username:陈晓明
sex:1
birthday:Wed Nov 27 00:00:00 CST 2019
address:null]
```
可以看到sql语句变为了`SELECT * FROM user`

# sql片段(重点)

将上边实现的动态sql判断代码块抽取出来，组成一个sql片段。其它的statement中就可以引用sql片段。


## 定义sql片段

```xml
<!-- 定义sql片段
id：sql片段的唯 一标识

经验：是基于单表来定义sql片段，这样话这个sql片段可重用性才高
在sql片段中不要包括 where
 -->
<sql id="query_user_where">
    <if test="userCustom!=null">
        <if test="userCustom.sex!=null and userCustom.sex!=''">
            AND user.sex = #{userCustom.sex}
        </if>
        <if test="userCustom.username!=null and userCustom.username!=''">
            AND user.username LIKE '%${userCustom.username}%'
        </if>
    </if>
</sql>
```

## 引用sql片段

```xml
<!-- 用户信息综合查询
    #{userCustom.sex}:取出pojo包装对象中性别值
    ${userCustom.username}：取出pojo包装对象中用户名称
 -->
<select id="findUserList" parameterType="com.iot.mybatis.po.UserQueryVo"
        resultType="com.iot.mybatis.po.UserCustom">
    SELECT * FROM user
    <!--  where 可以自动去掉条件中的第一个and -->
    <where>
        <!-- 引用sql片段 的id，如果refid指定的id不在本mapper文件中，需要前边加namespace -->
        <include refid="query_user_where"></include>
        <!-- 在这里还可以引用其它的sql片段  -->
    </where>
</select>
```

# foreach标签

向sql传递数组或List，mybatis使用foreach解析

在用户查询列表和查询总数的statement中增加多个id输入查询。两种方法，sql语句如下：

- `SELECT * FROM USER WHERE id=1 OR id=10 OR id=16`
- `SELECT * FROM USER WHERE id IN(1,10,16)`

一个使用OR,一个使用IN


## 在输入参数类型中添加`List<Integer> ids`传入多个id

```java
public class UserQueryVo {

    //传入多个id
   private List<Integer> ids;

    public List<Integer> getIds() {
        return ids;
    }

    public void setIds(List<Integer> ids) {
        this.ids = ids;
    }
}
```

## 修改mapper.xml

```xml
 select * from user
 <where>
    <if test="ids!=null">
        <!-- 使用 foreach遍历传入ids
        collection：指定输入 对象中集合属性
        item：每个遍历生成对象中
        open：开始遍历时拼接的串
        close：结束遍历时拼接的串
        separator：遍历的两个对象中需要拼接的串
        -->
        <!-- 使用实现下边的sql拼接：
        AND (id=1 OR id=10 OR id=16)
        -->
        <foreach collection="ids" item="user_id" open="AND (" close=")" separator="or">
            <!-- 每个遍历需要拼接的串 -->
            id=#{user_id}
        </foreach>
        
        <!-- 实现  “ and id IN(1,10,16)”拼接 -->
        <!-- <foreach collection="ids" item="user_id" open="and id IN(" close=")" separator=",">
            每个遍历需要拼接的串
            #{user_id}
        </foreach> -->
    </if>
</where>
```


## 测试代码

在`testFindUserList`中加入

```java
//传入多个id
List<Integer> ids = new ArrayList<Integer>();
ids.add(1);
ids.add(10);
ids.add(16);
//将ids通过userQueryVo传入statement中
userQueryVo.setIds(ids);
```
