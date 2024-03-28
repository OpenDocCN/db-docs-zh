> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-buffer-pool-resize.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-buffer-pool-resize.html)

#### 17.8.3.1 配置 InnoDB 缓冲池大小

您可以在离线或服务器运行时配置`InnoDB`缓冲池大小。本节描述的行为适用于两种方法。有关在线配置缓冲池大小的更多信息，请参见在线配置 InnoDB 缓冲池大小。

当增加或减少`innodb_buffer_pool_size`时，操作是以块为单位进行的。块大小由`innodb_buffer_pool_chunk_size`配置选项定义，默认值为`128M`。有关更多信息，请参见配置 InnoDB 缓冲池块大小。

缓冲池大小必须始终等于或是`innodb_buffer_pool_chunk_size` * `innodb_buffer_pool_instances`的倍数。如果您将`innodb_buffer_pool_size`配置为不等于或不是`innodb_buffer_pool_chunk_size` * `innodb_buffer_pool_instances`的倍数的值，缓冲池大小将自动调整为等于或是`innodb_buffer_pool_chunk_size` * `innodb_buffer_pool_instances`的倍数的值。

在以下示例中，`innodb_buffer_pool_size`设置为`8G`，`innodb_buffer_pool_instances`设置为`16`。`innodb_buffer_pool_chunk_size`为`128M`，这是默认值。

`8G`是一个有效的`innodb_buffer_pool_size`值，因为`8G`是`innodb_buffer_pool_instances=16` * `innodb_buffer_pool_chunk_size=128M`的倍数，即`2G`。

```sql
$> mysqld --innodb-buffer-pool-size=8G --innodb-buffer-pool-instances=16
```

```sql
mysql> SELECT @@innodb_buffer_pool_size/1024/1024/1024;
+------------------------------------------+
| @@innodb_buffer_pool_size/1024/1024/1024 |
+------------------------------------------+
|                           8.000000000000 |
+------------------------------------------+
```

在这个例子中，`innodb_buffer_pool_size` 设置为 `9G`，而 `innodb_buffer_pool_instances` 设置为 `16`。`innodb_buffer_pool_chunk_size` 是 `128M`，这是默认值。在这种情况下，`9G` 不是 `innodb_buffer_pool_instances=16` * `innodb_buffer_pool_chunk_size=128M` 的倍数，因此 `innodb_buffer_pool_size` 被调整为 `10G`，这是 `innodb_buffer_pool_chunk_size` * `innodb_buffer_pool_instances` 的倍数。

```sql
$> mysqld --innodb-buffer-pool-size=9G --innodb-buffer-pool-instances=16
```

```sql
mysql> SELECT @@innodb_buffer_pool_size/1024/1024/1024;
+------------------------------------------+
| @@innodb_buffer_pool_size/1024/1024/1024 |
+------------------------------------------+
|                          10.000000000000 |
+------------------------------------------+
```

##### 配置 InnoDB 缓冲池块大小

`innodb_buffer_pool_chunk_size` 可以以 1MB（1048576 字节）为单位增加或减少，但只能在启动时修改，在命令行字符串或 MySQL 配置文件中。

命令行：

```sql
$> mysqld --innodb-buffer-pool-chunk-size=134217728
```

配置文件：

```sql
[mysqld]
innodb_buffer_pool_chunk_size=134217728
```

当修改 `innodb_buffer_pool_chunk_size` 时，以下条件适用：

+   如果新的 `innodb_buffer_pool_chunk_size` 值 * `innodb_buffer_pool_instances` 大于当前缓冲池大小在缓冲池初始化时，`innodb_buffer_pool_chunk_size` 被截断为 `innodb_buffer_pool_size` / `innodb_buffer_pool_instances`。

    例如，如果缓冲池初始化大小为 `2GB`（2147483648 字节），有 `4` 个缓冲池实例，以及 `1GB`（1073741824 字节）的块大小，块大小被截断为等于 `innodb_buffer_pool_size` / `innodb_buffer_pool_instances` 的值，如下所示：

    ```sql
    $> mysqld --innodb-buffer-pool-size=2147483648 --innodb-buffer-pool-instances=4
    --innodb-buffer-pool-chunk-size=1073741824;
    ```

    ```sql
    mysql> SELECT @@innodb_buffer_pool_size;
    +---------------------------+
    | @@innodb_buffer_pool_size |
    +---------------------------+
    |                2147483648 |
    +---------------------------+

    mysql> SELECT @@innodb_buffer_pool_instances;
    +--------------------------------+
    | @@innodb_buffer_pool_instances |
    +--------------------------------+
    |                              4 |
    +--------------------------------+

    # Chunk size was set to 1GB (1073741824 bytes) on startup but was
    # truncated to innodb_buffer_pool_size / innodb_buffer_pool_instances

    mysql> SELECT @@innodb_buffer_pool_chunk_size;
    +---------------------------------+
    | @@innodb_buffer_pool_chunk_size |
    +---------------------------------+
    |                       536870912 |
    +---------------------------------+
    ```

