# Spring学习笔记(11)Spring的JdbcTemplate

---
[TOC]

---

# pom.xml
```xml
<properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <!-- mysql connector版本号-->
        <mysql.version>5.1.47</mysql.version>
        <!-- spring版本号 -->
        <spring.version>5.1.5.RELEASE</spring.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>
        <dependency>
            <groupId>com.mchange</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.5.4</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/commons-dbcp/commons-dbcp -->
        <dependency>
            <groupId>commons-dbcp</groupId>
            <artifactId>commons-dbcp</artifactId>
            <version>1.4</version>
        </dependency>
    </dependencies>
```

# 配置数据源
```xml
    <!--加载配置文件jdbc.properties-->
    <context:property-placeholder location="classpath:jdbc.properties"/>
```
## Spring中默认的数据源
```xml
    <!--spring内置数据源-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="${jdbc.driverClass}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
```
## C3P0数据源
```xml
    <!--c3p0数据源-->
    <bean id="dataSource1" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driverClass}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
```
## DBCP数据源
```xml
    <!--DBCP数据源-->
    <bean id="dataSource2" class="org.apache.commons.dbcp.BasicDataSource">
        <property name="driverClassName" value="${jdbc.driverClass}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
```

# JdbcTemplate的增删改查

## 配置数据库的操作模板 JdbcTemplate
```xml
    <!-- 配置一个数据库的操作模板： JdbcTemplate -->
    <!--spring的默认数据源-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!--注入数据源-->
        <property name="dataSource" ref="dataSource"/>
    </bean>
```
## 获取对象
```java
// 1.加载spring配置
ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
// 2.根据id获取bean对象
JdbcTemplate jdbcTemplate = (JdbcTemplate) ac.getBean("jdbcTemplate");
// 3.执行操作
```

## 保存操作
```java
// 3.1 保存操作
jdbcTemplate.update("insert into account(name,money) values ('ffff',77777.0)");
```

## 修改操作
```java
// 3.2 修改操作
jdbcTemplate.update("update account set money=money-? where id=?",300,11);
```

## 删除操作
```java
// 3.3 删除操作
jdbcTemplate.update("delete account where id=?",11);
```

## 查询所有操作
```java
// 3.4 查询所用操作
List<Account> accounts = jdbcTemplate.query("select * from account where money>?",new AccountRowMapper(),500);
for (Account account:accounts) {
    System.out.println(account);
}
```

```java
class AccountRowMapper implements RowMapper<Account>{
    @Override
    public Account mapRow(ResultSet resultSet, int i) throws SQLException {
        Account account = new Account();
        account.setId(resultSet.getInt("id"));
        account.setName(resultSet.getString("name"));
        account.setMoney(resultSet.getFloat("money"));
        return account;
    }
}
```

## 查询一个操作
```java
// 3.5 查询一个操作
List<Account> account = jdbcTemplate.query("select * from account where id=?",new AccountRowMapper(),23);
System.out.println(account.isEmpty()?"没有结果":account.get(0));
```

## 查询一行一列操作
```java
// 3.6 查询返回一行一列操作
// 查询返回一行一列：使用聚合函数，在不使用 group by 字句时，都是返回一行一列。最常用的就是分页中获取总记录条数
int count = jdbcTemplate.queryForObject("select count(*) from account where money>?",Integer.class,500);
System.out.println(count);
```

# 在dao中使用JdbcTemplate

## 实体类
- Account
```java
public class Account {
    public int id;
    public String name;
    public float money;

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

    public float getMoney() {
        return money;
    }

    public void setMoney(float money) {
        this.money = money;
    }

    @Override
    public String toString() {
        return "Account{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", money=" + money +
                '}';
    }
}
```

## 账户接口
- IAccountDao
```java
public interface IAccountDao {
    /**
     * 根据id查询账户信息
     * @param id
     * @return
     */
    Account findAccountById(Integer id);

    /**
     * 根据name查询账户信息
     * @param name
     * @return
     */
    Account findAccountByName(String name);
    /**
     * 更新账户信息
     * @param account
     */
    void updateAccount(Account account);
}
```

## 第一种方式:在dao中定义JdbcTemplate

账户持久层实现类
- AccountDaoImpl
```java
public class AccountDaoImpl implements IAccountDao {
    JdbcTemplate jdbcTemplate;

    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    /**
     * 根据id查询账户信息
     *
     * @param id
     * @return
     */
    @Override
    public Account findAccountById(Integer id) {
        List<Account> list = jdbcTemplate.query("select * from account where id=?",new AccountRowMapper(),id);
        return list.isEmpty()?null:list.get(0);
    }

    /**
     * 根据name查询账户信息
     *
     * @param name
     * @return
     */
    @Override
    public Account findAccountByName(String name) {
        List<Account> list = jdbcTemplate.query("select * from account where name=?",new AccountRowMapper(),name);
        if (list.isEmpty()){
            return null;
        }
        if (list.size()>1){
            throw new RuntimeException("结果集不唯一");
        }
        return list.get(0);
    }

    /**
     * 更新账户信息
     *
     * @param account
     */
    @Override
    public void updateAccount(Account account) {
        jdbcTemplate.update("update account set money=?,name=? where id=?",account.getMoney(),account.getName(),account.getId());
    }
}

class AccountRowMapper implements RowMapper<Account> {
    @Override
    public Account mapRow(ResultSet resultSet, int i) throws SQLException {
        Account account = new Account();
        account.setId(resultSet.getInt("id"));
        account.setName(resultSet.getString("name"));
        account.setMoney(resultSet.getFloat("money"));
        return account;
    }
}
```
 - 配置
