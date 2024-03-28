# 17.20.5 InnoDB memcached 插件的安全注意事项

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-memcached-security.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-memcached-security.html)

注意

在将`daemon_memcached`插件部署到生产服务器之前，请参考本节，甚至在包含敏感数据的 MySQL 实例上的测试服务器上。

因为**memcached**默认不使用身份验证机制，而可选的 SASL 身份验证不如传统的 DBMS 安全措施强大，因此只在使用`daemon_memcached`插件的 MySQL 实例中保留非敏感数据，并将使用此配置的任何服务器与潜在入侵者隔离开。不要允许来自 Internet 的**memcached**访问这些服务器；只允许来自受防火墙保护的内部网络的访问，最好是来自可以限制成员资格的子网。

#### 通过使用 SASL 对 memcached 进行密码保护

SASL 支持提供了保护 MySQL 数据库免受通过**memcached**客户端的未经身份验证访问的能力。本节介绍如何启用具有`daemon_memcached`插件的 SASL。这些步骤几乎与为传统**memcached**服务器启用 SASL 执行的步骤相同。

SASL 代表“简单身份验证和安全层”，这是一种为基于连接的协议添加身份验证支持的标准。**memcached**在 1.4.3 版本中添加了 SASL 支持。

SASL 身份验证仅受二进制协议支持。

**memcached**客户端只能访问在`innodb_memcache.containers`表中注册的`InnoDB`表。尽管 DBA 可以对这些表施加访问限制，但无法控制通过**memcached**应用程序的访问。因此，提供了 SASL 支持以控制与`daemon_memcached`插件关联的`InnoDB`表的访问。

以下部分显示了如何构建、启用和测试启用 SASL 的`daemon_memcached`插件。

#### 构建和启用具有 InnoDB memcached 插件的 SASL

默认情况下，MySQL 发布包中不包含启用 SASL 的`daemon_memcached`插件，因为启用 SASL 的`daemon_memcached`插件需要使用 SASL 库构建**memcached**。要启用 SASL 支持，请下载 MySQL 源代码，并在下载 SASL 库后重新构建`daemon_memcached`插件：

1.  安装 SASL 开发和实用程序库。例如，在 Ubuntu 上，使用**apt-get**获取这些库：

    ```sql
    sudo apt-get -f install libsasl2-2 sasl2-bin libsasl2-2 libsasl2-dev libsasl2-modules
    ```

1.  通过将`ENABLE_MEMCACHED_SASL=1`添加到**cmake**选项中，可以构建具有 SASL 功能的`daemon_memcached`插件共享库。**memcached**还提供*简单的明文密码支持*，这有助于测试。要启用简单的明文密码支持，请指定`ENABLE_MEMCACHED_SASL_PWDB=1` **cmake**选项。

    总之，添加以下三个**cmake**选项：

    ```sql
    cmake ... -DWITH_INNODB_MEMCACHED=1 -DENABLE_MEMCACHED_SASL=1 -DENABLE_MEMCACHED_SASL_PWDB=1
    ```

1.  安装`daemon_memcached`插件，如第 17.20.3 节，“设置 InnoDB memcached 插件”所述。

1.  配置一个用户名和密码文件。（此示例使用**memcached**简单明文密码支持。）

    1.  在文件中创建一个名为`testname`的用户，并将密码定义为`testpasswd`：

        ```sql
        echo "testname:testpasswd:::::::" >/home/jy/memcached-sasl-db
        ```

    1.  配置`MEMCACHED_SASL_PWDB`环境变量，通知`memcached`用户名称和密码文件的位置：

        ```sql
        export MEMCACHED_SASL_PWDB=/home/jy/memcached-sasl-db
        ```

    1.  通知`memcached`使用明文密码：

        ```sql
        echo "mech_list: plain" > /home/jy/work2/msasl/clients/memcached.conf
        export SASL_CONF_PATH=/home/jy/work2/msasl/clients
        ```

1.  通过在`daemon_memcached_option`配置参数中编码**memcached** `-S`选项，通过重新启动 MySQL 服务器启用 SASL：

    ```sql
    mysqld ... --daemon_memcached_option="-S"
    ```

1.  为了测试设置，使用支持 SASL 的客户端，如[SASL-enabled libmemcached](https://code.launchpad.net/~trond-norbye/libmemcached/sasl)。

    ```sql
    memcp --servers=localhost:11211 --binary  --username=testname
      --password=*password* myfile.txt

    memcat --servers=localhost:11211 --binary --username=testname
      --password=*password* myfile.txt
    ```

    如果指定了不正确的用户名或密码，则操作将被拒绝，并显示`memcache error AUTHENTICATION FAILURE`消息。在这种情况下，请检查`memcached-sasl-db`文件中设置的明文密码，以验证您提供的凭据是否正确。

还有其他测试使用**memcached**进行 SASL 身份验证的方法，但上述描述的方法是最直接的。
