> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-memcached-ddl.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-memcached-ddl.html)

#### 17.20.6.6 在底层 InnoDB 表上执行 DML 和 DDL 语句

您可以通过标准 SQL 接口访问底层的`InnoDB`表（默认为`test.demo_test`）。但是，存在一些限制：

+   当查询通过**memcached**接口访问的表时，请记住**memcached**操作可以配置为定期提交，而不是在每次写操作后立即提交。此行为由`daemon_memcached_w_batch_size`选项控制。如果此选项设置为大于`1`的值，请使用`READ UNCOMMITTED`查询以查找刚刚插入的行。

    ```sql
    mysql> SET SESSSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

    mysql> SELECT * FROM demo_test;
    +------+------+------+------+-----------+------+------+------+------+------+------+
    | cx   | cy   | c1   | cz   | c2        | ca   | CB   | c3   | cu   | c4   | C5   |
    +------+------+------+------+-----------+------+------+------+------+------+------+
    | NULL | NULL | a11  | NULL | 123456789 | NULL | NULL |   10 | NULL |    3 | NULL |
    +------+------+------+------+-----------+------+------+------+------+------+------+
    ```

+   当使用 SQL 修改通过**memcached**接口访问的表时，您可以配置**memcached**操作定期启动新事务，而不是每次读操作都启动事务。此行为由`daemon_memcached_r_batch_size`选项控制。如果此选项设置为大于`1`的值，则使用 SQL 对表进行的更改不会立即对**memcached**操作可见。

+   对于事务中的所有操作，`InnoDB`表都会被 IS（意向共享）或 IX（意向排他）锁定。如果您将`daemon_memcached_r_batch_size`和`daemon_memcached_w_batch_size`从默认值`1`大幅增加，那么表在每个操作之间很可能被锁定，从而阻止对表的 DDL 语句的执行。
