> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-memory-per-fragment.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-memory-per-fragment.html)

#### 25.6.16.46 ndbinfo memory_per_fragment 表

+   memory_per_fragment 表：注释

+   memory_per_fragment 表：示例

`memory_per_fragment` 表提供有关单个片段内存使用情况的信息。请参阅本节后面的 注释 以了解如何使用此信息查找 `NDB` 表使用了多少内存。

`memory_per_fragment` 表包含以下列：

+   `fq_name`

    此片段的名称

+   `parent_fq_name`

    此片段的父级名称

+   `type`

    用于此片段的字典对象类型（`Object::Type`，在 NDB API 中）之一：`System table`、`User table`、`Unique hash index`、`Hash index`、`Unique ordered index`、`Ordered index`、`Hash index trigger`、`Subscription trigger`、`Read only constraint`、`Index trigger`、`Reorganize trigger`、`Tablespace`、`Log file group`、`Data file`、`Undo file`、`Hash map`、`Foreign key definition`、`Foreign key parent trigger`、`Foreign key child trigger` 或 `Schema transaction`。

    您还可以通过在 **mysql** 客户端中执行 `TABLE` `ndbinfo.dict_obj_types` 来获取此列表。

+   `table_id`

    此表的表 ID

+   `node_id`

    此节点的节点 ID

+   `block_instance`

    NDB 内核块实例 ID；您可以使用此数字从 `threadblocks` 表中获取有关特定线程的信息。

+   `fragment_num`

    片段 ID（数字）

+   `fixed_elem_alloc_bytes`

    为固定大小元素分配的字节数

+   `fixed_elem_free_bytes`

    分配给固定大小元素的页面中剩余的空闲字节数

+   `fixed_elem_size_bytes`

    每个固定大小元素的字节长度

+   `fixed_elem_count`

    固定大小元素的数量

+   `fixed_elem_free_count`

    用于固定大小元素的空闲行数

+   `var_elem_alloc_bytes`

    为可变大小元素分配的字节数

+   `var_elem_free_bytes`

    分配给可变大小元素的页面中剩余的空闲字节数

+   `var_elem_count`

    可变大小元素的数量

+   `hash_index_alloc_bytes`

    为哈希索引分配的字节数

##### memory_per_fragment 表：注释

`memory_per_fragment` 表包含系统中每个表片段副本和每个索引片段副本的一行；这意味着，例如，当`NoOfReplicas=2`时，通常每个片段都有两个片段副本。只要所有数据节点都在运行并连接到集群，这一点就成立；对于缺失的数据节点，它承载的片段副本没有行。

`memory_per_fragment` 表的列可以根据其功能或目的分组如下：

+   *键列*：`fq_name`、`type`、`table_id`、`node_id`、`block_instance`和`fragment_num`

+   *关系列*：`parent_fq_name`

+   *固定大小存储列*：`fixed_elem_alloc_bytes`、`fixed_elem_free_bytes`、`fixed_elem_size_bytes`、`fixed_elem_count`和`fixed_elem_free_count`

+   *可变大小存储列*：`var_elem_alloc_bytes`、`var_elem_free_bytes`和`var_elem_count`

+   *哈希索引列*：`hash_index_alloc_bytes`

`parent_fq_name` 和 `fq_name` 列可用于识别与表关联的索引。类似的模式对象层次结构信息在其他`ndbinfo`表中也可用。

表和索引片段副本以 32KB 页面分配`DataMemory`。这些内存页面的管理如下所示：

+   *固定大小页*：这些存储在给定片段中存储的行的固定大小部分。每行都有一个固定大小部分。

+   *可变大小页*：这些为片段中的行存储可变大小部分。每行具有一个或多个可变大小部分，一个或多个动态列（或两者都有）的行具有可变大小部分。

+   *哈希索引页*：这些被分配为 8 KB 子页，存储主键哈希索引结构。

`NDB`表中的每行都有一个固定大小部分，包括行头和一个或多个固定大小列。该行还可能包含一个或多个变量大小部分引用，一个或多个磁盘部分引用，或两者都有。每行还有一个主键哈希索引条目（对应于每个`NDB`表中的隐藏主键）。

