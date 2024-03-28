# 34.5 维护

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-oci-marketplace-maintenance.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-oci-marketplace-maintenance.html)

该产品是用户管理的，意味着您负责升级和维护。

### 升级 MySQL

现有安装是基于 RPM 的，要升级 MySQL 服务器，请参见第 3.7 节，“在 Unix/Linux 上升级 MySQL 二进制或基于包的安装”。

您可以使用 `scp` 将所需的 RPM 复制到 OCI 计算实例，或者从 OCI 对象存储中复制，如果您已配置访问权限。文件存储也是一个选择。更多信息，请参见[文件存储和 NFS](https://docs.cloud.oracle.com/iaas/Content/File/Concepts/filestorageoverview.htm)。

### 备份和恢复

MySQL Enterprise Backup 是首选的备份和恢复解决方案。更多信息，请参见备份到云存储。

有关 MySQL Enterprise Backup 的信息，请参见开始使用 MySQL Enterprise Backup。

有关默认 MySQL 备份和恢复的信息，请参见第九章，*备份和恢复*。
