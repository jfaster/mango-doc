.. _全局表名:

全局表名
========

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