从前述内容我们可以看到，每个表片段和索引片段一起分配了根据此处计算的`DataMemory`数量：

```sql
DataMemory =
  (*number_of_fixed_pages* + *number_of_var_pages*) * 32KB
    + *number_of_hash_pages* * 8KB
```

由于`fixed_elem_alloc_bytes`和`var_elem_alloc_bytes`始终是 32768 字节的倍数，我们可以进一步确定`*`number_of_fixed_pages`* = fixed_elem_alloc_bytes / 32768`和`*`number_of_var_pages`* = var_elem_alloc_bytes / 32768`。`hash_index_alloc_bytes`始终是 8192 字节的倍数，因此`*`number_of_hash_pages`* = hash_index_alloc_bytes / 8192`。

固定大小页具有内部标头和多个固定大小槽，每个槽可以包含一行的固定大小部分。给定行的固定大小部分的大小取决于模式，并由`fixed_elem_size_bytes`列提供；每页的固定大小槽数量可以通过计算总槽数量和总页数来确定，如下所示：

```sql
*fixed_slots* = fixed_elem_count + fixed_elem_free_count

*fixed_pages* = fixed_elem_alloc_bytes / 32768

*slots_per_page* = total_slots / total_pages
```

`fixed_elem_count`实际上是给定表碎片的行数，因为每行有 1 个固定元素，`fixed_elem_free_count`是分配的页中所有固定大小空槽的总数。`fixed_elem_free_bytes`等于`fixed_elem_free_count * fixed_elem_size_bytes`。

一个碎片可以有任意数量的固定大小页；当固定大小页上的最后一行被删除时，该页将释放到`DataMemory`页池中。固定大小页可以被碎片化，分配的页比实际使用的固定大小槽数量多。您可以通过比较所需页数和分配的页数来检查是否是这种情况，计算方法如下：

```sql
*fixed_pages_required* = 1 + (fixed_elem_count / *slots_per_page*)

fixed_page_utilization = *fixed_pages_required* / *fixed_pages*
```

可变大小页具有内部标头，并使用剩余空间存储一个或多个可变大小行部分；存储的部分数量取决于模式和实际存储的数据。由于并非所有模式或行都有可变大小部分，`var_elem_count`可能小于`fixed_elem_count`。碎片中所有可变大小页上的总可用空间由`var_elem_free_bytes`列显示；因为此空间可能分布在多个页上，所以不能必然用于存储特定大小的条目。根据需要重新组织每个可变大小页，以适应插入、更新和删除可变大小行部分的变化大小；如果给定行部分增长到超出所在页的大小，则可以将其移动到另一页。

可变大小页的利用率可以按照这里所示进行计算：

```sql
*var_page_used_bytes* =  var_elem_alloc_bytes - var_elem_free_bytes

*var_page_utilisation* = var_page_used_bytes / var_elem_alloc_bytes

*avg_row_var_part_size* = *var_page_used_bytes* / fixed_elem_count
```

我们可以按照以下方式获得每行的平均可变部分大小：

```sql
*avg_row_var_part_size* = *var_page_used_bytes* / fixed_elem_count
```

次要唯一索引在内部实现为具有以下模式的独立表：

+   *主键*：基表中的索引列。

+   *值*：基表中的主键列。

这些表像普通表一样分布和碎片化。这意味着它们的碎片副本使用固定、可变和哈希索引页，就像任何其他`NDB`表一样。

次序有序索引与基表相同方式进行碎片化和分布。有序索引片段是维护平衡树的 T 树结构，其中包含按索引列暗示的顺序包含行引用。由于树包含引用而不是实际数据，因此 T 树存储成本不取决于索引列的大小或数量，而是取决于行数。树是使用固定大小节点结构构建的，每个节点可以包含多个行引用；所需的节点数量取决于表中的行数，以及表示排序所需的树结构。在`memory_per_fragment`表中，我们可以看到有序索引仅分配固定大小页面，因此像往常一样，此表中相关列如下所列：

+   `fixed_elem_alloc_bytes`: 这等于固定大小页面的数量乘以 32768。

+   `fixed_elem_count`: 正在使用的 T 树节点数。

