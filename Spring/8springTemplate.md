## 持久层
```java
public class AccountDaoImp extends JdbcDaoSupport implements AccountDao{

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
            getJdbcTemplate().update("update springaccount set name=? ,account=? where id=?",new BeanPropertyRowMapper<Account>(Account.class),account.getName(),account.getAccount(),account.getId());
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }
}
```
## 手写JdbcDaoSupport
```java
public class JdbcDaoSupport {
    /**
     * 此类用于抽取dao中的重复代码
     */
    private JdbcTemplate jdbcTemplate;


    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public JdbcTemplate getJdbcTemplate() {
        return jdbcTemplate;
    }

    public void setDataSource(DataSource dataSource) {
        if (jdbcTemplate == null){
            jdbcTemplate = createJdbcTemplate(dataSource);
        }
    }

    private JdbcTemplate createJdbcTemplate(DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }
}
```
* 注：持久层也可以直接继承JdbcDaoSupport，导入包import org.springframework.jdbc.core.support.JdbcDaoSupport;
* 两种方式的区别是，自己写的JdbcDaoSupport类可以使用注解注入