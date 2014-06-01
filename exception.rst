常见异常
========

没有可读参数异常
________________

**NotReadableParameterException** 表示没有可读参数异常，请看下面的举例说明:

.. code-block:: java

    @DB
    interface UserDao {
        @SQL("insert into user(uid) values(:1)")
        public int add();

        @SQL("select uid from user where uid in (:1)")
        public int[] gets();
    }

上面的代码中，UserDao中的add()方法和gets()方法都没有参数，但在修饰它们的@SQL中，都使用了 ``:1`` 来引用方法的第一个参数，
所以它们都会抛出NotReadableParameterException，同时有提示信息 ``parameter :1 is not readable`` 。

没有可读属性异常
________________

**NotReadablePropertyException** 表示没有可读属性异常，请看下面的举例说明:

.. code-block:: java

    @DB
    interface UserDao {
        @SQL("insert into user(uid) values (:1.b)")
        public int add(A a);
    }

上面的代码中，UserDao中有参数为A的add方法，这里的A类中没有b属性，但在修饰add方法的@SQL中，使用了 ``:1.b`` 来引用A类中的b属性，
所以它会抛出NotReadablePropertyException，并提示相关信息。

错误的返回结果数量异常
______________________

**IncorrectResultSetColumnCountException** 表示错误的返回结果数量异常，请看下面的举例说明:

.. code-block:: java

    @DB
    interface PersonDao {
        @SQL("select id, name from person where id=:1")
        public int get(int id);
    }

上面的代码中，PersonDao中的get方法的返回参数为int，表示只会从ResultSet中取出唯一值，但是在@SQL中使用了 ``select id, name ...`` 取id与name，
所以它会抛出IncorrectResultSetColumnCountException，并提示相关信息。
