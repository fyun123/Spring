## 相关概念
1. Jointpoint（连接点）：是指那些被拦截到的点，在spring中，这些点指的是方法，因为spring只支持方法类型的连接点。
2. Pointcut（切入点）：是指我们要对哪些Jointpoint进行拦截的定义。
3. Advice（通知/增强）：是指拦截到Jointpoint之后所要做的事情就是通知。通知的类型：前置通知、后置通知、异常通知、最终通知、环绕通知。
4. Introduction（引介）：是一种特殊的通知在不修改代码的前提下，Introduction可以在运行期为类动态地添加一些方法或Field。
5. Target（目标对象）：代理的目标对象。
6. Weaving（织入）：是指把增强应用到目标对象来创建新的代理对象的过程。spring采用动态代理织入，而AspectJ采用编译器织入和类装载期织入。
7. Proxy（代理）：一个类被AOP织入增强后，就产生一个结果代理类。
8. Aspect（切面）：是切入点和通知（引介）的结合。
## 导入依赖jar包
```java
<packaging>jar</packaging>

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.9.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.6</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>5.2.9.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>commons-dbutils</groupId>
        <artifactId>commons-dbutils</artifactId>
        <version>1.7</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.21</version>
    </dependency>
    <dependency>
        <groupId>c3p0</groupId>
        <artifactId>c3p0</artifactId>
        <version>0.9.1.2</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```
## 编写工具类
1. 编写连接的工具类
它用于从数据源中获取一个连接，并且实现和线程的绑定
```java
public class ConnectionUtils {

    private ThreadLocal<Connection> tl = new ThreadLocal<>();

    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    /**
     * 获取当前线程上的连接
     * @return
     */
    public Connection getThreadConnection() {
        try{
            //1.先从ThreadLocal上获取
            Connection conn = tl.get();
            //2.判断当前线程上是否有连接
            if (conn == null) {
                //3.从数据源中获取一个连接，并且存入ThreadLocal中
                conn = dataSource.getConnection();
                tl.set(conn);
            }
            //4.返回当前线程上的连接
            return conn;
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    /**
     * 把连接和线程解绑
     */
    public void removeConnection(){
        tl.remove();
    }
}
```
2. 编写和事务管理相关工具类
开启事务，提交事务，回滚事务，释放连接
```java
public class TransactionManager {

    private ConnectionUtils connectionUtils;

    public void setConnectionUtils(ConnectionUtils connectionUtils) {
        this.connectionUtils = connectionUtils;
    }

    public void beginTransaction(){
        try{
            connectionUtils.getThreadConnection().setAutoCommit(false);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    public void commit(){
        try{
            connectionUtils.getThreadConnection().commit();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    public void rollback(){
        try{
            connectionUtils.getThreadConnection().rollback();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    public void release(){
        try{
            connectionUtils.getThreadConnection().close();
            connectionUtils.removeConnection();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```
