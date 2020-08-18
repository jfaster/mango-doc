排序与分页
==========

准备工作
________

在数据库 **mango_example** 中创建下面的表

.. code-block:: sql

    DROP TABLE IF EXISTS `message`;
    CREATE TABLE `message` (
      `id` int(11) NOT NULL AUTO_INCREMENT,
      `uid` int(11) NOT NULL,
      `content` int(11) NOT NULL,
      PRIMARY KEY (`id`),
      KEY `key_uid` (`uid`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

创建与message表对应的POJO类Message

.. code-block:: java

    public class Message {

        private int id;
        private int uid;
        private int content;

        // 各个属性对应的get与set方法必须加上，这里省略掉了
    }

MySQL排序与分页
_______________


.. code-block:: java

	public class MySQLPageMain {

	  public static void main(String[] args) {
	    String driverClassName = "com.mysql.jdbc.Driver";
	    String url = "jdbc:mysql://localhost:3306/mango_example";
	    String username = "root"; // 这里请使用您自己的用户名
	    String password = "root"; // 这里请使用您自己的密码
	    DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
	    Mango mango = Mango.newInstance(ds); // 使用数据源初始化mango

	    MessageDao dao = mango.create(MessageDao.class);
	    dao.deleteAll();
	    int uid = 100;
	    for (int i = 0; i < 10; i++) {
	      Message msg = new Message();
	      msg.setUid(uid);
	      msg.setContent(i + 1);
	      dao.insert(msg);
	    }

	    int pageNum = 2;
	    int pageSize = 3;

	    // 等价于 order by id desc
	    Sort sort = Sort.by(Direction.DESC, "id");
	    List<Message> msgs = dao.getMessages(uid, sort);
	    System.out.println("msgs: " + msgs);

	    // 等价于 limit pgeNum * pageSize, pageSize
	    Page page = Page.of(pageNum, pageSize);
	    msgs = dao.getMessages(uid, page);
	    System.out.println("msgs: " + msgs);

	    // 等价于 order by id desc limit pgeNum * pageSize, pageSize
	    page = Page.of(pageNum, pageSize, Sort.by(Direction.DESC, "id"));
	    msgs = dao.getMessages(uid, page);
	    System.out.println("msgs: " + msgs);
	  }

	  @DB
	  interface MessageDao {

	    @ReturnGeneratedId
	    @SQL("insert into message(uid, content) values(:uid, :content)")
	    int insert(Message msg);

	    @SQL("select id, uid, content from message where uid=:1")
	    List<Message> getMessages(int uid, Sort sort);

	    @SQL("select id, uid, content from message where uid=:1")
	    List<Message> getMessages(int uid, Page page);

	    @SQL("delete from message")
	    void deleteAll();

	  }

	}


上面的代码中，我们首先往message表中插入了10条数据，然后进行3次查询操作：

1. 排序查询，等价于执行SQL： **select id, uid, content from message where uid=:1 order by id**

2. 分页查询，等价于执行SQL： **select id, uid, content from message where uid=:1 limit 6, 3**

3. 排序+分页查询，等价于执行SQL： **select id, uid, content from message where uid=:1 order by id limit 6, 3**

注：分页从0开始，pageNum=2表示第3页。

Sort类
______

`Sort <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/page/Sort.java>`_ 类用于排序。

.. code-block:: java

    // 等价于 order by id asc
    Sort.by("id");


    // 等价于 order by id desc
    Sort.by(Direction.DESC, "id");


    // 等价于 order by id asc, name asc
    Sort.by("id", "name");


    // 等价于 order by id asc, name desc
    Sort.by(Order.by("id"), Order.desc("name"));


Page类
______

`Page <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/page/Page.java>`_ 类用于排序与分页。

.. code-block:: java

    // 等价于 limit pageNum * pageSize, pageSize
    Page.of(pageNum, pageSize);


    // 等价于 order by id desc limit pageNum * pageSize, pageSize
    Page.of(pageNum, pageSize, Direction.DESC, "id");


    // 等价于 order by id asc, name desc limit pageNum * pageSize, pageSize
    Page.of(pageNum, pageSize, Sort.by(Order.by("id"), Order.desc("name")));


PageResult类
____________

`PageResult <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/page/PageResult.java>`_ 类用于分页查询时获取数据总量（主要用于实现前端分页显示）

.. code-block:: java

	@DB
	interface MessageDao {

	  @SQL("select id, uid, content from message where uid=:1")
	  List<Message> getMessages(int uid, Page page);

	  @SQL("select id, uid, content from message where uid=:1")
	  PageResult<Message> getMessagesWithTotal(int uid, Page page);

	}

上面的代码中：

**getMessages** 方法只返回分页查询到的数据，不包含数据总量。

**getMessagesWithTotal** 方法返回PageResult对象，PageResult对象的getData方法返回分页查询到的数据，PageResult对象的getTotal方法则返回数据总量。


非MySQL排序与分页
_________________

mango框架的排序与分页目前支持MySQL与Oracle这两个主流数据库，在不进行任何设置时，默认用户使用MySQL数据库。

如需使用Oracle数据库，则需手动调用mango.setPageHandler(new `OraclePageHandler <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/page/OraclePageHandler.java>`_ ())。

如需使用其他数据库，则需自己实现 `PageHandler <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/page/PageHandler.java>`_ 接口：

.. code-block:: java

	public interface PageHandler {

	  void handlePage(BoundSql boundSql, Page page);

	  void handleSort(BoundSql boundSql, Sort sort);

	  void handleCount(BoundSql boundSql);

	}


查看完整示例代码和表结构
________________________

**排序与分页** 的所有代码和表结构均可以在 `mango-example <https://github.com/jfaster/mango-example/tree/master/src/main/java/org/jfaster/mango/example/page>`_ 中找到。






