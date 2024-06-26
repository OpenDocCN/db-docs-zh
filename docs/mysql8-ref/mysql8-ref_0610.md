# 10.10.2 MyISAM 关键缓存

> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisam-key-cache.html`](https://dev.mysql.com/doc/refman/8.0/en/myisam-key-cache.html)

10.10.2.1 共享关键缓存访问

10.10.2.2 多个关键缓存

10.10.2.3 中点插入策略

10.10.2.4 索引预加载

10.10.2.5 关键缓存块大小

10.10.2.6 重构关键缓存

为了最小化磁盘 I/O，`MyISAM`存储引擎利用了许多数据库管理系统使用的策略。它使用缓存机制将最常访问的表块保留在内存中：

+   对于索引块，维护一个称为关键缓存（或关键缓冲区）的特殊结构。该结构包含多个块缓冲区，其中放置了最常用的索引块。

+   对于数据块，MySQL 不使用特殊缓存。相反，它依赖于本机操作系统文件系统缓存。

本节首先描述了`MyISAM`关键缓存的基本操作。然后讨论了改进关键缓存性能的特性，以及使您能够更好地控制缓存操作的功能：

+   多个会话可以同时访问缓存。

+   您可以设置多个关键缓存并将表索引分配给特定缓存。

要控制关键缓存的大小，请使用`key_buffer_size`系统变量。如果将此变量设置为零，则不使用关键缓存。如果`key_buffer_size`的值太小，无法分配最小数量的块缓冲区（8），则也不使用关键缓存。

当关键缓存不可用时，索引文件仅使用操作系统提供的本机文件系统缓冲区进行访问。（换句话说，表索引块的访问采用与表数据块相同的策略。）

索引块是对`MyISAM`索引文件的连续访问单元。通常，索引块的大小等于索引 B 树节点的大小。（索引在磁盘上使用 B 树数据结构表示。树底部的节点是叶节点。叶节点上面的节点是非叶节点。）

关键缓存结构中的所有块缓冲区大小相同。这个大小可以等于、大于或小于表索引块的大小。通常这两个值中的一个是另一个的倍数。

当需要访问任何表索引块的数据时，服务器首先检查它是否在关键缓存的某个块缓冲区中可用。如果是，服务器访问关键缓存中的数据而不是在磁盘上。也就是说，它从缓存中读取或写入数据，而不是从磁盘读取或写入数据。否则，服务器选择一个包含不同表索引块（或块）的缓存块缓冲区，并将数据替换为所需表索引块的副本。一旦新的索引块在缓存中，索引数据就可以被访问。

如果选择要替换的块已被修改，该块被视为“脏”。在这种情况下，在替换之前，其内容被刷新到其来源的表索引中。

通常，服务器遵循 LRU（最近最少使用）策略：在选择要替换的块时，它选择最近最少使用的索引块。为了更容易做出这个选择，关键缓存模块维护了一个特殊列表（LRU 链），按使用时间排序所有已使用的块。当访问一个块时，它是最近使用的，并被放在列表的末尾。当需要替换块时，列表开头的块是最近最少使用的，成为首选的驱逐候选。

`InnoDB`存储引擎也使用 LRU 算法来管理其缓冲池。请参阅第 17.5.1 节，“缓冲池”。
