.. _拦截器:

拦截器
======

拦截器是mango框架中一个非常重要的功能，它为mango框架提供了扩展插件的可能。

当我们为mango框架添加拦截器后，最终执行的SQL会依次通过这些拦截器后再被执行。
所以我们可以通过拦截器对最终执行的SQL进行各种操作，如修改SQL，记录SQL等。

通过自定义拦截器，我们可以为mango框架扩展出许许多多的插件，最为常用的 :ref:`分页查询` 插件就可以通过拦截器来实现。

mango框架将拦截器分为了两类：查询拦截器与更新拦截器。

下面我们分别对这两类拦截器进行说明。

准备工作
________

在数据库 **mango_example** 中创建下面的表

.. code-block:: sql

    DROP TABLE IF EXISTS `message`;
    CREATE TABLE `message` (
      `id` int(11) NOT NULL AUTO_INCREMENT,
      `uid` int(11) NOT NULL,
      `content` varchar(100) NOT NULL,
      PRIMARY KEY (`id`),
      KEY `key_uid` (`uid`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

创建与message表对应的POJO类Message

.. code-block:: java

    public class Message {

        private int id;
        private int uid;
        private String content;

        // 各个属性对应的get与set方法必须加上，这里省略掉了
    }


查询拦截器
__________

查询拦截器，顾名思义，所有查询请求最终需要执行的SQL都需要依次通过查询拦截器处理。

下面是一个最简单的查询拦截器使用实例：

.. code-block:: java
	:emphasize-lines: 11,23

	public class QueryInterceptorMain {

	    public static void main(String[] args) {
	        String driverClassName = "com.mysql.jdbc.Driver";
	        String url = "jdbc:mysql://localhost:3306/mango_example";
	        String username = "root"; // 这里请使用您自己的用户名
	        String password = "root"; // 这里请使用您自己的密码
	        DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
	        Mango mango = Mango.newInstance(ds); // 使用数据源初始化mango

	        mango.addInterceptor(new MyQueryInterceptor());

	        MessageDao dao = mango.create(MessageDao.class);
	        int uid = 100;
	        Message msg = new Message();
	        msg.setUid(uid);
	        msg.setContent("hello");

	        dao.insert(msg);
	        dao.getMessages(uid);
	    }

	    static class MyQueryInterceptor extends QueryInterceptor {

	        public void interceptQuery(
	                BoundSql boundSql,
	                List<Parameter> parameters,
	                DataSource dataSource) {

	            System.out.println("sql: " + boundSql.getSql());
	            System.out.println("args: " + boundSql.getArgs());
	        }

	    }

	    @DB
	    interface MessageDao {

	        @ReturnGeneratedId
	        @SQL("insert into message(uid, content) values(:uid, :content)")
	        public int insert(Message msg);

	        @SQL("select id, uid, content from message where uid=:1")
	        public List<Message> getMessages(int uid);

	    }

	}

请注意上面代码中高亮的两行：

我们首先通过继承 `QueryInterceptor <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/interceptor/QueryInterceptor.java>`_ 类，实现了自己的拦截器MyQueryInterceptor；然后在初始化mango对象时，我们调用addInterceptor方法将我们的拦截器MyQueryInterceptor添加到了mango对象中。

这样当我们调用MessageDao中的getMessages方法进行查询时，最终将被执行的SQL会交由我们的拦截器MyQueryInterceptor处理。在MyQueryInterceptor中，我们只是简单输出了最终将被执行的SQL。

运行代码，输出结果如下：

.. code-block:: xml

	sql: select id, uid, content from message where uid=?
	args: [100]

这里需要注意的是QueryInterceptor中的interceptQuery方法有3个输入参数：

1. BoundSql封装了最终将要被执行的SQL
2. List<Parameter>封装了被执行方法的参数列表信息
3. DataSource则是SQL执行时所使用的数据源


更新拦截器
__________

更新拦截器，顾名思义，所有更新请求最终需要执行的SQL都需要依次通过更新拦截器处理。

下面是一个最简单的更新拦截器使用实例：

.. code-block:: java
	:emphasize-lines: 11,22

	public class UpdateInterceptorMain {

	    public static void main(String[] args) {
	        String driverClassName = "com.mysql.jdbc.Driver";
	        String url = "jdbc:mysql://localhost:3306/mango_example";
	        String username = "root"; // 这里请使用您自己的用户名
	        String password = "root"; // 这里请使用您自己的密码
	        DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
	        Mango mango = Mango.newInstance(ds); // 使用数据源初始化mango

	        mango.addInterceptor(new MyUpdateInterceptor());

	        MessageDao dao = mango.create(MessageDao.class);
	        int uid = 100;
	        Message msg = new Message();
	        msg.setUid(uid);
	        msg.setContent("hello");

	        dao.insert(msg);
	    }

	    static class MyUpdateInterceptor extends UpdateInterceptor {

	        public void interceptUpdate(
	                BoundSql boundSql,
	                List<Parameter> parameters,
	                SQLType sqlType,
	                DataSource dataSource) {

	            System.out.println("sql: " + boundSql.getSql());
	            System.out.println("args: " + boundSql.getArgs());
	        }

	    }

	    @DB
	    interface MessageDao {

	        @ReturnGeneratedId
	        @SQL("insert into message(uid, content) values(:uid, :content)")
	        public int insert(Message msg);

	    }

	}

请注意上面代码中高亮的两行：

我们首先通过继承 `UpdateInterceptor <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/interceptor/UpdateInterceptor.java>`_ 类，实现了自己的拦截器MyUpdateInterceptor；然后在初始化mango对象时，我们调用addInterceptor方法将我们的拦截器MyUpdateInterceptor添加到了mango对象中。

这样当我们调用MessageDao中的insert方法进行查询时，最终将被执行的SQL会交由我们的拦截器MyUpdateInterceptor处理。在MyUpdateInterceptor中，我们只是简单输出了最终将被执行的SQL。

运行代码，输出结果如下：

.. code-block:: xml

	sql: insert into message(uid, content) values(?, ?)
	args: [100, hello]

这里需要注意的是UpdateInterceptor中的interceptUpdate方法有4个输入参数：

1. BoundSql封装了最终将要被执行的SQL
2. List<Parameter>封装了被执行方法的参数列表信息
3. SQLType封装了被执行SQL的类型
4. DataSource则是SQL执行时所使用的数据源

查看完整示例代码和表结构
________________________

**拦截器** 的所有代码和表结构均可以在 `mango-example <https://github.com/jfaster/mango-example/tree/master/src/main/java/org/jfaster/mango/example/interceptor>`_ 中找到。
