# 17.20.7 InnoDB memcached 插件和复制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-memcached-replication.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-memcached-replication.html)

因为`daemon_memcached`插件支持 MySQL 的二进制日志，源服务器通过**memcached**接口可以进行备份复制，平衡读取工作负载，并实现高可用性。所有**memcached**命令都支持二进制日志记录。

您无需在副本服务器上设置`daemon_memcached`插件。此配置的主要优势是增加源端的写入吞吐量。复制机制的速度不受影响。

以下各节显示了在使用`daemon_memcached`插件进行 MySQL 复制时如何使用二进制日志功能。假定您已完成第 17.20.3 节“设置 InnoDB memcached 插件”中描述的设置。

#### 启用 InnoDB memcached 二进制日志

1.  要在 MySQL 的二进制日志中使用`daemon_memcached`插件，请在源服务器上启用`innodb_api_enable_binlog`配置选项。此选项只能在服务器启动时设置。您还必须在源服务器上使用`--log-bin`选项启用 MySQL 的二进制日志。您可以将这些选项添加到 MySQL 配置文件中，或者在**mysqld**命令行中添加。

    ```sql
    mysqld ... --log-bin -–innodb_api_enable_binlog=1
    ```

1.  配置源服务器和副本服务器，如第 19.1.2 节“基于二进制日志文件位置的复制设置”中所述。

1.  使用**mysqldump**创建源数据快照，并将快照同步到副本服务器。

    ```sql
    source $> mysqldump --all-databases --lock-all-tables > dbdump.db
    replica $> mysql < dbdump.db
    ```

1.  在源服务器上，执行`SHOW MASTER STATUS`以获取源二进制日志坐标。

    ```sql
    mysql> SHOW MASTER STATUS;
    ```

1.  在副本服务器上，使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（MySQL 8.0.23 之前）设置使用源二进制日志坐标的副本服务器。

    ```sql
    mysql> CHANGE MASTER TO
           MASTER_HOST='localhost',
           MASTER_USER='root',
           MASTER_PASSWORD='',
           MASTER_PORT = 13000,
           MASTER_LOG_FILE='0.000001,
           MASTER_LOG_POS=114;

    Or from MySQL 8.0.23:
    mysql> CHANGE REPLICATION SOURCE TO
           SOURCE_HOST='localhost',
           SOURCE_USER='root',
           SOURCE_PASSWORD='',
           SOURCE_PORT = 13000,
           SOURCE_LOG_FILE='0.000001,
           SOURCE_LOG_POS=114;
    ```

1.  启动副本。

    ```sql
    mysql> START SLAVE;
    Or from MySQL 8.0.22:
    mysql> START REPLICA;
    ```

    如果错误日志输出类似于以下内容，则副本已准备好进行复制。

    ```sql
    2013-09-24T13:04:38.639684Z 49 [Note] Replication I/O thread: connected to
    source 'root@localhost:13000', replication started in log '0.000001'
    at position 114
    ```

#### 测试 InnoDB memcached 复制配置

该示例演示了如何使用**memcached**和 telnet 测试**InnoDB** **memcached**复制配置，以插入、更新和删除数据。使用 MySQL 客户端验证源服务器和副本服务器上的结果。

该示例使用了`innodb_memcached_config.sql`配置脚本在`daemon_memcached`插件的初始设置期间创建的`demo_test`表。`demo_test`表包含一个示例记录。

1.  使用`set`命令插入一个具有键`test1`、标志值`10`、过期值`0`、cas 值 1 和值`t1`的记录。

    ```sql
    telnet 127.0.0.1 11211
    Trying 127.0.0.1...
    Connected to 127.0.0.1.
    Escape character is '^]'.
    set test1 10 0 1
    t1
    STORED
    ```

1.  在源服务器上，检查记录是否插入到`demo_test`表中。假设`demo_test`表之前未被修改，应该有两条记录。一个具有键`AA`的示例记录，以及刚刚插入的具有键`test1`的记录。`c1`列映射到键，`c2`列映射到值，`c3`列映射到标志值，`c4`列映射到 cas 值，`c5`列映射到过期时间。过期时间设置为 0，因为未使用。

    ```sql
    mysql> SELECT * FROM test.demo_test;
    +-------+--------------+------+------+------+
    | c1    | c2           | c3   | c4   | c5   |
    +-------+--------------+------+------+------+
    | AA    | HELLO, HELLO |    8 |    0 |    0 |
    | test1 | t1           |   10 |    1 |    0 |
    +-------+--------------+------+------+------+
    ```

1.  检查验证相同记录是否被复制到副本服务器。

    ```sql
    mysql> SELECT * FROM test.demo_test;
    +-------+--------------+------+------+------+
    | c1    | c2           | c3   | c4   | c5   |
    +-------+--------------+------+------+------+
    | AA    | HELLO, HELLO |    8 |    0 |    0 |
    | test1 | t1           |   10 |    1 |    0 |
    +-------+--------------+------+------+------+
    ```

