> 原文：[`dev.mysql.com/doc/refman/8.0/en/clone-plugin-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/clone-plugin-installation.html)

#### 7.6.7.1 安装克隆插件

本节描述了如何安装和配置克隆插件。对于远程克隆操作，克隆插件必须安装在捐赠者和接收者 MySQL 服务器实例上。

有关安装或卸载插件的一般信息，请参见 Section 7.6.1, “Installing and Uninstalling Plugins”。

要使服务器可用，插件库文件必须位于 MySQL 插件目录中（由 `plugin_dir` 系统变量命名的目录）。如有必要，在服务器启动时设置 `plugin_dir` 的值，告诉服务器插件目录的位置。

插件库文件基本名称为 `mysql_clone.so`。文件名后缀因平台而异（例如，Unix 和类 Unix 系统使用 `.so`，Windows 使用 `.dll`）。

要在服务器启动时加载插件，请使用 `--plugin-load-add` 选项命名包含插件的库文件。使用此插件加载方法，每次服务器启动时都必须提供该选项。例如，将以下行放入您的 `my.cnf` 文件中，根据需要调整插件库文件名扩展名以适应您的平台。（插件库文件名扩展名取决于您的平台。Unix 和类 Unix 系统通常使用 `.so`，Windows 使用 `.dll`。）

```sql
[mysqld]
plugin-load-add=mysql_clone.so
```

修改 `my.cnf` 后，重新启动服务器以使新设置生效。

注意

`--plugin-load-add` 选项在升级过程中重新启动服务器时无法用于加载克隆插件。例如，在从先前的 MySQL 版本升级到 MySQL 8.0 后，尝试使用 `plugin-load-add=mysql_clone.so` 重新启动服务器会导致以下错误：[ERROR] [MY-013238] [Server] Error installing plugin 'clone': Cannot install during upgrade。解决方法是在尝试使用 `plugin-load-add=mysql_clone.so` 启动服务器之前先升级服务器。

或者，要在运行时加载插件，请使用以下语句，根据需要调整平台的 `.so` 后缀：

```sql
INSTALL PLUGIN clone SONAME 'mysql_clone.so';
```

`INSTALL PLUGIN` 加载插件，并在 `mysql.plugins` 系统表中注册，以使插件在每次正常服务器启动时自动加载，无需 `--plugin-load-add`。

要验证插件安装情况，请检查信息模式`PLUGINS`表，或者使用`SHOW PLUGINS`语句（参见 Section 7.6.2, “Obtaining Server Plugin Information”）。例如：

```sql
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS
       FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME = 'clone';
+------------------------+---------------+
| PLUGIN_NAME            | PLUGIN_STATUS |
+------------------------+---------------+
| clone                  | ACTIVE        |
+------------------------+---------------+
```

如果插件初始化失败，请检查服务器错误日志以获取克隆或插件相关的诊断消息。

如果插件之前已经使用`INSTALL PLUGIN`注册过，或者使用`--plugin-load-add`加载过，您可以在服务器启动时使用`--clone`选项来控制插件的激活状态。例如，要在启动时加载插件并防止在运行时被移除，可以使用以下选项：

```sql
[mysqld]
plugin-load-add=mysql_clone.so
clone=FORCE_PLUS_PERMANENT
```

如果您希望防止服务器在没有克隆插件的情况下运行，请使用`--clone`并设置值为`FORCE`或`FORCE_PLUS_PERMANENT`，以强制服务器启动失败，如果插件初始化失败。

有关插件激活状态的更多信息，请参阅 Controlling Plugin Activation State。
