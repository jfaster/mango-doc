表分片与数据库分片(分表与分库)
==============================

表分片
______

表分片通常也被称为分表，散表。
当某张表的数据量很大时，sql执行效率都会变低，这时通常会把大表拆分成多个小表，以提高sql执行效率。
我们将这种大表拆分成多个小表的策略称之为表分片。

先来看一段mango框架中表分片的代码:

.. code-block:: java

    @DB(table = "t_order")
    @Sharding(tableShardingStrategy = TableShardingOrderDao.OrderTableShardingStrategy.class)
    public interface TableShardingOrderDao {

        @SQL("insert into #table(id, uid, price, status) values(:id, :uid, :price, :status)")
        public void addOrder(@TableShardingBy("uid") Order order);

        @SQL("select id, uid, price, status from #table where uid = :1")
        public List<Order> getOrdersByUid(@TableShardingBy int uid);

        class OrderTableShardingStrategy implements TableShardingStrategy<Integer> {

            @Override
            public String getTargetTable(String table, Integer shardingParameter) {
                return table + "_" + (shardingParameter % 2);
            }

        }

    }

上面的代码实现了所有的表分片逻辑，以上面的代码为例，总结一下mango框架实现表分片的3个步骤：

1. 填写@DB注解中的table参数，这个参数将启用 :ref:`全局表名`，上面代码的全局表名是t_order
2. 引入 `@Sharding <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/annotation/Sharding.java>`_ 注解，并填写@Sharding注解中的tableShardingStrategy参数，这个参数的作用是定义表分片策略，上面代码使用了自定义的表分片策略OrderTableShardingStrategy
3. 使用 `@TableShardingBy <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/annotation/TableShardingBy.java>`_ 注解指定对表分片策略传入的参数。上面的代码中，调用 ``addOrder(@TableShardingBy("uid") Order order)`` 方法时，会使用order对象中的uid属性作为参数传递给第2步中的表分片策略，而调用 ``getOrdersByUid(@TableShardingBy int uid)`` 方法时，会使用uid作为参数传递给第2步中的表分片策略

上面的3个步骤步中，最核心的是第2步中的表分片策略。mango框架使用@Sharding注解中的tableShardingStrategy参数来指定表分片策略，tableShardingStrategy参数接受任何实现了 `TableShardingStrategy <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/sharding/TableShardingStrategy.java>`_ 接口的类。

我们来看一下TableShardingStrategy接口的定义:

.. code-block:: java

    public interface TableShardingStrategy<T> {
    
        public String getTargetTable(String table, T shardingParameter);

    }

TableShardingStrategy接口非常简单，只有一个getTargetTable方法，其中:

* 输入参数table，对应的是全局表名
* 输入参数shardingParameter，接收被@TableShardingBy注解修饰的参数，shardingParameter的类型是泛型，将由实现类根据@TableShardingBy修饰的参数确定具体类型
* 输出则为真正的表名
 
以上面的OrderTableShardingStrategy表分片策略为例：

* 输入参数table将被传入字符串"t_order"
* 输入参数shardingParameter则会分两种情况，在调用 ``addOrder(@TableShardingBy("uid") Order order)`` 方法时，shardingParameter会被传入order对象中的uid属性，而在调用 ``getOrdersByUid(@TableShardingBy int uid)`` 方法时，shardingParameter会被传入参数uid
* 当uid为偶数时，使用t_order_0表，当uid为奇数时，使用t_order_1表

数据库分片
__________

数据库分片通常也被称为分库，散库等。
当我们在某个库中，把某张大表拆分成多个小表后还不能满足性能要求，这时我们需要把一部分拆分的表挪到另外一个库中，以提高sql执行效率。

先来看一段mango框架中数据库分片的代码:

.. code-block:: java

    @DB()
    @Sharding(databaseShardingStrategy = DatabaseShardingOrderDao.OrderDatabaseShardingStrategy.class)
    public interface DatabaseShardingOrderDao {

        @SQL("insert into t_order(id, uid, price, status) values(:id, :uid, :price, :status)")
        public void addOrder(@DatabaseShardingBy("uid") Order order);

        @SQL("select id, uid, price, status from t_order where uid = :1")
        public List<Order> getOrdersByUid(@DatabaseShardingBy int uid);

        class OrderDatabaseShardingStrategy implements DatabaseShardingStrategy<Integer> {

            @Override
            public String getDatabase(Integer shardingParameter) {
                return shardingParameter < 1000 ? "db1" : "db2";
            }

        }

    }