+   `fixed_elem_size_bytes`: 每个 T 树节点的字节数。

+   `fixed_elem_free_count`: 分配的页面中可用的 T 树节点槽位数。

+   `fixed_elem_free_bytes`: 这等于`fixed_elem_free_count * fixed_elem_size_bytes`。

如果页面中的空闲空间是碎片化的，则页面将被整理。`OPTIMIZE TABLE`可用于整理表的可变大小页面；这将在页面之间移动行的可变大小部分，以便一些整个页面可以被释放以供重新使用。

##### memory_per_fragment 表：示例

+   获取有关片段和内存使用情况的一般信息

+   查找表及其索引

+   查找模式元素分配的内存

+   查找表及其所有索引分配的内存

+   查找每行分配的内存

+   查找每行使用的总内存

+   查找每个元素分配的内存

+   按元素查找每行分配的平均内存

+   查找每行分配的平均内存

+   查找表格每行分配的平均内存

+   按每个模式元素查找使用的内存

+   按每个模式元素查找使用的平均内存

+   按元素查找每行使用的平均内存

+   查找每行使用的总平均内存

对于以下示例，我们创建一个简单的表格，包含三个整数列，其中一个具有主键，一个具有唯一索引，一个没有索引，以及一个没有索引的`VARCHAR`列，如下所示：

```sql
mysql> CREATE DATABASE IF NOT EXISTS test;
Query OK, 1 row affected (0.06 sec)

mysql> USE test;
Database changed

mysql> CREATE TABLE t1 (
 ->    c1 BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
 ->    c2 INT,
 ->    c3 INT UNIQUE,
 -> )  ENGINE=NDBCLUSTER;
Query OK, 0 rows affected (0.27 sec)
```

创建表格后，我们插入包含随机数据的 50,000 行；生成和插入这些行的确切方法在实际上没有任何区别，我们将完成方法的实现留给用户作为练习。

###### 获取片段和内存使用的一般信息

此查询显示每个片段的内存使用情况的一般信息：

```sql
mysql> SELECT
 ->   fq_name, node_id, block_instance, fragment_num, fixed_elem_alloc_bytes,
 ->   fixed_elem_free_bytes, fixed_elem_size_bytes, fixed_elem_count,
 ->   fixed_elem_free_count, var_elem_alloc_bytes, var_elem_free_bytes,
 ->   var_elem_count
 -> FROM ndbinfo.memory_per_fragment
 -> WHERE fq_name = "test/def/t1"\G
*************************** 1\. row ***************************
               fq_name: test/def/t1
               node_id: 5
        block_instance: 1
          fragment_num: 0
fixed_elem_alloc_bytes: 1114112
 fixed_elem_free_bytes: 11836
 fixed_elem_size_bytes: 44
      fixed_elem_count: 24925
 fixed_elem_free_count: 269
  var_elem_alloc_bytes: 1245184
   var_elem_free_bytes: 32552
        var_elem_count: 24925
*************************** 2\. row ***************************
               fq_name: test/def/t1
               node_id: 5
        block_instance: 1
          fragment_num: 1
fixed_elem_alloc_bytes: 1114112
 fixed_elem_free_bytes: 5236
 fixed_elem_size_bytes: 44
      fixed_elem_count: 25075
 fixed_elem_free_count: 119
  var_elem_alloc_bytes: 1277952
   var_elem_free_bytes: 54232
        var_elem_count: 25075
*************************** 3\. row ***************************
               fq_name: test/def/t1
               node_id: 6
        block_instance: 1
          fragment_num: 0
fixed_elem_alloc_bytes: 1114112
 fixed_elem_free_bytes: 11836
 fixed_elem_size_bytes: 44
      fixed_elem_count: 24925
 fixed_elem_free_count: 269
  var_elem_alloc_bytes: 1245184
   var_elem_free_bytes: 32552
        var_elem_count: 24925
*************************** 4\. row ***************************
               fq_name: test/def/t1
               node_id: 6
        block_instance: 1
          fragment_num: 1
fixed_elem_alloc_bytes: 1114112
 fixed_elem_free_bytes: 5236
 fixed_elem_size_bytes: 44
      fixed_elem_count: 25075
 fixed_elem_free_count: 119
  var_elem_alloc_bytes: 1277952
   var_elem_free_bytes: 54232
        var_elem_count: 25075 4 rows in set (0.12 sec)
```

