> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-binlog-encryption-encryption-keys.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-binlog-encryption-encryption-keys.html)

#### 19.3.2.2 二进制日志加密密钥

用于加密日志文件的文件密码的二进制日志加密密钥是为每个 MySQL 服务器实例专门生成的 256 位密钥，使用 MySQL 服务器的密钥环服务生成（参见第 8.4.4 节，“MySQL 密钥环”）。密钥环服务处理二进制日志加密密钥的创建、检索和删除。服务器实例仅为自身创建和删除生成的密钥，但如果存储在密钥环中，则可以读取为其他实例生成的密钥，例如通过文件复制克隆的服务器实例的情况。

重要提示

MySQL 服务器实例的二进制日志加密密钥必须包含在您的备份和恢复程序中，因为如果丢失了解密当前和保留的二进制日志文件或中继日志文件的文件密码所需的密钥，可能无法启动服务器。

密钥环中二进制日志加密密钥的格式如下：

```sql
MySQLReplicationKey_{UUID}_{SEQ_NO}
```

例如：

```sql
MySQLReplicationKey_00508583-b5ce-11e8-a6a5-0010e0734796_1
```

`{UUID}` 是由 MySQL 服务器生成的真实 UUID（`server_uuid` 系统变量的值）。`{SEQ_NO}` 是二进制日志加密密钥的序列号，对于在服务器上生成的每个新密钥，该序列号递增 1。

目前在服务器上使用的二进制日志加密密钥称为二进制日志主密钥。当前二进制日志主密钥的序列号存储在密钥环中。二进制日志主密钥用于加密每个新日志文件的文件密码，这是一个特定于用于加密文件数据的日志文件的随机生成的 32 字节文件密码。文件密码使用 AES-CBC（AES 密码块链接模式）使用 256 位二进制日志加密密钥和随机初始化向量（IV）进行加密，并存储在日志文件的文件头中。文件数据使用从文件密码生成的 256 位密钥和从文件密码生成的随机数也生成的随机数使用 AES-CTR（AES 计数器模式）进行加密。如果知道用于加密文件密码的二进制日志加密密钥，则可以使用 OpenSSL 密码工具包中提供的工具离线解密加密文件。

如果您使用文件复制来克隆一个具有加密功能的 MySQL 服务器实例，使其二进制日志文件和中继日志文件被加密，请确保也复制了密钥环，以便克隆服务器可以从源服务器读取二进制日志加密密钥。当在克隆服务器上激活加密（无论是在启动时还是随后），克隆服务器会意识到与复制文件一起使用的二进制日志加密密钥包括源服务器的生成的 UUID。它会自动生成一个新的二进制日志加密密钥，使用自己生成的 UUID，并将其用于加密后续的二进制日志文件和中继日志文件的文件密码。复制的文件将继续使用源服务器的密钥进行读取。
