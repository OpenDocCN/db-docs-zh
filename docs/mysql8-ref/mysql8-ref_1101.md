> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-plugins.html`](https://dev.mysql.com/doc/refman/8.0/en/show-plugins.html)

#### 15.7.7.25 显示插件语句

```sql
SHOW PLUGINS
```

`显示插件`显示有关服务器插件的信息。

`显示插件`输出示例：

```sql
mysql> SHOW PLUGINS\G
*************************** 1\. row ***************************
   Name: binlog
 Status: ACTIVE
   Type: STORAGE ENGINE
Library: NULL
License: GPL
*************************** 2\. row ***************************
   Name: CSV
 Status: ACTIVE
   Type: STORAGE ENGINE
Library: NULL
License: GPL
*************************** 3\. row ***************************
   Name: MEMORY
 Status: ACTIVE
   Type: STORAGE ENGINE
Library: NULL
License: GPL
*************************** 4\. row ***************************
   Name: MyISAM
 Status: ACTIVE
   Type: STORAGE ENGINE
Library: NULL
License: GPL
...
```

`显示插件`输出具有以下列：

+   `名称`

    在`安装插件`和`卸载插件`等语句中用于引用插件的名称。

+   `状态`

    插件状态，其中之一为`活动`，`非活动`，`已禁用`，`正在删除`或`已删除`。

+   `类型`

    插件的类型，如`存储引擎`，`INFORMATION_SCHEMA`或`认证`。

+   `库`

    插件共享库文件的名称。这是在`安装插件`和`卸载插件`等语句中用于引用插件文件的名称。此文件位于由`plugin_dir`系统变量命名的目录中。如果库名称为`NULL`，则插件已编译并且无法使用`卸载插件`卸载。

+   `许可证`

    插件的许可证（例如，`GPL`）。

对于使用`安装插件`安装的插件，`名称`和`库`值也在`mysql.plugin`系统表中注册。

有关形成`显示插件`显示的信息基础的插件数据结构的信息，请参阅 MySQL 插件 API。

插件信息也可以从`INFORMATION_SCHEMA`的`.PLUGINS`表中获取。请参阅第 28.3.22 节，“INFORMATION_SCHEMA PLUGINS 表”。
