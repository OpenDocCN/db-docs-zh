> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-function-loadable.html`](https://dev.mysql.com/doc/refman/8.0/en/create-function-loadable.html)

#### 15.7.4.1 可加载函数的 CREATE FUNCTION 语句

```sql
CREATE [AGGREGATE] FUNCTION [IF NOT EXISTS] *function_name*
    RETURNS {STRING|INTEGER|REAL|DECIMAL}
    SONAME *shared_library_name*
```

这个语句加载了名为*`function_name`*的可加载函数。（`CREATE FUNCTION`也用于创建存储函数；请参阅 Section 15.1.17, “CREATE PROCEDURE and CREATE FUNCTION Statements”.）

可加载函数是通过新函数扩展 MySQL 的一种方式，其工作方式类似于本机（内置）MySQL 函数，如`ABS()`或`CONCAT()`。请参阅添加可加载函数。

*`function_name`*是应在 SQL 语句中使用的名称来调用函数。`RETURNS`子句指示函数返回值的类型。`DECIMAL`是`RETURNS`后的合法值，但当前`DECIMAL`函数返回字符串值，应该像`STRING`函数一样编写。

`IF NOT EXISTS`可以防止出现错误，如果已经存在具有相同名称的可加载函数。它*不*会防止出现错误，如果已经存在具有相同名称的内置函数。`IF NOT EXISTS`支持从 MySQL 8.0.29 开始的`CREATE FUNCTION`语句。另请参阅函数名称解析。

如果指定了`AGGREGATE`关键字，则表示该函数是一个聚合（组）函数。聚合函数的工作方式与本机 MySQL 聚合函数（如`SUM()`或`COUNT()`）完全相同。

*`shared_library_name`*是包含实现函数代码的共享库文件的基本名称。该文件必须位于插件目录中。此目录由`plugin_dir`系统变量的值给出。有关更多信息，请参阅 Section 7.7.1, “Installing and Uninstalling Loadable Functions”.

`CREATE FUNCTION`需要对`mysql`系统模式具有`INSERT`权限，因为它向`mysql.func`系统表添加一行以注册函数。

`CREATE FUNCTION`还将函数添加到提供有关已安装可加载函数的运行时信息的性能模式`user_defined_functions`表中。请参阅 Section 29.12.21.10, “The user_defined_functions Table”。

注意

与`mysql.func`系统表类似，性能模式`user_defined_functions`表列出使用`CREATE FUNCTION`安装的可加载函数。与`mysql.func`表不同，`user_defined_functions`表还列出服务器组件或插件自动安装的可加载函数。这种差异使得`user_defined_functions`比`mysql.func`更适合检查已安装的可加载函数。

在正常启动序列期间，服务器加载在`mysql.func`表中注册的函数。如果使用`--skip-grant-tables`选项启动服务器，则表中注册的函数不会加载且不可用。

注意

要升级与可加载函数关联的共享库，请发出`DROP FUNCTION`语句，升级共享库，然后发出`CREATE FUNCTION`语句。如果您先升级共享库，然后使用`DROP FUNCTION`，服务器可能会意外关闭。
