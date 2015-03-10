语法蜜糖
========

全局表名
________

我们可以把表名定义在@DB注解的table参数中作为全局表名，这样我们就可以通过#table来使用全局表名了，请看下面的示例代码:

.. code-block:: java

    package org.jfaster.mango.example;

    import org.jfaster.mango.annotation.DB;
    import org.jfaster.mango.annotation.SQL;

    @DB(table = "card")
    public interface CardDao {

        @SQL("select content from #table where id=:1")
        public String getContentById(int id);

        @SQL("insert into #table values(:1, :2)")
        public int insert(int id, String content);

    }

变量重命名
__________

默认情况下mango使用变量在方法中出现的位置（从1开始）来命名参数，请看下面的代码:

.. code-block:: java

	@SQL("insert into card(id, content) values(:1, :2)")
    public int insert(int id, String content);

在调用insert方法时，id变量通过:1引用，content变量则通过:2引用。
有没有办法对:1与:2进行重命名呢？

答案是使用@org.jfaster.mango.annotation.Rename注解:

.. code-block:: java

    @SQL("insert into card(id, content) values(:id, :c)")
    public int insert(@Rename("id") int id, @Rename("c") String content);

请看上面的代码，通过@Rename注解，我们将id的引用由:1重命名为:id，将content的引用由:2重命名为:c。

需要注意的是，经过@Rename注解处理后，就不能再通过变量在方法中的序号引用变量了，例如上面的代码中，我们能通过:id引用id变量，但不能再通过:1引用id变量了。

属性自动匹配
____________

我们先来看一段通过对象传入参数的代码:

.. code-block:: java

    @SQL("insert into user(uid, name) values(:1.uid, :1.name)")
    public int insert(User user);

从上面的代码我们注意到，每次通过对象的方式传入参数时，都需要使用:1.xxx来引用需要的字段，上面的例子是:1.uid和:1.name，当有很多字段需要通过:1.xxx引用时，会显得不够简练。

下面的代码能完成一样的功能，但会显得更加简练:

.. code-block:: java

    @SQL("insert into user(uid, name) values(:uid, :name)")
    public int insert(User user);

上面的代码会通过“属性自动匹配”技术，在sql编译时，将:uid与:name自动转换为:1.uid与:1.name。