+   缓冲池大小必须始终等于或是`innodb_buffer_pool_chunk_size` * `innodb_buffer_pool_instances`的倍数。如果您更改`innodb_buffer_pool_chunk_size`，`innodb_buffer_pool_size`会自动调整为等于或是`innodb_buffer_pool_chunk_size` * `innodb_buffer_pool_instances`的值。此调整发生在缓冲池初始化时。以下示例演示了这种行为：

    ```sql
    # The buffer pool has a default size of 128MB (134217728 bytes)

    mysql> SELECT @@innodb_buffer_pool_size;
    +---------------------------+
    | @@innodb_buffer_pool_size |
    +---------------------------+
    |                 134217728 |
    +---------------------------+

    # The chunk size is also 128MB (134217728 bytes)

    mysql> SELECT @@innodb_buffer_pool_chunk_size;
    +---------------------------------+
    | @@innodb_buffer_pool_chunk_size |
    +---------------------------------+
    |                       134217728 |
    +---------------------------------+

    # There is a single buffer pool instance

    mysql> SELECT @@innodb_buffer_pool_instances;
    +--------------------------------+
    | @@innodb_buffer_pool_instances |
    +--------------------------------+
    |                              1 |
    +--------------------------------+

    # Chunk size is decreased by 1MB (1048576 bytes) at startup
    # (134217728 - 1048576 = 133169152):

    $> mysqld --innodb-buffer-pool-chunk-size=133169152

    mysql> SELECT @@innodb_buffer_pool_chunk_size;
    +---------------------------------+
    | @@innodb_buffer_pool_chunk_size |
    +---------------------------------+
    |                       133169152 |
    +---------------------------------+

    # Buffer pool size increases from 134217728 to 266338304
    # Buffer pool size is automatically adjusted to a value that is equal to
    # or a multiple of innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances

    mysql> SELECT @@innodb_buffer_pool_size;
    +---------------------------+
    | @@innodb_buffer_pool_size |
    +---------------------------+
    |                 266338304 |
    +---------------------------+
    ```

    该示例演示了具有多个缓冲池实例的相同行为：

    ```sql
    # The buffer pool has a default size of 2GB (2147483648 bytes)

    mysql> SELECT @@innodb_buffer_pool_size;
    +---------------------------+
    | @@innodb_buffer_pool_size |
    +---------------------------+
    |                2147483648 |
    +---------------------------+

    # The chunk size is .5 GB (536870912 bytes)

    mysql> SELECT @@innodb_buffer_pool_chunk_size;
    +---------------------------------+
    | @@innodb_buffer_pool_chunk_size |
    +---------------------------------+
    |                       536870912 |
    +---------------------------------+

    # There are 4 buffer pool instances

    mysql> SELECT @@innodb_buffer_pool_instances;
    +--------------------------------+
    | @@innodb_buffer_pool_instances |
    +--------------------------------+
    |                              4 |
    +--------------------------------+

    # Chunk size is decreased by 1MB (1048576 bytes) at startup
    # (536870912 - 1048576 = 535822336):

    $> mysqld --innodb-buffer-pool-chunk-size=535822336

    mysql> SELECT @@innodb_buffer_pool_chunk_size;
    +---------------------------------+
    | @@innodb_buffer_pool_chunk_size |
    +---------------------------------+
    |                       535822336 |
    +---------------------------------+

    # Buffer pool size increases from 2147483648 to 4286578688
    # Buffer pool size is automatically adjusted to a value that is equal to
    # or a multiple of innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances

    mysql> SELECT @@innodb_buffer_pool_size;
    +---------------------------+
    | @@innodb_buffer_pool_size |
    +---------------------------+
    |                4286578688 |
    +---------------------------+
    ```

    当更改`innodb_buffer_pool_chunk_size`时需要小心，因为更改此值可能会增加缓冲池的大小，如上面的示例所示。在更改`innodb_buffer_pool_chunk_size`之前，请计算对`innodb_buffer_pool_size`的影响，以确保最终的缓冲池大小是可接受的。

