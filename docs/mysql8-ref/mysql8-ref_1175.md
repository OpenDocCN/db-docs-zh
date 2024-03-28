> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-temporary-tablespace.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-temporary-tablespace.html)

#### 17.6.3.5 临时表空间

`InnoDB` 使用会话临时表空间和全局临时表空间。

##### 会话临时表空间

会话临时表空间存储用户创建的临时表和优化器创建的内部临时表，当 `InnoDB` 配置为磁盘上内部临时表的存储引擎时。从 MySQL 8.0.16 开始，用于磁盘上内部临时表的存储引擎是 `InnoDB`。（以前，存储引擎由 `internal_tmp_disk_storage_engine` 的值确定。）

会话临时表空间从临时表空间池中分配给会话，当首次请求创建磁盘上的临时表时。每个会话最多分配两个表空间，一个用于用户创建的临时表，另一个用于优化器创建的内部临时表。分配给会话的临时表空间用于会话创建的所有磁盘上的临时表。会话断开连接时，其临时表空间被截断并释放回池中。服务器启动时创建一个包含 10 个临时表空间的池。池的大小永远不会缩小，表空间会根据需要自动添加到池中。会话临时表空间在正常关闭或初始化中止时被移除。会话临时表空间文件在创建时为五个页面大小，并具有 `.ibt` 文件扩展名。

为会话临时表空间保留了 40 万个空间 ID 范围。因为会话临时表空间池每次服务器启动时都会重新创建，所以会话临时表空间的空间 ID 在服务器关闭时不会持久化，并且可能被重用。

`innodb_temp_tablespaces_dir` 变量定义了会话临时表空间创建的位置。默认位置是数据目录中的 `#innodb_temp` 目录。如果无法创建临时表空间池，则拒绝启动。

```sql
$> cd *BASEDIR*/data/#innodb_temp
$> ls
temp_10.ibt  temp_2.ibt  temp_4.ibt  temp_6.ibt  temp_8.ibt
temp_1.ibt   temp_3.ibt  temp_5.ibt  temp_7.ibt  temp_9.ibt
```

在基于语句的复制（SBR）模式下，在副本上创建的临时表位于一个会话临时表空间中，该表空间仅在 MySQL 服务器关闭时截断。

`INNODB_SESSION_TEMP_TABLESPACES` 表提供有关会话临时表空间的元数据。

信息模式 `INNODB_TEMP_TABLE_INFO` 表提供有关在 `InnoDB` 实例中活动的用户创建的临时表的元数据。

##### 全局临时表空间

全局临时表空间（`ibtmp1`）存储对用户创建的临时表所做更改的回滚段。

`innodb_temp_data_file_path`变量定义了全局临时表空间数据文件的相对路径、名称、大小和属性。如果未为`innodb_temp_data_file_path`指定任何值，则默认行为是在`innodb_data_home_dir`目录中创建一个名为`ibtmp1`的单个自动扩展数据文件。初始文件大小略大于 12MB。

全局临时表空间在正常关闭或中止初始化时被移除，并在每次服务器启动时重新创建。全局临时表空间在创建时接收一个动态生成的空间 ID。如果无法创建全局临时表空间，则拒绝启动。如果服务器意外停止，则不会移除全局临时表空间。在这种情况下，数据库管理员可以手动移除全局临时表空间或重新启动 MySQL 服务器。重新启动 MySQL 服务器会自动移除并重新创建全局临时表空间。

全局临时表空间不能位于原始设备上。

信息模式`FILES`表提供有关全局临时表空间的元数据。发出类似以下查询以查看全局临时表空间元数据：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.FILES WHERE TABLESPACE_NAME='innodb_temporary'\G
```

默认情况下，全局临时表空间数据文件是自动扩展的，并根据需要增加大小。

要确定全局临时表空间数据文件是否自动扩展，请检查`innodb_temp_data_file_path`设置：

```sql
mysql> SELECT @@innodb_temp_data_file_path;
+------------------------------+
| @@innodb_temp_data_file_path |
+------------------------------+
| ibtmp1:12M:autoextend        |
+------------------------------+
```

要检查全局临时表空间数据文件的大小，请使用类似以下查询检查信息模式`FILES`表：

```sql
mysql> SELECT FILE_NAME, TABLESPACE_NAME, ENGINE, INITIAL_SIZE, TOTAL_EXTENTS*EXTENT_SIZE
       AS TotalSizeBytes, DATA_FREE, MAXIMUM_SIZE FROM INFORMATION_SCHEMA.FILES
       WHERE TABLESPACE_NAME = 'innodb_temporary'\G
*************************** 1\. row ***************************
      FILE_NAME: ./ibtmp1
TABLESPACE_NAME: innodb_temporary
         ENGINE: InnoDB
   INITIAL_SIZE: 12582912
 TotalSizeBytes: 12582912
      DATA_FREE: 6291456
   MAXIMUM_SIZE: NULL
```

`TotalSizeBytes`显示全局临时表空间数据文件的当前大小。有关其他字段值的信息，请参阅第 28.3.15 节，“信息模式 FILES 表”。

或者，在操作系统上检查全局临时表空间数据文件大小。全局临时表空间数据文件位于由`innodb_temp_data_file_path`变量定义的目录中。

要回收全局临时表空间数据文件占用的磁盘空间，重新启动 MySQL 服务器。重新启动服务器会根据`innodb_temp_data_file_path`定义的属性删除并重新创建全局临时表空间数据文件。

为了限制全局临时表空间数据文件的大小，配置`innodb_temp_data_file_path`以指定最大文件大小。例如：

```sql
[mysqld]
innodb_temp_data_file_path=ibtmp1:12M:autoextend:max:500M
```

配置`innodb_temp_data_file_path`需要重新启动服务器。
