# 15.1.35 DROP VIEW Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/drop-view.html`](https://dev.mysql.com/doc/refman/8.0/en/drop-view.html)

```sql
DROP VIEW [IF EXISTS]
    *view_name* [, *view_name*] ...
    [RESTRICT | CASCADE]
```

`DROP VIEW`用于移除一个或多个视图。对于每个视图，您必须具有`DROP`权限。

如果参数列表中命名的任何视图不存在，则该语句将失败，并显示一个错误，指示无法删除的不存在视图的名称，并且不会进行任何更改。

注意

在 MySQL 5.7 及更早版本中，如果参数列表中命名的任何视图不存在，`DROP VIEW`会返回错误，但也会删除列表中存在的所有视图。由于 MySQL 8.0 中行为的更改，当在 MySQL 5.7 复制源服务器上复制到 MySQL 8.0 副本时，对于部分完成的`DROP VIEW`操作会失败。为避免此失败场景，在`DROP VIEW`语句中使用`IF EXISTS`语法可防止出现视图不存在的错误。更多信息，请参见 Section 15.1.1, “Atomic Data Definition Statement Support”。

`IF EXISTS`子句可防止出现视图不存在的错误。当使用此子句时，对于每个不存在的视图会生成一个`NOTE`。参见 Section 15.7.7.42, “SHOW WARNINGS Statement”。

`RESTRICT`和`CASCADE`，如果给定，将被解析并忽略。