## 配置xml文件
1. 配置spring中基于xml的配置步骤
   1. 把通知bean也交给spring来管理
   2. 使用aop:config标签来表明开始aop的配置
   3. 使用aop:aspect标签表明配置切面
        id属性：是给切面提供一个唯一标识
        ref属性：是指通知类bean的id
   4. 在aop:aspect标签内部使用对应标签来配置通知的类型
      1. aop:before：表示配置前置通知
      2. method属性：用于指定类中哪个方法是前置通知
      3. pointcut属性：用于指定切入点表达式，该表达式的含义指的是对业务层中哪些方法增强
      4. 切入点表达式的写法：
         1. 关键字：execution（表达式）
         2. 表达式：
            1. 访问修饰符   返回值     包名.包名....类名.方法名(参数列表)
            2. 标准的表达式写法：public void com.whut.service.imp.AccountServiceImp.transfer(String,String,float)
            3. 访问修饰符可以省略：void com.whut.service.imp.AccountServiceImp.transfer(String,String,float)
            4. 返回值可以使用通配符，表示任意返回值：* com.whut.service.imp.AccountServiceImp.transfer(String,String,float)
            5. 包名可以使用通配符，表示任意包，但是有几个包需要写几个*.：* *.*.*.*.AccountServiceImp.transfer(String,String,float)
            6. 包名可以使用..表示当前包及其子包：* *..AccountServiceImp.transfer(String,String,float)
            7. 类名和方法名都可以使用\*来实现通配：* *..*.*(String,String,float)
            8. 参数列表：
               1. 可以直接写数据类型
                  1. 基本类型直接写名称     int
                     1. 引用类型写包名.类名的方式   java.lang.String
               2. 可以使用通配符，但是必须要有参数：* *..*.*(*,*,*)
               3. 可以使用..表示有无参数均可，有参数可以是任意类型
            9. 全通配写法：* *..*.*(..)
            10. 实际开发中通常写法：切到业务层下的所有方法
                1.  * com.whut.service.imp.*.*(..))
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--配置spring的ioc，把service对象配置进来-->
    <bean id="accountService" class="com.whut.service.imp.AccountServiceImp">
        <!-- 注入Dao-->
        <property name="accountDao" ref="accountDao"></property>
    </bean>
    <!--配置Dao对象-->
    <bean id="accountDao" class="com.whut.dao.imp.AccountDaoImp">
        <!--注入QueryRunner-->
        <property name="queryRunner" ref="queryRunner"></property>
        <!--注入ConnectionUtils -->
        <property name="connectionUtils" ref="connectionUtils"></property>
    </bean>
    <!--配置QueryRunner-->
    <bean id="queryRunner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype"></bean>

    <bean id="txManager" class="com.whut.utils.TransactionManager">
        <!--注入ConnectionUtils -->
        <property name="connectionUtils" ref="connectionUtils"></property>
    </bean>
    <bean id="connectionUtils" class="com.whut.utils.ConnectionUtils">
        <!--注入数据源-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <!--配置数据源-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <!--连接数据库必备信息-->
        <property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/my_db?serverTimezone=UTC"/>
        <property name="user" value="root"/>
        <property name="password" value="fy123"/>
    </bean>
    <!--配置aop-->
    <aop:config>
        <!--配置切面-->
        <aop:aspect id="transactionAdvice" ref="txManager">
            <aop:before method="beginTransaction" pointcut="execution(public void com.whut.service.imp.AccountServiceImp.transfer(..))"></aop:before>
            <aop:after-returning method="commit" pointcut="execution(public void com.whut.service.imp.AccountServiceImp.transfer(..))"></aop:after-returning>
            <aop:after-throwing method="rollback" pointcut="execution(public void com.whut.service.imp.AccountServiceImp.transfer(..))"></aop:after-throwing>
            <aop:after method="release" pointcut="execution(public void com.whut.service.imp.AccountServiceImp.transfer(..))"></aop:after>
        </aop:aspect>
    </aop:config>
</beans>
```
## 通用化切入点表达式
1. 此标签写在aop:aspect内部，只能当前切面使用
2. 此标签写在aop:aspect外部，所有切面使用
```java
<aop:config>
    <!--配置切面-->
    <aop:aspect id="transactionAdvice" ref="txManager">
        <!--配置切入点表达式-->
        <aop:pointcut id="pt" expression="execution(* com.whut.service.imp.*.*(..))"/>
        <aop:before method="beginTransaction" pointcut-ref="pt"/>
        <aop:after-returning method="commit" pointcut-ref="pt"/>
        <aop:after-throwing method="rollback" pointcut-ref="pt"/>
        <aop:after method="release" pointcut-ref="pt"/>
    </aop:aspect>
</aop:config>
```
## 环绕通知
1. 配置aop
```java
<aop:config>
        <!--配置切面-->
        <aop:aspect id="transactionAdvice" ref="txManager">
            <aop:pointcut id="pt" expression="execution(* com.whut.service.imp.*.*(..))"/>
            <aop:around method="aroundAdvice" pointcut-ref="pt"/>
        </aop:aspect>
    </aop:config>
```
2. 在事务管理类中添加方法
```java
public Object aroundAdvice(ProceedingJoinPoint proceedingJoinPoint) throws SQLException {
    Object rtValue = null;
    try{
        Object[] args = proceedingJoinPoint.getArgs();

        System.out.println("开启事务");
        connectionUtils.getThreadConnection().setAutoCommit(false);

        rtValue = proceedingJoinPoint.proceed(args);//明确调用业务层方法（切入点方法）

        System.out.println("提交");
        connectionUtils.getThreadConnection().commit();

        return rtValue;
    }catch (Throwable t){
        System.out.println("回滚");
        connectionUtils.getThreadConnection().rollback();
        throw new RuntimeException(t);
    }finally {
        System.out.println("释放连接");
        connectionUtils.getThreadConnection().close();
        connectionUtils.removeConnection();
    }
}
```