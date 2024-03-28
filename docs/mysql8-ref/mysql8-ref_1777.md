> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-dict-obj-types.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-dict-obj-types.html)

#### 25.6.16.26 ndbinfo dict_obj_types 表

`dict_obj_types` 表是一个静态表，列出了 NDB 内核中使用的可能的字典对象类型。这些类型与 NDB API 中的 `Object::Type` 中定义的类型相同。

`dict_obj_types` 表包含以下列：

+   `type_id`

    此类型的类型 ID

+   `type_name`

    此类型的名称