###### 查找表格及其索引

此查询可用于查找特定表格及其索引：

```sql
mysql> SELECT fq_name
 -> FROM ndbinfo.memory_per_fragment
 -> WHERE fq_name = 'test/def/t1' OR parent_fq_name='test/def/t1'
 -> GROUP BY fq_name;
+----------------------+
| fq_name              |
+----------------------+
| test/def/t1          |
| sys/def/13/PRIMARY   |
| sys/def/13/c3        |
| sys/def/13/c3$unique |
+----------------------+
4 rows in set (0.13 sec)

mysql> SELECT COUNT(*) FROM t1;
+----------+
| COUNT(*) |
+----------+
|    50000 |
+----------+
1 row in set (0.00 sec)
```

###### 查找模式元素分配的内存

此查询显示每个模式元素分配的内存（在所有副本中总计）：

```sql
mysql> SELECT
 ->   fq_name AS Name,
 ->   SUM(fixed_elem_alloc_bytes) AS Fixed,
 ->   SUM(var_elem_alloc_bytes) AS Var,
 ->   SUM(hash_index_alloc_bytes) AS Hash,
 ->   SUM(fixed_elem_alloc_bytes+var_elem_alloc_bytes+hash_index_alloc_bytes) AS Total
 -> FROM ndbinfo.memory_per_fragment
 -> WHERE fq_name = 'test/def/t1' OR parent_fq_name='test/def/t1'
 -> GROUP BY fq_name;
+----------------------+---------+---------+---------+----------+
| Name                 | Fixed   | Var     | Hash    | Total    |
+----------------------+---------+---------+---------+----------+
| test/def/t1          | 4456448 | 5046272 | 1425408 | 10928128 |
| sys/def/13/PRIMARY   | 1966080 |       0 |       0 |  1966080 |
| sys/def/13/c3        | 1441792 |       0 |       0 |  1441792 |
| sys/def/13/c3$unique | 3276800 |       0 | 1425408 |  4702208 |
+----------------------+---------+---------+---------+----------+
4 rows in set (0.11 sec)
```

###### 查找为表格及所有索引分配的内存

可以使用以下查询获取为表格及其所有索引分配的内存总和（在所有副本中总计）：

```sql
mysql> SELECT
 ->   SUM(fixed_elem_alloc_bytes) AS Fixed,
 ->   SUM(var_elem_alloc_bytes) AS Var,
 ->   SUM(hash_index_alloc_bytes) AS Hash,
 ->   SUM(fixed_elem_alloc_bytes+var_elem_alloc_bytes+hash_index_alloc_bytes) AS Total
 -> FROM ndbinfo.memory_per_fragment
 -> WHERE fq_name = 'test/def/t1' OR parent_fq_name='test/def/t1';
+----------+---------+---------+----------+
| Fixed    | Var     | Hash    | Total    |
+----------+---------+---------+----------+
| 11141120 | 5046272 | 2850816 | 19038208 |
+----------+---------+---------+----------+
1 row in set (0.12 sec)
```

这是前一个查询的简化版本，仅显示表格使用的总内存：

```sql
mysql> SELECT
 ->   SUM(fixed_elem_alloc_bytes+var_elem_alloc_bytes+hash_index_alloc_bytes) AS Total
 -> FROM ndbinfo.memory_per_fragment
 -> WHERE fq_name = 'test/def/t1' OR parent_fq_name='test/def/t1';
+----------+
| Total    |
+----------+
| 19038208 |
+----------+
1 row in set (0.12 sec)
```

###### 查找每行分配的内存

以下查询显示每行分配的总内存（在所有副本中）：

