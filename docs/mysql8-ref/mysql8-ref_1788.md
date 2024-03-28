> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-hash-maps.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-hash-maps.html)

#### 25.6.16.37 ndbinfo hash_maps 表

+   `id`

    哈希映射的唯一 ID

+   `version`

    哈希映射版本（整数）

+   `state`

    哈希映射状态；参见 Object::State 获取数值和描述。

+   `fq_name`

    哈希映射的完全限定名称

`hash_maps`表实际上是一个视图，由四列组成，这四列与`dict_obj_info`表的列名相同，如下所示：

```sql
CREATE VIEW hash_maps AS
  SELECT id, version, state, fq_name
  FROM dict_obj_info
  WHERE type=24;  # Hash map; defined in dict_obj_types
```

更多信息请参阅`dict_obj_info`的描述。

`hash_maps`表在 NDB 8.0.29 中添加。
