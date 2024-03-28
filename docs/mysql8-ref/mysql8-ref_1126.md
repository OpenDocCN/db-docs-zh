> 原文：[`dev.mysql.com/doc/refman/8.0/en/reset-persist.html`](https://dev.mysql.com/doc/refman/8.0/en/reset-persist.html)

#### 15.7.8.7 RESET PERSIST 语句

```sql
RESET PERSIST [[IF EXISTS] *system_var_name*]
```

`RESET PERSIST`从数据目录中的`mysqld-auto.cnf`选项文件中删除持久化的全局系统变量设置。移除持久化系统变量会导致该变量不再从`mysqld-auto.cnf`在服务器启动时初始化。有关持久化系统变量和`mysqld-auto.cnf`文件的更多信息，请参见第 7.1.9.3 节，“持久化系统变量”。

在 MySQL 8.0.32 之前，此语句不适用于变量名包含点字符（`.`）的变量，例如`MyISAM`多键缓存变量和组件注册的变量。（Bug #33417357）

执行`RESET PERSIST`所需的权限取决于要移除的系统变量类型：

+   对于动态系统变量，此语句需要`SYSTEM_VARIABLES_ADMIN`权限（或已弃用的`SUPER`权限）。

+   对于只读系统变量，此语句需要`SYSTEM_VARIABLES_ADMIN`和`PERSIST_RO_VARIABLES_ADMIN`权限。

参见第 7.1.9.1 节，“系统变量权限”。

根据变量名和`IF EXISTS`子句是否存在，`RESET PERSIST`语句有以下形式：

+   要从`mysqld-auto.cnf`中移除所有持久化变量，请使用`RESET PERSIST`而不命名任何系统变量：

    ```sql
    RESET PERSIST;
    ```

    你必须拥有权限来移除`mysqld-auto.cnf`中包含的动态和只读系统变量，如果这两种变量都存在。

+   要从`mysqld-auto.cnf`中移除特定的持久化变量，请在语句中命名它：

    ```sql
    RESET PERSIST *system_var_name*;
    ```

    这包括插件系统变量，即使插件当前未安装。如果变量不存在于文件中，则会发生错误。

+   要从`mysqld-auto.cnf`中移除特定的持久化变量，但是如果该变量不存在于文件中，则产生警告而不是错误，请在先前的语法中添加一个`IF EXISTS`子句：

    ```sql
    RESET PERSIST IF EXISTS *system_var_name*;
    ```

`RESET PERSIST`不受`persisted_globals_load`系统变量值的影响。

`RESET PERSIST`会影响性能模式`persisted_variables`表的内容，因为表内容对应于`mysqld-auto.cnf`文件的内容。另一方面，因为`RESET PERSIST`不会改变变量值，所以在服务器重新启动之前，它不会对性能模式`variables_info`表的内容产生影响。

关于清除其他服务器操作状态的`RESET`语句变体的信息，请参阅第 15.7.8.6 节，“RESET Statement”。
