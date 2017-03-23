.. _分页查询:

分页查询
========

在生产开发中，我们经常遇到一种查询方式就是分页查询。

一般情况下，分页查询需要下面两个步骤：

1. 查询数据总量，用于计算一共有多少页数据
2. 查询某一页的数据列表

为简化分页查询，mango框架通过 :ref:`拦截器` 的方式，为MySQL与Oracle数据库提供了分页查询插件。

下面我们以MySQL为例，为大家阐述如何使用mango分页插件。

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

MySQL分页实例
_____________


.. code-block:: java
	:emphasize-lines: 11,24,38

	public class MySQLPageMain {

	    public static void main(String[] args) {
	        String driverClassName = "com.mysql.jdbc.Driver";
	        String url = "jdbc:mysql://localhost:3306/mango_example";
	        String username = "root"; // 这里请使用您自己的用户名
	        String password = "root"; // 这里请使用您自己的密码
	        DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
	        Mango mango = Mango.newInstance(ds); // 使用数据源初始化mango

	        mango.addInterceptor(new MySQLPageInterceptor());
	        MessageDao dao = mango.create(MessageDao.class);
	        int uid = 100;

	        for (int i = 0; i < 10; i++) {
	            Message msg = new Message();
	            msg.setUid(uid);
	            msg.setContent("hello");
	            dao.insert(msg);
	        }

	        int pageNum = 2;
	        int pageSize = 3;
	        Page page = Page.create(pageNum, pageSize);
	        List<Message> msgs = dao.getMessages(uid, page);
	        System.out.println("total: " + page.getTotal());
	        System.out.println("msgs: " + msgs);
	    }

	    @DB
	    interface MessageDao {

	        @ReturnGeneratedId
	        @SQL("insert into message(uid, content) values(:uid, :content)")
	        public int insert(org.jfaster.mango.example.interceptor.Message msg);

	        @SQL("select id, uid, content from message where uid=:1")
	        public List<Message> getMessages(int uid, Page page);

	    }

	}

上面的代码中，我们首先往message表中插入了10条数据，然后进行分页查询，每3条数据为1页，取第2页数据。

上面的分页代码，使用了mango框架内置的分页拦截器 `MySQLPageInterceptor <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/plugin/page/MySQLPageInterceptor.java>`_ 与分页类 `Page <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/plugin/page/Page.java>`_。

简单说明一下分页原理：

1. 执行MessageDao中的getMessages方法时，传入page对象，page对象中的pageNum属性指定分页号（从1开始），page对象中的pageSize属性指定每页数据数量
2. MySQLPageInterceptor拦截器根据原始SQL会自动生成一个获得数据总量的SQL，执行该SQL获取到数据总量后，把数据总量放入page对象的total属性中
3. MySQLPageInterceptor拦截器改写原始SQL，添加limit关键字和分页信息
4. 执行改写后的SQL，获得分页数据列表

查看完整示例代码和表结构
________________________

**分页查询** 的所有代码和表结构均可以在 `mango-example <https://github.com/jfaster/mango-example/tree/master/src/main/java/org/jfaster/mango/example/page>`_ 中找到。






