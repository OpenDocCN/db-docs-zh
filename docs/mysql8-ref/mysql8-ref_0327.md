# 7.7.1 安装和卸载可加载函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/function-loading.html`](https://dev.mysql.com/doc/refman/8.0/en/function-loading.html)

如其名称所示，可加载函数必须在服务器中加载后才能使用。MySQL 支持在服务器启动期间自动加载函数以及随后的手动加载。

在加载可加载函数时，有关其的信息可如第 7.7.2 节“获取有关可加载函数的信息”所述获得。

+   安装可加载函数

+   卸载可加载函数

+   重新安装或升级可加载函数

#### 安装可加载函数

要手动加载可加载函数，请使用`CREATE FUNCTION`语句。例如：

```sql
CREATE FUNCTION metaphon
  RETURNS STRING
  SONAME 'udf_example.so';
```

文件基本名称取决于您的平台。Unix 和类 Unix 系统的常见后缀是`.so`，Windows 的是`.dll`。

`CREATE FUNCTION`具有以下效果：

+   它将函数加载到服务器中，使其立即可用。

+   它在`mysql.func`系统表中注册函数，使其在服务器重新启动时保持持久性。因此，`CREATE FUNCTION`需要对`mysql`系统数据库的`INSERT`权限。

+   它将函数添加到性能模式`user_defined_functions`表中，该表提供有关已安装可加载函数的运行时信息。参见第 7.7.2 节“获取有关可加载函数的信息”。

可加载函数的自动加载发生在正常服务器启动序列期间：

+   在`mysql.func`表中注册的函数已安装。

+   在启动时安装的组件或插件可能会自动安装相关函数。

+   自动函数安装将函数添加到性能模式`user_defined_functions`表中，该表提供有关已安装函数的运行时信息。

如果服务器使用`--skip-grant-tables`选项启动，则`mysql.func`表中注册的函数不会被加载，也无法使用。这不适用于组件或插件自动安装的函数。

#### 卸载可加载函数

要删除可加载函数，请使用`DROP FUNCTION`语句。例如：

```sql
DROP FUNCTION metaphon;
```

`DROP FUNCTION`有以下影响：

+   它会卸载函数使其不可用。

+   它会从`mysql.func`系统表中删除函数。因此，`DROP FUNCTION`需要对`mysql`系统数据库的`DELETE`权限。由于函数不再在`mysql.func`表中注册，服务器在后续重新启动时不会加载该函数。

+   它会从性能模式`user_defined_functions`表中删除提供有关已安装可加载函数的运行时信息的函数。

`DROP FUNCTION`不能用于删除由组件或插件自动安装而不是使用`CREATE FUNCTION`安装的可加载函数。这样的函数在卸载安装它的组件或插件时也会自动删除。

#### 重新安装或升级可加载函数

要重新安装或升级与可加载函数关联的共享库，请发出`DROP FUNCTION`语句，升级共享库，然后发出`CREATE FUNCTION`语句。如果先升级共享库，然后使用`DROP FUNCTION`，服务器可能会意外关闭。
