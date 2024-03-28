# 7.5.1 安装和卸载组件

> 译文：[`dev.mysql.com/doc/refman/8.0/en/component-loading.html`](https://dev.mysql.com/doc/refman/8.0/en/component-loading.html)

组件必须在服务器中加载后才能使用。MySQL 支持在运行时手动加载组件和在服务器启动期间自动加载组件。

在加载组件时，可以按照 Section 7.5.2, “Obtaining Component Information”中描述的方式获取有关组件的信息。

`INSTALL COMPONENT`和`UNINSTALL COMPONENT` SQL 语句可实现组件的加载和卸载。例如：

```sql
INSTALL COMPONENT 'file://component_validate_password';
UNINSTALL COMPONENT 'file://component_validate_password';
```

加载程序服务处理组件的加载和卸载，并在`mysql.component`系统表中注册已加载的组件。

组件操作的 SQL 语句会影响服务器操作和`mysql.component`系统表，具体如下：

+   `INSTALL COMPONENT`会将组件加载到服务器中。组件会立即生效。加载程序服务还会在`mysql.component`系统表中注册已加载的组件。对于后续的服务器重新启动，加载程序服务会在启动序列中加载`mysql.component`中列出的任何组件。即使服务器使用`--skip-grant-tables`选项启动，也会发生这种情况。可选的`SET`子句允许在安装组件时设置组件系统变量的值。

+   `UNINSTALL COMPONENT`会停用组件并从服务器中卸载它们。加载程序服务还会从`mysql.component`系统表中注销这些组件，以便服务器在后续重新启动时不再在启动序列中加载它们。

与服务器插件的对应`INSTALL PLUGIN`语句相比，组件的`INSTALL COMPONENT`语句具有显著优势，不需要知道任何特定于平台的文件名后缀来命名组件。这意味着给定的`INSTALL COMPONENT`语句可以在各个平台上统一执行。

安装组件时，可能还会自动安装相关的可加载函数。如果是这样，卸载组件时也会自动卸载这些函数。
