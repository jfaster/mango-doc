.. _快速开始:

快速开始
========

本文的目的是让您快速的对mango框架有一个概括的了解。

接下来我们将使用mango框架连接MySQL数据库，然后在测试表里插入hello world字符串，最后再将其读取并打印出来。

这将是一段属于mango框架的hello world代码。

添加依赖包
__________

如果您使用maven，请将mango和mysql-connector-java的依赖添加进pom.xml文件即可

.. code-block:: none

    <dependency>
        <groupId>org.jfaster</groupId>
        <artifactId>mango</artifactId>
        <version>1.3.3</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.29</version>
    </dependency>

如果您不使用maven，请手动下载 `mango-1.3.3.jar <http://search.maven.org/remotecontent?filepath=org/jfaster/mango/1.3.3/mango-1.3.3.jar>`_ 和 `mysql-connector-java-5.1.29 <http://search.maven.org/remotecontent?filepath=mysql/mysql-connector-java/5.1.29/mysql-connector-java-5.1.29.jar>`_ ，并将他们放入工程的classpath下。


数据库准备
__________

首先您需要一台能自主访问的MySQL数据库，本文使用的MySQL数据库搭建在本地，使用默认的3306端口。

登陆MySQL服务器，创建数据库mango_example:

.. code-block:: sql

    create database mango_example;

在mango_example数据库中创建表fruit:

.. code-block:: sql

    use mango_example;
    CREATE TABLE `fruit` (
        `id` int(11) NOT NULL AUTO_INCREMENT,
        `name` varchar(20) NOT NULL,
        `num` int(11) NOT NULL,
        PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

fruit表中有id，name，num三个字段，其中id是自增主键，不用关心。接下来我们会使用mango框架对name，num字段进行读写操作。

创建HelloWorld类
________________

在org.jfaster.mango.example.quickstar包下创建HelloWorld类:

.. code-block:: java

    package org.jfaster.mango.example.quickstar;

    public class HelloWorld {

        public static void main(String[] args) {

        }

    }

这个类目前只有一个空的main函数，接下来我们将在HelloWorld类中慢慢添加代码，来实现对数据库的操作。

书写插入与查询方法
__________________

使用mango框架的主要工作就是书写dao接口，mango框架会通过java的动态代理“生成”相应的数据层访问代码。

为了方便阅读代码，我们将接口以内部类的形式放入HelloWorld类中:

.. code-block:: java

    package org.jfaster.mango.example.quickstar;

    import org.jfaster.mango.annotation.DB;
    import org.jfaster.mango.annotation.SQL;

    public class HelloWorld {

        public static void main(String[] args) {

        }

        @DB
        interface FruitDao {

            // 插入数据
            @SQL("insert into fruit(name, num) values(:1, :2)")
            public void add(String name, int num);

             // 根据name取num的总和
            @SQL("select sum(num) from fruit where name=:1")
            public int getTotalNum(String name);

        }

    }

如果您对FruitDao接口有疑问，请进一步阅读 :ref:`基本操作` 。

构造数据源并初始化mango对象
___________________________

mango框架对java标准数据源javax.sql.DataSource进行了简单实现，所以这里构造数据源不需要引入第三方jar包。

初始化数据源需要4个参数:

* **driverClassName**: 驱动程序类名，这里我们使用MySQL驱动，所以类名是 *com.mysql.jdbc.Driver* 。
* **url**: 连接数据库的url，这里我们将连接到本地MySQL的mango_example库，所以地址为 *jdbc:mysql://localhost:3306/mango_example* 。
* **username**: 数据库用户名，这里我们使用root作为用户名。
* **password**: 用户名所对应的密码，这里我们使用root作为密码。

初始化mango对象只需要数据源即可，请看下面代码:

.. code-block:: java

    package org.jfaster.mango.example.quickstar;

    import org.jfaster.mango.annotation.DB;
    import org.jfaster.mango.annotation.SQL;
    import org.jfaster.mango.datasource.DriverManagerDataSource;
    import org.jfaster.mango.operator.Mango;

    import javax.sql.DataSource;

    public class HelloWorld {

        public static void main(String[] args) {
            String driverClassName = "com.mysql.jdbc.Driver";
            String url = "jdbc:mysql://localhost:3306/mango_example";
            String username = "root"; // 这里请使用您自己的用户名
            String password = "root"; // 这里请使用您自己的密码
            DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
            Mango mango = Mango.newInstance(ds); // 使用数据源初始化mango
        }

        @DB
        interface FruitDao {

            // 插入数据
            @SQL("insert into fruit(name, num) values(:1, :2)")
            public void add(String name, int num);

             // 根据name取num的总和
            @SQL("select sum(num) from fruit where name=:1")
            public int getTotalNum(String name);

        }

    }

获取dao并调用插入与查询方法
___________________________

.. code-block:: java

    package org.jfaster.mango.example.quickstar;

    import org.jfaster.mango.annotation.DB;
    import org.jfaster.mango.annotation.SQL;
    import org.jfaster.mango.datasource.DriverManagerDataSource;
    import org.jfaster.mango.operator.Mango;

    import javax.sql.DataSource;

    public class HelloWorld {

        public static void main(String[] args) {
            String driverClassName = "com.mysql.jdbc.Driver";
            String url = "jdbc:mysql://localhost:3306/mango_example";
            String username = "root"; // 这里请使用您自己的用户名
            String password = "root"; // 这里请使用您自己的密码
            DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
            Mango mango = Mango.newInstance(ds); // 使用数据源初始化mango

            FruitDao dao = mango.create(FruitDao.class);
            String name = "apple";
            int num = 7;
            dao.add(name, num);
            System.out.println(dao.getTotalNum(name));
        }

        @DB
        interface FruitDao {

            // 插入数据
            @SQL("insert into fruit(name, num) values(:1, :2)")
            public void add(String name, int num);

             // 根据name取num的总和
            @SQL("select sum(num) from fruit where name=:1")
            public int getTotalNum(String name);

        }

    }

运行上面代码，控制台中将输出 *7* ，同时您的数据库中会被插入一行name=apple，num=7的数据。

如果再运行一次，控制台中将输出 *14* ，同时您的数据库中会再被插入一行name=apple，num=7的数据。

查看完整示例代码
________________

和 **快速开始** 相关的所有代码均可以在 `mango-example <https://github.com/jfaster/mango-example/tree/master/src/main/java/org/jfaster/mango/example/quickstart>`_ 中找到。
