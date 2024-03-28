> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-tablespace-autoextend-size.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-tablespace-autoextend-size.html)

#### 17.6.3.9 表空间 AUTOEXTEND_SIZE 配置

默认情况下，当每个表文件或通用表空间需要额外空间时，表空间根据以下规则逐步扩展：

+   如果表空间小于一个区段的大小，则每次扩展一个页面。

+   如果表空间大于 1 个区段但小于 32 个区段，则每次扩展一个区段。

+   如果表空间大于 32 个区段，则每次扩展四个区段。

有关区段大小的信息，请参阅 Section 17.11.2, “File Space Management”。

从 MySQL 8.0.23 开始，可以通过指定`AUTOEXTEND_SIZE`选项来配置每个表文件或通用表空间的扩展量。配置更大的扩展大小可以帮助避免碎片化，并促进大量数据的摄入。

要为每个表文件表空间配置扩展大小，请在`CREATE TABLE`或`ALTER TABLE`语句中指定`AUTOEXTEND_SIZE`大小：

```sql
CREATE TABLE t1 (c1 INT) AUTOEXTEND_SIZE = 4M;
```

```sql
ALTER TABLE t1 AUTOEXTEND_SIZE = 8M;
```

要为通用表空间配置扩展大小，请在`CREATE TABLESPACE`或`ALTER TABLESPACE`语句中指定`AUTOEXTEND_SIZE`大小：

```sql
CREATE TABLESPACE ts1 AUTOEXTEND_SIZE = 4M;
```

```sql
ALTER TABLESPACE ts1 AUTOEXTEND_SIZE = 8M;
```

注意

`AUTOEXTEND_SIZE`选项也可用于创建撤销表空间，但是撤销表空间的扩展行为有所不同。有关更多信息，请参阅 Section 17.6.3.4, “Undo Tablespaces”。

`AUTOEXTEND_SIZE`设置必须是 4M 的倍数。指定不是 4M 的倍数的`AUTOEXTEND_SIZE`设置会返回错误。

`AUTOEXTEND_SIZE`的默认设置为 0，这会导致表空间根据上述默认行为进行扩展。

允许的最大`AUTOEXTEND_SIZE`为 4GB。表空间的最大大小在 Section 17.22, “InnoDB Limits”中有描述。

最小的`AUTOEXTEND_SIZE`设置取决于`InnoDB`页面大小，如下表所示：

| InnoDB 页面大小 | 最小 AUTOEXTEND_SIZE |
| --- | --- |
| `4K` | `4M` |
| `8K` | `4M` |
| `16K` | `4M` |
| `32K` | `8M` |
| `64K` | `16M` |

默认的`InnoDB`页面大小为 16K（16384 字节）。要确定 MySQL 实例的`InnoDB`页面大小，请查询`innodb_page_size`设置：

```sql
mysql> SELECT @@GLOBAL.innodb_page_size;
+---------------------------+
| @@GLOBAL.innodb_page_size |
+---------------------------+
|                     16384 |
+---------------------------+
```

当更改表空间的`AUTOEXTEND_SIZE`设置后，随后发生的第一个扩展会将表空间大小增加到`AUTOEXTEND_SIZE`设置的倍数。随后的扩展将按配置的大小进行。

当使用非零 `AUTOEXTEND_SIZE` 设置创建文件表空间或通用表空间时，表空间将以指定的 `AUTOEXTEND_SIZE` 大小初始化。

不能使用 `ALTER TABLESPACE` 来配置文件表空间的 `AUTOEXTEND_SIZE`。必须使用 `ALTER TABLE`。

对于在文件表空间中创建的表，`SHOW CREATE TABLE` 仅在将 `AUTOEXTEND_SIZE` 配置为非零值时显示该选项。

要确定任何 `InnoDB` 表空间的 `AUTOEXTEND_SIZE`，请查询信息模式 `INNODB_TABLESPACES` 表。例如：

```sql
mysql> SELECT NAME, AUTOEXTEND_SIZE FROM INFORMATION_SCHEMA.INNODB_TABLESPACES 
       WHERE NAME LIKE 'test/t1';
+---------+-----------------+
| NAME    | AUTOEXTEND_SIZE |
+---------+-----------------+
| test/t1 |         4194304 |
+---------+-----------------+

mysql> SELECT NAME, AUTOEXTEND_SIZE FROM INFORMATION_SCHEMA.INNODB_TABLESPACES 
       WHERE NAME LIKE 'ts1';
+------+-----------------+
| NAME | AUTOEXTEND_SIZE |
+------+-----------------+
| ts1  |         4194304 |
+------+-----------------+
```

注意

`AUTOEXTEND_SIZE` 的值为 0，这是默认设置，意味着表空间根据上述默认表空间扩展行为进行扩展。
