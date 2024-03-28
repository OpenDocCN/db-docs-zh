> 原文：[`dev.mysql.com/doc/refman/8.0/en/key-cache-block-size.html`](https://dev.mysql.com/doc/refman/8.0/en/key-cache-block-size.html)

#### 10.10.2.5 关键缓存块大小

可以使用`key_cache_block_size`变量来指定单个关键缓存的块缓冲区大小。这允许调整索引文件的 I/O 操作性能。

当读取缓冲区的大小等于本机操作系统 I/O 缓冲区的大小时，I/O 操作的最佳性能可以实现。但是，将关键节点的大小设置为 I/O 缓冲区的大小并不总是确保获得最佳的整体性能。当读取大叶节点时，服务器会拉取许多不必要的数据，有效地阻止了对其他叶节点的读取。

要控制`MyISAM`表的`.MYI`索引文件中块的大小，请在服务器启动时使用`--myisam-block-size`选项。
