> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-user-defined-functions-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-user-defined-functions-table.html)

#### 29.12.21.10 user_defined_functions 表

`user_defined_functions` 表包含每个由组件或插件自动注册的可加载函数或由 `CREATE FUNCTION` 语句手动注册的函数的行。有关添加或删除表行的操作信息，请参阅 Section 7.7.1, “安装和卸载可加载函数”。

注意

`user_defined_functions` 表的名称源自其创立时用于现在称为可加载函数（即用户定义函数或 UDF）的函数类型的术语。

`user_defined_functions` 表具有以下列：

+   `UDF_NAME`

    SQL 语句中引用的函数名称。如果函数是由 `CREATE FUNCTION` 语句注册并正在卸载过程中，则值为 `NULL`。

+   `UDF_RETURN_TYPE`

    函数返回值类型。值为 `int`、`decimal`、`real`、`char` 或 `row` 中的一个。

+   `UDF_TYPE`

    函数类型。值为 `function`（标量）或 `aggregate` 中的一个。

+   `UDF_LIBRARY`

    包含可执行函数代码的库文件的名称。该文件位于由 `plugin_dir` 系统变量命名的目录中。如果函数是由组件或插件而不是由 `CREATE FUNCTION` 语句注册，则值为 `NULL`。

+   `UDF_USAGE_COUNT`

    当前函数使用计数。用于判断当前是否有语句访问该函数。

`user_defined_functions` 表具有以下索引：

+   主键为 (`UDF_NAME`)

不允许对 `user_defined_functions` 表使用 `TRUNCATE TABLE`。

`mysql.func` 系统表还列出已安装的可加载函数，但仅列出使用 `CREATE FUNCTION` 安装的函数。`user_defined_functions` 表列出使用 `CREATE FUNCTION` 安装的可加载函数，以及由组件或插件自动安装的可加载函数。这种差异使得 `user_defined_functions` 更适合用于检查已安装的可加载函数。
