.. _查询映射:

查询映射
========

查询映射指的是：将使用select语句从数据库表中查询到的字段，映射到原生对象或者自定义对象的属性中。

映射到原生对象
______________

当我们通过select语句只查询数据库表中的一列字段时，我们可以将该列字段的数据映射到原生对象中。

所谓原生对象指的是：String，int等基本对象，它们能和数据库中的varchar，int等类型一一对应。

下面是映射到原生对象的实例：

.. code-block:: java

    @SQL("select name from user where id = :1")
    public String getNameById(int id);

    @SQL("select id from user limit :1")
    public List<Integer> getIdsLimit(int limit);


映射到自定义对象的属性中
________________________

当我们通过select语句查询数据库表中的多列字段时，我们可以将这些字段的数据映射到自定义对象的属性中。

默认映射规则
^^^^^^^^^^^^

当列字段名称与对象属性名称满足下面的匹配规则时，该列字段的数据将会映射到自定义对象的该属性中：

1. 将对象属性名称记做P  
2. 将P由驼峰式规则变为下划线规则，得到新的名称Q（如：P="userId"则Q="user_id"）
3. 如果某个列字段名称与名称P或Q相等（忽略大小写），则匹配满足，能完成映射
   
下面是匹配规则实例表格：

============    ================    ========
对象属性名称    数据库列字段名称    是否匹配
============    ================    ========
userAge         user_age            匹配
userAge         userAge             匹配
userAge         USERAGE             匹配
userage         userAge             匹配
user_age        userage             不匹配
user_age        userAge             不匹配
============    ================    ========

下面是默认映射规则的实例代码：

.. code-block:: java

    public class MappingUser {
        private int id;
        private String name;
        private int userAge;
        private Date updateTime;

        // 各个属性对应的get与set方法必须加上，这里省略掉了
    }

.. code-block:: java

    @SQL("select id, name, user_age, update_time from mapping_user where id = :1")
    public MappingUser getMappingUserById(int id);

    @SQL("select id, name, user_age, update_time from mapping_user where id in (:1)")
    public List<MappingUser> getMappingUsersById(List<Integer> ids);

自定义映射规则
^^^^^^^^^^^^^^

一般来说，我们可以使用默认映射规则完成绝大部分匹配。如果遇到默认映射规则无法完成的匹配，我们可以使用自定义映射规则。

.. code-block:: java

    public class MappingUser2 {
        private int userId;
        private String userName;
        private int userAge;
        private Date updateTime;

        // 各个属性对应的get与set方法必须加上，这里省略掉了
    }

请看上面的类，属性userId和userName显然不能与列字段id和name完成默认规则匹配。这时我们可以用使用注解 **@org.jfaster.mango.annotation.Results** 和注解 **@org.jfaster.mango.annotation.Result** 来完成自定义映射规则匹配。

下面是自定义映射规则的实例：

.. code-block:: java

    @Results({
            @Result(column = "id", property = "userId"),
            @Result(column = "name", property = "userName")
    })
    @SQL("select id, name, user_age, update_time from mapping_user where id = :1")
    public MappingUser2 getMappingUser2ById(int id);

    @Results({
            @Result(column = "id", property = "userId"),
            @Result(column = "name", property = "userName")
    })
    @SQL("select id, name, user_age, update_time from mapping_user where id in (:1)")
    public List<MappingUser2> getMappingUsers2ById(List<Integer> ids);

手动映射
^^^^^^^^

无论是默认映射规则还是自定义映射规则都是通过反射的形式进行列字段到对象属性的映射。
mango提供了抽象类 **org.jfaster.mango.jdbc.AbstractRowMapper**，继承该类可以实现手动映射。

下面是手动映射的实例：

.. code-block:: java

    public class UserMapper extends AbstractRowMapper<MappingUser> {
    
        @Override
        public MappingUser mapRow(ResultSet rs, int rowNum) throws SQLException {
            MappingUser u = new MappingUser();
            u.setId(rs.getInt("id"));
            u.setName(rs.getString("name"));
            u.setUserAge(rs.getInt("user_age"));
            u.setUpdateTime(rs.getTimestamp("update_time"));
            return u;
        }

    }

.. code-block:: java

    @Mapper(UserMapper.class)
    @SQL("select id, name, user_age, update_time from mapping_user where id = :1")
    public MappingUser getMappingUserByIdMapper(int id);
    
    @Mapper(UserMapper.class)
    @SQL("select id, name, user_age, update_time from mapping_user where id in (:1)")
    public List<MappingUser> getMappingUsersByIdMapper(List<Integer> ids);

查看完整示例代码和表结构
________________________

**查询映射** 的所有代码和表结构均可以在 `mango-example <https://github.com/jfaster/mango-example/tree/master/src/main/java/org/jfaster/mango/example/mapping>`_ 中找到。