```sql
mysql> SELECT
 ->   SUM(fixed_elem_alloc_bytes+var_elem_alloc_bytes+hash_index_alloc_bytes)
 ->   /
 ->   SUM(fixed_elem_count) AS Total_alloc_per_row
 -> FROM ndbinfo.memory_per_fragment
 -> WHERE fq_name = 'test/def/t1';
+---------------------+
| Total_alloc_per_row |
+---------------------+
|            109.2813 |
+---------------------+
1 row in set (0.12 sec)
```

###### 查找每行使用的总内存

要获取每行使用的总内存（在所有副本中），我们需要将总内存使用量除以行数，即基本表的`fixed_elem_count`，如下所示：

```sql
mysql> SELECT
 ->   SUM(
 ->     (fixed_elem_alloc_bytes - fixed_elem_free_bytes)
 ->     + (var_elem_alloc_bytes - var_elem_free_bytes)
 ->     + hash_index_alloc_bytes
 ->   )
 ->   /
 ->   SUM(fixed_elem_count)
 ->   AS total_in_use_per_row
 -> FROM ndbinfo.memory_per_fragment
 -> WHERE fq_name = 'test/def/t1';
+----------------------+
| total_in_use_per_row |
+----------------------+
|             107.2042 |
+----------------------+
1 row in set (0.12 sec)
```

###### 查找每个元素分配的内存

每个模式元素分配的内存（在所有副本中总共）可以使用以下查询找到：

```sql
mysql> SELECT
 ->   fq_name AS Name,
 ->   SUM(fixed_elem_alloc_bytes) AS Fixed,
 ->   SUM(var_elem_alloc_bytes) AS Var,
 ->   SUM(hash_index_alloc_bytes) AS Hash,
 ->   SUM(fixed_elem_alloc_bytes + var_elem_alloc_bytes + hash_index_alloc_bytes)
 ->     AS Total_alloc
 -> FROM ndbinfo.memory_per_fragment
 -> WHERE fq_name = 'test/def/t1' OR parent_fq_name='test/def/t1'
 -> GROUP BY fq_name;
+----------------------+---------+---------+---------+-------------+
| Name                 | Fixed   | Var     | Hash    | Total_alloc |
+----------------------+---------+---------+---------+-------------+
| test/def/t1          | 4456448 | 5046272 | 1425408 |    10928128 |
| sys/def/13/PRIMARY   | 1966080 |       0 |       0 |     1966080 |
| sys/def/13/c3        | 1441792 |       0 |       0 |     1441792 |
| sys/def/13/c3$unique | 3276800 |       0 | 1425408 |     4702208 |
+----------------------+---------+---------+---------+-------------+
4 rows in set (0.11 sec)
```

###### 按元素查找每行分配的平均内存

要获取每个模式元素分配的平均每行内存（在所有副本中），我们使用子查询每次获取基本表固定元素计数以获得每行平均值，因为索引的`fixed_elem_count`不一定与基本表相同，如下所示：

```sql
mysql> SELECT
 ->   fq_name AS Name,
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS Table_rows,
 ->
 ->   SUM(fixed_elem_alloc_bytes)
 ->   /
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS Avg_fixed_alloc,
 ->
 ->   SUM(var_elem_alloc_bytes)
 ->   /
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') as Avg_var_alloc,
 ->
 ->   SUM(hash_index_alloc_bytes)
 ->   /
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') as Avg_hash_alloc,
 ->
 ->   SUM(fixed_elem_alloc_bytes+var_elem_alloc_bytes+hash_index_alloc_bytes)
 ->   /
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') as Avg_total_alloc
 ->
 -> FROM ndbinfo.memory_per_fragment
 -> WHERE fq_name = 'test/def/t1' or parent_fq_name='test/def/t1'
 -> GROUP BY fq_name;
+----------------------+------------+-----------------+---------------+----------------+-----------------+
| Name                 | Table_rows | Avg_fixed_alloc | Avg_var_alloc | Avg_hash_alloc | Avg_total_alloc |
+----------------------+------------+-----------------+---------------+----------------+-----------------+
| test/def/t1          |     100000 |         44.5645 |       50.4627 |        14.2541 |        109.2813 |
| sys/def/13/PRIMARY   |     100000 |         19.6608 |        0.0000 |         0.0000 |         19.6608 |
| sys/def/13/c3        |     100000 |         14.4179 |        0.0000 |         0.0000 |         14.4179 |
| sys/def/13/c3$unique |     100000 |         32.7680 |        0.0000 |        14.2541 |         47.0221 |
+----------------------+------------+-----------------+---------------+----------------+-----------------+
4 rows in set (0.70 sec)
```

