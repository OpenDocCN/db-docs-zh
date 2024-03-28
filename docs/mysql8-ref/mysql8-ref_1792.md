> [`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-locks-per-fragment.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-locks-per-fragment.html)

#### 25.6.16.41 The ndbinfo locks_per_fragment Table

`locks_per_fragment` 表提供了关于锁索赔请求计数以及这些请求的结果的信息，以每个片段为基础，作为 `operations_per_fragment` 和 `memory_per_fragment` 的伴随表。该表还显示了自片段或表创建以来成功和失败等待锁的总时间，或自最近一次重新启动以来的时间。

`locks_per_fragment` 表包含以下列：

+   `fq_name`

    完全限定表名

+   `parent_fq_name`

    父对象的完全限定名称

+   `type`

    表类型；请参阅文本以获取可能的值

+   `table_id`

    表 ID

+   `node_id`

    报告节点 ID

+   `block_instance`

    LDM 实例 ID

+   `fragment_num`

    片段标识符

+   `ex_req`

    排他锁请求已开始

+   `ex_imm_ok`

    立即授予的排他锁请求

+   `ex_wait_ok`

    等待后授予的排他锁请求

+   `ex_wait_fail`

    未授予的排他锁请求

+   `sh_req`

    已开始的共享锁请求

+   `sh_imm_ok`

    立即授予的共享锁请求

+   `sh_wait_ok`

    等待后授予的共享锁请求

+   `sh_wait_fail`

    未授予共享锁请求

+   `wait_ok_millis`

    等待授予的锁请求花费的时间，以毫秒为单位

+   `wait_fail_millis`

    等待失败的锁请求花费的时间，以毫秒为单位

##### 注意

`block_instance` 指的是内核块的一个实例。与块名���一起，此数字可用于在 `threadblocks` 表中查找给定实例。

`fq_name` 是以*`database`*/*`schema`*/*`name`*格式表示的完全限定数据库对象名称，例如 `test/def/t1` 或 `sys/def/10/b$unique`。

`parent_fq_name` 是此对象的父对象（表）的完全限定名称。

`table_id` 是由 `NDB` 生成的表的内部 ID。这是其他 `ndbinfo` 表中显示的相同内部表 ID；它也可在 **ndb_show_tables** 的输出中看到。

`type` 列显示表的类型。这始终是 `System table`、`User table`、`Unique hash index`、`Hash index`、`Unique ordered index`、`Ordered index`、`Hash index trigger`、`Subscription trigger`、`Read only constraint`、`Index trigger`、`Reorganize trigger`、`Tablespace`、`Log file group`、`Data file`、`Undo file`、`Hash map`、`Foreign key definition`、`Foreign key parent trigger`、`Foreign key child trigger` 或 `Schema transaction` 中的一个。

所有列`ex_req`、`ex_req_imm_ok`、`ex_wait_ok`、`ex_wait_fail`、`sh_req`、`sh_req_imm_ok`、`sh_wait_ok`和`sh_wait_fail`中显示的值代表自表或片段创建以来的累积请求次数，或自此节点上次重新启动以来的时间，以后者为准。对于`wait_ok_millis`和`wait_fail_millis`列中显示的时间值也是如此。

每个锁请求被认为是正在进行中，或以某种方式已经完成（即成功或失败）。这意味着以下关系是正确的：

```sql
ex_req >= (ex_req_imm_ok + ex_wait_ok + ex_wait_fail)

sh_req >= (sh_req_imm_ok + sh_wait_ok + sh_wait_fail)
```

当前正在进行的请求数量是当前未完成请求的数量，可以按照以下所示找到：

```sql
[exclusive lock requests in progress] =
    ex_req - (ex_req_imm_ok + ex_wait_ok + ex_wait_fail)

[shared lock requests in progress] =
    sh_req - (sh_req_imm_ok + sh_wait_ok + sh_wait_fail)
```

失败的等待表示一个中止的事务，但中止可能或可能不是由于锁等待超时引起的。您可以按照以下所示获取等待锁时中止的总数：

```sql
[aborts while waiting for locks] = ex_wait_fail + sh_wait_fail
```
