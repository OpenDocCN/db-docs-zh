> 原文：[`dev.mysql.com/doc/refman/8.0/en/test-pluggable-authentication.html`](https://dev.mysql.com/doc/refman/8.0/en/test-pluggable-authentication.html)

#### 8.4.1.12 测试可插拔认证

MySQL 包含一个测试插件，用于检查帐户凭据并将成功或失败记录到服务器错误日志中。这是一个可加载的插件（非内置），必须在使用之前安装。

测试插件源代码与服务器源代码分开，不同于内置的本机插件，因此可以作为一个相对简单的示例来演示如何编写可加载的认证插件。

注意

此插件旨在用于测试和开发目的，不适用于生产环境或暴露在公共网络上的服务器。

以下表格显示了插件和库文件的名称。文件名后缀可能在您的系统上有所不同。文件必须位于由`plugin_dir`系统变量命名的目录中。

**表 8.28 测试认证的插件和库名称**

| 插件或文件 | 插件或文件名 |
| --- | --- |
| 服务器端插件 | `test_plugin_server` |
| 客户端插件 | `auth_test_plugin` |
| 库文件 | `auth_test_plugin.so` |

以下各节提供了特定于测试可插拔认证的安装和使用信息：

+   安装测试可插拔认证

+   卸载测试可插拔认证

+   使用测试可插拔认证

有关 MySQL 中可插拔认证的一般信息，请参见第 8.2.17 节，“可插拔认证”。

##### 安装测试可插拔认证

本节描述了如何安装服务器端测试认证插件。有关安装插件的一般信息，请参见第 7.6.1 节，“安装和卸载插件”。

要被服务器使用，插件库文件必须位于 MySQL 插件目录中（由`plugin_dir`系统变量命名的目录）。如有必要，在服务器启动时通过设置`plugin_dir`的值来配置插件目录位置。

要在服务器启动时加载插件，请使用`--plugin-load-add`选项命名包含插件的库文件。使用此插件加载方法，每次服务器启动时都必须提供该选项。例如，将这些行放入服务器的`my.cnf`文件中，根据需要调整`.so`后缀以适应您的平台：

```sql
[mysqld]
plugin-load-add=auth_test_plugin.so
```

修改`my.cnf`后，重新启动服务器以使新设置生效。

或者，要在运行时加载插件，请使用此语句，根据需要调整`.so`后缀以适应您的平台：

```sql
INSTALL PLUGIN test_plugin_server SONAME 'auth_test_plugin.so';
```

`INSTALL PLUGIN`立即加载插件，并在`mysql.plugins`系统表中注册它，以使服务器在每次后续正常启动时加载它，而无需`--plugin-load-add`。

要验证插件安装，请检查 Information Schema `PLUGINS`表或使用`SHOW PLUGINS`语句（参见 Section 7.6.2, “Obtaining Server Plugin Information”）。例如：

```sql
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS
       FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME LIKE '%test_plugin%';
+--------------------+---------------+
| PLUGIN_NAME        | PLUGIN_STATUS |
+--------------------+---------------+
| test_plugin_server | ACTIVE        |
+--------------------+---------------+
```

如果插件初始化失败，请检查服务器错误日志以获取诊断消息。

要将 MySQL 帐户与测试插件关联，请参阅 Using Test Pluggable Authentication。

##### 卸载测试可插拔认证

用于卸载测试认证插件的方法取决于您安装它的方式：

+   如果您在服务器启动时使用`--plugin-load-add`选项安装了插件，请在不带该选项的情况下重新启动服务器。

+   如果您使用`INSTALL PLUGIN`语句在运行时安装插件，则它将在服务器重新启动时保持安装状态。要卸载它，请使用`UNINSTALL PLUGIN`：

    ```sql
    UNINSTALL PLUGIN test_plugin_server;
    ```

##### 使用测试可插拔认证

要使用测试认证插件，请创建一个帐户并在`IDENTIFIED WITH`子句中命名该插件：

```sql
CREATE USER 'testuser'@'localhost'
IDENTIFIED WITH test_plugin_server
BY '*testpassword*';
```

测试认证插件还需要创建代理用户，如下所示：

```sql
CREATE USER testpassword@localhost;
GRANT PROXY ON testpassword@localhost TO testuser@localhost;
```

然后在连接到服务器时为该帐户提供`--user`和`--password`选项。例如：

```sql
$> mysql --user=testuser --password
Enter password: *testpassword*
```

该插件从客户端接收的密码并将其与存储在`mysql.user`系统表中帐户行的`authentication_string`列中的值进行比较。如果两个值匹配，插件将`authentication_string`值作为新的有效用户 ID 返回。

你可以查看服务器错误日志，查看是否有关于身份验证成功的消息（注意密码被报告为“user”）：

```sql
[Note] Plugin test_plugin_server reported:
'successfully authenticated user *testpassword*'
```
