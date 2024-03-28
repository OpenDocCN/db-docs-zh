> 原文：[`dev.mysql.com/doc/refman/8.0/en/socket-pluggable-authentication.html`](https://dev.mysql.com/doc/refman/8.0/en/socket-pluggable-authentication.html)

#### 8.4.1.10 套接字对等凭证可插拔认证

服务器端`auth_socket`认证插件用于认证通过 Unix 套接字文件从本地主机连接的客户端。该插件使用`SO_PEERCRED`套接字选项来获取运行客户端程序的用户信息。因此，该插件只能在支持`SO_PEERCRED`选项的系统上使用，例如 Linux。

此插件的源代码可作为一个相对简单的示例，演示如何编写一个可加载的认证插件。

以下表显示了插件和库文件名。文件必须位于由`plugin_dir`系统变量命名的目录中。

**表 8.26 套接字对等凭证认证的插件和库名称**

| 插件或文件 | 插件或文件名 |
| --- | --- |
| 服务器端插件 | `auth_socket` |
| 客户端插件 | 无，请参阅讨论 |
| 库文件 | `auth_socket.so` |

以下各节提供了特定于套接字可插拔认证的安装和使用信息：

+   安装套接字可插拔认证

+   卸载套接字可插拔认证

+   使用套接字可插拔认证

有关 MySQL 中可插拔认证的一般信息，请参阅第 8.2.17 节，“可插拔认证”。

##### 安装套接字可插拔认证

本节描述了如何安装套接字认证插件。有关安装插件的一般信息，请参阅第 7.6.1 节，“安装和卸载插件”。

要使服务器可用，插件库文件必须位于 MySQL 插件目录中（由`plugin_dir`系统变量命名的目录）。如有必要，通过在服务器启动时设置`plugin_dir`的值来配置插件目录位置。

要在服务器启动时加载插件，请使用`--plugin-load-add`选项来命名包含插件的库文件。使用这种插件加载方法，选项必须在每次服务器启动时给出。例如，将以下行放入服务器的`my.cnf`文件中：

```sql
[mysqld]
plugin-load-add=auth_socket.so
```

修改`my.cnf`后，重新启动服务器以使新设置生效。

或者，要在运行时加载插件，请使用此语句：

```sql
INSTALL PLUGIN auth_socket SONAME 'auth_socket.so';
```

`INSTALL PLUGIN`立即加载插件，并在`mysql.plugins`系统表中注册它，以使服务器在每次后续正常启动时加载它，而无需`--plugin-load-add`。

要验证插件安装情况，请检查信息模式`PLUGINS`表，或使用`SHOW PLUGINS`语句（参见第 7.6.2 节，“获取服务器插件信息”）。例如：

```sql
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS
       FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME LIKE '%socket%';
+-------------+---------------+
| PLUGIN_NAME | PLUGIN_STATUS |
+-------------+---------------+
| auth_socket | ACTIVE        |
+-------------+---------------+
```

如果插件初始化失败，请检查服务器错误日志以获取诊断消息。

要将 MySQL 帐户与套接字插件关联，请参阅使用套接字可插拔认证。

##### 卸载套接字可插拔认证

卸载套接字可插拔认证插件的方法取决于您安装插件的方式：

+   如果您在服务器启动时使用`--plugin-load-add`选项安装了插件，请在不带该选项的情况下重新启动服务器。

+   如果您在运行时使用`INSTALL PLUGIN`语句安装了插件，则在服务器重新启动时仍然安装。要卸载它，请使用`UNINSTALL PLUGIN`：

    ```sql
    UNINSTALL PLUGIN auth_socket;
    ```

##### 使用套接字可插拔认证

套接字插件检查套接字用户名（操作系统用户名）是否与客户端程序指定给服务器的 MySQL 用户名匹配。如果名称不匹配，则插件将检查套接字用户名是否与`mysql.user`系统表行的`authentication_string`列中指定的名称匹配。如果找到匹配项，则插件允许连接。`authentication_string`值可以使用`CREATE USER`或`ALTER USER`中的`IDENTIFIED ...AS`子句指定。

假设为一个名为`valerie`的操作系统用户创建了一个用于通过套接字文件从本地主机进行认证的 MySQL 帐户，该用户将通过`auth_socket`插件进行认证：

```sql
CREATE USER 'valerie'@'localhost' IDENTIFIED WITH auth_socket;
```

如果本地主机上的用户使用登录名 `stefanie` 调用 **mysql** 并使用选项 `--user=valerie` 通过套接字文件连接，服务器将使用 `auth_socket` 对客户端进行身份验证。插件确定 `--user` 选项值 (`valerie`) 与客户端用户名 (`stephanie`) 不同，因此拒绝连接。如果名为 `valerie` 的用户尝试同样的操作，插件会发现用户名和 MySQL 用户名都是 `valerie`，允许连接。但是，即使是 `valerie`，如果使用不同的协议（如 TCP/IP）进行连接，插件也会拒绝连接。

要允许 `valerie` 和 `stephanie` 操作系统用户通过使用账户的套接字文件连接访问 MySQL，可以通过两种方式实现：

+   在创建账户时分别命名这两个用户，一个在 `CREATE USER` 后面，另一个在认证字符串中：

    ```sql
    CREATE USER 'valerie'@'localhost' IDENTIFIED WITH auth_socket AS 'stephanie';
    ```

+   如果您已经使用 `CREATE USER` 为单个用户创建了账户，可以使用 `ALTER USER` 来添加第二个用户：

    ```sql
    CREATE USER 'valerie'@'localhost' IDENTIFIED WITH auth_socket;
    ALTER USER 'valerie'@'localhost' IDENTIFIED WITH auth_socket AS 'stephanie';
    ```

要访问账户，`valerie` 和 `stephanie` 在连接时都需要指定 `--user=valerie`。
