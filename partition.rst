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

	@DB(table = "user", tablePartition = IntegerModTenTablePartition.class)
	public interface UserDao {

	    @SQL("insert into #table(uid, name) values(:1, :2)")
	    public void addUser(@TableShardBy int uid, String name);

	    @SQL("select uid, name from #table where uid = :1")
	    public User getUser(@TableShardBy int uid);

	}

上面的代码实现了所有的分表逻辑，以上面的代码为例，总结一下实现分表的三个步骤：

1. 填写@DB注解中的table参数，这个参数将启用 :ref:`全局表名`，上面代码的全局表名是user
2. 填写@DB注解中的tablePartition参数，这个参数的作用是定义分表策略，上面代码使用了mango内部提供的IntegerModTenTablePartition，表明对输入参数进行模十分表
3. 使用 `@TableShardBy <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/annotation/TableShardBy.java>`_ 注解指定对分表策略传入的参数，上面的代码会将uid作为参数传递给第2步中的分表策略。需要注意的是IntegerModTenTablePartition分表策略只支持Integer或int类型的参数。

以调用 userDao.getUser(88) 为例，最后被执行的sql为 select uid, name from user_8 where uid = 88 。

下面是mango框架内部提供的8种分表策略:

1. 整数模十分表，支持Integer或int参数类型，实现类为 `IntegerModTenTablePartition <http://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/partition/IntegerModTenTablePartition.java>`_
2. 长整模十分表，支持Long或long参数类型，实现类为 `LongModTenTablePartition <http://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/partition/LongModTenTablePartition.java>`_
3. 字符串模十分表，支持String参数类型，实现类为 `StringModTenTablePartition <http://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/partition/StringModTenTablePartition.java>`_
4. 模十分表，支持Integer或int或Long或long或String参数类型，实现类为 `ModTenTablePartition <http://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/partition/ModTenTablePartition.java>`_
5. 整数模百分表，支持Integer或int参数类型，实现类为 `IntegerModHundredTablePartition <http://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/partition/IntegerModHundredTablePartition.java>`_
6. 长整模百分表，支持Long或long参数类型，实现类为 `LongModHundredTablePartition <http://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/partition/LongModHundredTablePartition.java>`_
7. 字符串模百分表，支持String参数类型，实现类为 `StringModHundredTablePartition <http://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/partition/StringModHundredTablePartition.java>`_
8. 模百分表，支持Integer或int或Long或long或String参数类型，实现类为 `ModHundredTablePartition <http://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/partition/ModHundredTablePartition.java>`_

自定义分表
^^^^^^^^^^

mango框架内部提供的8种分表策略，但这并不能满足所有要求，当我们需要其他的分表策略时，我们就需要自定义分表策略类。

分表策略类通过@DB注解中的tablePartition参数传入，tablePartition参数接受任何实现了 `TablePartition <http://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/partition/TablePartition.java>`_ 接口的类，所以我们可以通过自己实现TablePartition接口，来完成任何自定义分表策略。

我们先来看一下TablePartition接口的定义:

.. code-block:: java

	public interface TablePartition<T> {

	    public String getPartitionedTable(String table, T shardParam, int type);

	}

TablePartition接口非常简单，只有一个getPartitionedTable方法，其中:

* 输入参数table，对应的是全局表名
* 输入参数shardParam，接收被@TableShardBy注解修饰的参数，shardParam的类型是泛型，将由实现类确定具体类型
* 输入参数type，接收@TableShardBy注解中的type参数，该参数用于复杂的分表策略
* 输出则为真正的表名

我们来实现一个简单的模五分表策略:

.. code-block:: java

	public class ModFiveTablePartition implements TablePartition<Integer> {

	    @Override
	    public String getPartitionedTable(String table, Integer shardParam, int type) {
	        return table + "_" + (shardParam % 5);
	    }

	}

如果我们用这个自定义的分表策略ModFiveTablePartition替换之前的ModTenTablePartition，再次调用 userDao.getUser(88)，最后被执行的sql将变为 select uid, name from user_3 where uid = 88 。

分库
____

分库通常也被称为散库，数据源路由等。
当我们在某个库中，把某张大表拆分成多个小表后还不能满足性能要求，这时我们需要把一部分拆分的表挪到另外一个库中，以提高sql执行效率。

