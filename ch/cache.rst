集成cache
=========

SimpleCacheHandler抽象类简介
____________________________

mango自身不依赖任何缓存工具，mango对外提供SimpleCacheHandler抽象类，您只需实现SimpleCacheHandler抽象类，并在其中填写适当的缓存操作代码（memcached，redis，直接内存等均可），就能享受mango带来的缓存操作便利。

SimpleCacheHandler抽象类需要实现的抽象方法如下:

.. code-block:: java

    public abstract Object get(String key);

    public abstract Map<String, Object> getBulk(Set<String> keys);

    public abstract void set(String key, Object value, int exptimeSeconds);

    public abstract void delete(String key);


CacheHandler抽象类一共有4个需要实现的抽象方法，它们分别对应着封装缓存的操作:

* ``Object get(String key)`` ，根据单个key值从缓存中查找数据。
* ``Map<String, Object> getBulk(Set<String> keys)`` ，根据多个key值从缓存中查找数据，返回key-value对应的map。
* ``void set(String key, Object value, int exptimeSeconds)``，向缓存中设置数据，其中exptimeSeconds为缓存失效时间，单位为秒。
* ``void delete(String key)`` ，根据单个key值从缓存中删除数据。

实现SimpleCacheHandler抽象类
____________________________

为了展示的简单性，下面的代码使用了ConcurrentHashMap对SimpleCacheHandler抽象类进行了实现，在生产环境中，您可以通过Memcache或Redis实现SimpleCacheHandler抽象类。

.. code-block:: java

    package org.jfaster.mango.example.cache;

    import org.jfaster.mango.operator.cache.SimpleCacheHandler;

    import java.util.HashMap;
    import java.util.Map;
    import java.util.Set;
    import java.util.concurrent.ConcurrentHashMap;

    public class CacheHandlerImpl extends SimpleCacheHandler {

        private ConcurrentHashMap<String, Object> cache = new ConcurrentHashMap<String, Object>();

        @Override
        public Object get(String key) {
            return cache.get(key);
        }

        @Override
        public Map<String, Object> getBulk(Set<String> keys) {
            Map<String, Object> map = new HashMap<String, Object>();
            for (String key : keys) {
                map.put(key, cache.get(key));
            }
            return map;
        }

        @Override
        public void set(String key, Object value, int expires) {
            cache.put(key, value);
        }

        @Override
        public void delete(String key) {
            cache.remove(key);
        }

    }

初始化mango对象
_______________

.. code-block:: java

    DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
    Mango mango = Mango.newInstance(ds); 
    mango.setDefaultCacheHandler(new CacheHandlerImpl());

正常初始化mango对象后，只需要通过setDefaultCacheHandler方法传入一个实现了CacheHandler接口的对象即可。

.. _单key取单值:

单key取单值
___________

使用场景
^^^^^^^^

我们有一张user表，表里有两个字段uid和name，其中uid是唯一主键，用来唯一标识用户的身份，name用于标识用户的名字。
对user表的操作有4个：增，删，改，查，由于user表的查找压力很大，所以我需要根据uid进行缓存，缓存方式如下:

* 增：插入新的user数据，不需要操作缓存。
* 删：根据uid删除user数据，清空uid对应的缓存。
* 改：根据uid更新user数据，清空uid对应的缓存。
* 查：根据uid从缓存中查找数据，如果找到直接返回，如果缓存中没有，从db中查找数据，如果db中有数据，将数据放入uid对应的缓存并返回，如果db中没有数据，直接返回null。

创建user表
^^^^^^^^^^

这里我们使用MySQL数据库:

