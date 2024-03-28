# 26.3.2 哈希和键分区的管理

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-management-hash-key.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-management-hash-key.html)

使用哈希或键分区的表在修改分区设置方面非常相似，与按范围或列表分区的表在许多方面不同。因此，本节仅讨论了对使用哈希或键分区的表进行修改。有关对按范围或列表分区的表添加和删除分区的讨论，请参见第 26.3.1 节，“范围和列表分区的管理”。

与可以从按`RANGE`或`LIST`分区的表中删除分区的方式不同，您无法像从中删除分区一样从按`HASH`或`KEY`分区的表中删除分区。但是，您可以使用`ALTER TABLE ... COALESCE PARTITION`合并`HASH`或`KEY`分区。假设一个包含有关客户数据的`clients`表被分成了 12 个分区，如下所示创建：

```sql
CREATE TABLE clients (
    id INT,
    fname VARCHAR(30),
    lname VARCHAR(30),
    signed DATE
)
PARTITION BY HASH( MONTH(signed) )
PARTITIONS 12;
```

要将分区数从 12 减少到 8，请执行以下`ALTER TABLE`语句：

```sql
mysql> ALTER TABLE clients COALESCE PARTITION 4;
Query OK, 0 rows affected (0.02 sec)
```

`COALESCE`在使用`HASH`、`KEY`、`LINEAR HASH`或`LINEAR KEY`分区的表上同样有效。以下是一个类似于前一个示例的示例，唯一不同之处在于表是通过`LINEAR KEY`分区的：

```sql
mysql> CREATE TABLE clients_lk (
 ->     id INT,
 ->     fname VARCHAR(30),
 ->     lname VARCHAR(30),
 ->     signed DATE
 -> )
 -> PARTITION BY LINEAR KEY(signed)
 -> PARTITIONS 12;
Query OK, 0 rows affected (0.03 sec)

mysql> ALTER TABLE clients_lk COALESCE PARTITION 4;
Query OK, 0 rows affected (0.06 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

`COALESCE PARTITION`后面的数字是要合并到剩余部分中的分区数，换句话说，要从表中删除的分区数。

尝试删除比表中存在的分区更多的分区会导致如下错误：

```sql
mysql> ALTER TABLE clients COALESCE PARTITION 18;
ERROR 1478 (HY000): Cannot remove all partitions, use DROP TABLE instead
```

要将`clients`表的分区数从 12 增加到 18，请使用如下`ALTER TABLE ... ADD PARTITION`：

```sql
ALTER TABLE clients ADD PARTITION PARTITIONS 6;
```
