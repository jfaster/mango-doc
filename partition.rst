分表与分库
==========

分表
____

分表通常也被称为散表。
当某张表的数据量很大时，sql执行效率都会变低，这时通常会把大表拆分成多个小表，以提高sql执行效率。

快速分表
^^^^^^^^

先来看一段分表的代码:

.. code-block:: java

	@DB(table = "user", tablePartition = ModTenTablePartition.class)
	public interface UserDao {

	    @SQL("insert into #table(uid, name) values(:1, :2)")
	    public void addUser(@ShardBy int uid, String name);

	    @SQL("select uid, name from #table where uid = :1")
	    public User getUser(@ShardBy int uid);

	}

上面的代码实现了所有的分表逻辑，以上面的代码为例，总结一下实现分表的三个步骤：

1. 填写@DB注解中的table参数，这个参数将启用 :ref:`全局表名`，上面代码的全局表名是user
2. 填写@DB注解中的tablePartition参数，这个参数的作用是定义分表策略，上面代码使用了ModTenTablePartition，表明对输入参数进行模十分表
3. 使用@ShardBy注解指定对分表策略传入的参数，上面的代码会将uid作为参数传递给第2步中的分表策略

以调用 userDao.getUser(88) 为例，最后被执行的sql为 select uid, name from user_8 where uid = 88 。

mango框架内部实现了两种分表策略:

1. 模十分表，实现类为 `ModTenTablePartition <http://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/partition/ModTenTablePartition.java>`_
2. 模百分表，实现类为 `ModHundredTablePartition <http://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/partition/ModHundredTablePartition.java>`_

自定义分表
^^^^^^^^^^

mango框架内部自定义了两种最常见的分表策略类，但这并不能满足所有要求，当我们需要其他的分表策略时，我们就需要自定义分表策略类。

分表策略类通过@DB注解中的tablePartition参数传入，tablePartition参数接受任何实现了 `TablePartition <http://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/partition/TablePartition.java>`_ 接口的类，所以我们可以通过自己实现TablePartition接口，来完成任何自定义分表策略。

我们先来看一下TablePartition接口的定义:

.. code-block:: java

	public interface TablePartition {

	    public String getPartitionedTable(String table, Object shardParam);

	}

TablePartition接口非常简单，只有一个getPartitionedTable方法，其中:

* 输入参数table，对应的是全局表名
* 输入参数shardParam，对应的是被@ShardBy注解修饰的参数
* 输出则为真正的表名

还是以调用 userDao.getUser(88) 为例，输入参数table将为 user，输入参数shardParam将为 88，经过ModTenTablePartition的处理后，将会输出 user_88。

我们来实现一个简单的模五分表策略:

.. code-block:: java

	public class ModFiveTablePartition implements TablePartition {

	    @Override
	    public String getPartitionedTable(String table, Object shardParam) {
	        if (!(shardParam instanceof Integer)) {
	            throw new IllegalArgumentException("Parameter need Integer");
	        }
	        Integer num = (Integer) shardParam;
	        return table + "_" + (num % 5);
	    }

	}

如果我们用这个自定义的分表策略ModFiveTablePartition替换之前的ModTenTablePartition，再次调用 userDao.getUser(88)，最后被执行的sql将变为 select uid, name from user_3 where uid = 88 。

分库
____

分库通常也被称为散库，数据源路由等。
当我们在某个库中，把某张大表拆分成多个小表后还不能满足性能要求，这时我们需要把一部分拆分的表挪到另外一个库中，以提高sql执行效率。
先来看一段分库的代码:

.. code-block:: java

	@DB(table = "user", tablePartition = ModTenTablePartition.class,
	        dataSourceRouter = MyDataSourceRouter.class)
	public interface RouterUserDao {

	    @SQL("insert into #table(uid, name) values(:1, :2)")
	    public void addUser(@ShardBy int uid, String name);

	    @SQL("select uid, name from #table where uid = :1")
	    public User getUser(@ShardBy int uid);

	}

上面的代码除了名字和UserDao不一样之外，唯一的区别就是：填写了@DB注解中的dataSourceRouter参数，代码中的MyDataSourceRouter对应了一种分库策略。
dataSourceRouter参数接受任何实现了 `DataSourceRouter <http://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/partition/DataSourceRouter.java>`_ 接口的类。
我们来看一下DataSourceRouter接口的定义:

.. code-block:: java

	public interface DataSourceRouter {

	    public String getDataSourceName(Object shardParam);

	}

DataSourceRouter接口也非常简单，只有一个getDataSourceName方法，其中:

* 输入参数shardParam，对应的是被@ShardBy注解修饰的参数
* 输出则为要访问的数据源名称，这里需要用到 :ref:`多数据源` 的相关知识

最后来看MyDataSourceRouter分库策略:

.. code-block:: java

	public class MyDataSourceRouter implements DataSourceRouter {

	    @Override
	    public String getDataSourceName(Object shardParam) {
	        if (!(shardParam instanceof Integer)) {
	            throw new IllegalArgumentException("Parameter need Integer");
	        }
	        Integer num = (Integer) shardParam;
	        return num % 10 < 5 ? "datasource1" : "datasource2";
	    }

	}

上面的代码中，如果uid尾号为0-4的用户将使用datasource1数据源，uid尾号为5-9的用户将使用datasource2数据源。

汇总RouterUserDao与MyDataSourceRouter的信息。

1. 分表策略使用了模十分表：uid尾号为0的用户，信息被放入到了user_0中；uid尾号为1的用户，信息被放入到了user_1中，以此类推。
2. 分库策略则是：uid尾号为0-4的用户将使用datasource1数据源，uid尾号为5-9的用户将使用datasource2数据源。

合并上面的两条信息，可以得出结论：uid尾号为0-4的用户将使用datasource1数据源中对应的user_0,user_1,...,user_4表，uid尾号为5-9的用户将使用datasource2数据源中对应的user_5,user_6,...,user_9表。


查看完整示例代码
________________

和 **分表与分库** 相关的所有代码均可以在 `mango-example <https://github.com/jfaster/mango-example/tree/master/src/main/java/org/jfaster/mango/example/partition>`_ 中找到。















