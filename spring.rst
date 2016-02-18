集成到spring
============

在项目开发中，一般都会使用spring管理对象，进行依赖注入。
我们能通过mango自带的spring插件，便捷的将mango集成到spring中。

纯配置文件集成
______________

纯配置文件集成是最简单的集成方式，所有的集成操作均在spring配置文件中，由spring容器创建数据库源工厂，mango对象和扫描使用@DB注解修饰的DAO类。
下面是一个简单的纯配置文件集成实例:

.. code-block:: xml

	<beans>

	    <!-- 配置数据源工厂 -->
	    <bean id="dsf" class="org.jfaster.mango.datasource.SimpleDataSourceFactory">
	        <property name="dataSource">
	            <bean class="org.jfaster.mango.datasource.DriverManagerDataSource">
	                <constructor-arg index="0" value="com.mysql.jdbc.Driver" />
	                <constructor-arg index="1" value="jdbc:mysql://localhost:3306/mango_example" />
	                <constructor-arg index="2" value="root"/>
	                <constructor-arg index="3" value="root" />
	            </bean>
	        </property>
	    </bean>

	    <!-- 配置mango对象 -->
	    <bean id="mango" class="org.jfaster.mango.operator.Mango" factory-method="newInstance">
	        <property name="dataSourceFactory" ref="dsf" />
	    </bean>

	    <!-- 配置扫描使用@DB注解修饰的DAO类 -->
	    <bean class="org.jfaster.mango.plugin.spring.MangoConfigurer">
	        <property name="packages">
	            <list>
	                <!-- 扫描包名 -->
	                <value>org.jfaster.mango.example.spring</value>

	                <!-- <value>其他需要扫描的包</value> -->
	            </list>
	        </property>
	    </bean>

	</beans>

上面的配置主要包含3部分。

1. **配置数据源工厂**。DriverManagerDataSource数据源是mango框架对DataSource的简单实现，仅供测试使用，如果是生产环境，请使用c3p0，dbcp等高性能数据源。
2. **配置mango对象**。创建mango对象的最佳途径是通过Mango类的静态方法newInstance，所以使用了factory-method指定静态方法。
3. **配置扫描使用@DB注解修饰的DAO类**。配置后mango会自动扫描指定包下的所有类，识别出@DB注解修饰的DAO类，并将他自动加载到spring大工厂中，这样我们既能从ApplicationContext中直接getBean获得dao实例，也能将dao实例直接Autowired到所有由spring管理的类上。需要注意的是所有DAO类必须以DAO或Dao结尾，才能被扫描器识别。
   

配置文件加代码集成
__________________

一般情况下使用纯配置文件集成就能完成大多数项目需求，但在有些项目中，我们需要自己管理数据源工厂，以便在线上动态切换数据源。这时数据源工厂和mango对象对象就不能简单的交由spring管理，我们需要使用配置文件加代码的方式来完成集成。

下面是一个简单的配置文件加代码集成实例:

.. code-block:: xml

    <beans>

        <!-- 配置扫描使用@DB注解修饰的DAO类 -->
        <bean class="org.jfaster.mango.plugin.spring.MangoConfigurer">
            <property name="packages">
                <list>
                    <!-- 扫描包名 -->
                    <value>org.jfaster.mango.example.spring</value>

                    <!-- <value>其他需要扫描的包</value> -->
                </list>
            </property>

            <property name="factoryBeanClass" value="org.jfaster.mango.example.spring.MyMangoFactoryBean" />
        </bean>

    </beans>

.. code-block:: java

	public class MyMangoFactoryBean extends AbstractMangoFactoryBean {

	    @Override
	    public Mango createMango() {
	        String driverClassName = "com.mysql.jdbc.Driver";
	        String url = "jdbc:mysql://localhost:3306/mango_example";
	        String username = "root"; // 这里请使用您自己的用户名
	        String password = "root"; // 这里请使用您自己的密码
	        DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
	        Mango mango = Mango.newInstance(ds); // 使用数据源初始化mango
	        return mango;
	    }

	}    

上面的实例分为spring配置文件与代码两部分。在spring配置文件中只有扫描DAO类的配置，并多了一个对扫描器MangoConfigurer的factoryBeanClass属性的配置，factoryBeanClass的值是一个自定义的类MyMangoFactoryBean。代码部分，自定义类MyMangoFactoryBean继承了mango自带的抽象类org.jfaster.mango.plugin.spring.AbstractMangoFactoryBean，MyMangoFactoryBean通过实现createMango方法，实现用代码创建数据源工厂与mango对象。


查看完整示例代码
________________

和集成到spring的所有代码均可以在 `mango-example <https://github.com/jfaster/mango-example/tree/master/src/main/java/org/jfaster/mango/example/spring>`_ 中找到。










