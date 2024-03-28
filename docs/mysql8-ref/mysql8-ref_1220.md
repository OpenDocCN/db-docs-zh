# 17.9 InnoDB 表和页面压缩

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-compression.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-compression.html)

17.9.1 InnoDB 表压缩

17.9.2 InnoDB 页面压缩

本节提供关于`InnoDB`表压缩和`InnoDB`页面压缩功能的信息。页面压缩功能也被称为透明页面压缩。

使用`InnoDB`的压缩功能，您可以创建数据以压缩形式存储的表。压缩有助于提高原始性能和可伸缩性。压缩意味着在磁盘和内存之间传输的数据量更少，并且在磁盘和内存中占用的空间更少。对于具有二级索引的表，好处尤为明显，因为索引数据也被压缩。压缩对于 SSD 存储设备尤为重要，因为它们的容量往往比 HDD 设备低。
