## 开启注解AOP的支持
@EnableAspectJAutoProxy
```java
@ComponentScan("com.whut")
@Import(JdbcConfiguration.class)
@EnableAspectJAutoProxy
@PropertySource("classpath:jdbcConfig.properties")
public class SpringConfiguration {
}
```
## 切面注解
@Aspect
```java
@Component("txManager")
@Aspect
public class TransactionManager {

    @Pointcut("execution(* com.whut.service.imp.*.*(..))")
    private void pt(){}

    @Autowired
    private ConnectionUtils connectionUtils;

    @Before("pt()")
    public void beginTransaction(){
        try{
            System.out.println("开启事务");
            connectionUtils.getThreadConnection().setAutoCommit(false);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    @AfterReturning("pt()")
    public void commit(){
        try{
            System.out.println("提交");
            connectionUtils.getThreadConnection().commit();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    @AfterThrowing("pt()")
    public void rollback(){
        try{
            System.out.println("回滚");
            connectionUtils.getThreadConnection().rollback();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    @After("pt()")
    public void release(){
        try{
            System.out.println("释放连接");
            connectionUtils.getThreadConnection().close();
            connectionUtils.removeConnection();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
//    @Around("pt()")
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
}
```