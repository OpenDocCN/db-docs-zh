> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-schema-object-overview.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-schema-object-overview.html)

#### 30.4.3.26 模式对象概述视图

此视图总结了每个模式中的对象类型。默认情况下，按模式和对象类型对行进行排序。

注意

对于具有大量对象的 MySQL 实例，执行此视图可能需要很长时间。

`schema_object_overview` 视图具有以下列：

+   `db`

    模式名称。

+   `object_type`

    对象类型：`BASE TABLE`，`INDEX (*`index_type`*)`，`EVENT`，`FUNCTION`，`PROCEDURE`，`TRIGGER`，`VIEW`。

+   `count`

    给定类型的模式中对象的数量。
