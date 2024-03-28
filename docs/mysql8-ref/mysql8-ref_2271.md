# 34.4 连接

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-oci-marketplace-connecting.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-oci-marketplace-connecting.html)

本节描述了连接到 OCI 计算实例上部署的 MySQL 服务器的各种连接方法。

### 通过 SSH 连接

本节详细介绍了如何从类 UNIX 平台连接到 OCI 计算实例。有关使用 SSH 连接的更多信息，请参阅[使用 SSH 访问 Oracle Linux 实例](https://docs.oracle.com/en/cloud/iaas/compute-iaas-cloud/stcsg/accessing-oracle-linux-instance-using-ssh.html#GUID-D947E2CC-0D4C-43F4-B2A9-A517037D6C11)和[连接到您的实例](https://docs.cloud.oracle.com/iaas/Content/GSG/Tasks/testingconnection.htm)。

要通过 SSH 连接到运行在计算实例上的 Oracle Linux，请运行以下命令：

```sql
ssh opc@*computeIP*
```

其中`opc`是计算用户，*`computeIP`*是您计算实例的 IP 地址。

要查找为 root 用户创建的临时根密码，请运行以下命令：

```sql
sudo grep 'temporary password' /var/log/mysqld.log
```

要更改默认密码，请使用生成的临时密码登录服务器，使用以下命令：`mysql -uroot -p`。然后运行以下命令：

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
```

### 使用 MySQL 客户端连接

注意

要从本地 MySQL 客户端连接，您必须首先在 MySQL 服务器上创建一个允许远程登录的用户。

要从本地 MySQL 客户端连接到 MySQL 服务器，请在您的 shell 会话中运行以下命令：

```sql
mysql -uroot -p -h*computeIP*
```

其中*`computeIP`*是您计算实例的 IP 地址。

### 使用 MySQL Shell 连接

要从本地 MySQL Shell 连接到 MySQL 服务器，请运行以下命令启动您的 shell 会话：

```sql
mysqlsh \connect root@*computeIP*
```

其中*`computeIP`*是您计算实例的 IP 地址。

有关 MySQL Shell 连接的更多信息，请参阅 MySQL Shell 连接。

### 连接到工作台

要从 MySQL Workbench 连接到 MySQL 服务器，请参阅 MySQL Workbench 中的连接。
