# 12.9.3 utf8 字符集（utf8mb3 的弃用别名）

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-unicode-utf8.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-unicode-utf8.html)

MySQL 过去曾将`utf8`用作`utf8mb3`字符集的别名，但现在已弃用此用法；在 MySQL 8.0 中，`SHOW`语句和`INFORMATION_SCHEMA`表的列显示为`utf8mb3`。有关更多信息，请参见第 12.9.2 节，“utf8mb3 字符集（3 字节 UTF-8 Unicode 编码）”。

注意

MySQL 推荐的字符集是`utf8mb4`。所有新应用程序应该使用`utf8mb4`。

`utf8mb3`字符集已被弃用。`utf8mb3`将在 MySQL 8.0.x 及其后续 LTS 版本系列的生命周期中继续受支持，以及在 MySQL 8.0 中。

预计`utf8mb3`将在 MySQL 的未来主要版本中被移除。

由于更改字符集可能是一个复杂且耗时的任务，您应该立即开始准备使用`utf8mb4`来为新应用程序做好准备。有关转换现有使用 utfmb3 的应用程序的指导，请参见第 12.9.8 节，“在 3 字节和 4 字节 Unicode 字符集之间转换”。
