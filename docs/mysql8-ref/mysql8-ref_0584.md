# 10.5.5 InnoDB 表的批量数据加载

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizing-innodb-bulk-data-loading.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizing-innodb-bulk-data-loading.html)

这些性能提示补充了快速插入的一般准则，详见 Section 10.2.5.1, “优化 INSERT 语句”。

+   当将数据导入`InnoDB`时，关闭自动提交模式，因为它会为每次插入执行一次日志刷新到磁盘。在导入操作期间禁用自动提交，请使用`SET autocommit`和`COMMIT`语句包围它：

    ```sql
    SET autocommit=0;
    *... SQL import statements ...* COMMIT;
    ```

    **mysqldump**选项`--opt`创建的转储文件可以快速导入到`InnoDB`表中，即使没有用`SET autocommit`和`COMMIT`语句包装它们。

+   如果在次要键上有`UNIQUE`约束，则可以在导入会话期间暂时关闭唯一性检查以加快表导入速度：

    ```sql
    SET unique_checks=0;
    *... SQL import statements ...* SET unique_checks=1;
    ```

    对于大表，这样可以节省大量磁盘 I/O，因为`InnoDB`可以使用其更改缓冲区批量写入次要索引记录。确保数据不包含重复键。

+   如果您的表中有`FOREIGN KEY`约束，可以通过在导入会话期间关闭外键检查来加快表导入速度：

    ```sql
    SET foreign_key_checks=0;
    *... SQL import statements ...* SET foreign_key_checks=1;
    ```

    对于大��，这可以节省大量磁盘 I/O。

+   如果需要插入许多行，请使用多行`INSERT`语法以减少客户端和服务器之间的通信开销：

    ```sql
    INSERT INTO yourtable VALUES (1,2), (5,5), ...;
    ```

    这个提示适用于任何表的插入，不仅仅是`InnoDB`表。

+   在对具有自增列的表进行批量插入时，将`innodb_autoinc_lock_mode`设置为 2（交错）而不是 1（连续）。有关详细信息，请参阅 Section 17.6.1.6, “InnoDB 中的 AUTO_INCREMENT 处理”。

+   在执行批量插入时，按`PRIMARY KEY`顺序插入行会更快。`InnoDB`表使用聚簇索引，这使得按`PRIMARY KEY`顺序使用数据相对快速。按`PRIMARY KEY`顺序执行批量插入对于完全不适合缓冲池的表格尤为重要。

+   在将数据加载到`InnoDB`的`FULLTEXT`索引时，为了获得最佳性能，请按照以下步骤进行：

    1.  在表创建时定义一个名为`FTS_DOC_ID`的列，类型为`BIGINT UNSIGNED NOT NULL`，并创建一个名为`FTS_DOC_ID_INDEX`的唯一索引。例如：

        ```sql
        CREATE TABLE t1 (
        FTS_DOC_ID BIGINT unsigned NOT NULL AUTO_INCREMENT,
        title varchar(255) NOT NULL DEFAULT '',
        text mediumtext NOT NULL,
        PRIMARY KEY (`FTS_DOC_ID`)
        ) ENGINE=InnoDB;
        CREATE UNIQUE INDEX FTS_DOC_ID_INDEX on t1(FTS_DOC_ID);
        ```

    1.  将数据加载到表中。

    1.  在数据加载后创建`FULLTEXT`索引。

    注意

    在表创建时添加`FTS_DOC_ID`列时，请确保在更新`FULLTEXT`索引列时更新`FTS_DOC_ID`列，因为`FTS_DOC_ID`必须随着每个`INSERT`或`UPDATE`而单调递增。如果选择不在表创建时添加`FTS_DOC_ID`并让`InnoDB`为您管理 DOC ID，`InnoDB`会在下一次`CREATE FULLTEXT INDEX`调用时添加`FTS_DOC_ID`作为隐藏列。然而，这种方法需要重建表，可能会影响性能。

+   如果要将数据加载到一个*新的* MySQL 实例中，请考虑使用`ALTER INSTANCE {ENABLE|DISABLE} INNODB REDO_LOG`语法来禁用重做日志记录。禁用重做日志记录有助于加快数据加载速度，避免重做日志写入。更多信息，请参见禁用重做日志记录。

    警告

    此功能仅用于将数据加载到新的 MySQL 实例中。*不要在生产系统上禁用重做日志记录*。允许在禁用重做日志记录时关闭并重新启动服务器，但在禁用重做日志记录时发生意外服务器停止可能会导致数据丢失和实例损坏。

+   使用 MySQL Shell 导入数据。MySQL Shell 的并行表导入实用程序`util.importTable()`为大型数据文件提供了快速数据导入到 MySQL 关系表的功能。MySQL Shell 的转储加载实用程序`util.loadDump()`也提供了并行加载功能。请参见 MySQL Shell 实用程序。
