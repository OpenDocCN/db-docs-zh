> 原文：[`dev.mysql.com/doc/refman/8.0/en/administrative-connection-interface.html`](https://dev.mysql.com/doc/refman/8.0/en/administrative-connection-interface.html)

#### 7.1.12.2 管理连接管理

如在连接量管理中提到的，即使在用于普通连接的接口上已经建立了`max_connections`连接，MySQL 服务器也允许具有`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限）的用户进行单个管理连接，以满足执行管理操作的需要。

此外，从 MySQL 8.0.14 开始，服务器允许为管理连接专用一个 TCP/IP 端口，如下节所述。

+   管理界面特性

+   管理界面支持加密连接

##### 管理界面特性

管理连接界面具有以下特性：

+   服务器仅在启动时将`admin_address`系统变量设置为指示其 IP 地址时才启用接口。如果未设置`admin_address`，服务器将不维护任何管理界面。

+   `admin_port`系统变量指定接口 TCP/IP 端口号（默认为 33062）。

+   管理连接数量没有限制，但只允许具有`SERVICE_CONNECTION_ADMIN`权限的用户连接。

+   `create_admin_listener_thread`系统变量允许 DBA 在启动时选择管理界面是否有自己独立的线程。默认为`OFF`；即，用于主接口上普通连接的管理线程也处理管理界面的连接。

在服务器的`my.cnf`文件中，这些行启用了回环接口上的管理界面，并配置其使用端口号 33064（即与默认端口不同）：

```sql
[mysqld]
admin_address=127.0.0.1
admin_port=33064
```

MySQL 客户端程序通过指定适当的连接参数连接到主界面或管理界面。如果运行在本地主机上的服务器使用默认的 TCP/IP 端口号 3306 和 33062 用于主界面和管理界面，这些命令将连接到这些界面：

```sql
mysql --protocol=TCP --port=3306
mysql --protocol=TCP --port=33062
```

##### 管理界面支持加密连接

在 MySQL 8.0.21 之前，管理界面支持使用适用于主界面的连接加密配置进行加密连接。从 MySQL 8.0.21 开始，管理界面有自己的加密连接配置参数。这些参数对应于主界面参数，但允许独立配置管理界面的加密连接：

+   `admin_tls_*`xxx`*` 和 `admin_ssl_*`xxx`*` 系统变量类似于 `tls_*`xxx`*` 和 `ssl_*`xxx`*` 系统变量，但它们配置管理界面而不是主界面的 TLS 上下文。

+   `--admin-ssl` 选项类似于 `--ssl` 选项，但它启用或禁用管理界面而不是主界面上的加密连接支持。

    因为默认情况下启用了对加密连接的支持，通常不需要指定 `--admin-ssl`。从 MySQL 8.0.26 开始，`--admin-ssl` 已被弃用，并可能在未来的 MySQL 版本中移除。

有关配置连接加密支持的一般信息，请参阅 第 8.3.1 节，“配置 MySQL 使用加密连接” 和 第 8.3.2 节，“加密连接 TLS 协议和密码”。该讨论是针对主连接界面编写的，但管理连接界面的参数名称类似。结合该讨论和以下提供的信息，这些信息提供了管理界面特定的信息。

管理界面的 TLS 配置遵循以下规则：

+   如果启用了 `--admin-ssl`（默认情况下），管理界面支持加密连接。对于界面上的连接，适用的 TLS 上下文取决于是否配置了任何非默认的管理 TLS 参数：

    +   如果所有管理 TLS 参数都采用默认值，则管理界面将使用与主界面相同的 TLS 上下文。

    +   如果任何管理 TLS 参数具有非默认值，则管理界面使用由其自身参数定义的 TLS 上下文。（如果任何`admin_tls_*`xxx`*`或`admin_ssl_*`xxx`*`系统变量设置为与其默认值不同的值，则是这种情况。）如果无法从这些参数创建有效的 TLS 上下文，则管理界面将退回到主界面 TLS 上下文。

+   如果`--admin-ssl`被禁用（例如，通过指定`--admin-ssl=OFF`，则对管理界面的加密连接被禁用。即使管理 TLS 参数具有非默认值，因为禁用`--admin-ssl`优先。

    也可以在不指定`--admin-ssl`的否定形式的情况下禁用管理界面上的加密连接。将`admin_tls_version`系统变量设置为空值表示不支持任何 TLS 版本。例如，服务器`my.cnf`文件中的这些行禁用了管理界面上的加密连接：

    ```sql
    [mysqld]
    admin_tls_version=''
    ```

例子：

+   这个在服务器`my.cnf`文件中的配置启用了管理界面，但没有设置任何特定于该界面的 TLS 参数：

    ```sql
    [mysqld]
    admin_address=127.0.0.1
    ```

    结果，管理界面支持加密连接（因为当管理界面启用时，默认支持加密），并使用主界面 TLS 上下文。当客户端连接到管理界面时，他们应该使用与主界面上普通连接相同的证书和密钥文件。例如（在一行上输入命令）：

    ```sql
    mysql --protocol=TCP --port=33062
          --ssl-ca=ca.pem
          --ssl-cert=client-cert.pem
          --ssl-key=client-key.pem
    ```

+   这个服务器配置启用了管理界面，并设置了特定于该界面的 TLS 证书和密钥文件参数：

    ```sql
    [mysqld]
    admin_address=127.0.0.1
    admin_ssl_ca=admin-ca.pem
    admin_ssl_cert=admin-server-cert.pem
    admin_ssl_key=admin-server-key.pem
    ```

    结果，管理界面支持使用其自己的 TLS 上下文进行加密连接。当客户端连接到管理界面时，他们应该使用特定于该界面的证书和密钥文件。例如（在一行上输入命令）：

    ```sql
    mysql --protocol=TCP --port=33062
          --ssl-ca=admin-ca.pem
          --ssl-cert=admin-client-cert.pem
          --ssl-key=admin-client-key.pem
    ```
