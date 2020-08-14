事务
====

准备工作
_________

我们以一个交易的事例来说明如何在mango框架中使用事务。

首先我们需要一张账户表来存储用户的账户数据，其中：uid是表账户标识，money则是账户余额:

.. code-block:: sql

    DROP TABLE IF EXISTS `accounts`;
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

编程式事务
__________

下面是使用编程式事务的代码:

.. code-block:: java

	public class AccountsMain {

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

	        if (dao.getAccount(zhangsan) == null) {
	            dao.addAccount(zhangsan, 10000);
	        }
	        if (dao.getAccount(lisi) == null) {
	            dao.addAccount(lisi, 10000);
	        }

	        Transaction tx = TransactionFactory.newTransaction();
	        try {
	            dao.transferMoney(zhangsan, -money);
	            dao.transferMoney(lisi, money);
	        } catch (Throwable e) {
	            tx.rollback();
	            return;
	        }
	        tx.commit();
	    }

	}

上面的代码中zhangsan（张三）转给lisi（李四）100块。如果没有异常则事务提交，有异常则事务回滚。

使用TransactionTemplate
________________________

在直接使用编程式事务时，由于编码较为复杂，容易出错。所以Mango框架提供了TransactionTemplate工具类，用于简化编程式事务的书写。

下面是使用TransactionTemplate的代码:

.. code-block:: java

	public class AccountsMain2 {

	    public static void main(String[] args) {
	        String driverClassName = "com.mysql.jdbc.Driver";
	        String url = "jdbc:mysql://localhost:3306/mango_example";
	        String username = "root"; // 这里请使用您自己的用户名
	        String password = "root"; // 这里请使用您自己的密码
	        DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
	        Mango mango = Mango.newInstance(ds);

	        final AccountsDao dao = mango.create(AccountsDao.class);

	        final int zhangsan = 1;
	        final int lisi = 2;
	        final int money = 100;

	        if (dao.getAccount(zhangsan) == null) {
	            dao.addAccount(zhangsan, 10000);
	        }
	        if (dao.getAccount(lisi) == null) {
	            dao.addAccount(lisi, 10000);
	        }

	        TransactionTemplate.execute(new TransactionAction() {

	            @Override
	            public void doInTransaction(TransactionStatus status) {
	                dao.transferMoney(zhangsan, -money);
	                dao.transferMoney(lisi, money);
	            }
	        });

	    }

	}

上面的代码中zhangsan（张三）转给lisi（李四）100块。如果没有异常则事务提交，有异常则事务回滚。

查看完整示例代码和表结构
________________________

**事务** 的所有代码和表结构均可以在 `mango-example <https://github.com/jfaster/mango-example/tree/master/src/main/java/org/jfaster/mango/example/transaction>`_ 中找到。




