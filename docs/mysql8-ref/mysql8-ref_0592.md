# 10.6.2 MyISAM 表的批量数据加载

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizing-myisam-bulk-data-loading.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizing-myisam-bulk-data-loading.html)

这些性能提示补充了 Section 10.2.5.1, “Optimizing INSERT Statements”中快速插入的��般准则。

+   对于`MyISAM`表，如果数据文件中间没有删除的行，可以使用并发插入同时添加行，同时运行`SELECT`语句。参见 Section 10.11.3, “Concurrent Inserts”。

+   经过一些额外工作，可以使`LOAD DATA`在`MyISAM`表中运行得更快，尤其是当表中有许多索引时。使用以下过程：

    1.  执行`FLUSH TABLES`语句或**mysqladmin flush-tables**命令。

    1.  使用**myisamchk --keys-used=0 -rq *`/path/to/db/tbl_name`***来删除表中所有索引的使用。

    1.  使用`LOAD DATA`向表中插入数据。这不会更新任何索引，因此非常快。

    1.  如果将来只打算从表中读取数据，请使用**myisampack**对其进行压缩。参见 Section 18.2.3.3, “Compressed Table Characteristics”。

    1.  使用**myisamchk -rq *`/path/to/db/tbl_name`***重新创建索引。这会在将索引写入磁盘之前在内存中创建索引树，比在`LOAD DATA`期间更新索引要快得多，因为它避免了大量的磁盘查找。生成的索引树也是完全平衡的。

    1.  执行`FLUSH TABLES`语句或**mysqladmin flush-tables**命令。

    `LOAD DATA` 如果你要插入数据的`MyISAM`表是空的，它会自动执行前述优化。自动优化和显式使用该过程的主要区别在于，你可以让**myisamchk**为索引创建分配更多临时内存，而不是在执行`LOAD DATA`语句时让服务器为索引重建分配更多内存。

    你也可以通过以下语句来禁用或启用`MyISAM`表的非唯一索引，而不是使用**myisamchk**。如果使用这些语句，你可以跳过`FLUSH TABLES`操作：

    ```sql
    ALTER TABLE *tbl_name* DISABLE KEYS;
    ALTER TABLE *tbl_name* ENABLE KEYS;
    ```

+   为了加快对非事务表执行的多语句`INSERT`操作的速度，锁定你的表：

    ```sql
    LOCK TABLES a WRITE;
    INSERT INTO a VALUES (1,23),(2,34),(4,33);
    INSERT INTO a VALUES (8,26),(6,29);
    ...
    UNLOCK TABLES;
    ```

    这有助于性能，因为索引缓冲区仅在所有`INSERT`语句完成后一次性刷新到磁盘。通常情况下，会有与`INSERT`语句数量相同的索引缓冲区刷新。如果你可以使用单个`INSERT`插入所有行，则不需要显式的锁定语句。

    锁定还降低了多连接测试的总时间，尽管单个连接的最大等待时间可能会增加，因为它们在等待锁。假设五个客户端同时尝试执行插入操作如下：

    +   连接 1 执行 1000 次插入

    +   连接 2、3 和 4 执行 1 次插入

    +   连接 5 执行 1000 次插入

    如果不使用锁定，连接 2、3 和 4 会在 1 和 5 之前完成。如果使用锁定，连接 2、3 和 4 可能不会在 1 或 5 之前完成，但总时间应该快约 40%。

    在 MySQL 中，`INSERT`、`UPDATE`和`DELETE`操作非常快，但通过在进行超过约五次连续插入或更新的操作周围添加锁，你可以获得更好的整体性能。如果你进行了很多连续的插入操作，你可以偶尔执行一次`LOCK TABLES`，然后再执行一次`UNLOCK TABLES`（每 1000 行左右），以允许其他线程访问表。这仍然会带来良好的性能提升。

    对于加载数据，`INSERT`仍然比`LOAD DATA`慢得多，即使使用了刚刚概述的策略。

+   为了提高`MyISAM`表的性能，无论是对于`LOAD DATA`还是`INSERT`，都可以通过增加`key_buffer_size`系统变量来扩大键缓存。参见第 7.1.1 节，“配置服务器”。
