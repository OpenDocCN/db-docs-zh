# 7.1.14 网络命名空间支持

> 原文：[`dev.mysql.com/doc/refman/8.0/en/network-namespace-support.html`](https://dev.mysql.com/doc/refman/8.0/en/network-namespace-support.html)

网络命名空间是主机系统网络堆栈的逻辑副本。网络命名空间适用于设置容器或虚拟环境。每个命名空间都有自己的 IP 地址、网络接口、路由表等。默认或全局命名空间是主机系统物理接口存在的命名空间。

命名空间特定的地址空间可能会导致 MySQL 连接跨越命名空间时出现问题。例如，在容器或虚拟网络中运行的 MySQL 实例的网络地址空间可能与主机机器的地址空间不同。这可能会导致诸如来自一个命名空间中的地址的客户端连接似乎来自不同地址的 MySQL 服务器，即使客户端和服务器在同一台机器上运行。假设两个进程都在具有 IP 地址`203.0.113.10`的主机上运行，但使用不同的命名空间。连接可能产生如下结果：

```sql
$> mysql --user=admin --host=203.0.113.10 --protocol=tcp

mysql> SELECT USER();
+--------------------+
| USER()             |
+--------------------+
| admin@198.51.100.2 |
+--------------------+
```

在这种情况下，预期的`USER()`值为`admin@203.0.113.10`。如果连接发起的地址与其表面不符，这种行为可能会使正确分配帐户权限变得困难。

为解决此问题，MySQL 允许指定用于 TCP/IP 连接的网络命名空间，以便连接的两端使用约定的共同地址空间。

MySQL 8.0.22 及更高版本支持实现网络命名空间的平台上的网络命名空间。MySQL 内的支持适用于：

+   MySQL 服务器，**mysqld**。

+   X 插件。

+   **mysql**客户端和**mysqlxtest**测试套件客户端。（不支持其他客户端。它们必须从要连接的服务器的网络命名空间内调用。）

+   常规复制。

+   组复制，仅在使用 MySQL 通信堆栈建立组通信连接时（从 MySQL 8.0.27 开始）。

以下部分描述了如何在 MySQL 中使用网络命名空间：

+   主机系统先决条件

+   MySQL 配置

+   网络命名空间监控

#### 主机系统先决条件

在 MySQL 中使用网络命名空间支持之前，必须满足这些主机系统先决条件：

+   主机操作系统必须支持网络命名空间。（例如，Linux。）

+   MySQL 要使用的任何网络命名空间必须首先在主机系统上创建。

+   主机名解析必须由系统管理员配置以支持网络命名空间。

    注意

    已知的限制是，在 MySQL 中，主机名解析对于在特定网络命名空间主机文件中指定的名称不起作用。例如，如果在`red`命名空间中的主机名的地址在`/etc/netns/red/hosts`文件中指定，那么在服务器和客户端端都无法绑定到该名称。解决方法是使用 IP 地址而不是主机名。

+   系统管理员必须为支持网络命名空间的 MySQL 二进制文件启用`CAP_SYS_ADMIN`操作系统特权（**mysqld**，**mysql**，**mysqlxtest**）。

    重要提示

    启用`CAP_SYS_ADMIN`是一个安全敏感的操作，因为它使进程能够执行其他特权操作，除了设置命名空间。有关其影响的描述，请参见[`man7.org/linux/man-pages/man7/capabilities.7.html`](https://man7.org/linux/man-pages/man7/capabilities.7.html)。

    因为`CAP_SYS_ADMIN`必须由系统管理员显式启用，所以默认情况下 MySQL 二进制文件不具有启用网络命���空间支持。在启用之前，系统管理员应评估使用`CAP_SYS_ADMIN`运行 MySQL 进程的安全性影响。

以下示例中的说明设置了名为`red`和`blue`的网络命名空间。您选择的名称可能不同，主机系统上的网络地址和接口也可能不同。

在此处显示的命令要么作为`root`操作系统用户调用，要么在每个命令前加上**sudo**前缀。例如，如果你不是`root`用户，要调用**ip**或**setcap**命令，使用**sudo ip**或**sudo setcap**。

要配置网络命名空间，请使用**ip**命令。对于某些操作，**ip**命令必须在特定的命名空间内执行（该命名空间必须已经存在）。在这种情况下，命令应该这样开始：

```sql
ip netns exec *namespace_name*
```

例如，此命令在`red`命名空间内执行以启动环回接口：

```sql
ip netns exec red ip link set lo up
```

要添加名为`red`和`blue`的命名空间，每个命名空间都有自己的虚拟以太网设备用作命名空间之间的链接以及自己的环回接口：

```sql
ip netns add red
ip link add veth-red type veth peer name vpeer-red
ip link set vpeer-red netns red
ip addr add 192.0.2.1/24 dev veth-red
ip link set veth-red up
ip netns exec red ip addr add 192.0.2.2/24 dev vpeer-red
ip netns exec red ip link set vpeer-red up
ip netns exec red ip link set lo up

ip netns add blue
ip link add veth-blue type veth peer name vpeer-blue
ip link set vpeer-blue netns blue
ip addr add 198.51.100.1/24 dev veth-blue
ip link set veth-blue up
ip netns exec blue ip addr add 198.51.100.2/24 dev vpeer-blue
ip netns exec blue ip link set vpeer-blue up
ip netns exec blue ip link set lo up

# if you want to enable inter-subnet routing...
sysctl net.ipv4.ip_forward=1
ip netns exec red ip route add default via 192.0.2.1
ip netns exec blue ip route add default via 198.51.100.1
```

命名空间之间链接的图示如下： 

```sql
red              global           blue

192.0.2.2   <=>  192.0.2.1
(vpeer-red)      (veth-red)

                 198.51.100.1 <=> 198.51.100.2
                 (veth-blue)      (vpeer-blue)
```

要检查存在哪些命名空间和链接：

```sql
ip netns list
ip link list
```

要查看全局和命名空间的路由表：

```sql
ip route show
ip netns exec red ip route show
ip netns exec blue ip route show
```

要删除`red`和`blue`的链接和命名空间：

```sql
ip link del veth-red
ip link del veth-blue

ip netns del red
ip netns del blue

sysctl net.ipv4.ip_forward=0
```

为了使包含网络命名空间支持的 MySQL 二进制文件实际上可以使用命名空间，您必须授予它们`CAP_SYS_ADMIN`权限。以下**setcap**命令假定您已更改位置到包含 MySQL 二进制文件的目录（根据需要调整系统的路径名）：

```sql
cd /usr/local/mysql/bin
```

要将`CAP_SYS_ADMIN`权限授予适当的二进制文件：

```sql
setcap cap_sys_admin+ep ./mysqld
setcap cap_sys_admin+ep ./mysql
setcap cap_sys_admin+ep ./mysqlxtest
```

要检查`CAP_SYS_ADMIN`权限：

```sql
$> getcap ./mysqld ./mysql ./mysqlxtest
./mysqld = cap_sys_admin+ep
./mysql = cap_sys_admin+ep
./mysqlxtest = cap_sys_admin+ep
```

要删除`CAP_SYS_ADMIN`权限：

```sql
setcap -r ./mysqld
setcap -r ./mysql
setcap -r ./mysqlxtest
```

重要提示

如果重新安装之前已应用**setcap**的二进制文件，则必须再次使用**setcap**。例如，如果执行原地 MySQL 升级，未再次授予`CAP_SYS_ADMIN`权限将导致与命名空间相关的失败。服务器在尝试绑定到具有命名空间的地址���将失败，并显示此错误：

```sql
[ERROR] [MY-013408] [Server] setns() failed with error 'Operation not permitted'
```

使用`--network-namespace`选项调用的客户端会失败，如下所示：

```sql
ERROR: Network namespace error: Operation not permitted
```

#### MySQL 配置

假设前述主机系统先决条件已满足，MySQL 可以配置服务器端命名空间以监听（入站）连接的一侧，以及客户端端命名空间以进行出站连接。

在服务器端，`bind_address`、`admin_address`和`mysqlx_bind_address`系统变量具有扩展语法，用于指定用于给定 IP 地址或主机名监听传入连接的网络命名空间。要为地址指定命名空间，请添加斜杠和命名空间名称。例如，服务器的`my.cnf`文件可能包含以下行：

```sql
[mysqld]
bind_address = 127.0.1.1,192.0.2.2/red,198.51.100.2/blue
admin_address = 102.0.2.2/red
mysqlx_bind_address = 102.0.2.2/red
```

这些规则适用：

+   可以为 IP 地址或主机名指定网络命名空间。

+   无法为通配符 IP 地址指定网络命名空间。

+   对于给定的地址，网络命名空间是可选的。如果指定了，必须在地址后面立即添加`/*`ns`*`后缀。

+   没有`/*`ns`*`后缀的地址使用主机系统的全局命名空间。因此，全局命名空间是默认值。

+   带有`/*`ns`*`后缀的地址使用名为*`ns`*的命名空间。

+   主机系统必须支持网络命名空间，并且每个命名命名空间必须事先设置好。命名不存在的命名空间会产生错误。

+   `bind_address`和（自 MySQL 8.0.21 起）`mysqlx_bind_address`接受多个逗号分隔地址的列表，变量值可以指定全局命名空间中的地址、命名命名空间中的地址或混合地址。

如果在服务器启动过程中发生错误，尝试使用命名空间，则服务器将无法启动。如果在插件初始化过程中发生 X 插件的错误，导致无法绑定到任何地址，插件将在初始化序列中失败，服务器将无法加载它。

在客户端端，可以在以下情境中指定网络命名空间：

+   对于**mysql**客户端和**mysqlxtest**测试套件客户端，使用`--network-namespace`选项。例如：

    ```sql
    mysql --host=192.0.2.2 --network-namespace=red
    ```

    如果省略了`--network-namespace`选项，连接将使用默认（全局）命名空间。

+   对于从副本服务器到源服务器的复制连接，请使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（MySQL 8.0.23 之前），并指定`NETWORK_NAMESPACE`选项。例如：

    ```sql
    CHANGE REPLICATION SOURCE TO
      SOURCE_HOST = '192.0.2.2',
      NETWORK_NAMESPACE = 'red';
    ```

    如果省略了`NETWORK_NAMESPACE`选项，复制连接将使用默认（全局）命名空间。

以下示例设置了一个 MySQL 服务器，监听全局、`red` 和 `blue` 命名空间中的连接，并展示了如何配置从`red` 和 `blue` 命名空间连接的帐户。假设`red` 和 `blue` 命名空间已经像主机系统先决条件中所示创建。

1.  配置服务器以在多个命名空间中监听地址。将这些行放入服务器的`my.cnf`文件中并启动服务器：

    ```sql
    [mysqld]
    bind_address = 127.0.1.1,192.0.2.2/red,198.51.100.2/blue
    ```

    该值告诉服务器在全局命名空间中监听回环地址`127.0.0.1`，在`red`命名空间中监听地址`192.0.2.2`，在`blue`命名空间中监听地址`198.51.100.2`。

1.  连接到全局命名空间中的服务器，并创建具有连接权限的帐户，这些帐户可以从每个命名空间的地址空间连接：

    ```sql
    $> mysql -u root -h 127.0.0.1 -p
    Enter password: *root_password*

    mysql> CREATE USER 'red_user'@'192.0.2.2'
           IDENTIFIED BY '*red_user_password*';
    mysql> CREATE USER 'blue_user'@'198.51.100.2'
           IDENTIFIED BY '*blue_user_password*';
    ```

1.  验证您是否可以连接到每个命名空间中的服务器：

    ```sql
    $> mysql -u red_user -h 192.0.2.2 --network-namespace=red -p
    Enter password: *red_user_password*

    mysql> SELECT USER();
    +--------------------+
    | USER()             |
    +--------------------+
    | red_user@192.0.2.2 |
    +--------------------+
    ```

    ```sql
    $> mysql -u blue_user -h 198.51.100.2 --network-namespace=blue -p
    Enter password: *blue_user_password*

    mysql> SELECT USER();
    +------------------------+
    | USER()                 |
    +------------------------+
    | blue_user@198.51.100.2 |
    +------------------------+
    ```

    注意

    `USER()`可能返回不同的结果，如果您的 DNS 配置能够将地址解析为相应的主机名，而服务器未启用`skip_name_resolve`系统变量，则可能返回包含主机名而不是 IP 地址的值。

    您也可以尝试在不使用`--network-namespace`选项的情况下调用**mysql**，以查看连接尝试是否成功，以及`USER()`值如何受到影响。

#### 网络命名空间监控

为了复制监控目的，这些信息源具有显示连接适用的网络命名空间的列：

+   Performance Schema `replication_connection_configuration` 表。参见 Section 29.12.11.1, “The replication_connection_configuration Table”。

+   复制服务器连接元数据存储库。参见第 19.2.4.2 节，“复制元数据存储库”。

+   `SHOW REPLICA STATUS`（或在 MySQL 8.0.22 之前，`SHOW SLAVE STATUS`）语句。参见第 15.7.7.35 节，“SHOW REPLICA STATUS 语句”。
