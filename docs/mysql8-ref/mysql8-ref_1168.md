> 原文：[`dev.mysql.com/doc/refman/8.0/en/sorted-index-builds.html`](https://dev.mysql.com/doc/refman/8.0/en/sorted-index-builds.html)

#### 17.6.2.3 排序索引构建

`InnoDB`在创建或重建索引时执行批量加载，而不是逐个插入一个索引记录。这种索引创建方法也被称为排序索引构建。排序索引构建不支持空间索引。

索引构建有三个阶段。在第一阶段中，扫描聚集索引，生成索引条目并添加到排序缓冲区。当排序缓冲区变满时，条目被排序并写入临时中间文件。这个过程也被称为“运行”。在第二阶段，将一个或多个运行写入临时中间文件后，在文件中对所有条目执行合并排序。在第三和最后阶段，排序的条目被插入到 B 树中。

在引入排序索引构建之前，索引条目是使用插入 API 逐个插入到 B 树中的。这种方法涉及打开一个 B 树游标以找到插入位置，然后使用乐观插入将条目插入到 B 树页中。如果由于页面已满而插入失败，则会执行悲观插入，这涉及打开一个 B 树游标，并根据需要拆分和合并 B 树节点以为条目找到空间。构建索引的这种“自顶向下”方法的缺点是搜索插入位置的成本以及不断拆分和合并 B 树节点。

排序索引构建使用“自底向上”的方法构建索引。采用这种方法，B 树的所有级别都保存对最右叶页的引用。在必要的 B 树深度上分配最右叶页，并根据它们的排序顺序插入条目。一旦叶页已满，就会向父页附加一个节点指针，并为下一个插入分配一个兄弟叶页。这个过程会一直持续，直到所有条目都被插入，这可能导致插入到根级别。当分配一个兄弟页时，之前固定的叶页的引用会被释放，新分配的叶页成为最右叶页和新的默认插入位置。

##### 为未来索引增长保留 B 树页空间

为了为未来的索引增长留出空间，您可以使用`innodb_fill_factor`变量来保留 B-Tree 页面空间的百分比。例如，将`innodb_fill_factor`设置为 80 会在排序索引构建期间保留 B-Tree 页面中 20％的空间。此设置适用于 B-Tree 叶子和非叶子页面。它不适用于用于`TEXT`或`BLOB`条目的外部页面。保留的空间量可能不会完全按照配置，因为`innodb_fill_factor`值被解释为提示而不是硬限制。

##### 排序索引构建和全文索引支持

支持对全文索引进行排序。以前，SQL 用于向全文索引插入条目。

##### 排序索引构建和压缩表

对于压缩表，先前的索引创建方法将条目附加到压缩和未压缩页面。当修改日志（表示压缩页面上的空闲空间）变满时，压缩页面将被重新压缩。如果由于空间不足而导致压缩失败，则页面将被分割。通过排序索引构建，条目仅附加到未压缩页面。当未压缩页面变满时，它将被压缩。自适应填充用于确保在大多数情况下压缩成功，但如果压缩失败，则页面将被分割，并再次尝试压缩。此过程将持续，直到压缩成功。有关 InnoDB 表 B-Tree 页面压缩的更多信息，请参见第 17.9.1.5 节，“InnoDB 表的压缩工作原理”。

##### 排序索引构建和重做日志

在排序索引构建期间，重做日志被禁用。相反，存在一个检查点以确保索引构建可以经受意外退出或失败。检查点强制将所有脏页写入磁盘。在排序索引构建期间，会定期向页面清理器线程发出信号，以刷新脏页以确保检查点操作可以快速处理。通常情况下，当干净页面数量低于设定阈值时，页面清理器线程会刷新脏页。对于排序索引构建，脏页会被迅速刷新以减少检查点开销并并行化 I/O 和 CPU 活动。

##### 排序索引构建和优化器统计

排序索引构建可能导致与以前的索引创建方法生成的优化器统计数据不同。统计数据的差异不会影响工作负载性能，这是由于用于填充索引的不同算法所致。
