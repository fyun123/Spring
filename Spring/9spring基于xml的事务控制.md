## 添加依赖jar包
```java
<packaging>jar</packaging>

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.9.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.2.9.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.21</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-tx</artifactId>
        <version>5.2.9.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>5.2.9.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.6</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```
## 业务层
```java
public class AccountServiceImp implements AccountService {

    private AccountDao accountDao;

    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }

    @Override
    public Account findAccountById(int id) {
        return accountDao.findAccountById(id);
    }

    public void transfer(String sourceName, String targetName, float money) {
        Account sourceAccount = accountDao.findAccountByName(sourceName);
        Account targetAccount = accountDao.findAccountByName(targetName);
        sourceAccount.setAccount(sourceAccount.getAccount() - money);
        targetAccount.setAccount(targetAccount.getAccount() + money);
        accountDao.updateAccount(sourceAccount);
//        int a = 3 /0;
        accountDao.updateAccount(targetAccount);
    }
}
```
## 持久层
```java
public class AccountDaoImp extends JdbcDaoSupport implements AccountDao {


    @Override
    public Account findAccountByName(String name) {
        try {
            List<Account> accounts = getJdbcTemplate().query("select * from springaccount where name=?", new BeanPropertyRowMapper<Account>(Account.class),name);
            if (accounts == null || accounts.size() == 0){
                return null;
            }
            if (accounts.size() > 1){
                throw new RuntimeException("结果不唯一，数据有问题");
            }
            return accounts.get(0);
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    @Override
    public void updateAccount(Account account) {
        try {
            getJdbcTemplate().update("update springaccount set name=? ,account=? where id=?",account.getName(),account.getAccount(),account.getId());
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    @Override
    public Account findAccountById(int id) {
        try {
            List<Account> accounts = getJdbcTemplate().query("select * from springaccount where id=?", new BeanPropertyRowMapper<Account>(Account.class),id);
            return accounts.isEmpty() ? null : accounts.get(0);
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }
}
```
## bean.xml
1. 配置spring中基于xml申明事务控制的配置步骤
   1. 配置事务管理器
   2. 配置事务的通知
      1. 使用tx:advice标签表配置事务通知
         1. id属性：给事务通知提供一个唯一标识
         2. transaction-manager属性：给事务通知提供一个事务管理引用
      2. isolation：用于指定事务的隔离级别，默认值是DEFAULT，表示使用数据库的默认隔离级别。
      3. propagation：用于指定事务的传播行为，默认值是REQUIRED，表示一定会有事务，增删改的选择。查询方法可以选择SUPPORTS。
      4. read-only：用于指定事务是否只读。只有查询方法才能设置true。默认值是false，表示读写。
      5. timeout：用于指定事务的超时时间，默认值是-1，表示永不超时，如果指定了数量
      6. rollback-for：用于指定一个异常，当产生该异常时，事务回滚，产生其他异常时，事务不回滚。没有默认值，表示任何异常都回滚。
      7. no-rollback-for：用于指定一个异常，当产生改异常时，事务不回滚，产生其他异常时事务回滚。没有默认值。表示任何异常都回滚。
   3. 配置AOP中的通用切入点表达式
   4. 建立事务通知和切入点表达式的关系
   5. 配置事务的属性
      1. 是在事务通知tx:advice标签内部
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--配置spring的ioc，把service对象配置进来-->
    <bean id="accountService" class="com.whut.service.imp.AccountServiceImp">
        <!-- 注入Dao-->
        <property name="accountDao" ref="accountDao"/>
    </bean>
    <!--配置Dao对象-->
    <bean id="accountDao" class="com.whut.dao.imp.AccountDaoImp">
        <!--注入dataSource-->
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--配置数据源-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <!--连接数据库必备信息-->
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/my_db?serverTimezone=UTC"/>
        <property name="username" value="root"/>
        <property name="password" value="fy123"/>
    </bean>
    <!--配置spring中基于xml申明事务控制-->
    <!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--配置事务的通知-->
    <!--isolation：用于指定事务的隔离级别，默认值是DEFAULT，表示使用数据库的默认隔离级别。
        propagation：用于指定事务的传播行为，默认值是REQUIRED，表示一定会有事务，增删改的选择。查询方法可以选择SUPPORTS。
        read-only：用于指定事务是否只读。只有查询方法才能设置true。默认值是false，表示读写。
        timeout：用于指定事务的超时时间，默认值是-1，表示永不超时，如果指定了数量
        rollback-for：用于指定一个异常，当产生该异常时，事务回滚，产生其他异常时，事务不回滚。没有默认值，表示任何异常都回滚。
        no-rollback-for：用于指定一个异常，当产生改异常时，事务不回滚，产生其他异常时事务回滚。没有默认值。表示任何异常都回滚。
        -->

    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED" read-only="false"/>
            <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
        </tx:attributes>
    </tx:advice>

    <!--配置AOP-->
    <aop:config>
        <!--配置切入点表达式-->
        <aop:pointcut id="pt" expression="execution(* com.whut.service.imp.*.*(..))"/>
        <!--建立切入点表达式和事务通知的对应关系-->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pt"/>
    </aop:config>
</beans>
```