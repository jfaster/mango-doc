增删改查NoSQL
=============

为提高开发效率，mango框架提供了一整套NoSQL的方案，开发人员无需书写SQL即可完成绝大多数的数据库增删改查操作。

CrudDao接口
___________

为实现增删改查NoSQL，mango框架对开发人员提供了 `CrudDao <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/crud/CrudDao.java>`_ 接口，我们只需简单继承CrudDao接口，不需要书写任何SQL，即可获得常用的增删改查方法。

CrudDao定义如下：

.. code-block:: java

	public interface CrudDao<T, ID> extends Generic<T, ID> {

	  void add(T entity);

	  int addAndReturnGeneratedId(T entity);

	  void add(Collection<T> entities);

	  T getOne(ID id);

	  List<T> getMulti(List<ID> ids);

	  List<T> getAll();

	  long count();

	  int update(T entity);

	  int[] update(Collection<T> entities);

	  int delete(ID id);

	}

订单表增删改查实例
__________________

Order表
^^^^^^^

.. code-block:: sql

	CREATE TABLE `t_order` (
	  `id` int(11) NOT NULL AUTO_INCREMENT,
	  `uid` int(11) NOT NULL,
	  `status` int(11) NOT NULL,
	  PRIMARY KEY (`id`)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8;

Order对象
^^^^^^^^^

.. code-block:: java

	public class Order {

	  @ID
	  private Integer id;

	  private Integer uid;

	  private Integer status;

	  // 各个属性对应的get与set方法必须加上，这里省略掉了

	}

请注意，t_order表中id字段为自增主键，我们需要在Order类的id属性上使用 `@ID <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/annotation/ID.java>`_ 注解。

OrderDao
^^^^^^^^

.. code-block:: java

	@DB(table = "t_order")
	public interface OrderDao extends CrudDao<Order, Integer> {
	}

OrderDao只是简单的继承了 `CrudDao <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/crud/CrudDao.java>`_ 接口，就能获得常用的增删改查方法。

测试代码
^^^^^^^^

.. code-block:: java

	public class OrderDaoMain {

	  public static void main(String[] args) {
	    String driverClassName = "com.mysql.jdbc.Driver";
	    String url = "jdbc:mysql://localhost:3306/mango_example";
	    String username = "root"; // 这里请使用您自己的用户名
	    String password = "root"; // 这里请使用您自己的密码
	    DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
	    Mango mango = Mango.newInstance(ds); // 使用数据源初始化mango

	    OrderDao dao = mango.create(OrderDao.class);
	    Order order = new Order();
	    order.setUid(100);
	    order.setStatus(1);
	    dao.add(order);
	    int id = dao.addAndReturnGeneratedId(order);
	    order.setId(id);
	    System.out.println(dao.getOne(id));
	    order.setStatus(2);
	    dao.update(order);
	    System.out.println(dao.getOne(id));
	  }

	}

常用注解
________

OrderB表
^^^^^^^^

.. code-block:: sql

	CREATE TABLE `t_order_b` (
	  `id` int(11) NOT NULL,
	  `uid` int(11) NOT NULL,
	  `status` int(11) NOT NULL,
	  PRIMARY KEY (`id`)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8;

OrderB对象
^^^^^^^^^^

.. code-block:: java

	public class OrderB {

	  @ID(autoGenerateId = false)
	  private Integer id;

	  @Column("uid")
	  private Integer userId;

	  private Integer status;

	  @Ignore
	  private String address;

	  // 各个属性对应的get与set方法必须加上，这里省略掉了

	}

请注意：

* t_order_b表中id字段为非自增主键，我们需要在Order类的id属性上使用 `@ID <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/annotation/ID.java>`_ 注解，并设置autoGenerateId = false

* 使用 `@Column <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/annotation/Column.java>`_ 注解，将OrderB类的userId属性映射到t_order_b表的uid字段中

* t_order_b表中没有address字段，我们使用 `@Ignore <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/annotation/Ignore.java>`_ 注解，忽略OrderB类的address属性

自定义查询
__________

mango框架提供使用方法名的方式自定义查询，方法名必须以getBy,findBy,queryBy,selectBy开头

===============================    ========================================    ============================================
关键字                              样例                                         对应SQL       
===============================    ========================================    ============================================
And                                findByIdAndName                             … where id = :1 and name = :2
Or                                 findByIdOrName                              … where id = :1 or name = :2
Equals                             findById,findByIdEquals                     … where id = :1
Between                            findByStartDateBetween                      … where startDate between :1 and :2
LessThan                           findByAgeLessThan                           … where age < :1
LessThanEqual                      findByAgeLessThanEqual                      … where age <= :1
GreaterThan                        findByAgeGreaterThan                        … where age > :1
GreaterThanEqual                   findByAgeGreaterThanEqual                   … where age >= :1
IsNull                             findByAgeIsNull                             … where age is null
NotNull                            findByAgeNotNull                            … where age not null
OrderBy                            findByAgeOrderByIdDesc                      … where age = :1 order by id desc
Not                                findByLastnameNot                           … where lastname <> :1
In                                 findByAgeIn(Collection<Age> ages)           … where id = :1 or name = :2
NotIn                              findByAgeNotIn(Collection<Age> ages)        … where id = :1 or name = :2
True                               findByActiveTrue                            … where active = true
False                              findByActiveFalse                           … where active = false
===============================    ========================================    ============================================




