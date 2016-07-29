动态SQL
=======

字符串连接
__________

字符串连接的语法为 **#{参数}**，请看下面的代码:

.. code-block:: java

	@DB
	public interface UserDao {

	    @SQL("select uid, name from user order by #{:1}")
	    public List<User> getUsers(String orderBy);

	}

* 当orderBy传入uid时，sql被解析为：select uid, name from user order by uid
* 当orderBy传入name时，sql被解析为：select uid, name from user order by name

if语句
______

if语句主要存在两种形式，[xxx]表示xxx不出现或出现一次：

1. 不包含#elseif，语法为 **#if(表达式) 字符串 [ #else 字符串 ] #end**
2. 包含#elseif，语法为 **#if(表达式) 字符串 #elseif(表达式) 字符串 [ #else 字符串 ] #end**，这里的#elseif可以多个

不包含#elseif
^^^^^^^^^^^^^

.. code-block:: java

	@DB
	public interface UserDao {

	    @SQL("select uid, name from user where #if(:1>0) uid=:1 #else uid=-1 #end")
	    public User getUser(int uid);

	}

* 当uid传入10时，将执行：select uid, name from user where uid = 10
* 当uid传入0时，将执行：select uid, name from user where uid = -1


包含#elseif
^^^^^^^^^^^

.. code-block:: java

	@DB
	public interface UserDao {

	    @SQL("select uid, name from user where #if(:1>0) uid=:1 #elseif(:1==0) uid=1 #else uid=-1 #end")
	    public User getUser2(int uid);

	}

* 当uid传入10时，将执行：select uid, name from user where uid = 10
* 当uid传入0时，将执行：select uid, name from user where uid = 1
* 当uid传入-5时，将执行：select uid, name from user where uid = -1

查看完整示例代码和表结构
________________________

**动态SQL** 的所有代码和表结构均可以在 `mango-example <https://github.com/jfaster/mango-example/tree/master/src/main/java/org/jfaster/mango/example/dynamic>`_ 中找到。



