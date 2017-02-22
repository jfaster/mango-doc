实时统计
========

当我们使用mango框架访问数据库（或缓存）时，mango框架中的统计模块会实时的统计数据库（或缓存）的总访问次数，错误访问次数，平均响应时间，缓存命中率等数据。通过这些统计数据，我们能实时并全方位的了解我们的系统，方便我们更好的进行压力测试与线上实时性能监控。

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

使用web页面查看实时统计数据的方式简单明了，可以清晰的看到从服务开始运行到当前时间段之间所有统计信息的汇总。

我们来考虑下面问题：

假设服务已经稳定运行1天，某个DAO方法被执行了10000次，同时共花费了10000毫秒，这时我们不难得出该DAO方法的平均响应为：10000毫秒除以10000次等于 **1毫秒**。

突然第10001次dao方法执行出现问题，响应时间为200毫秒，这时dao方法的平均响应时间是为：10200毫秒除以10001约等于 **1.02毫秒**。虽然第10001次dao方法执行出现了问题，但由于累计了前面10000次的统计信息，我们很难通过总的平均响应时间看出第10001次dao方法的执行已出现性能问题。

为了使统计数据能更准确的反映最近一段时间dao方法调用的性能，我们不能累积从服务开始运行到当前时间段所有的统计数据，而是应该将统计数据周期性的输出（比如以10秒为一个周期输出统计数据），为此mango框架提供了StatMonitor接口。

StatMonitor接口的类全名为org.jfaster.mango.stat.StatMonitor

.. code-block:: java

    public interface StatMonitor {

        public int periodSecond();

        public void handleStat(long statBeginTime, long statEndTime, List<OperatorStat> stats) throws Exception;

    }

StatMonitor接口一共有两个需要实现的方法：

* ``int periodSecond()``，指定统计周期，单位为秒，比如当返回10时表示以10秒为一个统计周期。
* ``void handleStat(long statBeginTime, long statEndTime, List<OperatorStat> stats)``，每过一个统计周期会自动调用一次handleStat方法，所以我们可以将处理统计数据的逻辑放在handleStat方法中。参数statBeginTime为统计开始时间，参数statEndTime为统计结束时间，它们的单位都为毫秒；参数stats是一个OperatorStat对象的列表，对应前一个周期所有DAO方法的统计数据，有关OperatorStat的相信信息请查看 :ref:`OperatorStat类统计数据表格`。

有关StatMonitor接口的使用，请看下面的两个示例。

**示例1：以10秒为统计周期，如果有dao方法在10秒内平均响应时间大于10毫秒，则促发短信或邮件报警**

.. code-block:: java

    DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
    Mango mango = Mango.newInstance(ds); // 使用数据源初始化mango

    mango.setStatMonitor(new StatMonitor() {

        @Override
        public void handleStat(long statBeginTime, long statEndTime, List<OperatorStat> stats) throws Exception {
            for (OperatorStat stat : stats) {
                if (stat.getDatabaseAverageExecutePenalty() > TimeUnit.MILLISECONDS.toNanos(10)) {
                    // 有dao方法在10秒内平均响应时间大于10毫秒，促发短信或邮件报警
                    break;
                }
            }
        }

        @Override
        public int periodSecond() {
            return 10;
        }

    });

    // 后续创建DAO，执行DAO的代码


**示例2：以10秒为统计周期，输出所有dao方法的平均响应时间，执行总数，执行异常总数**

.. code-block:: java

    DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
    Mango mango = Mango.newInstance(ds);

    mango.setStatMonitor(new StatMonitor() {

        @Override
        public void handleStat(long statBeginTime, long statEndTime, List<OperatorStat> stats) throws Exception {
            StringBuilder data = new StringBuilder();
            data.append("Performance Statistics  [")
                    .append(format(statBeginTime))
                    .append("] - [")
                    .append(format(statEndTime))
                    .append("]")
                    .append("\n");
            data.append(String.format("%-36s%-12s%-12s%-12s%n",
                    "dao", "avg", "total", "error"));
            for (OperatorStat stat : stats) {
                if (stat.getDatabaseExecuteCount() > 0) { // 执行db数大于0才统计
                    String dao = stat.getMethod().getDeclaringClass().getSimpleName() + "." + stat.getMethod().getName();
                    data.append(String.format("%-36s%-12.1f%-12s%-12s%n",
                            dao,
                            (double) stat.getDatabaseAverageExecutePenalty() / (1000*1000), // 平均响应时间
                            stat.getDatabaseExecuteCount(), // 执行总数
                            stat.getDatabaseExecuteExceptionCount())); // 执行异常总数
                }
            }
            System.out.println(data);
        }

        @Override
        public int periodSecond() {
            return 10;
        }

        private String format(long time) {
            SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            return format.format(new Date(time));
        }

    });

    // 后续创建DAO，执行DAO的代码