1.  使用`set`命令将键更新为`new`的值。

    ```sql
    telnet 127.0.0.1 11211
    Trying 127.0.0.1...
    Connected to 127.0.0.1.
    Escape character is '^]'.
    set test1 10 0 2
    new
    STORED
    ```

    更新被复制到副本服务器（注意`cas`值也被更新）。

    ```sql
    mysql> SELECT * FROM test.demo_test;
    +-------+--------------+------+------+------+
    | c1    | c2           | c3   | c4   | c5   |
    +-------+--------------+------+------+------+
    | AA    | HELLO, HELLO |    8 |    0 |    0 |
    | test1 | new          |   10 |    2 |    0 |
    +-------+--------------+------+------+------+
    ```

1.  使用`delete`命令删除`test1`记录。

    ```sql
    telnet 127.0.0.1 11211
    Trying 127.0.0.1...
    Connected to 127.0.0.1.
    Escape character is '^]'.
    delete test1
    DELETED
    ```

    当`delete`操作被复制到副本时，副本上的`test1`记录也被删除。

    ```sql
    mysql> SELECT * FROM test.demo_test;
    +----+--------------+------+------+------+
    | c1 | c2           | c3   | c4   | c5   |
    +----+--------------+------+------+------+
    | AA | HELLO, HELLO |    8 |    0 |    0 |
    +----+--------------+------+------+------+
    ```

1.  使用`flush_all`命令从表中删除所有行。

    ```sql
    telnet 127.0.0.1 11211
    Trying 127.0.0.1...
    Connected to 127.0.0.1.
    Escape character is '^]'.
    flush_all
    OK
    ```

    ```sql
    mysql> SELECT * FROM test.demo_test;
    Empty set (0.00 sec)
    ```

1.  使用 telnet 连接到源服务器并输入两条新记录。

    ```sql
    telnet 127.0.0.1 11211
    Trying 127.0.0.1...
    Connected to 127.0.0.1.
    Escape character is '^]'
    set test2 10 0 4
    again
    STORED
    set test3 10 0 5
    again1
    STORED
    ```

1.  确认两条记录是否被复制到副本服务器。

    ```sql
    mysql> SELECT * FROM test.demo_test;
    +-------+--------------+------+------+------+
    | c1    | c2           | c3   | c4   | c5   |
    +-------+--------------+------+------+------+
    | test2 | again        |   10 |    4 |    0 |
    | test3 | again1       |   10 |    5 |    0 |
    +-------+--------------+------+------+------+
    ```

1.  使用`flush_all`命令从表中删除所有行。

    ```sql
    telnet 127.0.0.1 11211
    Trying 127.0.0.1...
    Connected to 127.0.0.1.
    Escape character is '^]'.
    flush_all
    OK
    ```

1.  检查确保`flush_all`操作在副本服务器上被复制。

    ```sql
    mysql> SELECT * FROM test.demo_test;
    Empty set (0.00 sec)
    ```

#### **InnoDB** **memcached**二进制日志注释

二进制日志格式：

+   大多数**memcached**操作都映射到 DML 语句（类似于插入、删除、更新）。由于 MySQL 服务器没有实际的 SQL 语句在处理，所有**memcached**命令（除了`flush_all`）使用基于行的复制（RBR）日志记录，这与任何服务器`binlog_format`设置无关。

+   **memcached**的`flush_all`命令映射到 MySQL 5.7 及更早版本的`TRUNCATE TABLE`命令。由于 DDL 命令只能使用基于语句的日志记录，`flush_all`命令通过发送`TRUNCATE TABLE`语句来复制。在 MySQL 8.0 及更高版本中，`flush_all`映射到`DELETE`，但仍通过发送`TRUNCATE TABLE`语句来复制。

事务：

+   事务的概念通常不是**memcached**应用的一部分。为了性能考虑，`daemon_memcached_r_batch_size`和`daemon_memcached_w_batch_size`用于控制读取和写入事务的批处理大小。这些设置不影响复制。在底层`InnoDB`表上的每个 SQL 操作在成功完成后被复制。

+   `daemon_memcached_w_batch_size`的默认值为`1`，这意味着每个**memcached**写操作立即提交。这个默认设置会产生一定的性能开销，以避免在源服务器和副本服务器上可见的数据不一致。复制的记录在副本服务器上始终立即可用。如果将`daemon_memcached_w_batch_size`设置为大于`1`的值，则通过**memcached**插入或更新的记录在源服务器上不会立即可见；在提交之前在源服务器上查看记录，请发出`SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED`。
