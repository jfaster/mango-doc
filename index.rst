
欢迎使用mango
=============

mango是一个轻量级数据层访问框架。它使用注解的形式将db操作与cache操作完美的结合起来，极大的缩减了数据层代码书写量，使开发人员能更好的专注业务逻辑开发。

下面是mango的一些特性:

* **采用接口与注解的形式定义DAO，完美结合db与cache操作**
* **轻量高效，具有和直接使用jdbc同样的响应速度**
* **支持动态sql，可以构造任意复杂的sql语句**
* **支持多数据源，分表，分库，事务**
* **内嵌“函数式调用”功能，能将任意复杂的对象，映射到数据库的表中**
* **高效详细的log统计，方便开发者随时了解自己的系统**
* **超级人性化的异常提示，使用简单，学习成本低**
* **独立jar包，不依赖其它jar包**

获得mango
_________

由于mango不依赖其它jar包，所以可以直接 `下载mango-1.2.5.jar <http://search.maven.org/remotecontent?filepath=org/jfaster/mango/1.2.5/mango-1.2.5.jar>`_ ，并将它放在工程的classpath下。

当然mango也已经上传到 `maven中心库 <http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.jfaster%22%20AND%20a%3A%22mango%22>`_ 中，如果您的工程在使用maven，那么只需要在pom.xml文件中添加下面的依赖就能使用mango的功能。

.. code-block:: none

    <dependency>
        <groupId>org.jfaster</groupId>
        <artifactId>mango</artifactId>
        <version>1.2.5</version>
    </dependency>

需要注意的是，只使用mango是无法连接数据库成功的，对于连接不同的数据库，您还需要添加相应的JDBC驱动，以连接MySQL数据库为例，您还需要用到 `mysql-connector-java <http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22mysql%22%20AND%20a%3A%22mysql-connector-java%22>`_ 。

如何使用mango
_____________

您可以去 :ref:`快速开始` 实现并运行第一个mango程序

您也可以去 :ref:`mango文档目录` 查看所有的文档

您还可以通过电子邮件 **mango@jfaster.org** 联系我，我们可以一起讨论关于mango的一切问题。

