> 原文：[`dev.mysql.com/doc/refman/8.0/en/key-cache-restructuring.html`](https://dev.mysql.com/doc/refman/8.0/en/key-cache-restructuring.html)

#### 10.10.2.6 重构关键缓存

可通过更新其参数值随时重构关键缓存。例如：

```sql
mysql> SET GLOBAL cold_cache.key_buffer_size=4*1024*1024;
```

如果您为`key_buffer_size`或`key_cache_block_size`关键缓存组件分配一个与组件当前值不同的值，则服务器会销毁缓存的旧结构，并根据新值创建一个新结构。如果缓存包含任何脏块，服务器会在销毁和重新创建缓存之前将它们保存到磁盘。如果更改其他关键缓存参数，则不会发生重构。

重构关键缓存时，服务器首先将任何脏缓冲区的内容刷新到磁盘。之后，缓存内容变得不可用。然而，重构不会阻塞需要使用分配给缓存的索引的查询。相反，服务器直接使用本地文件系统缓存访问表索引。文件系统缓存不如使用关键缓存高效，因此虽然查询会执行，但可以预期会有减速。在缓存重构后，它再次可用于缓存分配给它的索引，并且索引的文件系统缓存使用停止。
