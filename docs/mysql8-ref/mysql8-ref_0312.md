> 原文：[`dev.mysql.com/doc/refman/8.0/en/clone-plugin-encrypted-data.html`](https://dev.mysql.com/doc/refman/8.0/en/clone-plugin-encrypted-data.html)

#### 7.6.7.5 克隆加密数据

支持对加密数据进行克隆。以下要求适用：

+   在将远程数据克隆时，需要安全连接以确保未加密表空间密钥在网络上传输时的安全性。表空间密钥在捐赠者处解密后传输，并在接收者处使用接收者主密钥重新加密。如果没有加密连接可用或在 `CLONE INSTANCE` 语句中使用 `REQUIRE NO SSL` 子句，则会报告错误。有关为克隆配置加密连接的信息，请参见 为克隆配置加密连接。

+   当将数据克隆到使用本地管理的密钥环的本地数据目录时，启动克隆目录上的 MySQL 服务器时必须使用相同的密钥环。

+   当将数据克隆到使用本地管理的密钥环的远程数据目录（接收者目录）时，启动克隆目录上的 MySQL 服务器时必须使用接收者密钥环。

注意

`innodb_redo_log_encrypt` 和 `innodb_undo_log_encrypt` 变量设置在克隆操作进行时无法修改。

有关数据加密功能的信息，请参见 第 17.13 节，“InnoDB 数据静态加密”。