注意

为避免潜在的性能问题，块数（`innodb_buffer_pool_size` / `innodb_buffer_pool_chunk_size`）不应超过 1000。

##### 在线配置 InnoDB 缓冲池大小

`innodb_buffer_pool_size`配置选项可以使用`SET`语句动态设置，允许您调整缓冲池的大小而无需重新启动服务器。例如：

```sql
mysql> SET GLOBAL innodb_buffer_pool_size=402653184;
```

注意

缓冲池大小必须等于或是`innodb_buffer_pool_chunk_size` * `innodb_buffer_pool_instances`的倍数。更改这些变量设置需要重新启动服务器。

在调整缓冲池大小之前，应完成通过 `InnoDB` API 执行的活动事务和操作。在启动调整操作时，操作直到所有活动事务完成后才开始。一旦调整操作正在进行中，需要访问缓冲池的新事务和操作必须等待调整操作完成。唯一的例外是，在减小缓冲池大小时，允许并发访问缓冲池，当缓冲池碎片整理并在减小缓冲池大小时撤回页面时。允许并发访问的缺点是，在页面被撤回时可能导致可用页面的暂时短缺。

注意

如果在缓冲池调整操作开始后启动嵌套事务，可能会失败。

##### 监视在线缓冲池调整进度

`Innodb_buffer_pool_resize_status` 变量报告一个字符串值，指示缓冲池调整的进度；例如：

```sql
mysql> SHOW STATUS WHERE Variable_name='InnoDB_buffer_pool_resize_status';
+----------------------------------+----------------------------------+
| Variable_name                    | Value                            |
+----------------------------------+----------------------------------+
| Innodb_buffer_pool_resize_status | Resizing also other hash tables. |
+----------------------------------+----------------------------------+
```

从 MyQL 8.0.31 开始，您还可以使用 `Innodb_buffer_pool_resize_status_code` 和 `Innodb_buffer_pool_resize_status_progress` 状态变量监视在线缓冲池调整操作，这些变量报告数值，适合程序化监控。

`Innodb_buffer_pool_resize_status_code` 状态变量报告一个状态码，指示在线缓冲池调整操作的阶段。状态码包括：

+   0: 没有正在进行的调整操作

+   1: 开始调整大小

+   2: 禁用 AHI（自适应哈希索引）

+   3: 撤回块

+   4: 获取全局锁

+   5: 调整池

+   6: 调整哈希

+   7: 调整失败

`Innodb_buffer_pool_resize_status_progress` 状态变量报告一个百分比值，指示每个阶段的进度。百分比值在处理每个缓冲池实例后更新。当状态（由 `Innodb_buffer_pool_resize_status_code` 报告）从一个状态变为另一个状态时，百分比值将重置为 0。

以下查询返回一个字符串值，指示缓冲池调整的进度，一个代码，指示操作的当前阶段，以及该阶段的当前进度，表示为百分比值：

```sql
SELECT variable_name, variable_value 
 FROM performance_schema.global_status 
 WHERE LOWER(variable_name) LIKE "innodb_buffer_pool_resize%";
```

缓冲池调整进度也可以在服务器错误日志中看到。此示例显示了在增加缓冲池大小时记录的注释：

```sql
[Note] InnoDB: Resizing buffer pool from 134217728 to 4294967296. (unit=134217728)
[Note] InnoDB: disabled adaptive hash index.
[Note] InnoDB: buffer pool 0 : 31 chunks (253952 blocks) was added.
[Note] InnoDB: buffer pool 0 : hash tables were resized.
[Note] InnoDB: Resized hash tables at lock_sys, adaptive hash index, dictionary.
[Note] InnoDB: completed to resize buffer pool from 134217728 to 4294967296.
[Note] InnoDB: re-enabled adaptive hash index.
```

此示例显示了在减小缓冲池大小时记录的注释：

```sql
[Note] InnoDB: Resizing buffer pool from 4294967296 to 134217728. (unit=134217728)
[Note] InnoDB: disabled adaptive hash index.
[Note] InnoDB: buffer pool 0 : start to withdraw the last 253952 blocks.
[Note] InnoDB: buffer pool 0 : withdrew 253952 blocks from free list. tried to relocate 
0 pages. (253952/253952)
[Note] InnoDB: buffer pool 0 : withdrawn target 253952 blocks.
[Note] InnoDB: buffer pool 0 : 31 chunks (253952 blocks) was freed.
[Note] InnoDB: buffer pool 0 : hash tables were resized.
[Note] InnoDB: Resized hash tables at lock_sys, adaptive hash index, dictionary.
[Note] InnoDB: completed to resize buffer pool from 4294967296 to 134217728.
[Note] InnoDB: re-enabled adaptive hash index.
```

