> 原文：[`dev.mysql.com/doc/refman/8.0/en/drop-function-loadable.html`](https://dev.mysql.com/doc/refman/8.0/en/drop-function-loadable.html)

#### 15.7.4.2 DROP FUNCTION Statement for Loadable Functions

```sql
DROP FUNCTION [IF EXISTS] *function_name*
```

此语句删除名为 *`function_name`* 的可加载函数。(`DROP FUNCTION` 也用于删除存储函数；请参阅 Section 15.1.29, “DROP PROCEDURE and DROP FUNCTION Statements”.)

`DROP FUNCTION` 是 `CREATE FUNCTION` 的补充。它需要 `mysql` 系统模式的 `DELETE` 权限，因为它会从注册函数的 `mysql.func` 系统表中删除行。

`DROP FUNCTION` 还会从性能模式 `user_defined_functions` 表中删除提供有关已安装可加载函数的运行时信息的函数。请参阅 Section 29.12.21.10, “The user_defined_functions Table”.

在正常启动序列期间，服务器会加载在 `mysql.func` 表中注册的函数。因为 `DROP FUNCTION` 删除了被删除函数的 `mysql.func` 行，所以服务器在后续重新启动时不会加载该函数。

`DROP FUNCTION` 不能用于删除由组件或插件自动安装而不是使用 `CREATE FUNCTION` 安装的可加载函数。这样的函数在卸载安装它的组件或插件时也会自动删除。

注意

要升级与可加载函数关联的共享库，请发出 `DROP FUNCTION` 语句，升级共享库，然后发出 `CREATE FUNCTION` 语句。如果先升级共享库，然后使用 `DROP FUNCTION`，服务器可能会意外关闭。
