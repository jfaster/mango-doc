输出日志(输出SQL)
=================

mango框架拥有强大的日志处理功能，能通过下面任意一个“日志工具”输出日志（能查看执行的SQL与参数）:

* Slf4J
* Log4J2
* Log4J
* Console(控制台，也就是通过System.out输出)

使用Slf4J输出日志
_________________

准备配置文件
^^^^^^^^^^^^

**logback.xml** 配置文件如下，需要注意的是logger的name为org.jfaster.mango，level为debug。

.. code-block:: xml

	<configuration>
	    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
	        <encoder>
	            <pattern>%d{HH:mm:ss.SSS} - %msg%n</pattern>
	        </encoder>
	    </appender>

	    <logger name="org.jfaster.mango" level="debug" additivity="false">
	        <appender-ref ref="STDOUT"/>
	    </logger>

	    <root level="error">
	        <appender-ref ref="STDOUT" />
	    </root>
	</configuration>

Main方法运行程序
^^^^^^^^^^^^^^^^

有了前面的配置文件，当我们直接使用Main方法运行程序时，只需将下面的代码放到Main方法的最前面，即可让mango框架使用Slf4J输出日志信息。

.. code-block:: java

	MangoLogger.useSlf4JLogger();

上面的代码中MangoLogger的类全名为org.jfaster.mango.util.logging.MangoLogger

WEB容器运行程序
^^^^^^^^^^^^^^^

很多时候我们并不直接通过Main方法运行自己的程序，而是使用tomcat，jetty等WEB容器。这时我们只需将下面的“监听器”拷配到web.xml文件的最前面，即可让mango框架使用Slf4J输出日志信息。

.. code-block:: xml

	<listener>
	    <listener-class>org.jfaster.mango.plugin.listener.Slf4JLoggerListener</listener-class>
	</listener>

使用Log4J2输出日志
__________________

准备配置文件
^^^^^^^^^^^^

**log4j2.xml** 配置文件如下，需要注意的是Logger的name为org.jfaster.mango，level为debug。

.. code-block:: xml

	<?xml version="1.0" encoding="UTF-8"?>
	<Configuration status="WARN">
	    <Appenders>
	        <Console name="Console" target="SYSTEM_OUT">
	            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
	        </Console>
	    </Appenders>
	    <Loggers>
	        <Logger name="org.jfaster.mango" level="debug" additivity="false">
	            <AppenderRef ref="Console"/>
	        </Logger>
	        <Root level="error">
	            <AppenderRef ref="Console"/>
	        </Root>
	    </Loggers>
	</Configuration>

Main方法运行程序
^^^^^^^^^^^^^^^^

有了前面的配置文件，当我们直接使用Main方法运行程序时，只需将下面的代码放到Main方法的最前面，即可让mango框架使用Log4J2输出日志信息。

.. code-block:: java

	MangoLogger.useLog4J2Logger();

上面的代码中MangoLogger的类全名为org.jfaster.mango.util.logging.MangoLogger

WEB容器运行程序
^^^^^^^^^^^^^^^

很多时候我们并不直接通过Main方法运行自己的程序，而是使用tomcat，jetty等WEB容器。这时我们只需将下面的“监听器”拷配到web.xml文件的最前面，即可让mango框架使用Log4J2输出日志信息。

.. code-block:: xml

	<listener>
	    <listener-class>org.jfaster.mango.plugin.listener.Log4J2LoggerListener</listener-class>
	</listener>

使用Log4J输出日志
_________________

准备配置文件
^^^^^^^^^^^^

**log4j.xml** 配置文件如下，需要注意的是logger的name为org.jfaster.mango，level为debug。

.. code-block:: xml

	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
	<log4j:configuration>

	    <appender name="console" class="org.apache.log4j.ConsoleAppender">
	        <layout class="org.apache.log4j.PatternLayout">
	            <param name="ConversionPattern" value="[%d{dd HH:mm:ss,SSS\} %-5p] [%t] %c{2\} - %m%n" />
	        </layout>
	    </appender>

	    <logger name="org.jfaster.mango" additivity="false">
	        <level value="debug" />
	        <appender-ref ref="console" />
	    </logger>

	    <root>
	        <level value="info" />
	        <appender-ref ref="console"/>
	    </root>

	</log4j:configuration>

Main方法运行程序
^^^^^^^^^^^^^^^^

有了前面的配置文件，当我们直接使用Main方法运行程序时，只需将下面的代码放到Main方法的最前面，即可让mango框架使用Log4J输出日志信息。

.. code-block:: java

	MangoLogger.useLog4JLogger();

上面的代码中MangoLogger的类全名为org.jfaster.mango.util.logging.MangoLogger

WEB容器运行程序
^^^^^^^^^^^^^^^

很多时候我们并不直接通过Main方法运行自己的程序，而是使用tomcat，jetty等WEB容器。这时我们只需将下面的“监听器”拷配到web.xml文件的最前面，即可让mango框架使用Log4J输出日志信息。

.. code-block:: xml

	<listener>
	    <listener-class>org.jfaster.mango.plugin.listener.Log4JLoggerListener</listener-class>
	</listener>

使用控制台输出日志
__________________

准备配置文件
^^^^^^^^^^^^

使用System.out输出日志到控制台不需要配置文件

Main方法运行程序
^^^^^^^^^^^^^^^^

当我们直接使用Main方法运行程序时，只需将下面的代码放到Main方法的最前面，即可让mango框架使用控制台输出日志信息。

.. code-block:: java

	MangoLogger.useConsoleLogger();

上面的代码中MangoLogger的类全名为org.jfaster.mango.util.logging.MangoLogger

WEB容器运行程序
^^^^^^^^^^^^^^^

很多时候我们并不直接通过Main方法运行自己的程序，而是使用tomcat，jetty等WEB容器。这时我们只需将下面的“监听器”拷配到web.xml文件的最前面，即可让mango框架使用控制台输出日志信息。

.. code-block:: xml

	<listener>
	    <listener-class>org.jfaster.mango.plugin.listener.ConsoleLoggerListener</listener-class>
	</listener>