上面的代码实现了所有的数据库分片逻辑，以上面的代码为例，总结一下mango框架实现数据库分片的2个步骤：

1. 引入 `@Sharding <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/annotation/Sharding.java>`_ 注解，并填写@Sharding注解中的databaseShardingStrategy参数，这个参数的作用是定义数据库分片策略，上面代码使用了自定义的数据库分片策略OrderDatabaseShardingStrategy
2. 使用 `@DatabaseShardingBy <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/annotation/DatabaseShardingBy.java>`_ 注解指定对数据库分片策略传入的参数。上面的代码中，调用 ``addOrder(@DatabaseShardingBy("uid") Order order)`` 方法时，会使用order对象中的uid属性作为参数传递给第1步中的数据库分片策略，而调用 ``getOrdersByUid(@DatabaseShardingBy int uid)`` 方法时，会使用uid作为参数传递给第1步中的数据库分片策略

上面的2个步骤步中，最核心的是第1步中的数据库分片策略。mango框架使用@Sharding注解中的databaseShardingStrategy参数来指定数据库分片策略，databaseShardingStrategy参数接受任何实现了 `DatabaseShardingStrategy <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/sharding/DatabaseShardingStrategy.java>`_ 接口的类。

我们来看一下DatabaseShardingStrategy接口的定义:

.. code-block:: java

    public interface DatabaseShardingStrategy<T> {
    
        public String getDatabase(T shardingParameter);
    
    }

DatabaseShardingStrategy接口非常简单，只有一个getDatabase方法，其中:

* 输入参数shardingParameter，接收被@DatabaseShardingBy注解修饰的参数，shardingParameter的类型是泛型，将由实现类根据@DatabaseShardingBy修饰的参数确定具体类型
* 输出则为database名称
 
以上面的OrderDatabaseShardingStrategy数据库分片策略为例：

* 输入参数shardingParameter则会分两种情况，在调用 ``addOrder(@DatabaseShardingBy("uid") Order order)`` 方法时，shardingParameter会被传入order对象中的uid属性，而在调用 ``getOrdersByUid(@DatabaseShardingBy int uid)`` 方法时，shardingParameter会被传入参数uid
* 当uid小于1000时，使用的database为db1，当uid大于等于1000时，使用的database为db2

同时使用数据库分片与表分片
__________________________

我们将上面的数据库分片策略与表分片策略一起使用，形成同时使用数据库分片与表分片的代码：

.. code-block:: java

    @DB(table = "t_order")
    @Sharding(
            databaseShardingStrategy = ShardingOrderDao.OrderDatabaseShardingStrategy.class,
            tableShardingStrategy = ShardingOrderDao.OrderTableShardingStrategy.class
    )
    public interface ShardingOrderDao {

        @SQL("insert into #table(id, uid, price, status) values(:id, :uid, :price, :status)")
        public void addOrder(@DatabaseShardingBy("uid") @TableShardingBy("uid") Order order);

        @SQL("select id, uid, price, status from #table where uid = :1")
        public List<Order> getOrdersByUid(@DatabaseShardingBy @TableShardingBy int uid);

        class OrderDatabaseShardingStrategy implements DatabaseShardingStrategy<Integer> {

            @Override
            public String getDatabase(Integer uid) {
                return uid < 1000 ? "db1" : "db2";
            }

        }

        class OrderTableShardingStrategy implements TableShardingStrategy<Integer> {

            @Override
            public String getTargetTable(String table, Integer uid) {
                return table + "_" + (uid % 2);
            }

        }

    }

上面的代码中，数据库分片策略使用了OrderDatabaseShardingStrategy，即uid小于1000时使用的database为db1，uid大于等于1000时使用的database为db2。
表分片策略则使用了OrderTableShardingStrategy，即uid为偶数时使用t_order_0表，uid为奇数时使用t_order_1表。

