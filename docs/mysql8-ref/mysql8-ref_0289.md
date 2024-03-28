# 7.6.2 获取服务器插件信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/obtaining-plugin-information.html`](https://dev.mysql.com/doc/refman/8.0/en/obtaining-plugin-information.html)

有几种方法可以确定服务器中安装了哪些插件：

+   信息模式`PLUGINS`表为每个已加载的插件包含一行。任何具有`PLUGIN_LIBRARY`值为`NULL`的插件都是内置的，无法卸载。

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.PLUGINS\G
    *************************** 1\. row ***************************
               PLUGIN_NAME: binlog
            PLUGIN_VERSION: 1.0
             PLUGIN_STATUS: ACTIVE
               PLUGIN_TYPE: STORAGE ENGINE
       PLUGIN_TYPE_VERSION: 50158.0
            PLUGIN_LIBRARY: NULL
    PLUGIN_LIBRARY_VERSION: NULL
             PLUGIN_AUTHOR: Oracle Corporation
        PLUGIN_DESCRIPTION: This is a pseudo storage engine to represent the binlog in a transaction
            PLUGIN_LICENSE: GPL
               LOAD_OPTION: FORCE
    ...
    *************************** 10\. row ***************************
               PLUGIN_NAME: InnoDB
            PLUGIN_VERSION: 1.0
             PLUGIN_STATUS: ACTIVE
               PLUGIN_TYPE: STORAGE ENGINE
       PLUGIN_TYPE_VERSION: 50158.0
            PLUGIN_LIBRARY: ha_innodb_plugin.so
    PLUGIN_LIBRARY_VERSION: 1.0
             PLUGIN_AUTHOR: Oracle Corporation
        PLUGIN_DESCRIPTION: Supports transactions, row-level locking,
                            and foreign keys
            PLUGIN_LICENSE: GPL
               LOAD_OPTION: ON
    ...
    ```

+   `SHOW PLUGINS`语句为每个已加载的插件显示一行。任何具有`Library`值为`NULL`的插件都是内置的，无法卸载。

    ```sql
    mysql> SHOW PLUGINS\G
    *************************** 1\. row ***************************
       Name: binlog
     Status: ACTIVE
       Type: STORAGE ENGINE
    Library: NULL
    License: GPL
    ...
    *************************** 10\. row ***************************
       Name: InnoDB
     Status: ACTIVE
       Type: STORAGE ENGINE
    Library: ha_innodb_plugin.so
    License: GPL
    ...
    ```

+   `mysql.plugin`表显示已经通过`INSTALL PLUGIN`注册的插件。该表仅包含插件名称和库文件名称，因此提供的信息不如`PLUGINS`表或`SHOW PLUGINS`语句详细。
