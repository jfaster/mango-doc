动态SQL
=======

字符串连接
__________

字符串连接的语法为 **#{参数}**，请看下面的代码:

.. code-block:: java

    @SQL("select id, uid, content from message order by #{:1}")
    public List<Message> getMessages(String orderBy);

* 当orderBy传入id时，sql被解析为：select id, uid, content from message order by id
* 当orderBy传入uid时，sql被解析为：select id, uid, content from message order by uid
* 当orderBy传入content时，sql被解析为：select id, uid, content from message order by content


if语句
______

if语句主要存在两种形式，[xxx]表示xxx不出现或出现一次：

1. 不包含#elseif，语法为 **#if(表达式) 字符串 [ #else 字符串 ] #end**
2. 包含#elseif，语法为 **#if(表达式) 字符串 #elseif(表达式) 字符串 [ #else 字符串 ] #end**，这里的#elseif可以多个

不包含#elseif
^^^^^^^^^^^^^

.. code-block:: java

    @SQL("select uid, name from user where #if(:1>0) uid=:1 #else uid=-1 #end")
    public User getUser(int uid);

* 当uid传入10时，将执行：select uid, name from user where uid = 10
* 当uid传入0时，将执行：select uid, name from user where uid = -1


包含#elseif
^^^^^^^^^^^

.. code-block:: java

    @SQL("select uid, name from user where #if(:1>0) uid=:1 #elseif(:1=0) uid=1 #else uid=-1 #end")
    public User getUser(int uid);

* 当uid传入10时，将执行：select uid, name from user where uid = 10
* 当uid传入0时，将执行：select uid, name from user where uid = 1
* 当uid传入-5时，将执行：select uid, name from user where uid = -1