```xml
<!--配置dao-->
<bean id="accountDao" class="club.krislin.dao.impl.AccountDaoImpl">
    <!--注入JdbcTemplate-->
    <property name="jdbcTemplate" ref="jdbcTemplate"/>
</bean>
```

 - 测试
```java
    // 获取dao
    @Resource(name = "accountDao")
    private IAccountDao accountDao;
    @Test
    public void testDao(){
        Account account = accountDao.findAccountById(11);
        System.out.println(account);

        Account account1 = accountDao.findAccountByName("krislin");
        System.out.println(account1);

        Account account2 = new Account();
        account2.setId(11);
        account2.setName("wwww");
        account2.setMoney(2344f);
        accountDao.updateAccount(account2);
    }
```

## 第二种方式:让dao继承JdbcDaoSupport

JdbcDaoSupport 是 spring 框架为我们提供的一个类，该类中定义了一个 JdbcTemplate 对象，我们可以
直接获取使用，但是要想创建该对象，需要为其提供一个数据源,具体源码如下：
```java
public abstract class JdbcDaoSupport extends DaoSupport {
//定义对象
private JdbcTemplate jdbcTemplate;
//set 方法注入数据源，判断是否注入了，注入了就创建 JdbcTemplate
public final void setDataSource(DataSource dataSource) {
if (this.jdbcTemplate == null || dataSource != this.jdbcTemplate.getDataSource())
{ //如果提供了数据源就创建 JdbcTemplate
this.jdbcTemplate = createJdbcTemplate(dataSource);
initTemplateConfig();
}
}
//使用数据源创建 JdcbTemplate
protected JdbcTemplate createJdbcTemplate(DataSource dataSource) {
return new JdbcTemplate(dataSource);
}
//当然，我们也可以通过注入 JdbcTemplate 对象
public final void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
this.jdbcTemplate = jdbcTemplate;
initTemplateConfig();
}
//使用 getJdbcTmeplate 方法获取操作模板对象
public final JdbcTemplate getJdbcTemplate() {
return this.jdbcTemplate;
}
```

账户持久层实现类
- AccountDaoImpl2
```java
public class AccountDaoImpl2 extends JdbcDaoSupport implements IAccountDao{
    /**
     * 根据id查询账户信息
     *
     * @param id
     * @return
     */
    @Override
    public Account findAccountById(Integer id) {
        List<Account> list = getJdbcTemplate().query("select * from account where id=?",new AccountRowMapper(),id);
        return list.isEmpty()?null:list.get(0);
    }

    /**
     * 根据name查询账户信息
     *
     * @param name
     * @return
     */
    @Override
    public Account findAccountByName(String name) {
        List<Account> list = getJdbcTemplate().query("select * from account where name=?",new AccountRowMapper(),name);
        if (list.isEmpty()){
            return null;
        }
        if (list.size()>1){
            throw new RuntimeException("结果集不知一个");
        }
        return list.get(0);
    }

    /**
     * 更新账户信息
     *
     * @param account
     */
    @Override
    public void updateAccount(Account account) {
        getJdbcTemplate().update("update account set name=?,money=? where id=?",account.getName(),account.getMoney(),account.getId());
    }
}
```
- 配置
```xml
<!--配置dao2-->
<bean id="accountDao2" class="club.krislin.dao.impl.AccountDaoImpl2">
    <!--注入dataSource-->
    <property name="dataSource" ref="dataSource"/>
</bean>
```

- 测试
```java
// 获取dao2
@Resource(name = "accountDao2")
IAccountDao accountDao2;
@Test
public void testDao2(){
    Account account = accountDao2.findAccountById(11);
    System.out.println(account);

    Account account1 = accountDao2.findAccountByName("krislin");
    System.out.println(account1);

    Account account2 = new Account();
    account2.setId(11);
    account2.setName("fff");
    account2.setMoney(2344f);
    accountDao2.updateAccount(account2);
}
```

## 两版 Dao 有什么区别呢？
1. 第一种在 Dao 类中定义 JdbcTemplate 的方式，适用于所有配置方式（ xml 和注解都可以）。
2. 第二种让 Dao 继承 JdbcDaoSupport 的方式，只能用于基于 XML 的方式，注解用不了。