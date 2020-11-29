## 半注解实现
1. 配置spring中基于注解的申明事务控制的配置步骤
   1. 配置事务管理器
   2. 开启spring对注解事务的支持
   3. 在需要事务支持的地方使用@Transactional
   4. 建立事务通知和切入点表达式的关系
   5. 配置事务的属性：是在事务通知tx:advice标签内部
2. bean.xml
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!--配置spring创建容器时需要扫描的包-->
    <context:component-scan base-package="com.whut"/>
    <!--配置jdbcTemplate-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
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

    <!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--开启spring对注解事务的支持-->
    <tx:annotation-driven transaction-manager="transactionManager"/>

</beans>
```
3. 业务层
```java
@Service("accountService")
@Transactional(propagation = Propagation.SUPPORTS, readOnly = true) //只读型事务的配置
public class AccountServiceImp implements AccountService {

    @Autowired
    private AccountDao accountDao;


    @Override
    public Account findAccountById(int id) {
        return accountDao.findAccountById(id);
    }

    @Transactional(propagation = Propagation.REQUIRED, readOnly = false)    //读写型事务的配置
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
4. 持久层
```java
@Repository("accountDao")
public class AccountDaoImp implements AccountDao {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    public Account findAccountByName(String name) {
        try {
            List<Account> accounts = jdbcTemplate.query("select * from springaccount where name=?", new BeanPropertyRowMapper<Account>(Account.class),name);
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
            jdbcTemplate.update("update springaccount set name=? ,account=? where id=?",account.getName(),account.getAccount(),account.getId());
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    @Override
    public Account findAccountById(int id) {
        try {
            List<Account> accounts = jdbcTemplate.query("select * from springaccount where id=?", new BeanPropertyRowMapper<Account>(Account.class),id);
            return accounts.isEmpty() ? null : accounts.get(0);
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }
}
```
5. 测试方法
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:bean.xml")
public class AccountServiceTest {

    @Autowired
    private AccountService accountService;
    @Test
    public void testFindAllUsers(){

        accountService.transfer("张三","李四",100);
        Account user1 = accountService.findAccountById(1);
        Account user2 = accountService.findAccountById(2);
        System.out.printf(user1.toString());
        System.out.printf(user2.toString());

    }
}
```
## 纯注解实现
1. 创建spring的配置类
```java
@Configuration
@ComponentScan("com.whut")
@Import({JdbcConfig.class,TransactionConfig.class})
@PropertySource("classpath:jdbcConfig.properties")
@EnableTransactionManagement
public class SpringConfiguration {

}
```
2. 创建jdbc配置类
```java
public class JdbcConfig {

    @Value("${jdbc.driver}")
    private String driverClass;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String username;
    @Value("${jdbc.password}")
    private String password;

    /**
     * 创建JdbcTemplate
     * @param dataSource
     * @return
     */
    @Bean(name = "jdbcTemplate")
    public JdbcTemplate createJdbcTemplate(DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }

    /**
     * 创建DataSource
     * @return
     */
    @Bean(name = "dataSource")
    public DataSource createDataSource(){
        DriverManagerDataSource ds = new DriverManagerDataSource();
        ds.setDriverClassName(driverClass);
        ds.setUrl(url);
        ds.setUsername(username);
        ds.setPassword(password);
        return ds;
    }
}
```
3. 创建事务配置类
```java
public class TransactionConfig {
    @Bean(name = "transactionManager")
    public PlatformTransactionManager creatTransactionManager(DataSource dataSource){
        return new DataSourceTransactionManager(dataSource);
    }
}
```
4. 测试方法
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfiguration.class)
public class AccountServiceTest {

    @Autowired
    private AccountService accountService;
    @Test
    public void testFindAllUsers(){

        accountService.transfer("张三","李四",-800);
        Account user1 = accountService.findAccountById(1);
        Account user2 = accountService.findAccountById(2);
        System.out.printf(user1.toString());
        System.out.printf(user2.toString());

    }
}
```