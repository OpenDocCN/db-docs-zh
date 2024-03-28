# 6.8.3 zlib_decompress — 解压缩 mysqlpump ZLIB-压缩输出

> 原文：[`dev.mysql.com/doc/refman/8.0/en/zlib-decompress.html`](https://dev.mysql.com/doc/refman/8.0/en/zlib-decompress.html)

**zlib_decompress** 实用程序用于解压缩使用 ZLIB 压缩创建的 **mysqlpump** 输出。

注意

**zlib_decompress** 自 MySQL 8.0.34 起已被弃用；预计在未来的 MySQL 版本中将被移除。这是因为相关的 **mysqlpump** 实用程序在 MySQL 8.0.34 起已被弃用。

注意

如果 MySQL 配置时使用了 `-DWITH_ZLIB=system` 选项，则不会构建 **zlib_decompress**。在这种情况下，可以改用系统的 **openssl zlib** 命令。

像这样调用 **zlib_decompress**：

```sql
zlib_decompress *input_file* *output_file*
```

示例：

```sql
mysqlpump --compress-output=ZLIB > dump.zlib
zlib_decompress dump.zlib dump.txt
```

要查看帮助信息，请不带任何参数调用 **zlib_decompress**。

要解压缩 **mysqlpump** 的 LZ4 压缩输出，请使用 **lz4_decompress**。参见 Section 6.8.1, “lz4_decompress — 解压缩 mysqlpump LZ4-压缩输出”。