###### 按行查找每行分配的平均内存

每行分配的平均内存（在所有副本中总共）：

```sql
mysql> SELECT
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS Table_rows,
 ->
 ->   SUM(fixed_elem_alloc_bytes)
 ->   /
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS Avg_fixed_alloc,
 ->
 ->   SUM(var_elem_alloc_bytes)
 ->   /
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS Avg_var_alloc,
 ->
 ->   SUM(hash_index_alloc_bytes)
 ->   /
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS Avg_hash_alloc,
 ->
 ->   SUM(fixed_elem_alloc_bytes + var_elem_alloc_bytes + hash_index_alloc_bytes)
 ->   /
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS Avg_total_alloc
 ->
 -> FROM ndbinfo.memory_per_fragment
 -> WHERE fq_name = 'test/def/t1' OR parent_fq_name='test/def/t1';
+------------+-----------------+---------------+----------------+-----------------+
| Table_rows | Avg_fixed_alloc | Avg_var_alloc | Avg_hash_alloc | Avg_total_alloc |
+------------+-----------------+---------------+----------------+-----------------+
|     100000 |        111.4112 |       50.4627 |        28.5082 |        190.3821 |
+------------+-----------------+---------------+----------------+-----------------+
1 row in set (0.71 sec)
```

###### 查找表格每行分配的平均内存

要获取整个表中每行分配的平均内存（在所有副本中），我们可以使用这里显示的查询：

```sql
mysql> SELECT
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS table_rows,
 ->
 ->   SUM(fixed_elem_alloc_bytes + var_elem_alloc_bytes + hash_index_alloc_bytes)
 ->   /
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS avg_total_alloc
 ->
 -> FROM ndbinfo.memory_per_fragment
 -> WHERE fq_name = 'test/def/t1' OR parent_fq_name='test/def/t1';
+------------+-----------------+
| table_rows | avg_total_alloc |
+------------+-----------------+
|     100000 |        190.3821 |
+------------+-----------------+
1 row in set (0.33 sec)
```

###### 查找每个模式元素使用的内存

要获取所有副本中每个模式元素使用的内存，我们需要对每个元素的分配和空闲内存之间的差值求和，如下所示：

```sql
mysql> SELECT
 ->   fq_name AS Name,
 ->   SUM(fixed_elem_alloc_bytes - fixed_elem_free_bytes) AS fixed_inuse,
 ->   SUM(var_elem_alloc_bytes-var_elem_free_bytes) AS var_inuse,
 ->   SUM(hash_index_alloc_bytes) AS hash_memory,
 ->   SUM(  (fixed_elem_alloc_bytes - fixed_elem_free_bytes)
 ->       + (var_elem_alloc_bytes - var_elem_free_bytes)
 ->       + hash_index_alloc_bytes) AS total_alloc
 -> FROM ndbinfo.memory_per_fragment
 -> WHERE fq_name = 'test/def/t1' OR parent_fq_name='test/def/t1'
 -> GROUP BY fq_name;
+----------------------+-------------+-----------+---------+-------------+
| fq_name              | fixed_inuse | var_inuse | hash    | total_alloc |
+----------------------+-------------+-----------+---------+-------------+
| test/def/t1          |     4422304 |   4872704 | 1425408 |    10720416 |
| sys/def/13/PRIMARY   |     1950848 |         0 |       0 |     1950848 |
| sys/def/13/c3        |     1428736 |         0 |       0 |     1428736 |
| sys/def/13/c3$unique |     3212800 |         0 | 1425408 |     4638208 |
+----------------------+-------------+-----------+---------+-------------+
4 rows in set (0.13 sec)
```

###### 按每个模式元素查找使用的平均内存

这个查询获取所有副本中每个模式元素使用的平均内存：

