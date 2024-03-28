> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-ndb-sync-excluded-objects-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-ndb-sync-excluded-objects-table.html)

#### 29.12.12.2 `ndb_sync_excluded_objects` 表

此表提供有关`NDB`数据库对象的信息，这些对象无法在 NDB Cluster 字典和 MySQL 数据字典之间自动同步。

有关`NDB`数据库对象的示例信息，这些对象无法与 MySQL 数据字典同步：

```sql
mysql> SELECT * FROM performance_schema.ndb_sync_excluded_objects\G
*************************** 1\. row ***************************
SCHEMA_NAME: NULL
       NAME: lg1
       TYPE: LOGFILE GROUP
     REASON: Injected failure
*************************** 2\. row ***************************
SCHEMA_NAME: NULL
       NAME: ts1
       TYPE: TABLESPACE
     REASON: Injected failure
*************************** 3\. row ***************************
SCHEMA_NAME: db1
       NAME: NULL
       TYPE: SCHEMA
     REASON: Injected failure
*************************** 4\. row ***************************
SCHEMA_NAME: test
       NAME: t1
       TYPE: TABLE
     REASON: Injected failure
*************************** 5\. row ***************************
SCHEMA_NAME: test
       NAME: t2
       TYPE: TABLE
     REASON: Injected failure
*************************** 6\. row ***************************
SCHEMA_NAME: test
       NAME: t3
       TYPE: TABLE
     REASON: Injected failure
```

`ndb_sync_excluded_objects` 表具有以下列：

+   `SCHEMA_NAME`：未能同步的对象所在的模式（数据库）的名称；对于表空间和日志文件组，此处为`NULL`

+   `NAME`：未能同步的对象的名称；如果对象是模式，则为`NULL`

+   `TYPE`：未能同步的对象的类型；可以是`LOGFILE GROUP`、`TABLESPACE`、`SCHEMA`或`TABLE`之一

+   `REASON`：排除（阻止列入列表）对象的原因；即，未能同步此对象的原因

    可能的原因包括以下内容：

    +   `注入失败`

    +   `确定对象是否存在于 NDB 中失败`

    +   `确定对象是否存在于 DD 中失败`

    +   `在 DD 中删除对象失败`

    +   `获取分配给日志文件组的 undofiles 失败`

    +   `获取对象 ID 和版本失败`

    +   `在 DD 中安装对象失败`

    +   `获取分配给表空间的数据文件失败`

    +   `创建模式失败`

    +   `确定对象是否为本地表失败`

    +   `使表引用无效失败`

    +   `设置 NDB 对象的数据库名称失败`

    +   `获取表的额外元数据失败`

    +   `迁移带有额外元数据版本 1 的表失败`

    +   `从 DD 获取对象失败`

    +   `在 NDB 字典中表的定义已更改`

    +   `为表设置二进制日志记录失败`

    此列表可能不是详尽无遗的，并且可能会在未来的`NDB`版本中发生变化。

`ndb_sync_excluded_objects` 表在 NDB 8.0.21 中添加。