从 MySQL 8.0.31 开始，使用`--log-error-verbosity=3`启动服务器时，在在线缓冲池调整操作期间向错误日志记录额外信息。额外信息包括由`Innodb_buffer_pool_resize_status_code`报告的状态代码和由`Innodb_buffer_pool_resize_status_progress`报告的百分比进度值。

```sql
[Note] [MY-012398] [InnoDB] Requested to resize buffer pool. (new size: 1073741824 bytes)
[Note] [MY-013954] [InnoDB] Status code 1: Resizing buffer pool from 134217728 to 1073741824
(unit=134217728).
[Note] [MY-013953] [InnoDB] Status code 1: 100% complete
[Note] [MY-013952] [InnoDB] Status code 1: Completed
[Note] [MY-013954] [InnoDB] Status code 2: Disabling adaptive hash index.
[Note] [MY-011885] [InnoDB] disabled adaptive hash index.
[Note] [MY-013953] [InnoDB] Status code 2: 100% complete
[Note] [MY-013952] [InnoDB] Status code 2: Completed
[Note] [MY-013954] [InnoDB] Status code 3: Withdrawing blocks to be shrunken.
[Note] [MY-013953] [InnoDB] Status code 3: 100% complete
[Note] [MY-013952] [InnoDB] Status code 3: Completed
[Note] [MY-013954] [InnoDB] Status code 4: Latching whole of buffer pool.
[Note] [MY-013953] [InnoDB] Status code 4: 14% complete
[Note] [MY-013953] [InnoDB] Status code 4: 28% complete
[Note] [MY-013953] [InnoDB] Status code 4: 42% complete
[Note] [MY-013953] [InnoDB] Status code 4: 57% complete
[Note] [MY-013953] [InnoDB] Status code 4: 71% complete
[Note] [MY-013953] [InnoDB] Status code 4: 85% complete
[Note] [MY-013953] [InnoDB] Status code 4: 100% complete
[Note] [MY-013952] [InnoDB] Status code 4: Completed
[Note] [MY-013954] [InnoDB] Status code 5: Starting pool resize
[Note] [MY-013954] [InnoDB] Status code 5: buffer pool 0 : resizing with chunks 1 to 8.
[Note] [MY-011891] [InnoDB] buffer pool 0 : 7 chunks (57339 blocks) were added.
[Note] [MY-013953] [InnoDB] Status code 5: 100% complete
[Note] [MY-013952] [InnoDB] Status code 5: Completed
[Note] [MY-013954] [InnoDB] Status code 6: Resizing hash tables.
[Note] [MY-011892] [InnoDB] buffer pool 0 : hash tables were resized.
[Note] [MY-013953] [InnoDB] Status code 6: 100% complete
[Note] [MY-013954] [InnoDB] Status code 6: Resizing also other hash tables.
[Note] [MY-011893] [InnoDB] Resized hash tables at lock_sys, adaptive hash index, dictionary.
[Note] [MY-011894] [InnoDB] Completed to resize buffer pool from 134217728 to 1073741824.
[Note] [MY-011895] [InnoDB] Re-enabled adaptive hash index.
[Note] [MY-013952] [InnoDB] Status code 6: Completed
[Note] [MY-013954] [InnoDB] Status code 0: Completed resizing buffer pool at 220826  6:25:46.
[Note] [MY-013953] [InnoDB] Status code 0: 100% complete
```

##### 在线缓冲池调整内部机制

调整操作由后台线程执行。当增加缓冲池大小时，调整操作：

+   按`块`（块大小由`innodb_buffer_pool_chunk_size`定义）添加页面

+   将哈希表、列表和指针转换为在内存中使用新地址

+   将新页面添加到空闲列表中

在进行这些操作时，其他线程被阻止访问缓冲池。

当缩小缓冲池大小时，调整操作：

+   对缓冲池进行碎片整理并撤回（释放）页面

+   按`块`（块大小由`innodb_buffer_pool_chunk_size`定义）移除页面

+   将哈希表、列表和指针转换为在内存中使用新地址

在这些操作中，只有对缓冲池进行碎片整理和撤回页面允许其他线程同时访问缓冲池。
