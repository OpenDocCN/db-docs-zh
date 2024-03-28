> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-physical-structure.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-physical-structure.html)

#### 17.6.2.2 InnoDB 索引的物理结构

除了空间索引外，`InnoDB`索引是 B 树数据结构。空间索引使用 R 树，这是专门用于索引多维数据的数据结构。索引记录存储在它们的 B 树或 R 树数据结构的叶子页中。索引页的默认大小为 16KB。页面大小由 MySQL 实例初始化时的`innodb_page_size`设置确定。请参阅第 17.8.1 节，“InnoDB 启动配置”。

当新记录插入`InnoDB`的聚簇索引时，`InnoDB`尝试保留 1/16 的页面空间以供将来插入和更新索引记录。如果索引记录按顺序（升序或降序）插入，则生成的索引页约为 15/16 满。如果记录以随机顺序插入，则页面填充率为 1/2 至 15/16。

创建或重建 B 树索引时，`InnoDB`执行批量加载。这种索引创建方法称为排序索引构建。`innodb_fill_factor`变量定义了在排序索引构建期间填充每个 B 树页的空间百分比，剩余空间保留用于未来的索引增长。空间索引不支持排序索引构建。有关更多信息，请参阅第 17.6.2.3 节，“排序索引构建”。将`innodb_fill_factor`设置为 100 会使聚簇索引页中的 1/16 空间保留用于未来的索引增长。

如果`InnoDB`索引页的填充因子低于`MERGE_THRESHOLD`，默认为 50%，`InnoDB`会尝试收缩索引树以释放页面。`MERGE_THRESHOLD`设置适用于 B 树和 R 树索引。有关更多信息，请参阅第 17.8.11 节，“配置索引页合并阈值”。
