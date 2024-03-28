# 16.6 序列化字典信息（SDI）

> 原文：[`dev.mysql.com/doc/refman/8.0/en/serialized-dictionary-information.html`](https://dev.mysql.com/doc/refman/8.0/en/serialized-dictionary-information.html)

除了在数据字典中存储有关数据库对象的元数据外，MySQL 还以序列化形式存储它。这些数据被称为序列化字典信息（SDI）。`InnoDB`将 SDI 数据存储在其表空间文件中。`NDBCLUSTER`将 SDI 数据存储在 NDB 字典中。其他存储引擎将 SDI 数据存储在为给定表创建的`.sdi`文件中，该文件位于表的数据库目录中。SDI 数据以紧凑的`JSON`格式生成。

所有`InnoDB`表空间文件中都存在序列化字典信息（SDI），临时表空间和撤销表空间文件除外。`InnoDB`表空间文件中的 SDI 记录仅描述表空间中包含的表和表对象。

SDI 数据通过对表进行 DDL 操作或`CHECK TABLE FOR UPGRADE`来更新。当 MySQL 服务器升级到新版本时，SDI 数据不会被更新。

SDI 数据的存在提供了元数据冗余。例如，如果数据字典不可用，可以使用**ibd2sdi**工具直接从`InnoDB`表空间文件中提取对象元数据。

对于`InnoDB`，一个 SDI 记录需要一个索引页，默认大小为 16KB。但是，SDI 数据经过压缩以减少存储占用空间。

对于由多个表空间组成的分区`InnoDB`表，SDI 数据存储在第一个分区的表空间文件中。

MySQL 服务器在 DDL 操作期间使用内部 API 来创建和维护 SDI 记录。

`IMPORT TABLE`语句根据`.sdi`文件中包含的信息导入`MyISAM`表。有关更多信息，请参见 Section 15.2.6, “IMPORT TABLE Statement”。