.. _OperatorStat类统计数据表格:

OperatorStat类统计数据表格
__________________________

===============================    ========================    ============================================
属性                               类型                        描述           
===============================    ========================    ============================================
method                             java.lang.reflect.Method    SQL操作所在的方法
operatorType                       枚举类型OperatorType        SQL操作类型
isCacheable                        boolean                     是否使用缓存
initCount                          long                        SQL操作被初始化次数（一般情况为1）
databaseExecuteSuccessCount        long                        数据库执行成功数        
databaseExecuteExceptionCount      long                        数据库执行异常数
databaseExecuteCount               long                        数据库执行总数（成功数＋异常数）
databaseExecuteSuccessRate         double                      数据库执行成功率
databaseExecuteExceptionRate       double                      数据库执行异常率
databaseExecuteAveragePenalty      long                        数据库执行平均响应时间（单位为纳秒）
hitCount                           long                        缓存命中数
missCount                          long                        缓存丢失数
hitRate                            double                      缓存命中率
cacheGetSuccessCount               long                        缓存get操作成功数
cacheGetExceptionCount             long                        缓存get操作异常数
cacheGetCount                      long                        缓存get操作总数（成功数＋异常数）
cacheGetSuccessRate                double                      缓存get操作成功率
cacheGetExceptionRate              double                      缓存get操作异常率
cacheGetAveragePenalty             long                        缓存get操作平均响应时间（单位为纳秒）
cacheGetBulkSuccessCount           long                        缓存批量get操作成功数
cacheGetBulkExceptionCount         long                        缓存批量get操作异常数
cacheGetBulkCount                  long                        缓存批量get操作总数（成功数＋异常数）
cacheGetBulkSuccessRate            double                      缓存批量get操作成功率
cacheGetBulkExceptionRate          double                      缓存批量get操作异常率
cacheGetBulkAveragePenalty         long                        缓存批量get操作平均响应时间（单位为纳秒）
cacheSetSuccessCount               long                        缓存set操作成功数
cacheSetExceptionCount             long                        缓存set操作异常数
cacheSetCount                      long                        缓存set操作总数（成功数＋异常数）
cacheSetSuccessRate                double                      缓存set操作成功率
cacheSetExceptionRate              double                      缓存set操作异常率
cacheSetAveragePenalty             long                        缓存set操作平均响应时间（单位为纳秒）
cacheAddSuccessCount               long                        缓存add操作成功数
cacheAddExceptionCount             long                        缓存add操作异常数
cacheAddCount                      long                        缓存add操作总数（成功数＋异常数）
cacheAddSuccessRate                double                      缓存add操作成功率
cacheAddExceptionRate              double                      缓存add操作异常率
cacheAddAveragePenalty             long                        缓存add操作平均响应时间（单位为纳秒）
cacheDeleteSuccessCount            long                        缓存delete操作成功数
cacheDeleteExceptionCount          long                        缓存delete操作异常数
cacheDeleteCount                   long                        缓存delete操作总数（成功数＋异常数）
cacheDeleteSuccessRate             double                      缓存delete操作成功率
cacheDeleteExceptionRate           double                      缓存delete操作异常率
cacheDeleteAveragePenalty          long                        缓存delete操作平均响应时间（单位为纳秒）
cacheBatchDeleteSuccessCount       long                        缓存批量delete操作成功数
cacheBatchDeleteExceptionCount     long                        缓存批量delete操作异常数
cacheBatchDeleteCount              long                        缓存批量delete操作总数（成功数＋异常数）
cacheBatchDeleteSuccessRate        double                      缓存批量delete操作成功率
cacheBatchDeleteExceptionRate      double                      缓存批量delete操作异常率
cacheBatchDeleteAveragePenalty     long                        缓存批量delete操作平均响应时间（单位为纳秒）
===============================    ========================    ============================================



