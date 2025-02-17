# Spring学习笔记(8)AOP的底层实现

---

[TOC]

---

# 一.JDK动态代理

![](../../images/SpringAOP的底层实现JDK.jpg)

## 1.创建接口
- UserDao
```java
public interface UserDao {
    public void add();
    public void update();
}
```

## 2.创建接口
- UserDaoImpl
```java
public class UserDaoImpl implements UserDao {
    @Override
    public void add() {
        System.out.println("添加用户");
    }

    @Override
    public void update() {
        System.out.println("更新用户");
    }
}
```

## 3.创建生成代理的类
- JavaJDKProxy
```java
public class JavaJDKProxy {
    private UserDao userDao;

    public JavaJDKProxy(UserDao userDao){
        super();
        this.userDao=userDao;
    }

    /**
     * 创建userDao的代理类
     * 工厂模式
     * @return
     */
    public UserDao createProxy(){
        UserDao userDaoProxy =(UserDao) Proxy.newProxyInstance(userDao.getClass().getClassLoader(), userDao.getClass().getInterfaces(), new InvocationHandler() {
            /**
             * InvocationHandler 生成方式：
             * 1.匿名内部类的形式
             * 2.直接让当前类谁实现 InvocationHandler 接口， 然后创建代理的 InvocationHandler 写 this
             * 3.重写写一个类实现 InvocationHandler 接口， 在本类中通过的 new 的方式创建
             * @param proxy
             * @param method
             * @param args
             * @return
             * @throws Throwable
             */
            //代用目标对象的任何一个方法都相当于代用 invoke
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                //如果执行的 add 方法， 那么就记录日志
                if ("add".equals(method.getName())){
                    // 记录日志
                    System.out.println("------记录日志-----");
                    //然后执行 add 方法
                    /**
                     * method.invoke(obj,args): 对带有指定参数的指定对象调用由此 Method
                     * 对象表示的底层方法。
                     * 参数一： obj - 从中调用底层方法的对象
                     * 参数二： args 用于方法调用的参数
                     * 返回值： result:使用参数 args 在 obj 上指派该对象所表示方法的结果
                     */
                    Object result = method.invoke(userDao,args);
                    return result;
                }
                return method.invoke(userDao,args);
            }
        });
        return userDaoProxy;
    }
}
```

## 4.测试
- UserDaoTest
```java
public class UserDaoTest {
    @Test
    public void testUserDao(){
        // 生成目标对象
        UserDao userDao = new UserDaoImpl();

        // 创建代理对象
        UserDao proxy = new JavaJDKProxy(userDao).createProxy();

        //proxy.add();
        proxy.update();
    }
}
```

# 二.CGLIB 动态代理
CGLIB(Code Generation Library)是一个开源项目！ 是一个强大的， 高性能， 高质量的 Code 生成类库， 它可以
在运行期扩展 Java 类与实现 Java 接口。 Hibernate 支持它来实现 PO(Persistent Object 持久化对象)字节码的
动态生成
Hibernate 生成持久化类的 javassist.
CGLIB 生成代理机制:其实生成了一个真实对象的子类.（ 只能对类生成代理）
下载 cglib 的 jar 包.
* 现在做 cglib 的开发,可以不用直接引入 cglib 的包.已经在 spring 的核心中集成 cglib.

![](../../images/SpringAOP的底层实现CGLIB.jpg)

## 1.创建类
- ProductDao
```java
public class ProductDao {
    public void add(){
        System.out.println("增加商品");
    }

    public void update(){
        System.out.println("更新商品");
    }
}
```

## 2.生成代理类
- CglibProxy
```java
public class CglibProxy implements MethodInterceptor{
    private ProductDao productDao;

    public CglibProxy(ProductDao productDao) {
        this.productDao = productDao;
    }

    public ProductDao createProxy(){
        //1.使用CGLIB生成代理
        Enhancer enhancer = new Enhancer();
        //2.为其设置父类(因为采用继承机制,所以的指定父类)
        enhancer.setSuperclass(productDao.getClass());
        //3.设置回调:和javaProxyInvoke相似
        enhancer.setCallback(this);
        //4.创建代理
        return (ProductDao) enhancer.create();
    }

    /**
     *
     * @param o CGLIB根据指定父类生成的代理对象(是ProductDao的子类)
     * @param method 拦截的方法
     * @param objects 拦截方法的参数数组
     * @param methodProxy 方法的代理对象,用于执行父类的方法
     * @return
     * @throws Throwable
     */
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        if ("add".equals(method.getName())){
            //在执行 add 方法前执行记录日志
            System.out.println("----记录日志----");
            // 执行add方法
            Object result = methodProxy.invokeSuper(o,objects);
            // 执行add方法后的返回值
            return result;
        }
        // 执行的其他方法
        return methodProxy.invokeSuper(o,objects);
    }
}
```

## 3.测试
- ProductDaoTest
```java
public class ProductDaoTest {
    @Test
    public void testProductDao(){
        // 目标对象
        ProductDao productDao = new ProductDao();
        // 生成代理对象
        ProductDao proxy = new CglibProxy(productDao).createProxy();

        proxy.add();
        proxy.update();
    }
}
```

# 结论
Spring 框架,如果类实现了接口,就使用 JDK 的动态代理生成代理对象,如果这个类没有实现任何接口,使用CGLIB 生成代理对象.（ 底层会自动切换）


# spring 代理知识总结
Spring 在运行期， 生成动态代理对象， 不需要特殊的编译器
Spring AOP 的底层就是通过 JDK 动态代理或 CGLib 动态代理技术 为目标Bean 执行横向织入
- 1.若目标对象实现了若干接口， spring 使用 JDK 的 `java.lang.reflect.Proxy` 类代理。
- 2.若目标对象没有实现任何接口， spring 使用 CGLIB 库生成目标对象的子类。

- 程序中应优先对接口创建代理， 便于程序解耦维护
- 标记为 final 的方法， 不能被代理， 因为无法进行覆盖
- JDK 动态代理， 是针对接口生成子类， 接口中方法不能使用 final 修饰
- CGLib 是针对目标类生产子类， 因此类或方法 不能使 final 的
- Spring 只支持方法连接点， 不提供属性连接