> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-directory.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-directory.html)

#### 19.5.1.10 复制和 DIRECTORY 表选项

如果在源服务器的`CREATE TABLE`语句中使用了`DATA DIRECTORY`或`INDEX DIRECTORY`表选项，则该表选项也会在副本上使用。如果在副本主机文件系统中不存在相应的目录，或者存在但对副本 MySQL 服务器不可访问，则可能会出现问题。可以通过在副本上使用`NO_DIR_IN_CREATE`服务器 SQL 模式来覆盖此行为，这会导致副本在复制`CREATE TABLE`语句时忽略`DATA DIRECTORY`和`INDEX DIRECTORY`表选项。结果是`MyISAM`数据和索引文件将在表的数据库目录中创建。

查看更多信息，请参见第 7.1.11 节，“服务器 SQL 模式”。
