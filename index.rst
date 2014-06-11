
欢迎使用mango
=============

mango是一个轻量级数据层访问框架。它使用注解的形式将db操作与cache操作完美的结合起来，极大的缩减了数据层代码书写量，使开发人员能更好的专注业务逻辑开发。

下面是mango的一些特性:

* **完美结合db操作与cache操作**
* **轻量高效，独立jar包，不依赖其它jar包**
* **支持多数据源，散表**
* **超级人性化的异常提示**
* **使用简单，学习成本低**

获得mango
_________

由于mango不依赖其它jar包，所以可以直接 `下载mango-1.03.jar <http://search.maven.org/remotecontent?filepath=cc/concurrent/mango/1.03/mango-1.03.jar>`_ ，并将它放在工程的classpath下。

当然mango也已经上传到 `maven中心库 <http://search.maven.org/#artifactdetails%7Ccc.concurrent%7Cmango%7C1.03%7Cjar>`_ 中，如果您的工程在使用maven，那么只需要在pom.xml文件中添加下面的依赖就能使用mango的功能。

.. code-block:: none

    <dependency>
        <groupId>cc.concurrent</groupId>
        <artifactId>mango</artifactId>
        <version>1.03</version>
    </dependency>

需要注意的是，只使用mango是无法连接数据库成功的，对于连接不同的数据库，您还需要添加相应的JDBC驱动，以连接MySQL数据库为例，您还需要用到 `mysql-connector-java <http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22mysql%22%20AND%20a%3A%22mysql-connector-java%22>`_ 。

如何使用mango
_____________

您可以去 :ref:`快速开始` 实现并运行第一个mango程序

您也可以去 :ref:`mango文档目录` 查看所有的文档

您还可以通过电子邮件 **liangyanghe@gmail.com** 联系我，我们可以一起讨论关于mango的一切问题。

