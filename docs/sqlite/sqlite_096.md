# 从 SQLite 3.5.9 迁移到 3.6.0

> 原文：[`sqlite.com/35to36.html`](https://sqlite.com/35to36.html)

SQLite 版本 3.6.0（2008-07-16）包含了许多更改。与 SQLite 项目的惯例一样，大多数更改完全向后兼容。然而，版本 3.6.0 中的一些更改是不兼容的，可能需要修改应用程序代码和/或 makefile。本文是关于 SQLite 3.6.0 中更改的简介，特别关注不兼容的更改。

> **要点：**
> 
> +   数据库文件格式未更改。
> +   
> +   所有不兼容性都在晦涩的接口上，因此对大多数应用程序应该没有任何影响。

## 1.0 不兼容更改

不兼容的更改首先得到了覆盖，因为它们对维护者和程序员来说是最重要的。

### 1.1 不兼容更改概述

1.  对 sqlite3_vfs 对象的更改

    1.  xAccess 方法的签名已修改，以返回一个错误代码，并将其输出存储到一个由参数指向的整数中，而不是直接返回输出。此更改允许 xAccess()方法报告失败。与此签名更改相关联，还添加了一个新的扩展错误代码 SQLITE_IOERR_ACCESS。

    1.  sqlite3_vfs 的 xGetTempname 方法已被移除。取而代之的是，xOpen 方法被增强以在 filename 参数为 NULL 时打开一个自行创建的临时文件。

    1.  为 sqlite3_vfs 添加了 xGetLastError()方法，用于将特定于文件系统的错误消息和错误代码返回给 SQLite。

1.  对 sqlite3_io_methods 上的 xCheckReservedLock 方法的签名进行了修改，以便它返回一个错误代码并将其布尔结果存储到由参数指向的整数中。与此更改相关联，还添加了一个新的扩展错误代码 SQLITE_IOERR_CHECKRESERVEDLOCK。

1.  当 SQLite 移植到新的操作系统（即不包括 Unix、Windows 和 OS/2 在内的操作系统，这些操作系统的移植与核心一起提供）时，必须作为移植的一部分提供两个新的函数，sqlite3_os_init() 和 sqlite3_os_end()。

1.  IN 和 NOT IN 运算符在其右侧表达式处理 NULL 值的方式已经符合 SQL 标准和其他 SQL 数据库引擎的要求。

1.  在某些情况下，已经调整了 SELECT 语句的结果集的列名，以使其更像其他 SQL 数据库引擎的工作方式。

1.  编译时选项的更改：

    1.  不再识别 SQLITE_MUTEX_APPDEF 编译时参数。作为替代，可以使用 sqlite3_config() 的 SQLITE_CONFIG_MUTEX 运算符和 sqlite3_mutex_methods 对象在运行时创建替代的 互斥实现。

    1.  编译时选项 OS_UNIX、OS_WIN、OS_OS2、OS_OTHER 和 TEMP_STORE 已重命名，以在应用软件中帮助避免命名空间冲突。这些选项的新名称分别是：SQLITE_OS_UNIX、SQLITE_OS_WIN、SQLITE_OS_OS2、SQLITE_OS_OTHER 和 SQLITE_TEMP_STORE。

### 1.2 VFS 层的变更

SQLite 版本 3.5.0 引入了一个新的操作系统接口层，提供了对底层操作系统的抽象。这是一个重要的创新，已被证明在移植和维护 SQLite 中非常有帮助。然而，开发人员发现了版本 3.5.0 中引入的原始“虚拟文件系统”设计中的一些小缺陷，因此 SQLite 3.6.0 包含了一些小的不兼容更改来解决这些问题。

> **关键点：** SQLite 版本 3.6.0 中操作系统接口的不兼容更改仅影响极少数应用程序，这些应用程序使用 虚拟文件系统 接口或提供应用程序定义的 互斥实现 或使用其他不常见的编译时选项。SQLite 版本 3.6.0 引入的更改对于使用内置接口到 Unix、Windows 和 OS/2 并使用标准构建配置的绝大多数 SQLite 应用程序没有任何影响。

### 1.3 IN 操作符处理 NULL 值的变更

所有版本的 SQLite，包括版本 3.5.9 在内，都在处理 IN 和 NOT IN 操作符右侧的 NULL 值时出现问题。具体来说，SQLite 以前忽略了 IN 和 NOT IN 操作符右侧的 NULL 值。

假设我们有一个名为 X1 的表定义如下：

> ```sql
>   CREATE TABLE x1(x INTEGER);
>   INSERT INTO x1 VALUES(1);
>   INSERT INTO x1 VALUES(2);
>   INSERT INTO x1 VALUES(NULL);
> 
> ```

鉴于上述 X1 的定义，以下表达式在 SQLite 中历史上被评估为 FALSE，尽管正确答案实际上是 NULL：

> ```sql
>   3 IN (1,2,NULL)
>   3 IN (SELECT * FROM x1)
> 
> ```

类似地，以下表达式在历史上被评估为 TRUE，而实际上 NULL 才是正确答案：

> ```sql
>   3 NOT IN (1,2,NULL)
>   3 NOT IN (SELECT * FROM x1)
> 
> ```

根据 SQL:1999 标准，SQLite 的历史行为是错误的，并且与 MySQL 和 PostgreSQL 的行为不一致。版本 3.6.0 改变了 IN 和 NOT IN 操作符的行为，以符合标准并产生与其他 SQL 数据库引擎相同的结果。

> **关键点：** 处理 IN 和 NOT IN 操作符中 NULL 值的变更技术上是一个 bug 修复，而不是设计更改。然而，维护者应该检查确保在升级到版本 3.6.0 之前，应用程序不依赖于旧的、有缺陷的行为。

### 1.4 列命名规则的变更

通过连接子查询报告的列名已略微修改，以便更像其他数据库引擎。考虑以下查询：

> ```sql
>   CREATE TABLE t1(a);
>   CREATE TABLE t2(x);
>   SELECT * FROM (SELECT t1.a FROM t1 JOIN t2 ORDER BY t2.x LIMIT 1) ORDER BY 1;
> 
> ```

在版本 3.5.9 中，上述查询将返回一个名为 "t1.a" 的单列。在版本 3.6.0 中，列名仅为 "a"。

SQLite 从未对 SELECT 语句的结果集中列名做出任何承诺，除非列包含 AS 子句。因此，列名的此更改在技术上不算不兼容性。SQLite 只是从一个未定义的行为变更为另一个未定义的行为。尽管如此，许多应用程序依赖于 SQLite 的未指定列命名行为，因此此更改在不兼容更改子标题下进行讨论。

### 1.5 编译时选项的更改

SQLite 的编译时选项由 C 预处理宏控制。SQLite 版本 3.6.0 更改了部分这些宏的名称，以便所有特定于 SQLite 的 C 预处理宏都以 "SQLITE_" 前缀开头。这样做是为了减少与其他软件模块名称冲突的风险。

> **关键点：** 编译时选项的更改可能会影响那些对 SQLite 进行定制化构建的项目中的 makefile。这些更改对应用程序代码和大多数使用标准默认构建的 SQLite 项目应该没有影响。

## 2.0 完全向后兼容的增强功能

除了上述不兼容的更改外，SQLite 版本 3.6.0 还添加了以下向后兼容的更改和增强功能：

1.  新的 sqlite3_config() 接口允许应用程序在运行时自定义 SQLite 的行为。使用 sqlite3_config() 可能的自定义包括以下内容：

    1.  使用 sqlite3_mutex_methods 对象，并使用 SQLITE_CONFIG_MUTEX 动词指定替代互斥体实现。

    1.  使用 SQLITE_CONFIG_MALLOC 命令词和 sqlite3_mem_methods 对象指定替代的 malloc 实现。

    1.  使用 SQLITE_CONFIG_SINGLETHREAD、SQLITE_CONFIG_MULTITHREAD 和 SQLITE_CONFIG_SERIALIZED 部分或完全禁用互斥体的使用。

1.  为 sqlite3_open_v2()接口提供了新的标志 SQLITE_OPEN_NOMUTEX。

1.  新的 sqlite3_status()接口允许应用程序在运行时查询 SQLite 的性能状态。

1.  sqlite3_memory_used()和 sqlite3_memory_highwater()接口已弃用。现在可以通过 sqlite3_status()获得等效功能。

1.  sqlite3_initialize()接口可用于显式初始化 SQLite 子系统。调用某些接口时会自动调用 sqlite3_initialize()，因此不需要使用 sqlite3_initialize()，但建议这样做。

1.  sqlite3_shutdown()接口导致 SQLite 释放可能由 sqlite3_initialize()分配的任何系统资源（内存分配、互斥体、打开文件句柄）。

1.  sqlite3_next_stmt()接口允许应用程序发现与数据库连接关联的所有 prepared statements。

1.  添加了用于返回基础数据库文件大小的页面计数 PRAGMA。

1.  添加了新的 R*Tree 索引扩展。
