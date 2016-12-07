实时统计
========

实时统计是mango框架的一重要特性。mango框架的实时统计主要包含：数据库执行总次数，数据库执行失败次数，数据库执行平均速率，缓存命中率，缓存执行总次数，缓存执行失败次数，各种缓存操作平均速率等核心统计数据。通过这些统计数据，我们能实时并全方位的了解我们的系统，方便我们更好的进行压力测试与线上实时性能监控。

使用web页面查看实时统计数据
___________________________

最简单的查看实时统计的方法是使用web页面查看，如果我们在web项目中使用了mango框架，那么只需要将下面的配置文件添加到web项目的web.xml文件中，即可完成所有准备工作。

.. code-block:: xml

    <servlet>
        <servlet-name>mango-stat</servlet-name>
        <servlet-class>org.jfaster.mango.plugin.stats.MangoStatServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>mango-stat</servlet-name>
        <url-pattern>/mango-stat</url-pattern>
    </servlet-mapping>

上面的配置文件，我们使用了一个类全名为 **org.jfaster.mango.plugin.stats.MangoStatServlet** 的servlet，并将它映射到 **/mango-stat** 路径下，假设web服务的ip为127.0.0.1，端口号为8080，那门我们只需要用浏览器访问htt://127.0.0.1:8080/mango-stat即可查看到实时统计数据。

下面是使用浏览器访问实时统计web页面的截图：

.. image:: _static/mango-stat.png
    :width: 800px

您也可以checkout实时统计示例源码 `mango-stat-example <https://github.com/jfaster/mango-stat-example>`_ 自行运行示例代码并查看实时统计。

.. code-block:: bash

    git clone https://github.com/jfaster/mango-stat-example.git
    cd mango-stat-example
    mvn clean jetty:run
    # 使用浏览器访问htt://127.0.0.1:8080/mango-stat

周期性生成实时统计数据
______________________

todo
