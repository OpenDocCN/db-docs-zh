# SQLite Archiver

> 原文：[`www.sqlite.org/sqlar/`](https://www.sqlite.org/sqlar/)

文档登录☰主页 时间线 文件 分支 标签 工单 维基 # SQLAR - SQLite Archiver

此存储库包含“SQLite Archiver”程序的源代码。该程序（名为“sqlar”）的操作方式与“zip”类似，不同之处在于它构建的压缩归档存储在 SQLite 数据库中，而不是 ZIP 归档。

这样做的动机是看看与包含相同内容的 ZIP 归档相比，SQLite 数据库文件会大多少。答案取决于文件名，但估计增大约为 2%是一个合理的猜测。换句话说，将文件作为压缩的 BLOB 存储在 SQLite 数据库文件中，结果文件只比使用相同压缩的 ZIP 归档大约多出 2%。

## 编译

在 Unix 系统上，只需键入“make”。已包含 SQLite 源代码。构建需要 zlib 压缩库。

## 用法

要创建一个归档：

```sql
 sqlar ARCHIVE FILES... 
```

所有在 FILES 中命名的文件将被添加到归档中。如果归档中已存在具有相同名称的其他文件，则将其替换。如果 FILES 中的任何一个是目录，则将递归扫描该目录。

要查看归档的内容：

```sql
 sqlar -l ARCHIVE 
```

要提取归档的内容：

```sql
 sqlar -x ARCHIVE [FILES...] 
```

如果提供了 FILES 参数，则仅提取命名的文件。如果没有 FILES 参数，则提取所有文件。

所有命令都可以使用-v 进行详细输出。例如：

```sql
 sqlar -v ARCHIVE FILES..
    sqlar -lv ARCHIVE
    sqlar -xv ARCHIVE 
```

文件通常在存储为数据库中的 BLOB 之前使用 zlib 进行压缩。但是，如果文件是不可压缩的或者在命令行上使用了-n 选项，则文件将按照磁盘上的实际内容存储在数据库中，而不进行压缩。

## 存储

数据库模式如下：

```sql
 CREATE TABLE sqlar(
      name TEXT PRIMARY KEY,  -- name of the file
      mode INT,               -- access permissions
      mtime INT,              -- last modification time
      sz INT,                 -- original file size
      data BLOB               -- compressed content
    ); 
```

目录和空文件的 `sqlar.sz` 均为 0。可以通过 `sqlar.data IS NULL` 区分目录和空文件。如果 `sqlar.blob` 的长度小于 `sqlar.sz`，则文件已压缩；如果 `sqlar.blob` 的长度等于 `sqlar.sz`，则文件存储为纯文本。

符号链接的`sqlar.sz`设置为-1，链接目标存储在`sqlar.data`字段中。

SQLAR 使用 "zlib 格式" 进行压缩。ZIP 使用原始 deflate 格式。区别在于，zlib 格式包含一个两字节的压缩类型识别标头（0x78 0x9c）和一个 4 字节的校验和。因此，对于等效数据而言，SQLAR 的 "数据" 比 ZIP 多 6 字节。SQLAR 程序使用 zlib 格式，而不是稍微更小的原始 deflate 格式，因为这是 [zlib 文档](https://www.zlib.net/manual.html) 推荐的。

SQLAR 未来可能会扩展以支持除 deflate 外的其他压缩格式。如果是这样，数据字段将包含新的标头值，用于识别使用新格式压缩的条目。

## Fuse 文件系统

可以使用 "sqlarfs" 实用程序将 SQLite 存档文件作为 [Fuse 文件系统](http://fuse.sourceforge.net) 挂载，包括该项目。

要构建 "sqlarfs" 实用程序，请运行：

```sql
 make sqlarfs 
```

要将 SQLite 存档挂载为文件系统，请运行：

```sql
 mkdir ~/fuse
    ./sqlarfs ARCHIVE_NAME -f ~/fuse 
```

将 `ARCHIVE_NAME` 替换为要挂载的 SQLite 存档文件的文件名，当然了。`-f` 选项使得 `sqlarfs` 在前台运行，因此您可以通过简单地按中断键（通常是 Ctrl-C）来卸载文件系统。此页面由 Fossil 2.25 [fc8d476aca] 2024-06-19 18:26:12 生成，用时约 0.004 秒。
