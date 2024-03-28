# 17.9.2 InnoDB 页面压缩

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-page-compression.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-page-compression.html)

`InnoDB`支持位于 file-per-table 表空间中的表的页面级压缩。此功能称为*透明页压缩*。通过在`CREATE TABLE`或`ALTER TABLE`中指定`COMPRESSION`属性来启用页面压缩。支持的压缩算法包括`Zlib`和`LZ4`。

#### 支持的平台

页面压缩需要稀疏文件和空洞打孔支持。在 Windows 上，页面压缩受 NTFS 支持，在以下 MySQL 支持的 Linux 平台子集上，内核级别提供空洞打孔支持：

+   RHEL 7 及使用内核版本 3.10.0-123 或更高的衍生发行版

+   OEL 5.10（UEK2）内核版本 2.6.39 或更高

+   OEL 6.5（UEK3）内核版本 3.8.13 或更高

+   OEL 7.0 内核版本 3.8.13 或更高

+   SLE11 内核版本 3.0-x

+   SLE12 内核版本 3.12-x

+   OES11 内核版本 3.0-x

+   Ubuntu 14.0.4 LTS 内核版本 3.13 或更高

+   Ubuntu 12.0.4 LTS 内核版本 3.2 或更高

+   Debian 7 内核版本 3.2 或更高

注意

给定 Linux 发行版的所有可用文件系统可能不支持空洞打孔。

#### 页面压缩的工作原理

当写入页面时，使用指定的压缩算法对其进行压缩。压缩后的数据写入磁盘，其中空洞打孔机制释放页面末尾的空块。如果压缩失败，则按原样写出数据。

#### Linux 系统上的空洞打孔大小

在 Linux 系统上，文件系统块大小是用于空洞打孔的单位大小。因此，只有当页面数据可以压缩到小于或等于`InnoDB`页面大小减去文件系统块大小时，页面压缩才有效。例如，如果`innodb_page_size=16K`且文件系统块大小为 4K，则页面数据必须压缩到小于或等于 12K 才能进行空洞打孔。

#### Windows 系统上的空洞打孔大小

在 Windows 系统上，稀疏文件的基础架构基于 NTFS 压缩。空洞打孔大小是 NTFS 压缩单元，它是 NTFS 群集大小的 16 倍。群集大小及其压缩单元如下表所示：

**表 17.14 Windows NTFS 群集大小和压缩单元**

| 群集大小 | 压缩单元 |
| --- | --- |
| 512 字节 | 8 KB |
| 1 KB | 16 KB |
| 2 KB | 32 KB |
| 4 KB | 64 KB |

Windows 系统上的页面压缩仅在页面数据可以压缩到小于或等于`InnoDB`页面大小减去压缩单元大小时才有效。

默认的 NTFS 簇大小为 4KB，压缩单元大小为 64KB。这意味着对于开箱即用的 Windows NTFS 配置，页面压缩没有任何好处，因为最大的`innodb_page_size`也是 64KB。

要使页面压缩在 Windows 上工作，文件系统必须使用小于 4K 的簇大小，并且`innodb_page_size`必须至少是压缩单元大小的两倍。例如，要使页面压缩在 Windows 上工作，可以使用 512 字节的簇大小（具有 8KB 的压缩单元）构建文件系统，并使用大于 16K 的`innodb_page_size`值初始化`InnoDB`。

#### 启用页面压缩

要启用页面压缩，请在`CREATE TABLE`语句中指定`COMPRESSION`属性。例如：

```sql
CREATE TABLE t1 (c1 INT) COMPRESSION="zlib";
```

您还可以在`ALTER TABLE`语句中启用页面压缩。但是，`ALTER TABLE ... COMPRESSION`仅更新表空间压缩属性。设置新的压缩算法后对表空间的写入使用新设置，但要将新的压缩算法应用于现有页面，必须使用`OPTIMIZE TABLE`重建表。

```sql
ALTER TABLE t1 COMPRESSION="zlib";
OPTIMIZE TABLE t1;
```

#### 禁用页面压缩

要禁用页面压缩，请使用`ALTER TABLE`设置`COMPRESSION=None`。设置`COMPRESSION=None`后对表空间的写入不再使用页面压缩。要取消现有页面的压缩，必须在设置`COMPRESSION=None`后使用`OPTIMIZE TABLE`重建表。

```sql
ALTER TABLE t1 COMPRESSION="None";
OPTIMIZE TABLE t1;
```

#### 页面压缩元数据

页面压缩元数据位于信息模式`INNODB_TABLESPACES`表中，以下列中：

+   `FS_BLOCK_SIZE`：文件系统块大小，用于空洞打孔的单位大小。

+   `FILE_SIZE`：文件的表面大小，表示未压缩的文件的最大大小。

