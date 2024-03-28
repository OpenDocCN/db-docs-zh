# 6.8.1 lz4_decompress — 解压缩 mysqlpump LZ4 压缩输出

> 原文：[`dev.mysql.com/doc/refman/8.0/en/lz4-decompress.html`](https://dev.mysql.com/doc/refman/8.0/en/lz4-decompress.html)

**lz4_decompress** 实用程序用于解压缩使用 LZ4 压缩创建的 **mysqlpump** 输出。

注意

**lz4_decompress** 自 MySQL 8.0.34 起已被弃用；预计将在未来的 MySQL 版本中移除。这是因为相关的 **mysqlpump** 实用程序自 MySQL 8.0.34 起已被弃用。

注意

如果 MySQL 配置时使用了 `-DWITH_LZ4=system` 选项，**lz4_decompress** 将不会被构建。在这种情况下，可以使用系统的 **lz4** 命令。

调用 **lz4_decompress** 如下：

```sql
lz4_decompress *input_file* *output_file*
```

示例：

```sql
mysqlpump --compress-output=LZ4 > dump.lz4
lz4_decompress dump.lz4 dump.txt
```

要查看帮助信息，请不带参数调用 **lz4_decompress**。

要解压缩使用 ZLIB 压缩创建的 **mysqlpump** 输出，请使用 **zlib_decompress**。参见 第 6.8.3 节，“zlib_decompress — 解压缩 mysqlpump ZLIB 压缩输出”。
