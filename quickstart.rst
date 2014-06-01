.. _快速开始:

快速开始
========

本文的目的是让您快速的对mango有一个概括的了解。

接下来我们将使用mango连接MySQL数据库，然后在测试表里插入hello world字符串，最后再将其读取并打印出来。

这将是一段属于mango的hello world代码。

添加依赖包
__________

如果您使用maven，请将mango和mysql-connector-java的依赖添加进pom.xml文件即可

.. code-block:: none

    <dependency>
        <groupId>cc.concurrent</groupId>
        <artifactId>mango</artifactId>
        <version>1.02</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.29</version>
    </dependency>

如果您不使用maven，请手动下载 `mango-1.02.jar <http://search.maven.org/remotecontent?filepath=cc/concurrent/kid/1.02/kid-1.02.jar>`_ 和 `mysql-connector-java-5.1.29 <http://search.maven.org/remotecontent?filepath=mysql/mysql-connector-java/5.1.29/mysql-connector-java-5.1.29.jar>`_ ，并将他们放入工程的classpath下。


数据库准备
__________

首先您需要一台能自主访问的MySQL数据库，本文使用的MySQL数据库搭建在本地，使用默认的3306端口。

登陆MySQL服务器，创建数据库mango_db:

.. code-block:: sql

    mysql> create database mango_db;
    Query OK, 1 row affected (0.00 sec)


在mango_db数据库中创建表hello_world_table:

.. code-block:: sql

    mysql> use mango_db
    Database changed
    mysql> CREATE TABLE `hello_world_table` (
             `id` int(11) NOT NULL,
             `content` varchar(100) NOT NULL,
             PRIMARY KEY (`id`)
           );
    Query OK, 0 rows affected (0.01 sec)

hello_world_table中有id和content两个字段，接下来我们会使用mango将hello world写入content字段中，并通过id将其读取出来。

创建HelloWorld类
________________

在cc.concurrent.mango.example包下创建HelloWorld类:

.. code-block:: java

    package cc.concurrent.mango.example;

    public class HelloWorld {

        public static void main(String[] args) {

        }
        
    }

这个类目前只有一个空的main函数，接下来我们将在HelloWorld类中慢慢添加代码，来实现对数据库的操作。

书写插入与查找方法
__________________

使用mango的主要工作就是书写dao接口，mango会通过java的动态代理“生成”相应的数据层访问代码。

为了方便阅读代码，我们将接口以内部类的形式放入HelloWorld类中:

.. code-block:: java

    package cc.concurrent.mango.example;

    import cc.concurrent.mango.DB;
    import cc.concurrent.mango.SQL;

    public class HelloWorld {

        public static void main(String[] args) {

        }

        @DB
        static interface HelloWorldDao {

            @SQL("insert into hello_world_table(id, content) values(:1, :2)")
            public int add(int id, String content);

            @SQL("select content from hello_world_table where id=:1")
            public String getContentById(int id);

        }

    }

如果您对HelloWorldDao接口有疑问，请查阅 :ref:`db操作` 。

构造数据源并初始化mango对象
___________________________

mango对java标准数据源javax.sql.DataSource进行了简单实现，所以这里构造数据源不需要引入第三方jar包。

初始化数据源需要4个参数:

* **driverClassName**: 驱动程序类名，这里我们使用MySQL驱动，所以类名是 *com.mysql.jdbc.Driver* 。
* **url**: 连接数据库的url，这里我们将连接到本地MySQL的mango_db库，所以地址为 *jdbc:mysql://localhost:3306/mango_db* 。
* **username**: 数据库用户名，这里我们使用root作为用户名。
* **password**: 用户名所对应的密码，这里我们使用root作为密码。

初始化mango对象只需要数据源即可，请看下面代码:

.. code-block:: java

    package cc.concurrent.mango.example;

    import cc.concurrent.mango.DB;
    import cc.concurrent.mango.DriverManagerDataSource;
    import cc.concurrent.mango.Mango;
    import cc.concurrent.mango.SQL;

    import javax.sql.DataSource;

    public class HelloWorld {

        public static void main(String[] args) {
            String driverClassName = "com.mysql.jdbc.Driver";
            String url = "jdbc:mysql://localhost:3306/mango_db";
            String username = "root"; // 这里请使用您自己的用户名
            String password = "root"; // 这里请使用您自己的密码
            DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
            Mango mango = new Mango(ds); // 使用数据源初始化mango
            
        }

        @DB
        static interface HelloWorldDao {

            @SQL("insert into hello_world_table(id, content) values(:1, :2)")
            public int add(int id, String content);

            @SQL("select content from hello_world_table where id=:1")
            public String getContentById(int id);

        }

    } 

创建dao并调用插入与查找方法
___________________________

.. code-block:: java

    package cc.concurrent.mango.example;

    import cc.concurrent.mango.DB;
    import cc.concurrent.mango.DriverManagerDataSource;
    import cc.concurrent.mango.Mango;
    import cc.concurrent.mango.SQL;

    import javax.sql.DataSource;

    public class HelloWorld {

        public static void main(String[] args) {
            String driverClassName = "com.mysql.jdbc.Driver";
            String url = "jdbc:mysql://localhost:3306/mango_db";
            String username = "root"; // 这里请使用您自己的用户名
            String password = "root"; // 这里请使用您自己的密码
            DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
            Mango mango = new Mango(ds); // 使用数据源初始化mango

            HelloWorldDao dao = mango.create(HelloWorldDao.class);
            int id = 1;
            dao.add(id, "hello world");
            String content = dao.getContentById(id);
            System.out.println(content);
        }

        @DB
        static interface HelloWorldDao {

            @SQL("insert into hello_world_table(id, content) values(:1, :2)")
            public int add(int id, String content);

            @SQL("select content from hello_world_table where id=:1")
            public String getContentById(int id);

        }

    } 

运行上面代码，将在控制台中输出 *hello world* ，同时您的数据库中会被插入一行id=1，content=hello world的数据。
上面的代码只能正常运行一次，因为hello_world_table表中的id字段被定义为为了主键，所以再插入一次id=1的数据就会抛出异常。
如果想再次正确运行代码，只需要将id改为2或者其他在hello_world_table表中不存在的id即可。
