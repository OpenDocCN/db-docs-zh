# 34.2 在 Oracle 云基础设施上部署 MySQL EE

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-oci-marketplace-deploy.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-oci-marketplace-deploy.html)

在 Oracle 云基础设施上部署 MySQL EE，请执行以下操作：

1.  打开 OCI Marketplace 并选择 MySQL。

    显示了 MySQL 列表。

1.  单击“启动实例”开始应用程序启动过程。

    显示“创建计算实例”对话框。

    有关如何填写字段的信息，请参阅[创建 Linux 实例](https://docs.cloud.oracle.com/iaas/Content/Compute/Tasks/launchinginstance.htm)。

默认情况下，MySQL 服务器监听端口 3306，并配置有单个用户 root。

重要

部署完成并启动 MySQL 服务器后，您必须连接到计算实例并检索写入 MySQL 日志文件的默认 root 密码。

有关更多信息，请参阅使用 SSH 连接。

已安装以下 MySQL 软件：

+   MySQL 服务器 EE

+   MySQL 企业备份

+   MySQL Shell

+   MySQL 路由器

### MySQL 配置

出于安全考虑，以下内容已启用：

+   SELinux：有关更多信息，请参阅[配置和使用 SELinux](https://docs.oracle.com/en/operating-systems/oracle-linux/7/admin/ol7-s1-syssec.html)

+   `firewalld`：有关更多信息，请参阅[控制 firewalld 防火墙服务](https://docs.oracle.com/en/operating-systems/oracle-linux/7/security/ol7-implement-sec.html#ol7-firewalld-cfg)

已启用以下 MySQL 插件：

+   `thread_pool`

+   `validate_password`

启动时，会发生以下情况：

+   MySQL 服务器读取 `/etc/my.cnf` 和 `/etc/my.cnf.d/` 中所有名为 `*`*`*.cnf` 的文件。

+   `/etc/my.cnf.d/perf-tuning.cnf` 是由 `/usr/bin/mkcnf` 基于所选的 OCI 形状创建的。

    注意

    要禁用此机制，请删除 `/etc/systemd/system/mysqld.service.d/perf-tuning.conf`。

    针对以下形状配置了性能调整：

    +   VM.Standard2.1

    +   VM.Standard2.2

    +   VM.Standard2.4

    +   VM.Standard2.8

    +   VM.Standard2.16

    +   VM.Standard2.24

    +   VM.Standard.E2.1

    +   VM.Standard.E2.2

    +   VM.Standard.E2.4

    +   VM.Standard.E2.8

    +   VM.Standard.E3.Flex

    +   VM.Standard.E4.Flex

    +   BM.Standard2.52

    对于所有其他形状，使用 VM.Standard2.1 的调整。
