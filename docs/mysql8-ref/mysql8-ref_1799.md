> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-operations-per-fragment.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-operations-per-fragment.html)

#### 25.6.16.48 ndbinfo operations_per_fragment 表

`operations_per_fragment`表提供有关对各个片段和片段副本执行的操作以及一些操作结果的信息。

`operations_per_fragment`表包含以下列：

+   `fq_name`

    此片段的名称

+   `parent_fq_name`

    此片段的父级名称

+   `type`

    对象类型；请参阅文本以获取可能的值

+   `table_id`

    此表的表 ID

+   `node_id`

    此节点的节点 ID

+   `block_instance`

    内核块实例 ID

+   `fragment_num`

    片段 ID（数字）

+   `tot_key_reads`

    此片段副本的总键读取次数

+   `tot_key_inserts`

    此片段副本的总键插入次数

+   `tot_key_updates`

    此片段副本的总键更新次数

+   `tot_key_writes`

    此片段副本的总键写入次数

+   `tot_key_deletes`

    此片段副本的总键删除次数

+   `tot_key_refs`

    拒绝的键操作次数

+   `tot_key_attrinfo_bytes`

    所有`attrinfo`属性的总大小

+   `tot_key_keyinfo_bytes`

    所有`keyinfo`属性的总大小

+   `tot_key_prog_bytes`

    所有`attrinfo`属性携带的解释程序的总大小

+   `tot_key_inst_exec`

    由键操作的解释程序执行的指令总数

+   `tot_key_bytes_returned`

    从键读取操作返回的所有数据和元数据的总大小

+   `tot_frag_scans`

    此片段副本执行的扫描总数

+   `tot_scan_rows_examined`

    扫描检查的总行数

+   `tot_scan_rows_returned`

    返回给客户端的总行数

+   `tot_scan_bytes_returned`

    返回给客户端的数据和元数据的总大小

+   `tot_scan_prog_bytes`

    扫描操作的解释程序的总大小

+   `tot_scan_bound_bytes`

    有序索引扫描中使用的所有边界的总大小

+   `tot_scan_inst_exec`

    扫描执行的指令总数

+   `tot_qd_frag_scans`

    此片段副本扫描排队的次数

+   `conc_frag_scans`

    当前在此片段副本上活动的扫描数（不包括排队扫描）

+   `conc_qd_frag_scans`

    当前为此片段副本排队的扫描数

+   提交总数

    提交给此片段副本的行更改总数

##### 注意

`fq_name`包含此片段副本所属的模式对象的完全限定名称。目前具有以下格式：

+   基表：`*`DbName`*/def/*`TblName`*`

+   `BLOB`表：`*`DbName`*/def/NDB$BLOB_*`BaseTblId`*_*`ColNo`*`

+   有序索引：`sys/def/*`BaseTblId`*/*`IndexName`*`

+   唯一索引：`sys/def/*`BaseTblId`*/*`IndexName`*$unique`

唯一索引显示的`$unique`后缀是由**mysqld**添加的；对于由不同 NDB API 客户端应用程序创建的索引，可能会有所不同，或者不存在。

刚刚显示的完全限定对象名称的语法是一个内部接口，可能会在未来版本中更改。

考虑由以下 SQL 语句创建和修改的表`t1`：

```sql
CREATE DATABASE mydb;

USE mydb;

CREATE TABLE t1 (
  a INT NOT NULL,
  b INT NOT NULL,
  t TEXT NOT NULL,
  PRIMARY KEY (b)
) ENGINE=ndbcluster;

CREATE UNIQUE INDEX ix1 ON t1(b) USING HASH;
```

如果`t1`被分配表 ID 11，则这里显示的`fq_name`值为：

+   基本表：`mydb/def/t1`

+   `BLOB`表：`mydb/def/NDB$BLOB_11_2`

+   有序索引（主键）：`sys/def/11/PRIMARY`

+   唯一索引：`sys/def/11/ix1$unique`

对于索引或`BLOB`表，`parent_fq_name`列包含相应基本表的`fq_name`。对于基本表，此列始终为`NULL`。

`type`列显示了用于此片段的模式对象类型，可以取以下任一值：`System table`、`User table`、`Unique hash index`或`Ordered index`。`BLOB`表显示为`User table`。

`table_id`列值在任何给定时间都是唯一的，但如果相应对象已被删除，则可以重新使用。可以使用**ndb_show_tables**实用程序查看相同的 ID。

`block_instance`列显示此片段副本属于哪个 LDM 实例。您可以使用此信息从`threadblocks`表中获取有关特定线程的信息。第一个这样的实例始终编号为 0。

由于通常有两个片段副本，并假设是这样，每个`fragment_num`值应在表中出现两次，分别在来自同一节点组的两个不同数据节点上。

由于`NDB`不使用单键访问有序索引，因此有序索引操作不会增加`tot_key_reads`、`tot_key_inserts`、`tot_key_updates`、`tot_key_writes`和`tot_key_deletes`的计数。

注意

在使用`tot_key_writes`时，应注意在此上下文中，写操作会更新行（如果键存在），否则会插入新行。（这在`NDB`实现的`REPLACE` SQL 语句中有用途。）

`tot_key_refs`列显示 LDM 拒绝的键操作数量。通常，这种拒绝是由于重复键（插入）、未找到键错误（更新、删除和读取）或操作被用作谓词的解释程序拒绝的。

`tot_key_attrinfo_bytes` 和 `tot_key_keyinfo_bytes` 列计算的 `attrinfo` 和 `keyinfo` 属性是 `LQHKEYREQ` 信号的属性（参见 NDB 通信协议），用于由 LDM 发起的键操作。`attrinfo` 通常包含元组字段值（插入和更新）或投影规范（用于读取）；`keyinfo` 包含在此模式对象中定位给定元组所需的主键或唯一键。

`tot_frag_scans` 显示的值包括全扫描（检查每一行）和子集扫描。唯一索引和 `BLOB` 表永远不会被扫描，因此对于这些片段副本，该值以及其他与扫描相关的计数都为 0。

`tot_scan_rows_examined` 可能显示的行数少于给定片段副本中的总行数，因为有序索引扫描可能受到边界限制。此外，客户端可能选择在检查所有潜在匹配行之前结束扫描；例如，在使用包含 `LIMIT` 或 `EXISTS` 子句的 SQL 语句时会发生这种情况。`tot_scan_rows_returned` 总是小于或等于 `tot_scan_rows_examined`。

`tot_scan_bytes_returned` 包括在推送连接中返回给 NDB 内核中的 `DBSPJ` 块的投影。

`tot_qd_frag_scans` 可受到 `MaxParallelScansPerFragment` 数据节点配置参数的设置影响，该参数限制了在单个片段副本上可以同时执行的扫描数量。
