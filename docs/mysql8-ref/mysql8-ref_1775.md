> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-dict-obj-info.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-dict-obj-info.html)

#### 25.6.16.24 The ndbinfo dict_obj_info Table

`dict_obj_info` 表提供有关 `NDB` 数据字典（`DICT`）对象（如表和索引）的信息。 （可以查询 `dict_obj_types` 表以获取所有类型的列表。）此信息包括对象的类型、状态、父对象（如果有）和完全限定名称。

`dict_obj_info` 表包含以下列：

+   `type`

    `DICT` 对象类型；与 `dict_obj_types` 连接以获取名称

+   `id`

    对象标识符；对于磁盘数据撤销日志文件和数据文件，这与信息模式 `FILES` 表中的 `LOGFILE_GROUP_NUMBER` 列显示的值相同；对于撤销日志文件，它也与 `ndbinfo` `logbuffers` 和 `logspaces` 表中的 `log_id` 列显示的值相同

+   `version`

    对象版本

+   `state`

    对象状态；参见 Object::State 获取值和描述。

+   `parent_obj_type`

    父对象的类型（`dict_obj_types` 类型 ID）；0 表示对象没有父对象

+   `parent_obj_id`

    父对象 ID（如基表）；0 表示对象没有父对象

+   `fq_name`

    完全限定对象名称；对于表，格式为 `*`database_name`*/def/*`table_name`*`，对于主键，格式为 `sys/def/*`table_id`*/PRIMARY`，对于唯一键，格式为 `sys/def/*`table_id`*/*`uk_name`*$unique`
