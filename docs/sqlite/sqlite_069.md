# 1\. 概述

> 原文：[`sqlite.com/dbhash.html`](https://sqlite.com/dbhash.html)

**dbhash**（或在 Windows 上为 **dbhash.exe**）实用程序是一个命令行程序，用于计算 SQLite 数据库的架构和内容的 SHA1 哈希。

Dbhash 忽略额外的格式细节，仅对数据库架构和内容进行哈希。因此，即使数据库文件被修改，哈希也是恒定的：

+   清理

+   PRAGMA page_size

+   PRAGMA journal_mode

+   重建索引

+   分析

+   通过备份 API 复制

+   ... 等等

上述操作可能导致原始数据库文件发生巨大变化，因此在文件级别可能会导致非常不同的 SHA1 哈希。但由于这些操作未改变数据库文件中表示的内容，所以 dbhash 计算的哈希也不会改变。

Dbhash 可用于比较两个数据库，以确认它们在磁盘上的表示虽然相当不同，但它们是等效的。Dbhash 还可用于验证远程数据库的内容，而无需通过缓慢的链接传输整个远程数据库的内容。

# 2\. 使用

Dbhash 是一个命令行实用程序。要运行它，请在命令行提示符下输入 "dbhash"，然后输入一个或多个要进行哈希的 SQLite 数据库文件的名称。数据库的哈希将显示在标准输出上。例如：

```sql
drh@bella:~/sqlite/bld$ dbhash ~/Fossils/sqlite.fossil
8d3da9ff87196312aaa33076627ccb7943ef79e3 /home/drh/Fossils/sqlite.fossil

```

Dbhash 支持命令行选项，可以限制对数据库文件中进行哈希的表，或仅限制哈希到内容或仅架构。运行 "dbhash --help" 获取更多信息。

# 3\. 构建

要在 Unix 上构建 dbhash 实用程序的副本，请获取 SQLite 的官方源代码副本，并输入：

```sql
./configure
make dbhash

```

在 Windows 上，输入：

```sql
nmake /f makefile.msc dbhash.exe

```

`dbhash` 程序是由名为 [dbhash.c](https://www.sqlite.org/src/artifact?ci=trunk&filename=tool/dbhash.c) 的单个 C 代码文件实现的。要手动构建 `dbhash` 程序，只需编译 `dbhash.c` 源文件，并将其链接到 SQLite 库即可。
