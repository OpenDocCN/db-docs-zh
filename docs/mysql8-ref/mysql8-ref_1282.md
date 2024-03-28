> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-memcached-porting-memcached.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-memcached-porting-memcached.html)

#### 17.20.6.2 调整 memcached 应用以适配 InnoDB memcached 插件

在将现有的**memcached**应用程序调整为使用`daemon_memcached`插件时，请考虑 MySQL 和`InnoDB`表的这些方面：

+   如果键值长于几个字节，最好在`InnoDB`表的主键上使用数值自增列，并在包含**memcached**键值的列上创建唯一的二级索引可能更有效。这是因为如果主键值按排序顺序添加（就像使用自增值一样），`InnoDB`在大规模插入时表现最佳。主键值包含在二级索引中，如果主键是长字符串值，则会占用不必要的空间。

+   如果您使用**memcached**存储多种不同类别的信息，请考虑为每种数据类型设置单独的`InnoDB`表。在`innodb_memcache.containers`表中定义额外的表标识符，并使用`@@*`table_id`*.*`key`*`的表示法来存储和检索来自不同表的项目。物理上划分不同类型的信息允许您调整每个表的特性，以实现最佳的空间利用率、性能和可靠性。例如，您可以为保存博客文章的表启用压缩，但不为保存缩略图图像的表启用。您可能更频繁地备份一个表，因为它包含关键数据。您可能在经常用于生成报告的表上创建额外的二级索引。

+   最好为与**daemon_memcached**插件一起使用的表定义一个稳定的表定义，并永久保留这些表。对`innodb_memcache.containers`表的更改将在下次查询`innodb_memcache.containers`表时生效。容器表中的条目在启动时被处理，并且在使用`@@`表示法请求未识别的表标识符（由`containers.name`定义）时会被查询。因此，只要使用相关的表标识符，新条目就会立即可见，但对现有条目的更改需要在生效之前重新启动服务器。

+   当您使用默认的`innodb_only`缓存策略时，对`add()`、`set()`、`incr()`等的调用可能会成功，但仍会触发调试消息，如`while expecting 'STORED', got unexpected response 'NOT_STORED`。调试消息的出现是因为新值和更新值直接发送到`InnoDB`表，而不保存在内存缓存中，这是由于`innodb_only`缓存策略导致的。
