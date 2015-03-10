多数据源配置
============

**为了简单，本文的数据源使用了mango中的DriverManagerDataSource，DriverManagerDataSource只是一个用于测试的简单数据源，线上环境请使用第三方数据源** 。


单数据源
________

单数据源使用SimpleDataSourceFactory。

使用单数据源，不需要在@DB注解中定义dataSource参数，下面的代码使用了单一数据源:

.. code-block:: java

    DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
    Mango mango = Mango.newInstance(new SimpleDataSourceFactory(ds));

或者也可以简写为:

.. code-block:: java

    DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
    Mango mango = Mango.newInstance(ds);;

单数据源读写分离
________________

单数据源读写分离使用MasterSlaveDataSourceFactory。

请看下面的代码:

.. code-block:: java

    DataSource master = new DriverManagerDataSource(driverClassName, url, username, password);
    int slaveNum = 2;
    List<DataSource> slaves = new ArrayList<DataSource>();
    for (int i = 0; i < slaveNum; i++) {
        // 为了简单，参数与主库一致，实际情况下从库有不同的url，username，password
        slaves.add(new DriverManagerDataSource(driverClassName, url, username, password));
    }
    DataSourceFactory dsf = new MasterSlaveDataSourceFactory(master, slaves);
    Mango mango = Mango.newInstance(dsf);

多数据源
________

多数据源使用MultipleDataSourceFactory和SimpleDataSourceFactory相结合。

使用多数据源，需要在@DB注解中定义dataSource参数，下面的代码使用了多数据源:

.. code-block:: java

    HashMap<String, DataSourceFactory> factories = new HashMap<String, DataSourceFactory>();
    factories.put("dataSource1", new DriverManagerDataSource(driverClassName1, url1, username1, password1));
    factories.put("dataSource2", new DriverManagerDataSource(driverClassName2, url2, username2, password2));
    DataSourceFactory dsf = new MultipleDataSourceFactory(factories);
    Mango mango = Mango.newInstance(dsf);

在DAO中如果要使用数据源dataSource1，只需 ``@DB(dataSource = "dataSource1")`` 即可。

多数据源读写分离
________________

多数据源读写分离使用MultipleDataSourceFactory和MasterSlaveDataSourceFactory相结合。

请看下面的代码:

.. code-block:: java

    HashMap<String, DataSourceFactory> factories = new HashMap<String, DataSourceFactory>();
    for (int i = 1; i <= 2; i++) {
        DataSource master = new DriverManagerDataSource(driverClassNames, urls[i], usernames[i], passwords[i]);
        int slaveNum = 2;
        List<DataSource> slaves = new ArrayList<DataSource>();
        for (int j = 0; j < slaveNum; j++) {
            // 为了简单，参数与主库一致，实际情况下从库有不同的url，username，password
            slaves.add(new DriverManagerDataSource(driverClassName, urls[i], usernames[i], passwords[i]));
        }
        DataSourceFactory dsf = new MasterSlaveDataSourceFactory(master, slaves);
        factories.put("dataSource" + i, dsf);
    }
    DataSourceFactory dsf = new MultipleDataSourceFactory(factories);
    Mango mango = Mango.newInstance(dsf);
