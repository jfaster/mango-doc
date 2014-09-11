扩展
====

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

.. _表达式:

表达式
______

mango允许使用${...}的形式向sql中嵌入动态字符串，这点类似于jsp的参数渲染，不过jsp渲染出的最后结果是html，而mango渲染出的结果是sql。

请看下面的代码:

.. code-block:: java

    @SQL("select content from card order by ${:1}")
    public List<String> getContents(String order);

getContents中的order参数是String类型，当order传入字符串"id"时，sql语句被渲染为"select content from card order by id"，当order传入字符串"content"时，sql语句被渲染为"select content from card order by content"。

${...}中还支持运算，支持的运算符有+,-,*,/,%。

需要注意的是，${...}中的参数类型必须为String或Integer或int。

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
