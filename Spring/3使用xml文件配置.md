## 导入jar包(坐标)
```java
<packaging>jar</packaging>

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
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
## 编写实体类User
```java
public class User implements Serializable {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", birthday=" + birthday +
                ", sex='" + sex + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}
```
## 编写业务层实现类
```java
public class UserServiceImp implements UserService {

    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public List<User> findAllUsers() {
        return userDao.findAllUsers();
    }

    public User findUserById(Integer userId) {
        return userDao.findUserById(userId);
    }

    public void updateUser(User user) {
        userDao.updateUser(user);
    }

    public void addUser(User user) {
        userDao.addUser(user);
    }

    public void delUser(Integer userId) {
        userDao.delUser(userId);
    }
}
```
## 编写持久层实现类
```java
public class UserDaoImp implements UserDao {

    private QueryRunner queryRunner;

    public void setQueryRunner(QueryRunner queryRunner) {
        this.queryRunner = queryRunner;
    }

    public List<User> findAllUsers() {
        try {
            return queryRunner.query("select * from mybatisuser", new BeanListHandler<User>(User.class));
        }catch (Exception e){
            throw new RuntimeException(e);
        }

    }

    public User findUserById(Integer userId) {
        try {
            return queryRunner.query("select * from mybatisuser where id = ?", new BeanHandler<User>(User.class),userId);
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public void addUser(User user) {
        try {
            queryRunner.insert("insert into mybatisuser(username,birthday,sex,address) values(?,?,?,?)",new BeanHandler<User>(User.class),user.getUsername(),user.getBirthday(),user.getSex(),user.getAddress());
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public void updateUser(User user) {
        try {
            queryRunner.update("update mybatisuser set username=? ,birthday=?,sex=?,address=? where id=?",user.getUsername(),user.getBirthday(),user.getSex(),user.getAddress(),user.getId());
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public void delUser(Integer userId) {
        try {
            queryRunner.update("delete from mybatisuser where id=?",userId);
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }
}
```
## 配置bean.xml
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--配置service-->
    <bean id="userService" class="com.whut.service.imp.UserServiceImp">
        <!-- 注入Dao-->
        <property name="userDao" ref="userDao"></property>
    </bean>

    <!--配置Dao对象-->
    <bean id="userDao" class="com.whut.dao.imp.UserDaoImp">
        <!--注入QueryRunner-->
        <property name="queryRunner" ref="queryRunner"></property>
    </bean>

    <!--配置QueryRunner-->
    <bean id="queryRunner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype">
        <!--注入数据源-->
        <constructor-arg name="ds" ref="dataSource"></constructor-arg>
    </bean>

    <!--配置数据源-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <!--连接数据库必备信息-->
        <property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/my_db?serverTimezone=UTC"/>
        <property name="user" value="root"/>
        <property name="password" value="fy123"/>
    </bean>
</beans>
```
## 测试方法
```java
public class UserServiceTset {

    @Test
    public void testFindAllUsers(){
        //1. 获取容器
        ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2. 获取业务层对象
        UserService userService = ac.getBean("userService",UserService.class);
        //3. 执行方法
        List<User> allUsers = userService.findAllUsers();
        for (User user : allUsers){
            System.out.println(user);
        }
    }
    @Test
    public void testFindUserById(){
        //1. 获取容器
        ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2. 获取业务层对象
        UserService userService = ac.getBean("userService",UserService.class);
        //3. 执行方法
        User user = userService.findUserById(2);
        System.out.println(user);
    }
    @Test
    public void testAddUser(){
        User user = new User();
        user.setUsername("莫提斯");
        user.setBirthday(new Date());
        user.setSex("男");
        user.setAddress("神话突击者");
        //1. 获取容器
        ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2. 获取业务层对象
        UserService userService = ac.getBean("userService",UserService.class);
        //3. 执行方法
        userService.addUser(user);
    }
    @Test
    public void testUpdateUser(){
        User user = new User();
        user.setUsername("柯莱特");
        user.setBirthday(new Date());
        user.setSex("男");
        user.setAddress("流彩战士");
        user.setId(7);
        //1. 获取容器
        ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2. 获取业务层对象
        UserService userService = ac.getBean("userService",UserService.class);
        //3. 执行方法
        userService.updateUser(user);
    }
    @Test
    public void testDelUser(){
        //1. 获取容器
        ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2. 获取业务层对象
        UserService userService = ac.getBean("userService",UserService.class);
        //3. 执行方法
        userService.delUser(8);
    }
}
```