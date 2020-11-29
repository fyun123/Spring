## 配置文件bean.xml
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">
</bean>
```
## 创建Bean的三种方式
1. 使用默认构造函数创建
在Spring的配置文件中使用bean标签，配以id和class属性之后，且没有其他属性和标签时。采用的就是默认构造函数创建bean对象，此时如果类中没有默认构造函数，则对象无法创建。
```java
<bean id="accountService" class="com.whut.service.imp.AccountServiceImp"></bean>
```
2. 使用普通工厂中的方法创建对象（使用某个类中的方法创建对象，并存入spring容器）
```java
<bean id="instanceFactory" class="com.whut.factory.InstanceFactory"></bean>
<bean id="accountService" factory-bean="instanceFactory" factory-method="getAccountService"></bean>
```
3. 使用工厂中的静态方法创建对象（使用某个类中的静态方法创建对象，并存入spring容器）
```java
<bean id="accountService" class="com.whut.factory.StaticFactory" factory-method="getAccountService"></bean>
```
## bean的作用范围
1. bean标签的scope属性
   1. 作用：用于指定bean的作用范围
   2. 取值：常用的就是单例的和多例的
      1. singleton：单例的（默认值）
      2. prototype：多例的
      3. request：作用于web应用的请求范围
      4. session：作用于web应用的会话范围
      5. global-session：作用于集群环境的会话范围（全局会话范围），当不是集群环境时，它就是session
        ```java
         <bean id="accountService" class="com.whut.service.imp.AccountServiceImp" scope="prototype"></bean>
        ```
## bean对象的生命周期
1. 单例对象
   1. 出生：当容器创建时对象出生
   2. 活着：只要对象还存在，对象一直活着
   3. 死亡：容器销毁，对象消亡
   4. 总结：单例对象的生命周期和容器相同
2. 多例对象
   1. 出生：当我们使用对象时spring对象为我们创建
   2. 活着：对象只要是在使用过程单中就一直活着
   3. 死亡：当对象长时间不用，且没有别的对象引用时，由java的垃圾回收机制回收
```java
<bean id="accountService" class="com.whut.service.imp.AccountServiceImp" scope="singleton" init-method="init" destroy-method="destroy"></bean>
```
单例对象时，需手动关闭容器，才会执行销毁