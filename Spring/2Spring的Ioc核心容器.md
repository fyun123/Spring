## ApplicationContext的三个常用实现类
1. ClassPathXmlApplicationContext：它可以加载类路径下的配置文件，要求配置文件必须在类路径下。不在的话，加载不了。（更常用）
2. FileSystemXmlApplicationContext：它可以加载磁盘任意路径下的配置文件（必须有访问权限）
3. AnnotationConfigApplicationContext：它是用于读取注释创建容器的。
## 核心容器的两个接口引发出的问题
1. ApplicationContext：（单例对象适用）
   1. 它在构建核心容器时，创建对象采取的策略是采用立即加载的方式。也就是说，只要一读完配置文件马上就创建配置文件中配置的对象。
2. BeanFactory：（多例对象适用）
   1. 它在构建核心容器时，创建对象采取的策略是采用延迟加载的方式。也就是说，什么时候根据Id获取对象了，什么时候才真正创建对象。
## spring中的依赖注入
1. 依赖注入：Dependency Injection
2. IOC的作用：降低程序间的耦合（依赖关系）
3. 依赖关系的管理：以后都交给spring来维护
4. 在当前类需要用到其他类的对象，由spring为我们提供，我们只需要在配置文件中说明依赖关系的维护，就称之为依赖注入。
5. 依赖注入
   1. 能注入的数据，有三类
       1. 基本类型和String
       2. 其他bean类型（在配置文件中或者注解配置过的bean）
       3. 复杂类型/集合类型
   2. 注入的方式：有三种
       1. 使用构造函数提供
           1. 使用标签：constructor-arg
           2. 标签出现的位置：bean标签的内部
           3. 标签中的属性：
               1. type：用于指定注入的数据的数据类型，该数据类型也是构造函数中某个或某些参数的类型
               2. index：用于指定要注入的数据给构造函数中指定索引位置的参数赋值。索引的位置从0开始
               3. name：用于指定给构造函数中指定名称的参数赋值（常用）
               4. value：用于提供基本类型和String类型的数据
               5. ref：用于指定其他的bean类型数据。它指的就是在spring的IOC核心容器中出现过的bean对象。
            4. 优势：在获取bean对象时，注入数据是必须的操作，否则对象无法创建成功。
            5. 弊端：改变了bean对象的实例化方式，使我们在创建对象时，如果用不到这些数据，也必须提供。
        ```java
        <bean id="userService" class="com.whut.service.imp.UserServiceImp" scope="singleton" init-method="init" destroy-method="destory">
            <constructor-arg name="string" value="张三"></constructor-arg>
            <constructor-arg name="integer" value="12"></constructor-arg>
            <constructor-arg name="date" ref="now"></constructor-arg>
        </bean>
        <bean id="now" class="java.util.Date"></bean>
        <bean id="userDao" class="com.whut.dao.imp.UserDaoImp"></bean>
        ```
        在UserServiceImp中构造含参方法
        ```java
        private String string;
        private Integer integer;
        private Date date;
        public UserServiceImp(String string, Integer integer, Date date){
            this.string = string;
            this.integer = integer;
            this.date = date;
        }
        ```
       2. 使用set方法提供（更常用）
           1. 涉及的标签：property
           2. 出现的位置：bean标签的内部
           3. 标签的属性
              1. name：用于指定给构造函数中指定名称的参数赋值（常用）。
              2. value：用于提供基本类型和String类型的数据。
              3. ref：用于指定其他的bean类型数据。它指的就是在spring的IOC核心容器中出现过的bean对象。
           4. 优势：创建对象时没有明确的限制，可以直接使用默认构造函数。
           5. 弊端：如果有某个成员必须有值，则set方法无法保证一定注入，获取对象时有可能set方法没有执行。
            ```java
            <bean id="userService" class="com.whut.service.imp.UserServiceImp" init-method="init" destroy-method="destory">
                <property name="string" value="李四"></property>
                <property name="integer" value="21"></property>
                <property name="date" ref="now"></property>
            </bean>
            <bean id="now" class="java.util.Date"></bean>
            <bean id="userDao" class="com.whut.dao.imp.UserDaoImp"></bean>
            ```
            在UserServiceImp中生成setter方法
            ```java
            private String string;
            private Integer integer;
            private Date date;

            public void setString(String string) {
            this.string = string;
            }

            public void setInteger(Integer integer) {
                this.integer = integer;
            }

            public void setDate(Date date) {
                this.date = date;
            }
            ```
           6. 复杂类型/集合类型的注入
               1. 用于给List结构集合注入的标签
                    list    array   set
               2. 用于给Map结构集合注入的标签
                    map     props  
               3. 结构相同，标签可以互换
            ```java
            <property name="myStrs">
            <array>
                <value>AAA</value>
                <value>张三</value>
                <value>A12</value>
            </array>
            </property>

            <property name="myLsit">
                <list>
                    <value>AAA</value>
                    <value>张三</value>
                    <value>A12</value>
                </list>
            </property>

            <property name="mySet">
                <set>
                    <value>AAA</value>
                    <value>张三</value>
                    <value>A12</value>
                </set>
            </property>

            <property name="myMap">
                <map>
                    <entry key="testA" value="AAA"></entry>
                    <entry key="name" value="张三"></entry>
                    <entry key="age">
                        <value>13</value>
                    </entry>
                </map>
            </property>

            <property name="myProperties">
                <props>
                    <prop key="学历">高中</prop>
                    <prop key="years">2019</prop>
                </props>
            </property>
            ```
       3. 使用注解提供
          1. bean.xml配置
             1. 告知spring在创建容器时要扫描的包，配置所需要的标签不是在beans的约束中，而是一个名称为context名称空间和约束中
                ```java
                <?xml version="1.0" encoding="UTF-8"?>
                <beans xmlns="http://www.springframework.org/schema/beans"
                    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                    xmlns:context="http://www.springframework.org/schema/context"
                    xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context.xsd">

                    <context:annotation-config/>

                </beans>
                ```
          2. 用于创建对象的：他们的作用就和在xml配置文件中编写的一个\<bean>标签是一样的
             1. Component
                1. 作用：用于把当前类对象存入spring
                2. 属性：
                   1. value：用于指定bean的id。当我们不写时，它的默认值是当前类名，且首字母改小写
             2. Controller：一般用在表现层
             3. Service：一般用在业务层
             4. Repository：一般用在持久层
             5. 以上三个注解的作用和属性值与Component是一模一样。他们三个是spring框架为我们提供明确的三层使用的注解，使我们的三层对象更加清晰。
          3. 用于注入数据的：他们的作用就和在xml配置文件中编写的一个\<property>标签是一样的
             1. Autowired
                1. 作用：自动按照类型注入，只要容器中有唯一的一个bean对象类型和要注入的变量类型匹配，就可以注入成功。如果IOC容器中没有任何bean的类型和要注入的变量类型匹配，则报错。如果IOC容器中有多个类型匹配时，再根据变量名进行注入，若没有匹配的变量名，则注入失败。
                2. 出现位置：可以在变量上，也可以是方法上
                3. 细节：在使用注解注入时，set方法就不是必须的了
             2. Qualifier
                1. 作用：在按照类中注入的基础上再按照名称注入。它在给类成员注入时不能单独使用。但是在给方法参数注入时可以
                2. 属性
                   1. value：用于指定注入bean的id。
             3. Resource
                1. 作用：直接按照bean的id注入。它可以单独使用
                2. 属性
                   1. name：用于指定bean的id
             4. 以上三种注入都只能注入其他bean类型的数据，而基本类型和String类型无法使用上述注解实现。另外，集合类型的注入只能通过xml来实现。
             5. value
                1. 作用：用于注入基本类型和String类型的数据
                2. 属性:
                   1. value：用于指定数据的值，它可以使用spring中spEL（也就是spring的el表达式）
                   2. spEL的写法：${表达式}
          4. 用于改变作用范围的：他们的作用就和在xml配置文件中编写的一个\<scope>标签是一样的
             1. scope：
                1. 作用：用于指定bean的作用范围
                2. 属性：value：指定取值的范围（singleton、prototype）
          5. 和生命周期相关的：他们的作用就和在xml配置文件中使用init-method和destroy-method标签是一样的
             1. PerDestroy：用于指定销毁方法
             2. PostConstruct：用于指定初始化方法
          6. 业务层
            ```java
            @Service("userService")
            public class UserServiceImp implements UserService {

                @Autowired
                private UserDao userDao = null;

                public void saveUser() {
                    userDao.saveUser();
                }

            }
            ```
          7. 持久层
            ```java
            @Repository("userDao")
            public class UserDaoImp implements UserDao {

                @Override
                public void saveUser() {
                    System.out.println("保存用户1111111");
                }

            }
            ```
          8. 测试
           ```java
            public class Client {
                public static void main(String[] args) {
                    //1.获取核心容器对象
                    ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("bean.xml");
                    //2. 根据id获取bean对象
                    UserService userService = (UserService) applicationContext.getBean("userService");
                    System.out.println(userService);
                    userService.saveUser();
                }
            }
            ```
          9. 注意
             1.  @Autowired注释写在方法上方报错：Exception in thread "main" org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'userService': Injection of autowired dependencies failed; nested exception is java.lang.NullPointerException
             2.  解决方法：将@Autowired注释写在变量上方
            