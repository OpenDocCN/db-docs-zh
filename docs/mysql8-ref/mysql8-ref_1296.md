# 17.22 InnoDB 限制

> 译文：[`dev.mysql.com/doc/refman/8.0/en/innodb-limits.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-limits.html)

本节描述了 `InnoDB` 表、索引、表空间以及 `InnoDB` 存储引擎的其他方面的限制。

+   一张表最多可以包含 1017 列。虚拟生成列也包括在此限制内。

+   一张表最多可以包含 64 个次要索引。

+   对于使用 `DYNAMIC` 或 `COMPRESSED` 行格式的 `InnoDB` 表，索引键前缀长度限制为 3072 字节。

    对于使用 `REDUNDANT` 或 `COMPACT` 行格式的 `InnoDB` 表，索引键前缀长度限制为 767 字节。例如，假设在 `TEXT` 或 `VARCHAR` 列上使用了超过 191 个字符的 列前缀 索引，在 `utf8mb4` 字符集和每个字符最大 4 个字节的情况下，可能会达到这个限制。

    尝试使用超过限制的索引键前缀长度将返回错误。

    如果通过在创建 MySQL 实例时指定 `innodb_page_size` 选项将 `InnoDB` 页大小 降低为 8KB 或 4KB，那么基于 16KB 页大小的 3072 字节限制，索引键的最大长度将按比例降低。也就是说，当页大小为 8KB 时，最大索引键长度为 1536 字节，当页大小为 4KB 时，最大索引键长度为 768 字节。

    适用于索引键前缀的限制也适用于完整列索引键。

+   最多允许为多列索引设置 16 列。超过限制将返回错误。

    ```sql
    ERROR 1070 (42000): Too many key parts specified; max 16 parts allowed
    ```

+   行的最大大小，不包括存储在页外的任何可变长度列，对于 4KB、8KB、16KB 和 32KB 的页大小略小于半页。例如，默认的 `innodb_page_size` 为 16KB 时，最大行大小约为 8000 字节。然而，对于 64KB 的 `InnoDB` 页大小，最大行大小约为 16000 字节。`LONGBLOB` 和 `LONGTEXT` 列必须小于 4GB，包括 `BLOB` 和 `TEXT` 列在内的总行大小必须小于 4GB。

    如果一行长度小于半页，所有内容都存储在页面内。如果超过半页，变长列将选择外部离页存储，直到行适合半页为止，如第 17.11.2 节，“文件空间管理”中所述。

+   尽管`InnoDB`内部支持大于 65535 字节的行大小，但 MySQL 本身对所有列的组合大小施加了 65535 的行大小限制。请参见第 10.4.7 节，“表列数和行大小限制”。

+   在一些较旧的操作系统上，文件大小必须小于 2GB。这不是`InnoDB`的限制。如果您需要一个大的系统表空间，请使用多个较小的数据文件进行配置，而不是一个大的数据文件，或者将表数据分布在每个表一个文件和通用表空间数据文件中。

+   `InnoDB`日志文件的组合最大大小为 512GB。

+   最小表空间大小略大于 10MB。最大表空间大小取决于`InnoDB`页面大小。

    **表 17.31 InnoDB 最大表空间大小**

    | InnoDB 页面大小 | 最大表空间大小 |
    | --- | --- |
    | 4KB | 16TB |
    | 8KB | 32TB |
    | 16KB | 64TB |
    | 32KB | 128TB |
    | 64KB | 256TB |

    最大表空间大小也是表的最大大小。

+   一个`InnoDB`实例支持最多 2³²（4294967296）个表空间，其中少数表空间保留用于撤销和临时表。

+   共享表空间支持最多 2³²（4294967296）个表。

+   表空间文件的路径，包括文件名，在 Windows 上不能超过`MAX_PATH`限制。在 Windows 10 之前，`MAX_PATH`限制为 260 个字符。从 Windows 10，版本 1607 开始，`MAX_PATH`限制已从常见的 Win32 文件和目录函数中移除，但您必须启用新的行为。

+   有关并发读写事务相关的限制，请参见第 17.6.6 节，“撤销日志”。
