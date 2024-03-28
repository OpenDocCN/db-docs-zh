> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-disabling-tablespace-path-validation.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-disabling-tablespace-path-validation.html)

#### 17.6.3.7 禁用表空间路径验证

在启动时，`InnoDB` 会扫描由 `innodb_directories` 变量定义的目录，以查找表空间文件。发现的表空间文件的路径会与数据字典中记录的路径进行验证。如果路径不匹配，则会更新数据字典中的路径。

`innodb_validate_tablespace_paths` 变量是在 MySQL 8.0.21 中引入的，允许禁用表空间路径验证。此功能适用于表空间文件未移动的环境。禁用路径验证可以提高在具有大量表空间文件的系统上的启动时间。如果 `log_error_verbosity` 设置为 3，在禁用表空间路径验证时启动时会打印以下消息：

```sql
[InnoDB] Skipping InnoDB tablespace path validation. 
Manually moved tablespace files will not be detected!
```

警告

在移动表空间文件后以禁用表空间路径验证的方式启动服务器可能导致未定义的行为。