组合数据库分片策略与表分片策略得到如下规则：

1. uid小于1000并且uid为偶数时，使用db1中的t_order_0表
2. uid小于1000并且uid为奇数时，使用db1中的t_order_1表
3. uid大于等于1000并且uid为偶数时，使用db2中的t_order_0表
4. uid大于等于1000并且uid为奇数时，使用db2中的t_order_1表

精简分片代码
____________

下面的代码同样实现了同时使用数据库分片与表分片，不过更加简洁。

.. code-block:: java

    @DB(table = "t_order")
    @Sharding(shardingStrategy = ShardingOrder2Dao.OrderShardingStrategy.class)
    public interface ShardingOrder2Dao {

        @SQL("insert into #table(id, uid, price, status) values(:id, :uid, :price, :status)")
        public void addOrder(@ShardingBy("uid") Order order);

        @SQL("select id, uid, price, status from #table where uid = :1")
        public List<Order> getOrdersByUid(@ShardingBy int uid);

        class OrderShardingStrategy implements ShardingStrategy<Integer, Integer> {

            @Override
            public String getDatabase(Integer uid) {
                return uid < 1000 ? "db1" : "db2";
            }

            @Override
            public String getTargetTable(String table, Integer uid) {
                return table + "_" + (uid % 2);
            }

        }

    }

上面的代码中，引入了@ShardingBy注解，@ShardBy=@DataSourceShardBy+@TableShardBy。

多维度分片策略
______________

上面的所有的代码我们都使用uid作为分片策略的计算参数，我们称之为一维分片策略。

考虑下面一个问题，当我们把数据库分片信息与表分片信息保存到order表中id字段的头部时，我们不但能把uid作为分片策略的计算参数，也能把id作为分片策略的计算参数。但@Sharding注解放在类上时，我们只能要么选择uid作为分片策略的计算参数，要们选择id作为分片策略的计算参数。这时我们需要将@Sharding注解下移到方法上，不同的方法指定不同的分片策略，实现多维度分片策略。

请看下面的代码：

.. code-block:: java

    @DB(table = "t_order")
    public interface ShardingOrder3Dao {

        @SQL("insert into #table(id, uid, price, status) values(:id, :uid, :price, :status)")
        @Sharding(shardingStrategy = ShardingOrder3Dao.OrderUidShardingStrategy.class)
        public void addOrder(@ShardingBy("uid") Order order);

        @SQL("select id, uid, price, status from #table where uid = :1")
        @Sharding(shardingStrategy = ShardingOrder3Dao.OrderUidShardingStrategy.class)
        public List<Order> getOrdersByUid(@ShardingBy int uid);

        @SQL("select id, uid, price, status from #table where id = :1")
        @Sharding(shardingStrategy = OrderIdShardingStrategy.class)
        public Order getOrderById(@ShardingBy String id);

        class OrderUidShardingStrategy implements ShardingStrategy<Integer, Integer> {

            @Override
            public String getDatabase(Integer uid) {
                return uid < 1000 ? "db1" : "db2";
            }

            @Override
            public String getTargetTable(String table, Integer uid) {
                return table + "_" + (uid % 2);
            }

        }

        class OrderIdShardingStrategy implements ShardingStrategy<String, String> {

            @Override
            public String getDatabase(String orderId) {
                return "db" + orderId.substring(0, 1);
            }

            @Override
            public String getTargetTable(String table, String orderId) {
                return table + "_" + orderId.substring(1, 2);
            }

        }

    }

上面的代码中，``addOrder(@ShardingBy("uid") Order order)`` 方法与 ``getOrdersByUid(@ShardingBy int uid)`` 方法使用了以uid作为参数的分片策略OrderUidShardingStrategy，而 ``getOrderById(@ShardingBy String id)`` 方法则使用了以id作为参数的分片策略OrderIdShardingStrategy。

查看完整示例代码和表结构
________________________

**表分片与数据库分片** 的所有代码和表结构均可以在 `mango-example <https://github.com/jfaster/mango-example/tree/master/src/main/java/org/jfaster/mango/example/sharding>`_ 中找到。

