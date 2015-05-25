事务
====

我们以一个交易的事例来说明如何在mango框架中使用事务。

首先我们需要一张账户表来存储用户的账户数据，其中：uid是表账户标识，money则是账户余额:

.. code-block:: sql

    CREATE TABLE `accounts` (
	    `uid` int(11) NOT NULL,
	    `money` int(11) NOT NULL,
	     PRIMARY KEY (`uid`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8

接着书写AccountsDao:

.. code-block:: java

    @DB
    public interface AccountsDao {

	    @SQL("update accounts set money = money + :2 where uid = :1")
	    boolean addMoney(int uid, int inc);

    }

最后是使用事务的代码:

.. code-block:: java

	public class AccountsDaoRunner {

	    public static void main(String[] args) {
	        String driverClassName = "com.mysql.jdbc.Driver";
	        String url = "jdbc:mysql://localhost:3306/mango_example";
	        String username = "root"; // 这里请使用您自己的用户名
	        String password = "root"; // 这里请使用您自己的密码
	        DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
	        Mango mango = Mango.newInstance(ds);

	        AccountsDao dao = mango.create(AccountsDao.class);

	        int zhangsan = 1;
	        int lisi = 2;
	        int money = 100;

	        Transaction tx = TransactionFactory.newTransaction();
	        try {
	            dao.addMoney(zhangsan, -money);
	            dao.addMoney(lisi, money);
	            tx.commit();
	        } catch (Throwable e) {
	            tx.rollback();
	        }
	    }

	}

上面的代码中zhangsan（张三）转给lisi（李四）100块。如果没有异常则事务提交，有异常则事务回滚。