先来看一段分库的代码:

.. code-block:: java

	@DB(table = "user", dataSourceRouter = MyDataSourceRouter.class)
	public interface DataSourceRouterUserDao {

	    @SQL("insert into #table(uid, name) values(:1, :2)")
	    public void addUser(@DataSourceShardBy int uid, String name);

	    @SQL("select uid, name from #table where uid = :1")
	    public User getUser(@DataSourceShardBy int uid);

	}

上面的代码实现了所有的分库逻辑，以上面的代码为例，总结一下实现分表的三个步骤：

1. 填写@DB注解中的table参数，这个参数将启用 全局表名，上面代码的全局表名是user
2. 填写@DB注解中的dataSourceRouter参数，这个参数的作用是定义分库策略，上面代码使用了自定义分库策略MyDataSourceRouter
3. 使用 `@DataSourceShardBy <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/annotation/DataSourceShardBy.java>`_ 注解指定对分库策略传入的参数，上面的代码会将uid作为参数传递给第2步中的分库策略。

@DB注解中的dataSourceRouter参数接受任何实现了 `DataSourceRouter <http://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/partition/DataSourceRouter.java>`_ 接口的类。
我们来看一下DataSourceRouter接口的定义:

.. code-block:: java

	public interface DataSourceRouter<T> {

	    public String getDataSourceName(T shardParam, int type);

	}

DataSourceRouter接口也非常简单，只有一个getDataSourceName方法，其中:

* 输入参数shardParam，接收被@DataSourceShardBy注解修饰的参数，shardParam的类型是泛型，将由实现类确定具体类型
* 输出则为要访问的数据源名称，这里需要用到 :ref:`多数据源` 的相关知识

最后来看MyDataSourceRouter分库策略:

.. code-block:: java

	public class MyDataSourceRouter implements DataSourceRouter<Integer> {

	    @Override
	    public String getDataSourceName(Integer shardParam, int type) {
	        return shardParam % 10 < 5 ? "datasource1" : "datasource2";
	    }

	}

上面的代码中，如果uid尾号为0-4的用户将使用datasource1数据源，uid尾号为5-9的用户将使用datasource2数据源。

混合使用分库分表
________________

我们将上面的分库与分表策略一起使用，形成混合使用分库分表的代码：

.. code-block:: java

	@DB(
	        table = "user",
	        dataSourceRouter = MyDataSourceRouter.class,
	        tablePartition = IntegerModTenTablePartition.class
	)
	public interface UserDao {

	    @SQL("insert into #table(uid, name) values(:1, :2)")
	    public void addUser(@DataSourceShardBy @TableShardBy int uid, String name);

	    @SQL("select uid, name from #table where uid = :1")
	    public User getUser(@DataSourceShardBy @TableShardBy int uid);

	}

上面的代码中，分库策略使用了自定义的MyDataSourceRouter，分表策略则使用了IntegerModTenTablePartition模十分表。

组合分库加分表策略得到如下规则：

uid尾号为0,1,2,3,4的用户将使用datasource1数据源中对应的user_0,user_1,user_2,user_3,user_4表，uid尾号为5,6,7,8,9的用户将使用datasource2数据源中对应的user_5,user_6,user_7,user_8,user_9表。

@ShardBy注解
____________

一般情况下，分库分表都使用一个参数，如上面的UserDao，使用uid进行分库与分表。在uid上使用@DataSourceShardBy，@TableShardBy两个注解显得不够简洁，所以mango框架引入了@ShardBy注解。

**@ShardBy=@DataSourceShardBy+@TableShardBy**，所以上面的UserDao可以简化为如下代码：

.. code-block:: java

	@DB(
	        table = "user",
	        dataSourceRouter = MyDataSourceRouter.class,
	        tablePartition = IntegerModTenTablePartition.class
	)
	public interface User2Dao {

	    @SQL("insert into #table(uid, name) values(:1, :2)")
	    public void addUser(@ShardBy int uid, String name);

	    @SQL("select uid, name from #table where uid = :1")
	    public User getUser(@ShardBy int uid);

	}

查看完整示例代码
________________

和 **分表与分库** 相关的所有代码均可以在 `mango-example <https://github.com/jfaster/mango-example/tree/master/src/main/java/org/jfaster/mango/example/partition>`_ 中找到。















