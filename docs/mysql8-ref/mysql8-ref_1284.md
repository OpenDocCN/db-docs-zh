> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-memcached-txn.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-memcached-txn.html)

#### 17.20.6.4 控制 InnoDB memcached 插件的事务行为

与传统的**memcached**不同，`daemon_memcached`插件允许您控制通过`add`、`set`、`incr`等调用生成的数据值的耐久性。默认情况下，通过**memcached**接口写入的数据会存储到磁盘，并且调用`get`会从磁盘返回最新的值。尽管默认行为并不提供最佳的原始性能，但与`InnoDB`表的 SQL 接口相比，仍然快速。

随着您使用`daemon_memcached`插件的经验增加，您可以考虑放宽非关键数据类的耐久性设置，以便在发生故障时可能会丢失一些更新的值，或者返回略有过时的数据。

##### 提交频率

耐久性和原始性能之间的一个权衡是新数据和更改数据的提交频率。如果数据很关键，应立即提交，以便在意外退出或故障时安全。如果数据不那么关键，例如在意外退出后重置的计数器或您可以承受丢失的日志数据，您可能更喜欢更少的提交频率可用的更高原始吞吐量。

当**memcached**操作在底层`InnoDB`表中插入、更新或删除数据时，如果`daemon_memcached_w_batch_size=1`，更改可能会立即提交到`InnoDB`表中，或者稍后一段时间（如果`daemon_memcached_w_batch_size`的值大于 1）。在任何情况下，更改都无法回滚。如果您增加`daemon_memcached_w_batch_size`的值以避免在繁忙时期产生高 I/O 开销，当工作负载减少时，提交可能变得不频繁。作为一项安全措施，后台线程会定期自动提交通过**memcached** API 进行的更改。这个间隔由`innodb_api_bk_commit_interval`配置选项控制，默认设置为`5`秒。

当**memcached**操作在底层`InnoDB`表中插入或更新数据时，由于新值仍然保留在内存缓存中，即使尚未在 MySQL 端提交，其他**memcached**请求立即可以看到更改的数据。

##### 事务隔离

当 memcached 操作（如 `get` 或 `incr`）导致基础 `InnoDB` 表上的查询或 DML 操作时，您可以控制操作是否看到表中最新写入的数据，仅看到已提交的数据，或事务隔离级别的其他变化。使用 `innodb_api_trx_level` 配置选项来控制此功能。此选项指定的数字值对应于隔离级别，如 `REPEATABLE READ`。有关其他设置的信息，请参阅 `innodb_api_trx_level` 选项的描述。

严格的隔离级别确保您检索的数据不会被回滚或突然更改，导致后续查询返回不同的值。然而，严格的隔离级别需要更大的锁定开销，可能会导致等待。对于不使用长时间事务的 NoSQL 风格应用程序，通常可以使用默认隔离级别或切换到较不严格的隔禅级别。

##### 禁用 memcached 的 DML 操作的行锁。

当 **memcached** 请求通过 `daemon_memcached` 插件导致对 `InnoDB` 表的查询或 DML 操作时，可以使用 `innodb_api_disable_rowlock` 选项来禁用行锁。默认情况下，`innodb_api_disable_rowlock` 设置为 `OFF`，这意味着 **memcached** 请求行锁用于 `get` 和 `set` 操作。当 `innodb_api_disable_rowlock` 设置为 `ON` 时，**memcached** 请求表锁而不是行锁。

`innodb_api_disable_rowlock` 选项不是动态的。必须在启动时在 **mysqld** 命令行上指定，或在 MySQL 配置文件中输入。

##### 允许或禁止 DDL。

默认情况下，可以在 `daemon_memcached` 插件使用的表上执行 DDL 操作，如 `ALTER TABLE`。为避免这些表用于高吞吐量应用程序时潜在的减速，启用 `innodb_api_enable_mdl` 来禁用这些表上的 DDL 操作。当通过 **memcached** 和 SQL 访问相同的表时，此选项不太合适，因为它会阻止对表进行 `CREATE INDEX` 语句的操作，这对运行报表查询可能很重要。

##### 将数据存储在磁盘、内存中，或两者兼而有之。

`innodb_memcache.cache_policies` 表指定是否将通过 **memcached** 接口写入的数据存储到磁盘（`innodb_only`，默认）；仅存��在内存中，如传统的 **memcached**（`cache_only`）；或两者兼而有之（`caching`）。

使用`caching`设置，如果**memcached**在内存中找不到一个键，它会在`InnoDB`表中搜索该值。在`caching`设置下返回的`get`调用的值可能已经过时，如果这些值在`InnoDB`表中更新但尚未从内存缓存中过期。

缓存策略可以独立设置为`get`、`set`（包括`incr`和`decr`）、`delete`和`flush`操作。

例如，您可以允许`get`和`set`操作同时查询或更新表和**memcached**内存缓存（使用`caching`设置），同时使`delete`、`flush`或两者仅在内存副本上操作（使用`cache_only`设置）。这样，删除或刷新项目仅会使项目从缓存中过期，并且下次请求项目时将从`InnoDB`表中返回最新值。

```sql
mysql> SELECT * FROM innodb_memcache.cache_policies;
+--------------+-------------+-------------+---------------+--------------+
| policy_name  | get_policy  | set_policy  | delete_policy | flush_policy |
+--------------+-------------+-------------+---------------+--------------+
| cache_policy | innodb_only | innodb_only | innodb_only   | innodb_only  |
+--------------+-------------+-------------+---------------+--------------+

mysql> UPDATE innodb_memcache.cache_policies SET set_policy = 'caching'
       WHERE policy_name = 'cache_policy';
```

`innodb_memcache.cache_policies`的值只在启动时读取。在更改此表中的值后，卸载并重新安装`daemon_memcached`插件以确保更改生效。

```sql
mysql> UNINSTALL PLUGIN daemon_memcached;

mysql> INSTALL PLUGIN daemon_memcached soname "libmemcached.so";
```
