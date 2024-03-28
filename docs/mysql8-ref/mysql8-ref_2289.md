# A.16 MySQL 8.0 FAQ：InnoDB 更改缓冲区

> 原文：[`dev.mysql.com/doc/refman/8.0/en/faqs-innodb-change-buffer.html`](https://dev.mysql.com/doc/refman/8.0/en/faqs-innodb-change-buffer.html)

A.16.1\. [什么类型的操作会修改辅助索引并导致更改缓冲？](https://dev.mysql.com/doc/refman/8.0/en/faqs-innodb-change-buffer.html#faq-innodb-change-buffer-operations)

A.16.2\. [InnoDB 更改缓冲区的好处是什么？](https://dev.mysql.com/doc/refman/8.0/en/faqs-innodb-change-buffer.html#faq-innodb-change-buffer-benefits)

A.16.3\. [更改缓冲区支持其他类型的索引吗？](https://dev.mysql.com/doc/refman/8.0/en/faqs-innodb-change-buffer.html#faq-innodb-change-buffer-index-types)

A.16.4\. [InnoDB 为更改缓冲区使用了多少空间？](https://dev.mysql.com/doc/refman/8.0/en/faqs-innodb-change-buffer.html#faq-innodb-change-buffer-space-max-size)

A.16.5\. [如何确定更改缓冲区的当前大小？](https://dev.mysql.com/doc/refman/8.0/en/faqs-innodb-change-buffer.html#faq-innodb-change-buffer-current-size)

A.16.6\. [何时发生更改缓冲区合并？](https://dev.mysql.com/doc/refman/8.0/en/faqs-innodb-change-buffer.html#faq-innodb-change-buffer-merging)

A.16.7\. [何时刷新更改缓冲区？](https://dev.mysql.com/doc/refman/8.0/en/faqs-innodb-change-buffer.html#faq-innodb-change-buffer-flush-time)

A.16.8\. [何时应该使用更改缓冲区？](https://dev.mysql.com/doc/refman/8.0/en/faqs-innodb-change-buffer.html#faq-innodb-change-buffer-when-to-enable)

A.16.9\. [更改缓冲区不应该在什么情况下使用？](https://dev.mysql.com/doc/refman/8.0/en/faqs-innodb-change-buffer.html#faq-innodb-change-buffer-when-to-disable)

A.16.10\. [我在哪里可以找到有关更改缓冲区的更多信息？](https://dev.mysql.com/doc/refman/8.0/en/faqs-innodb-change-buffer.html#faq-innodb-change-buffer-info)

| **A.16.1.** | 什么类型的操作会修改辅助索引并导致更改缓冲？ |
| --- | --- |
|  | `INSERT`、`UPDATE`和`DELETE`操作可以修改辅助索引。如果受影响的索引页不在缓冲池中，则更改可以在更改���冲区中缓冲。 |
| **A.16.2.** | `InnoDB`更改缓冲区的好处是什么？ |
|  | 当辅助索引页不在缓冲池中时，缓冲辅助索引更改可以避免需要立即从磁盘读入受影响的索引页的昂贵随机访问 I/O 操作。缓冲的更改可以稍后批量应用，当页面被其他读取操作读入缓冲池时。 |
| **A.16.3.** | 更改缓冲区支持其他类型的索引吗？ |
|  | 不支持更改缓冲区的主键索引。聚簇索引、全文索引和空间索引不受支持。全文索引有自己的缓存机制。 |
| **A.16.4.** | `InnoDB`为更改缓冲区使用了多少空间？ |
|  | 在 MySQL 5.6 中引入`innodb_change_buffer_max_size`配置选项之前，系统表空间中磁盘上的变更缓冲的最大大小为`InnoDB`缓冲池大小的 1/3。在 MySQL 5.6 及更高版本中，`innodb_change_buffer_max_size`配置选项将变更缓冲的最大大小定义为总缓冲池大小的百分比。默认情况下，`innodb_change_buffer_max_size`设置为 25。最大设置为 50。如果操作会导致磁盘上的变更缓冲超过定义的限制，则`InnoDB`不会缓冲该操作。变更缓冲页不需要在缓冲池中持久存在，可能会被 LRU 操作驱逐。 |
| **A.16.5.** | 如何确定变更缓冲的当前大小？ |

|  | 变更缓冲的当前大小由`SHOW ENGINE INNODB STATUS \G`报告，在`INSERT BUFFER AND ADAPTIVE HASH INDEX`标题下。例如：

```sql
-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
-------------------------------------
Ibuf: size 1, free list len 0, seg size 2, 0 merges
```

相关数据点包括：

+   `size`：变更缓冲中使用的页面数。变更缓冲大小等于`seg size - (1 + free list len)`。`1 +`值代表变更缓冲头页面。

+   `seg size`：变更缓冲的大小，以页面为单位。

有关监视变更缓冲状态的信息，请参见 Section 17.5.2，“变更缓冲”。 |

| **A.16.6.** | 变更缓冲合并发生在什么时候？ |
| --- | --- |
|  |

+   当页面被读入缓冲池时，在页面可用之前，缓冲的更改在读取完成后合并。

+   变更缓冲合并作为后台任务执行。`innodb_io_capacity`参数设置了`InnoDB`后台任务（如从变更缓冲合并数据）执行的 I/O 活动的上限。

+   在崩溃恢复期间执行变更缓冲合并。当索引页面被读入缓冲池时，从变更缓冲（在系统表空间中）应用更改到辅助索引的叶子页面。

+   变更缓冲是完全持久的，可以在系统崩溃时幸存下来。重新启动后，变更缓冲合并操作将作为正常操作的一部分恢复。

+   可以通过使用`--innodb-fast-shutdown=0`在慢速服务器关闭过程中强制执行变更缓冲的完全合并。

|

| **A.16.7.** | 变更缓冲何时被刷新？ |
| --- | --- |
|  | 更新的页面由刷新缓冲池中其他页面的相同刷新机制刷新。 |
| **A.16.8.** | 何时应该使用变更缓冲？ |
|  | 变更缓冲区是一项旨在减少随着索引变大而无法再适应`InnoDB`缓冲池而产生的随机 I/O 的功能。通常情况下，当整个数据集无法适应缓冲池时，当有大量修改次要索引页的 DML 活动，或者当有许多次要索引经常被 DML 活动修改时，应该使用变更缓冲区。 |
| **A.16.9.** | 何时不应使用变更缓冲区？ |
|  | 如果整个数据集适应`InnoDB`缓冲池，如果次要索引相对较少，或者如果使用固态存储，其中随机读取速度与顺序读取速度几乎相同，您可能考虑禁用变更缓冲区。在进行配置更改之前，建议您使用代表性工作负载运行测试，以确定禁用变更缓冲区是否提供任何好处。 |
| **A.16.10.** | 我在哪里可以找到有关变更缓冲区的更多信息？ |
|  | 参见第 17.5.2 节，“变更缓冲区”。 |
