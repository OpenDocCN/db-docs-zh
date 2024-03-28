# 15.1.1 原子数据定义语句支持

> 原文：[`dev.mysql.com/doc/refman/8.0/en/atomic-ddl.html`](https://dev.mysql.com/doc/refman/8.0/en/atomic-ddl.html)

MySQL 8.0 支持原子数据定义语言（DDL）语句。这一特性被称为*原子 DDL*。原子 DDL 语句将与 DDL 操作相关的数据字典更新、存储引擎操作和二进制日志写入合并为单个原子操作。该操作要么被提交，适用更改被持久化到数据字典、存储引擎和二进制日志中，要么被回滚，即使服务器在操作过程中停止。

注意

*原子 DDL*不是*事务性 DDL*。DDL 语句，无论是原子的还是其他的，都会隐式结束当前会话中活动的任何事务，就好像在执行该语句之前执行了`COMMIT`一样。这意味着 DDL 语句不能在另一个事务中执行，在事务控制语句中（如`START TRANSACTION ... COMMIT`）中执行，或者与同一事务中的其他语句组合。

MySQL 8.0 中引入的 MySQL 数据字典使原子 DDL 成为可能。在早期的 MySQL 版本中，元数据存储在元数据文件、非事务表和存储引擎特定的字典中，这需要中间提交。MySQL 数据字典提供的集中式、事务性元数据存储消除了这一障碍，使得重构 DDL 语句操作成为原子操作成为可能。

本节中以下主题描述了原子 DDL 特性：

+   支持的 DDL 语句

+   原子 DDL 特性

+   DDL 语句行为变化

+   存储引擎支持

+   查看 DDL 日志

#### 支持的 DDL 语句

原子 DDL 特性支持表和非表 DDL 语句。与表相关的 DDL 操作需要存储引擎支持，而非表 DDL 操作则不需要。目前，只有`InnoDB`存储引擎支持原子 DDL。

+   支持的表 DDL 语句包括对数据库、表空间、表和索引的`CREATE`、`ALTER`和`DROP`语句，以及`TRUNCATE TABLE`语句。

+   支持的非表 DDL 语句包括：

    +   `CREATE`和`DROP`语句，以及（如果适用）用于存储过程、触发器、视图和可加载函数的`ALTER`语句。

    +   帐户管理语句：用户和角色的`CREATE`，`ALTER`，`DROP`，以及如果适用的`RENAME`语句，以及`GRANT`和`REVOKE`语句。

原子 DDL 功能不支持以下语句：

+   与`InnoDB`不同的存储引擎涉及的与表相关的 DDL 语句。

+   `INSTALL PLUGIN`和`UNINSTALL PLUGIN`语句。

+   `INSTALL COMPONENT`和`UNINSTALL COMPONENT`语句。

+   `CREATE SERVER`，`ALTER SERVER`和`DROP SERVER`语句。

#### 原子 DDL 特性

原子 DDL 语句的特性包括以下内容：

+   元数据更新，二进制日志写入和存储引擎操作（如果适用）被合并为单个原子操作。

+   在 DDL 操作期间，SQL 层没有中间提交。

+   如果适用：

    +   数据字典，例程，事件和可加载函数缓存的状态与 DDL 操作的状态一致，这意味着缓存将根据 DDL 操作是否成功完成或回滚而更新。

    +   DDL 操作涉及的存储引擎方法不执行中间提交，并且存储引擎将自身注册为 DDL 操作的一部分。

    +   存储引擎支持 DDL 操作的重做和回滚，这是在 DDL 操作的*后 DDL*阶段执行的。

+   DDL 操作的可见行为是原子的，这改变了一些 DDL 语句的行为。请参阅 DDL 语句行为的变化。

#### DDL 语句行为的变化

本节描述了由于引入原子 DDL 支持而导致的 DDL 语句行为的变化。

+   如果所有命名表使用支持原子 DDL 的存储引擎，则`DROP TABLE`操作是完全原子的。该语句要么成功删除所有表，要么回滚。

    如果命名表不存在，则`DROP TABLE`将失败并显示错误，无论存储引擎如何，都不会进行任何更改。下面的示例演示了行为的变化，`DROP TABLE`语句因为命名表不存在而失败：

    ```sql
    mysql> CREATE TABLE t1 (c1 INT);
    mysql> DROP TABLE t1, t2;
    ERROR 1051 (42S02): Unknown table 'test.t2'
    mysql> SHOW TABLES;
    +----------------+
    | Tables_in_test |
    +----------------+
    | t1             |
    +----------------+
    ```

    在引入原子 DDL 之前，`DROP TABLE`对于不存在的命名表报告错误，但对于存在的命名表成功：

    ```sql
    mysql> CREATE TABLE t1 (c1 INT);
    mysql> DROP TABLE t1, t2;
    ERROR 1051 (42S02): Unknown table 'test.t2'
    mysql> SHOW TABLES;
    Empty set (0.00 sec)
    ```

    注意

    由于这种行为变化，当在 MySQL 5.7 复制源服务器上复制到 MySQL 8.0 副本时，部分完成的`DROP TABLE`语句会失败。为避免此失败场景，在`DROP TABLE`语句中使用`IF EXISTS`语法，以防止对不存在的表发生错误。

+   如果所有表使用支持原子 DDL 的存储引擎，则`DROP DATABASE`是原子的。该语句要么成功删除所有对象，要么回滚。然而，从文件系统中删除数据库目录是最后发生的，不是原子操作的一部分。如果由于文件系统错误或服务器停止而导致无法删除数据库目录，则不会回滚`DROP DATABASE`事务。

+   对于不使用支持原子 DDL 的存储引擎的表，表删除发生在原子`DROP TABLE`或`DROP DATABASE`事务之外。这种表删除会单独写入二进制日志，这限制了在中断的`DROP TABLE`或`DROP DATABASE`操作中存储引擎、数据字典和二进制日志之间的差异至多为一个表。对于删除多个表的操作，不使用支持原子 DDL 的存储引擎的表会在支持原子 DDL 的表之前被删除。

+   对于使用支持原子 DDL 的存储引擎的表，`CREATE TABLE`、`ALTER TABLE`、`RENAME TABLE`、`TRUNCATE TABLE`、`CREATE TABLESPACE`和`DROP TABLESPACE`操作在操作过程中如果服务器停止，要么完全提交，要么回滚。在早期的 MySQL 版本中，这些操作的中断可能导致存储引擎、数据字典和二进制日志之间的差异，或留下孤立文件。只有所有命名表使用支持原子 DDL 的存储引擎时，`RENAME TABLE`操作才是原子的。

+   从 MySQL 8.0.21 开始，在支持原子 DDL 的存储引擎上，当使用基于行的复制时，`CREATE TABLE ... SELECT` 语句被记录为二进制日志中的一个事务。以前，它被记录为两个事务，一个用于创建表，另一个用于插入数据。在两个事务之间或插入数据时发生服务器故障可能导致复制一个空表。随着原子 DDL 支持的引入，`CREATE TABLE ... SELECT` 语句现在对基于行的复制是安全的，并且允许在基于 GTID 的复制中使用。

    在支持原子 DDL 和外键约束的存储引擎上，在使用基于行的复制时，不允许在 `CREATE TABLE ... SELECT` 语句中创建外键约束。可以稍后使用 `ALTER TABLE` 添加外键约束。

    当 `CREATE TABLE ... SELECT` 作为原子操作应用时，插入数据时会在表上持有元数据锁，这会阻止在操作期间对表的并发访问。

+   如果命名视图不存在，`DROP VIEW` 失败，并且不会进行任何更改。行为变更在这个例子中得到展示，`DROP VIEW` 语句因为命名视图不存在而失败：

    ```sql
    mysql> CREATE VIEW test.viewA AS SELECT * FROM t;
    mysql> DROP VIEW test.viewA, test.viewB;
    ERROR 1051 (42S02): Unknown table 'test.viewB'
    mysql> SHOW FULL TABLES IN test WHERE TABLE_TYPE LIKE 'VIEW';
    +----------------+------------+
    | Tables_in_test | Table_type |
    +----------------+------------+
    | viewA          | VIEW       |
    +----------------+------------+
    ```

    在引入原子 DDL 之前，`DROP VIEW` 对不存在的命名视图返回错误，但对存在的命名视图成功：

    ```sql
    mysql> CREATE VIEW test.viewA AS SELECT * FROM t;
    mysql> DROP VIEW test.viewA, test.viewB;
    ERROR 1051 (42S02): Unknown table 'test.viewB'
    mysql> SHOW FULL TABLES IN test WHERE TABLE_TYPE LIKE 'VIEW';
    Empty set (0.00 sec)
    ```

    注意

    由于这种行为变更，在 MySQL 5.7 复制源服务器上部分完成的 `DROP VIEW` 操作在 MySQL 8.0 复制品上复制时会失败。为避免这种失败场景，在 `DROP VIEW` 语句中使用 `IF EXISTS` 语法，以防止对不存在的视图发生错误。

+   部分执行账户管理语句不再被允许。如果发生错误，账户管理语句要么对所有命名用户成功，要么回滚并且没有效果。在早期的 MySQL 版本中，命名多个用户的账户管理语句可能对一些用户成功，对另一些用户失败。

    行为变更在这个例子中得到展示，第二个 `CREATE USER` 语句返回错误但失败，因为它无法对所有命名用户成功。

    ```sql
    mysql> CREATE USER userA;
    mysql> CREATE USER userA, userB;
    ERROR 1396 (HY000): Operation CREATE USER failed for 'userA'@'%'
    mysql> SELECT User FROM mysql.user WHERE User LIKE 'user%';
    +-------+
    | User  |
    +-------+
    | userA |
    +-------+
    ```

    在引入原子 DDL 之前，第二个`CREATE USER`语句对于不存在的命名用户返回错误，但对于已存在的命名用户成功：

    ```sql
    mysql> CREATE USER userA;
    mysql> CREATE USER userA, userB;
    ERROR 1396 (HY000): Operation CREATE USER failed for 'userA'@'%'
    mysql> SELECT User FROM mysql.user WHERE User LIKE 'user%';
    +-------+
    | User  |
    +-------+
    | userA |
    | userB |
    +-------+
    ```

    注意

    由于这种行为变化，MySQL 5.7 复制源服务器上部分完成的帐户管理语句在 MySQL 8.0 副本上复制时会失败。为避免此失败场景，在帐户管理语句中使用`IF EXISTS`或`IF NOT EXISTS`语法，适当地防止与命名用户相关的错误。

#### 存储引擎支持

目前，只有`InnoDB`存储引擎支持原子 DDL。不支持原子 DDL 的存储引擎不受 DDL 原子性的约束。涉及免除的存储引擎的 DDL 操作仍然可能引入不一致性，当操作被中断或仅部分完成时可能发生。

为支持 DDL 操作的重做和回滚，`InnoDB`将 DDL 日志写入`mysql.innodb_ddl_log`表，这是一个隐藏的数据字典表，驻留在`mysql.ibd`数据字典表空间中。

要查看在 DDL 操作期间写入`mysql.innodb_ddl_log`表的 DDL 日志，请启用`innodb_print_ddl_logs`配置选项。有关更多信息，请参见查看 DDL 日志。

注意

对于对`mysql.innodb_ddl_log`表的更改的重做日志会立即刷新到磁盘，不受`innodb_flush_log_at_trx_commit`设置的影响。立即刷新重做日志可以避免数据文件被 DDL 操作修改，但由于这些操作导致的对`mysql.innodb_ddl_log`表的重做日志未持久化到磁盘。这种情况可能导致回滚或恢复时出现错误。

`InnoDB`存储引擎在阶段中执行 DDL 操作。DDL 操作，如`ALTER TABLE`可能在*提交*阶段之前多次执行*准备*和*执行*阶段。

1.  *准备*: 创建所需对象并将 DDL 日志写入`mysql.innodb_ddl_log`表。DDL 日志定义了如何前滚和回滚 DDL 操作。

1.  *执行*: 执行 DDL 操作。例如，执行`CREATE TABLE`操作的创建例程。

1.  *提交*: 更新数据字典并提交数据字典事务。

1.  *后 DDL*：从`mysql.innodb_ddl_log`表中重放和删除 DDL 日志。为了确保可以安全地执行回滚而不引入不一致性，文件操作（如重命名或删除数据文件）在这个最终阶段执行。此阶段还会从`mysql.innodb_dynamic_metadata`数据字典表中删除`DROP TABLE`、`TRUNCATE TABLE`和其他重建表的 DDL 操作的动态元数据。

无论 DDL 操作是提交还是回滚，在*后 DDL*阶段都会重放和删除`mysql.innodb_ddl_log`表中的 DDL 日志。只有在服务器在 DDL 操作期间停止时，DDL 日志才应该保留在`mysql.innodb_ddl_log`表中。在这种情况下，恢复后会重放和删除 DDL 日志。

在恢复情况下，当服务器重新启动时，DDL 操作可能会提交或回滚。如果在 DDL 操作的*提交*阶段执行的数据字典事务存在于重做日志和二进制日志中，则认为操作成功，并向前滚动。否则，当`InnoDB`重放数据字典重做日志时，不完整的数据字典事务将被回滚，DDL 操作将被回滚。

#### 查看 DDL 日志

要查看写入`mysql.innodb_ddl_log`数据字典表的与涉及`InnoDB`存储引擎的原子 DDL 操作相关的 DDL 日志，请启用`innodb_print_ddl_logs`以使 MySQL 将 DDL 日志写入`stderr`。根据主机操作系统和 MySQL 配置，`stderr`可能是错误日志、终端或控制台窗口。请参见第 7.4.2.2 节，“默认错误日志目标配置”。

`InnoDB`将 DDL 日志写入`mysql.innodb_ddl_log`表，以支持 DDL 操作的重做和回滚。`mysql.innodb_ddl_log`表是一个隐藏的数据字典表，驻留在`mysql.ibd`数据字典表空间中。与其他隐藏的数据字典表一样，在非调试版本的 MySQL 中无法直接访问`mysql.innodb_ddl_log`表。（参见第 16.1 节，“数据字典模式”。）`mysql.innodb_ddl_log`表的结构对应于以下定义：

```sql
CREATE TABLE mysql.innodb_ddl_log (
  id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  thread_id BIGINT UNSIGNED NOT NULL,
  type INT UNSIGNED NOT NULL,
  space_id INT UNSIGNED,
  page_no INT UNSIGNED,
  index_id BIGINT UNSIGNED,
  table_id BIGINT UNSIGNED,
  old_file_path VARCHAR(512) COLLATE utf8mb4_bin,
  new_file_path VARCHAR(512) COLLATE utf8mb4_bin,
  KEY(thread_id)
);
```

+   `id`: 用于标识 DDL 日志记录的唯一标识符。

+   `thread_id`：每个 DDL 日志记录都被分配一个`thread_id`，用于重放和删除属于特定 DDL 操作的 DDL 日志。涉及多个数据文件操作的 DDL 操作会生成多个 DDL 日志记录。

+   `type`：DDL 操作类型。类型包括`FREE`（删除索引树）、`DELETE`（删除文件）、`RENAME`（重命名文件）或`DROP`（从`mysql.innodb_dynamic_metadata`数据字典表中删除元数据）。

+   `space_id`：表空间 ID。

+   `page_no`：包含分配信息的页面；例如，索引树根页面。

+   `index_id`：索引 ID。

+   `table_id`：表 ID。

+   `old_file_path`：旧表空间文件路径。用于创建或删除表空间文件的 DDL 操作；也用于重命名表空间的 DDL 操作。

+   `new_file_path`：新表空间文件路径。用于重命名表空间文件的 DDL 操作。

这个示例演示了启用`innodb_print_ddl_logs`以查看写入`strderr`的 DDL 日志，用于`CREATE TABLE`操作。

```sql
mysql> SET GLOBAL innodb_print_ddl_logs=1;
mysql> CREATE TABLE t1 (c1 INT) ENGINE = InnoDB;
```

```sql
[Note] [000000] InnoDB: DDL log insert : [DDL record: DELETE SPACE, id=18, thread_id=7,
space_id=5, old_file_path=./test/t1.ibd]
[Note] [000000] InnoDB: DDL log delete : by id 18
[Note] [000000] InnoDB: DDL log insert : [DDL record: REMOVE CACHE, id=19, thread_id=7,
table_id=1058, new_file_path=test/t1]
[Note] [000000] InnoDB: DDL log delete : by id 19
[Note] [000000] InnoDB: DDL log insert : [DDL record: FREE, id=20, thread_id=7,
space_id=5, index_id=132, page_no=4]
[Note] [000000] InnoDB: DDL log delete : by id 20
[Note] [000000] InnoDB: DDL log post ddl : begin for thread id : 7
[Note] [000000] InnoDB: DDL log post ddl : end for thread id : 7
```
