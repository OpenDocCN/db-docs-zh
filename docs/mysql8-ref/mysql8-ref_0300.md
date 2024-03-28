> 原文：[`dev.mysql.com/doc/refman/8.0/en/ddl-rewriter-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/ddl-rewriter-installation.html)

#### 7.6.5.1 安装或卸载 ddl_rewriter

本节描述了如何安装或卸载`ddl_rewriter`插件。有关安装插件的一般信息，请参见 Section 7.6.1，“安装和卸载插件”。

注意

如果安装了`ddl_rewriter`插件，即使禁用了，也会涉及一些最小的开销。为避免这种开销，只在打算使用它的期间安装`ddl_rewriter`。

主要用例是修改从转储文件中恢复的语句，因此典型的使用模式是：1）安装插件；2）恢复转储文件或文件；3）卸载插件。

要被服务器使用，插件库文件必须位于 MySQL 插件目录中（由`plugin_dir`系统变量命名的目录）。如有必要，在服务器启动时通过设置`plugin_dir`的值来配置插件目录位置。

插件库文件基本名称为`ddl_rewriter`。文件名后缀因平台而异（例如，Unix 和类 Unix 系统为`.so`，Windows 为`.dll`）。

要安装`ddl_rewriter`插件，请使用`INSTALL PLUGIN`语句，根据需要调整您平台的`.so`后缀：

```sql
INSTALL PLUGIN ddl_rewriter SONAME 'ddl_rewriter.so';
```

要验证插件安装，请检查信息模式`PLUGINS`表或使用`SHOW PLUGINS`语句（参见 Section 7.6.2，“获取服务器插件信息”）。例如：

```sql
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS, PLUGIN_TYPE
       FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME LIKE 'ddl%';
+--------------+---------------+-------------+
| PLUGIN_NAME  | PLUGIN_STATUS | PLUGIN_TYPE |
+--------------+---------------+-------------+
| ddl_rewriter | ACTIVE        | AUDIT       |
+--------------+---------------+-------------+
```

如前述结果所示，`ddl_rewriter`被实现为审计插件。

如果插件初始化失败，请检查服务器错误日志以获取诊断消息。

一旦按照上述描述安装，`ddl_rewriter`将保持安装状态，直到卸载。要删除它，请使用`UNINSTALL PLUGIN`：

```sql
UNINSTALL PLUGIN ddl_rewriter;
```

如果安装了`ddl_rewriter`，您可以使用`--ddl-rewriter`选项来控制后续服务器启动时`ddl_rewriter`插件的激活。例如，要防止插件在运行时启用，请使用此选项：

```sql
[mysqld]
ddl-rewriter=OFF
```