.. code-block:: sql

    DROP TABLE IF EXISTS `user`;
    CREATE TABLE `user` (
      `uid` int(11) NOT NULL,
      `name` varchar(20) NOT NULL,
      PRIMARY KEY (`uid`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

创建User对象
^^^^^^^^^^^^

.. code-block:: java

    package org.jfaster.mango.example.cache;

    public class User {

        private int uid;
        private String name;

        public int getUid() {
            return uid;
        }

        public void setUid(int uid) {
            this.uid = uid;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            return "uid=" + uid + ", name=" + name;
        }
    }

书写DAO接口
^^^^^^^^^^^

.. code-block:: java

    package org.jfaster.mango.example.cache;

    import org.jfaster.mango.annotation.*;
    import org.jfaster.mango.operator.cache.Hour;

    @DB
    @Cache(prefix = "user", expire = Hour.class, num = 2)
    public interface SingleKeySingeValueDao {

        @CacheIgnored
        @SQL("insert into user(uid, name) values(:1, :2)")
        public int insert(int uid, String name);

        @SQL("delete from user where uid=:1")
        public int delete(@CacheBy int uid);

        @SQL("update user set name=:2 where uid=:1")
        public int update(@CacheBy int uid, String name);

        @SQL("select uid, name from user where uid=:1")
        public User getUser(@CacheBy int uid);

    }

上面的代码引入了3个新的注解:

* @Cache表示需要使用缓存，参数prefix表示key前缀，比如说传入uid=1，那么缓存中的key就等于user_1，参数expire表示缓存过期时间，Hour.class表示小时，配合后面的参数num＝2表示缓存过期的时间为2小时。
* @CacheBy用于修饰key后缀参数，在delete，update，getUser方法中@CacheBy都是修饰的uid，所以当传入uid=1时，缓存中的key就等于user_1。
* @CacheIgnored表示该方法不操作缓存。需要注意的是，如果使用了@Cache注解，@CacheBy和@CacheIgnored二者必须有一个存在。

编写测试代码
^^^^^^^^^^^^

.. code-block:: java

    package org.jfaster.mango.example.cache;

    import org.jfaster.mango.datasource.DriverManagerDataSource;
    import org.jfaster.mango.operator.Mango;

    import javax.sql.DataSource;

    public class SingleKeySingeValue {

        public static void main(String[] args) {
            String driverClassName = "com.mysql.jdbc.Driver";
            String url = "jdbc:mysql://localhost:3306/mango_example";
            String username = "root"; // 这里请使用您自己的用户名
            String password = "root"; // 这里请使用您自己的密码
            DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
            Mango mango = Mango.newInstance(ds);
            mango.setDefaultCacheHandler(new CacheHandlerImpl());

            SingleKeySingeValueDao dao = mango.create(SingleKeySingeValueDao.class);
            dao.insert(1, "ash");
            dao.insert(2, "lucy");
            System.out.println(dao.getUser(1));
            System.out.println(dao.getUser(2));
            dao.update(2, "lily");
            System.out.println(dao.getUser(2));
            dao.delete(1);
            System.out.println(dao.getUser(1));
        }

    }

运行上面的代码（运行代码前先保证user表中没有数据），得到如下输出::

    uid=1, name=ash
    uid=2, name=lucy
    uid=2, name=lily
    null

单key取多值
___________

使用场景
^^^^^^^^

我们有一张message表，表里有三个字段：id，uid和content，其中id是自增唯一主键，用来唯一标识消息，uid用于标识消息的所有者，1个uid可以对应多个消息，content则标识消息的内容。对message表的操作有4个：增，删，改，查，由于message表的查找压力很大，所以我需要根据uid进行缓存，缓存方式如下:

* 增：插入新的message数据，由于我们是根据uid取出消息列表，所以这里需要清空uid对应的缓存。
* 删：根据uid删除message数据，清空uid对应的缓存。
* 改：根据uid更新message数据，清空uid对应的缓存。
* 查：根据uid从缓存中查找消息列表（List或Set或数组），如果找到直接返回，如果缓存中没有，从db中查找列表，如果db中有数据，将数据放入uid对应的缓存并返回，如果db中没有数据，返回空列表。

创建message表
^^^^^^^^^^^^^

这里我们使用MySQL数据库:

.. code-block:: sql

    DROP TABLE IF EXISTS `message`;
    CREATE TABLE `message` (
      `id` int(11) NOT NULL AUTO_INCREMENT,
      `uid` int(11) NOT NULL,
      `content` varchar(100) NOT NULL,
      PRIMARY KEY (`id`),
      KEY `key_uid` (`uid`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

创建Message对象
^^^^^^^^^^^^^^^

.. code-block:: java

    package org.jfaster.mango.example.cache;

    public class Message {

        private int id;
        private int uid;
        private String content;

        public int getId() {
            return id;
        }

        public void setId(int id) {
            this.id = id;
        }

        public int getUid() {
            return uid;
        }

        public void setUid(int uid) {
            this.uid = uid;
        }

        public String getContent() {
            return content;
        }

        public void setContent(String content) {
            this.content = content;
        }

        @Override
        public String toString() {
            return "id=" + id + ", uid=" + uid + ", content=" + content;
        }
    }

书写DAO接口
^^^^^^^^^^^

.. code-block:: java

    package org.jfaster.mango.example.cache;

    import org.jfaster.mango.annotation.*;
    import org.jfaster.mango.operator.cache.Day;

    import java.util.List;

    @DB
    @Cache(prefix = "message", expire = Day.class)
    public interface SingleKeyMultiValuesDao {

        @ReturnGeneratedId
        @SQL("insert into message(uid, content) values(:1.uid, :1.content)")
        public int insert(@CacheBy("uid") Message message);

        @SQL("delete from message where uid=:1 and id=:2")
        public int delete(@CacheBy int uid, int id);

        @SQL("update message set content=:1.content where id=:1.id and uid=:1.uid")
        public int update(@CacheBy("uid") Message message);

        @SQL("select id, uid, content from message where uid=:1 order by id")
        public List<Message> getMessages(@CacheBy int uid);

    }

值得注意的是上面代码的 ``@CacheBy("uid") Message message`` ，它表示使用message对象的uid属性作为key后缀。

编写测试代码
^^^^^^^^^^^^

.. code-block:: java

    package org.jfaster.mango.example.cache;

    import org.jfaster.mango.datasource.DriverManagerDataSource;
    import org.jfaster.mango.operator.Mango;

    import javax.sql.DataSource;

    public class SingleKeyMultiValues {

        public static void main(String[] args) {
            String driverClassName = "com.mysql.jdbc.Driver";
            String url = "jdbc:mysql://localhost:3306/mango_example";
            String username = "root"; // 这里请使用您自己的用户名
            String password = "root"; // 这里请使用您自己的密码
            DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
            Mango mango = Mango.newInstance(ds);
            mango.setDefaultCacheHandler(new CacheHandlerImpl());

            SingleKeyMultiValuesDao dao = mango.create(SingleKeyMultiValuesDao.class);
            int uid = 1;
            Message message = newMessage(uid, "hello");
            Message message2 = newMessage(uid, "world");
            Message message3 = newMessage(uid, "boy");
            message.setId(dao.insert(message));
            message2.setId(dao.insert(message2));
            message3.setId(dao.insert(message3));
            System.out.println(dao.getMessages(uid));
            message3.setContent("girl");
            dao.update(message3);
            System.out.println(dao.getMessages(uid));
            dao.delete(uid, message.getId());
            System.out.println(dao.getMessages(uid));
        }

        private static Message newMessage(int uid, String content) {
            Message message = new Message();
            message.setUid(uid);
            message.setContent(content);
            return message;
        }

    }

运行上面的代码（运行代码前先保证message表中没有数据，有的话请先delete掉），得到如下输出::

    [id=1, uid=1, content=hello, id=2, uid=1, content=world, id=3, uid=1, content=boy]
    [id=1, uid=1, content=hello, id=2, uid=1, content=world, id=3, uid=1, content=girl]
    [id=2, uid=1, content=world, id=3, uid=1, content=girl]

多key取多值
___________

扩展单key取单值
^^^^^^^^^^^^^^^

我们对 :ref:`单key取单值` 的使用场景进行扩展，增加一个批量查找的操作:

* 批量查找：根据uid列表从缓存中查找数据，得到命中数据与丢失数据，从db中查找丢失数据，然后和命中数据合在一起返回。

书写DAO接口
^^^^^^^^^^^

.. code-block:: java

    package org.jfaster.mango.example.cache;

    import org.jfaster.mango.annotation.*;
    import org.jfaster.mango.operator.cache.Hour;

    import java.util.List;

    @DB
    @Cache(prefix = "user", expire = Hour.class, num = 2)
    public interface MultiKeysMultiValuesDao {

        @CacheIgnored
        @SQL("insert into user(uid, name) values(:1, :2)")
        public int insert(int uid, String name);

        @SQL("delete from user where uid=:1")
        public int delete(@CacheBy int uid);

        @SQL("update user set name=:2 where uid=:1")
        public int update(@CacheBy int uid, String name);

        @SQL("select uid, name from user where uid=:1")
        public User getUser(@CacheBy int uid);

        @SQL("select uid, name from user where uid in (:1)")
        public List<User> getUsers(@CacheBy List<Integer> uids);

    }

前面的4个增删改查方法和 :ref:`单key取单值` 一样，新增 ``public List<User> getUsers(@CacheBy List<Integer> uids)`` 。

编写测试代码
^^^^^^^^^^^^

.. code-block:: java

    package org.jfaster.mango.example.cache;

    import org.jfaster.mango.datasource.DriverManagerDataSource;
    import org.jfaster.mango.operator.Mango;

    import javax.sql.DataSource;
    import java.util.Arrays;

    public class MultiKeysMultiValues {

        public static void main(String[] args) {
            String driverClassName = "com.mysql.jdbc.Driver";
            String url = "jdbc:mysql://localhost:3306/mango_example";
            String username = "root"; // 这里请使用您自己的用户名
            String password = "root"; // 这里请使用您自己的密码
            DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
            Mango mango = Mango.newInstance(ds);
            mango.setDefaultCacheHandler(new CacheHandlerImpl());

            MultiKeysMultiValuesDao dao = mango.create(MultiKeysMultiValuesDao.class);
            dao.insert(100, "ash");
            dao.insert(200, "lucy");
            dao.insert(300, "lily");
            System.out.println(dao.getUsers(Arrays.asList(100, 200, 300)));
        }

    }

运行上面的代码（运行代码前先保证user表中没有数据），得到如下输出::

    [uid=100, name=ash, uid=200, name=lucy, uid=300, name=lily]

多个参数组成单个key
___________________

考虑上面的user表，如果我们需要通过uid和name两个字段来作为缓存呢？

下面两种方式都能实现::

    @SQL("select uid, name from user where uid=:1 and name=:2")
    public User getByUidAndName(@CacheBy int uid, @CacheBy String name);

    @SQL("select uid, name from user where uid=:1.uid and name=:1.name")
    public User getByUidAndName(@CacheBy("uid,name") User user);


查看完整示例代码和表结构
________________________

**cache集成** 的所有代码和表结构均可以在 `mango-example <https://github.com/jfaster/mango-example/tree/master/src/main/java/org/jfaster/mango/example/cache>`_ 中找到。
