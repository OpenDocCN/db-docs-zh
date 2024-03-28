> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-extract-schema-from-file-name.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-extract-schema-from-file-name.html)

#### 30.4.5.1 extract_schema_from_file_name() 函数

给定文件路径名，返回表示模式名称的路径组件。此函数假定文件名位于模式目录中。因此，它不适用于使用其自己的`DATA_DIRECTORY`表选项定义的分区或表。

此函数在从包含文件路径名的性能模式中提取文件 I/O 信息时非常有用。它提供了一种方便的方式来显示模式名称，这比完整路径名更容易理解，并且可以用于与对象模式名称进行连接。

##### 参数

+   `path VARCHAR(512)`: 数据文件的完整路径，用于提取模式名称。

##### 返回值

一个`VARCHAR(64)`值。

##### 示例

```sql
mysql> SELECT sys.extract_schema_from_file_name('/usr/local/mysql/data/world/City.ibd');
+---------------------------------------------------------------------------+
| sys.extract_schema_from_file_name('/usr/local/mysql/data/world/City.ibd') |
+---------------------------------------------------------------------------+
| world                                                                     |
+---------------------------------------------------------------------------+
```
