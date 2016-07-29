.. _参数绑定:

参数绑定
========

参数绑定指的是：将接口参数绑定到SQL指定的位置中，也即向SQL中传入参数。

.. _序号绑定:

序号绑定
________

序号绑定指的是将接口参数的序号绑定到SQL指定的位置中。
参数的序号从1开始，:1表示使用第1个参数，:2表示使用第2个参数，以此类推。
下面是序号绑定的实例：

.. code-block:: java

    @SQL("insert into binding_user(uid, name, age) values(:1, :2, :3)")
    public void addUserByIndex(int uid, String name, int age);

请看上面的代码，当调用addUserByIndex方法时，:1将被替换为addUserByIndex方法的第1个参数，也就是uid；:2将替换为addUserByIndex方法的第2个参数，也就是name，:3将替换为addUserByIndex方法的第3个参数，也就是age。

最终mango内部将执行如下的JDBC代码绑定参数：

.. code-block:: java

    ps = conn.prepareStatement("insert into binding_user(uid, name, age) values(?, ?, ?)");
    ps.setInt(1, uid);
    ps.setString(2, name);
    ps.setInt(3, age);

重命名绑定
__________

使用序号绑定虽然很简便，但有时查看起来会不够直观，这时我们可以通过@Rename注解对参数进行重命名。
下面是重命名绑定的实例：

.. code-block:: java

    @SQL("insert into binding_user(uid, name, age) values(:uid, :name, :age)")
    public void addUserByRename(@Rename("uid") int uid, @Rename("name") String name, @Rename("age") int age);

需要注意的是，一旦使用@Rename注解对参数进行重命名，则不能再通过序号绑定参数，下面是一个不合法的实例：

.. code-block:: java

    @SQL("insert into binding_user(uid, name, age) values(:1, :name, :age)")
    public void addUserError(@Rename("uid") int uid, @Rename("name") String name, @Rename("age") int age);


列表参数绑定
____________

在SQL中使用in操作的时候，我们会使用到列表参数绑定。
下面是列表参数绑定的实例：

.. code-block:: java

    @SQL("select uid, name, age from binding_user where uid in (:1)")
    public List<BindingUser> getUsersByUids(List<Integer> uids);

需要注意的是， in (:1) 中的参数必须是List或Set或Array，同时返回参数也必须是List或Set或Array。


属性绑定
________

当接口参数传入的是自定义对象时，我们可以使用属性绑定。
下面是属性绑定的实例：

.. code-block:: java

    @SQL("insert into binding_user(uid, name, age) values(:1.uid, :1.name, :1.age)")
    public void addUserByObjIndex(BindingUser user);

需要注意的是，BindingUser类必须含有getUid()，getName()，getAge()方法，
因为:1.uid将调用getUid()方法获取参数，:1.name将调用getName()方法获取参数，:1.age将调用getAge()方法获取参数。
当然，我们依然可以对参数进行重命名：

.. code-block:: java

    @SQL("insert into binding_user(uid, name, age) values(:u.uid, :u.name, :u.age)")
    public void addUserByObjRename(@Rename("u") BindingUser user);

属性自动匹配
____________

在使用自定义对象时，使用:1.uid或:u.uid绑定参数会显得不够简练，mango实现了属性自动匹配功能，使SQL更加简练。
下面是属性自动匹配的实例：

.. code-block:: java

    @SQL("insert into binding_user(uid, name, age) values(:uid, :name, :age)")
    public void addUserByProperty(BindingUser user);

混合绑定
________

各种参数绑定混合使用：

.. code-block:: java

    @SQL("insert into binding_user(uid, name, age) values(:myuid, :name, :age)")
    public void addUserByMix(@Rename("myuid") int uid, BindingUser user);

查看完整示例代码和表结构
________________________

**参数绑定** 的所有代码和表结构均可以在 `mango-example <https://github.com/jfaster/mango-example/tree/master/src/main/java/org/jfaster/mango/example/binding>`_ 中找到。

