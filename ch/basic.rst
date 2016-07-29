.. _基本操作:

基本操作
========

准备工作
________

首先我们需要创建一张user表(这里选择的是MySQL数据库)，后续的所有内容将围绕这张表进行:

.. code-block:: sql

    DROP TABLE IF EXISTS `user`;
    CREATE TABLE `user` (
      `id` int(11) NOT NULL AUTO_INCREMENT,
      `name` varchar(25) DEFAULT NULL,
      `age` int(11) DEFAULT NULL,
      `gender` tinyint(1) DEFAULT NULL,
      `money` bigint(21) DEFAULT NULL,
      `update_time` timestamp NULL DEFAULT NULL,
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

我们还需要一个java类来对应这张表的内容:

.. code-block:: java

    public class User {
        private int id;
        private String name;
        private int age;
        private boolean gender;
        private Long money;
        private Date updateTime;
        
        // 各个属性对应的get与set方法必须加上，这里省略掉了
    }


更新
____

更新主要包含insert，delete与update这三种操作。

通常情况化更新操作支持四种类型的返回值：

1. void或java.lang.Void：不返回值
2. int或java.lang.Integer：返回有多少行数据受到了影响
3. long或java.lang.Long：返回有多少行数据受到了影响
4. boolean或java.lang.Boolean：false表示没有数据受到影响，true表示有一到多行数据受到影响

insert操作
^^^^^^^^^^

现在我们来书写向user表中插入数据的方法:

.. code-block:: java

    @DB
    public interface UserDao {

        @SQL("insert into user(name, age, gender, money, update_time) values(:1, :2, :3, :4, :5)")
        public void insertUser(String name, int age, boolean gender, long money, Date updateTime);

    }

请看上面代码。

@DB注解的全名为 **@org.jfaster.mango.annotation.DB** ，dao接口必须使用它来修饰，这样这个dao接口才能被mango框架接受。

@SQL注解的全名为 **@org.jfaster.mango.annotation.SQL** ，它被用来修饰下面的insertUser方法。

@SQL注解中是一个insert操作的sql语句，需要注意的是sql使用了 :ref:`序号绑定` 来传入参数，在调用insertUser方法时，:1将被替换为insertUser方法的第1个参数，也就是name；:2将替换为insertUser方法的第2个参数，也就是age，以此类推。序号绑定只是mango众多参数绑定方式中的一种，点击 :ref:`参数绑定` 可以了解到更多相关内容。

如果我们用注解 **@org.jfaster.mango.annotation.ReturnGeneratedId** 来修饰insertUser方法，那么insert操作只能支持两种类型的返回值:

1. int或java.lang.Integer：返回int类型的自增id
2. long或java.lang.Long：返回long类型的自增id

.. code-block:: java

    @DB
    public interface UserDao {

        @ReturnGeneratedId
        @SQL("insert into user(name, age, gender, money, update_time) " +
                "values(:1.name, :1.age, :1.gender, :1.money, :1.updateTime)")
        public int insertUser(User user);

    }

delete操作
^^^^^^^^^^

.. code-block:: java

    @DB
    public interface UserDao {

        @SQL("delete from user where id=:1")
        public int deleteUser(int id);

    }

update操作
^^^^^^^^^^

.. code-block:: java

    @DB
    public interface UserDao {

        @SQL("update user set name=:1.name, age=:1.age, gender=:1.gender, " +
            "money=:1.money, update_time=:1.updateTime where id=:1.id")
        public int updateUser(User user);

    }

查询
____

查询只包含一个select操作，但根据查询条件与返回结果的不同，查询方法的书写也会有一些不同。

查询单个属性
^^^^^^^^^^^^

.. code-block:: java

    @DB
    public interface UserDao {

        @SQL("select name from user where id = :1")
        public String getName(int id);

    }

查询自定义对象
^^^^^^^^^^^^^^

.. code-block:: java

    @DB
    public interface UserDao {

        @SQL("select id, name, age, gender, money, update_time from user where id = :1")
        public User getUser(int id);

    }

需要注意的是user表中的id, name, age, gender, money, update_time字段会分别被被映射到User对象的id, name, age, gender, money, updateTime属性中。点击 :ref:`查询映射` 可以了解到更多相关内容。

查询多行数据
^^^^^^^^^^^^

.. code-block:: java

    @DB
    public interface UserDao {

        @SQL("select id, name, age, gender, money, update_time from user where age=:1 order by id")
        public List<User> getUsersByAge(int age);

    }

使用in语句进行查询
^^^^^^^^^^^^^^^^^^

.. code-block:: java

    @DB
    public interface UserDao {

        @SQL("select id, name, age, gender, money, update_time from user where id in (:1)")
        public List<User> getUsersInList(List<Integer> ids);

    }

需要注意的是 ``in (:1)`` 中的参数必须是List或Set或Array，同时返回参数也必须是List或Set或Array。

批量更新
________

批量更新主要包含insert，delete与update这三种操作。

批量更新的输入只能有一个参数，参数的类型必须是List或Set或Array。

批量更新的输出支持三种类型的返回值：

1. void或java.lang.Void：不返回值
2. int或java.lang.Integer：返回累计有多少行数据受到了影响
3. int[]或java.lang.Integer[]：返回每条更新语句影响到了多少行数据

下面以批量插入为例:

.. code-block:: java

    @DB
    public interface UserDao {

        @SQL("insert into user(name, age, gender, money, update_time) " +
                "values(:1.name, :1.age, :1.gender, :1.money, :1.updateTime)")
        public int[] batchInsertUserList(List<User> userList);

    }

需要注意的是，mango内部有两种批量更新的实现，如果批量更新在同一个数据源的同一张表上完成，mango会使用jdbc原生的批量更新方法，否则mango会在内部进行循环更新。

查看完整示例代码和表结构
________________________

**基本操作** 的所有代码和表结构均可以在 `mango-example <https://github.com/jfaster/mango-example/tree/master/src/main/java/org/jfaster/mango/example/basic>`_ 中找到。