+   `ALLOCATED_SIZE`：文件的实际大小，即磁盘上分配的空间量。

注意

在类 Unix 系统上，`ls -l *`tablespace_name`*.ibd`显示文件的表面文件大小（等同于`FILE_SIZE`）以字节为单位。要查看磁盘上分配的实际空间量（等同于`ALLOCATED_SIZE`），请使用`du --block-size=1 *`tablespace_name`*.ibd`。`--block-size=1`选项以字节而不是块打印分配的空间，以便与`ls -l`输出进行比较。

使用 `SHOW CREATE TABLE` 查看当前页面压缩设置 (`Zlib`, `Lz4`, 或 `None`)。一个表可能包含具有不同压缩设置的页面。

在以下示例中，从信息模式 `INNODB_TABLESPACES` 表中检索了员工表的页面压缩元数据。

```sql
# Create the employees table with Zlib page compression

CREATE TABLE employees (
    emp_no      INT             NOT NULL,
    birth_date  DATE            NOT NULL,
    first_name  VARCHAR(14)     NOT NULL,
    last_name   VARCHAR(16)     NOT NULL,
    gender      ENUM ('M','F')  NOT NULL,
    hire_date   DATE            NOT NULL,
    PRIMARY KEY (emp_no)
) COMPRESSION="zlib";

# Insert data (not shown)

# Query page compression metadata in INFORMATION_SCHEMA.INNODB_TABLESPACES

mysql> SELECT SPACE, NAME, FS_BLOCK_SIZE, FILE_SIZE, ALLOCATED_SIZE FROM
       INFORMATION_SCHEMA.INNODB_TABLESPACES WHERE NAME='employees/employees'\G
*************************** 1\. row ***************************
SPACE: 45
NAME: employees/employees
FS_BLOCK_SIZE: 4096
FILE_SIZE: 23068672
ALLOCATED_SIZE: 19415040
```

员工表的页面压缩元数据显示，表面文件大小为 23068672 字节，而实��文件大小（带有页面压缩）为 19415040 字节。文件系统块大小为 4096 字节，这是用于打孔的块大小。

#### 识别使用页面压缩的表

要识别启用页面压缩的表，可以检查信息模式 `TABLES` 表的 `CREATE_OPTIONS` 列，查看定义了 `COMPRESSION` 属性的表：

```sql
mysql> SELECT TABLE_NAME, TABLE_SCHEMA, CREATE_OPTIONS FROM INFORMATION_SCHEMA.TABLES 
       WHERE CREATE_OPTIONS LIKE '%COMPRESSION=%';
+------------+--------------+--------------------+
| TABLE_NAME | TABLE_SCHEMA | CREATE_OPTIONS     |
+------------+--------------+--------------------+
| employees  | test         | COMPRESSION="zlib" |
+------------+--------------+--------------------+
```

`SHOW CREATE TABLE` 还显示了如果使用的话的 `COMPRESSION` 属性。

#### 页面压缩的限制和使用注意事项

+   如果文件系统块大小（或 Windows 上的压缩单元大小）* 2 > `innodb_page_size`，则禁用页面压缩。

+   不支持位于共享表空间中的表进行页面压缩，这包括系统表空间、临时表空间和一般表空间。

+   不支持撤销日志表空间进行页面压缩。

+   不支持重做日志页面进行页面压缩。

+   用于空间索引的 R 树页面不进行压缩。

+   属于压缩表 (`ROW_FORMAT=COMPRESSED`) 的页面保持不变。

+   在恢复期间，更新的页面以未压缩形式写出。

+   在不支持使用的压缩算法的服务器上加载页面压缩的表空间会导致 I/O 错误。

+   在降级到不支持页面压缩的早期版本的 MySQL 之前，需要取消对使用页面压缩功能的表进行解压缩。要对表进行解压缩，运行 `ALTER TABLE ... COMPRESSION=None` 和 `OPTIMIZE TABLE`。

+   如果在 Linux 和 Windows 服务器上都可用使用的压缩算法，则可以在页面压缩的表空间之间进行复制。

+   将一个页面压缩的表空间文件从一个主机移动到另一个主机时，需要使用一个能保留稀疏文件的实用程序来保留页面压缩。

+   在 Fusion-io 硬件上使用 NVMFS 可以实现更好的页面压缩效果，因为 NVMFS 设计用于利用打孔功能。

+   使用具有较大`InnoDB`页面大小和相对较小文件系统块大小的页面压缩功能可能导致写放大。例如，具有 64KB 的最大`InnoDB`页面大小和 4KB 文件系统块大小可能会提高压缩率，但也可能增加对缓冲池的需求，导致增加的 I/O 和潜在的写放大。
