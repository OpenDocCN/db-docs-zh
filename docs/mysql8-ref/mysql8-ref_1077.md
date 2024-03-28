> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-binary-logs.html`](https://dev.mysql.com/doc/refman/8.0/en/show-binary-logs.html)

#### 15.7.7.1 SHOW BINARY LOGS Statement

```sql
SHOW BINARY LOGS
SHOW MASTER LOGS
```

列出服务器上的二进制日志文件。此语句用作 Section 15.4.1.1, “PURGE BINARY LOGS Statement”中描述的过程的一部分，该过程显示了如何确定哪些日志可以被清除。`SHOW BINARY LOGS` 需要`REPLICATION CLIENT`权限（或已弃用的`SUPER`权限）。

加密的二进制日志文件具有一个 512 字节的文件头，其中存储了加密和解密文件所需的信息。这些信息包含在`SHOW BINARY LOGS`显示的文件大小中。`Encrypted`列显示二进制日志文件是否已加密。如果为服务器设置了`binlog_encryption=ON`，则二进制日志加密处于活动状态。如果在服务器运行时激活或停用二进制日志加密，则现有的二进制日志文件不会被加密或解密。

```sql
mysql> SHOW BINARY LOGS;
+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| binlog.000015 |    724935 |       Yes |
| binlog.000016 |    733481 |       Yes |
+---------------+-----------+-----------+
```

`SHOW MASTER LOGS` 等同于 `SHOW BINARY LOGS`。