```sql
mysql> SELECT
 ->   fq_name AS Name,
 ->
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS table_rows,
 ->
 ->   SUM(fixed_elem_alloc_bytes - fixed_elem_free_bytes)
 ->   /
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS avg_fixed_inuse,
 ->
 ->   SUM(var_elem_alloc_bytes - var_elem_free_bytes)
 ->   /
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS avg_var_inuse,
 ->
 ->   SUM(hash_index_alloc_bytes)
 ->   /
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS avg_hash,
 ->
 ->   SUM(
 ->       (fixed_elem_alloc_bytes - fixed_elem_free_bytes)
 ->     + (var_elem_alloc_bytes - var_elem_free_bytes) + hash_index_alloc_bytes)
 ->   /
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS avg_total_inuse
 ->
 -> FROM ndbinfo.memory_per_fragment
 -> WHERE fq_name = 'test/def/t1' OR parent_fq_name='test/def/t1'
 -> GROUP BY fq_name;
+----------------------+------------+-----------------+---------------+----------+-----------------+
| Name                 | table_rows | avg_fixed_inuse | avg_var_inuse | avg_hash | avg_total_inuse |
+----------------------+------------+-----------------+---------------+----------+-----------------+
| test/def/t1          |     100000 |         44.2230 |       48.7270 |  14.2541 |        107.2042 |
| sys/def/13/PRIMARY   |     100000 |         19.5085 |        0.0000 |   0.0000 |         19.5085 |
| sys/def/13/c3        |     100000 |         14.2874 |        0.0000 |   0.0000 |         14.2874 |
| sys/def/13/c3$unique |     100000 |         32.1280 |        0.0000 |  14.2541 |         46.3821 |
+----------------------+------------+-----------------+---------------+----------+-----------------+
4 rows in set (0.72 sec)
```

###### 按元素查找每行使用的平均内存

这个查询获取所有副本中每行按元素使用的平均内存：

```sql
mysql> SELECT
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS table_rows,
 ->
 ->   SUM(fixed_elem_alloc_bytes - fixed_elem_free_bytes)
 ->   /
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS avg_fixed_inuse,
 ->
 ->   SUM(var_elem_alloc_bytes - var_elem_free_bytes)
 ->   /
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS avg_var_inuse,
 ->
 ->   SUM(hash_index_alloc_bytes)
 ->   /
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS avg_hash,
 ->
 ->   SUM(
 ->     (fixed_elem_alloc_bytes - fixed_elem_free_bytes)
 ->     + (var_elem_alloc_bytes - var_elem_free_bytes)
 ->     + hash_index_alloc_bytes)
 ->   /
 ->   ( SELECT SUM(fixed_elem_count)
 ->     FROM ndbinfo.memory_per_fragment
 ->     WHERE fq_name='test/def/t1') AS avg_total_inuse
 ->
 -> FROM ndbinfo.memory_per_fragment
 -> WHERE fq_name = 'test/def/t1' OR parent_fq_name='test/def/t1';
+------------+-----------------+---------------+----------+-----------------+
| table_rows | avg_fixed_inuse | avg_var_inuse | avg_hash | avg_total_inuse |
+------------+-----------------+---------------+----------+-----------------+
|     100000 |        110.1469 |       48.7270 |  28.5082 |        187.3821 |
+------------+-----------------+---------------+----------+-----------------+
1 row in set (0.68 sec)
```

###### 查找每行使用的总平均内存

这个查询获取每行使用的总平均内存：

```sql
mysql> SELECT
 ->   SUM(
 ->     (fixed_elem_alloc_bytes - fixed_elem_free_bytes)
 ->     + (var_elem_alloc_bytes - var_elem_free_bytes)
 ->     + hash_index_alloc_bytes)
 ->   /
 ->   ( SELECT
 ->       SUM(fixed_elem_count)
 ->       FROM ndbinfo.memory_per_fragment
 ->       WHERE fq_name='test/def/t1') AS avg_total_in_use
 -> FROM ndbinfo.memory_per_fragment
 -> WHERE fq_name = 'test/def/t1' OR parent_fq_name='test/def/t1';
+------------------+
| avg_total_in_use |
+------------------+
|         187.3821 |
+------------------+
1 row in set (0.24 sec)
```
