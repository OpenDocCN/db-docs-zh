# 15.1.25 删除事件语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/drop-event.html`](https://dev.mysql.com/doc/refman/8.0/en/drop-event.html)

```sql
DROP EVENT [IF EXISTS] *event_name*
```

这个语句会删除名为*`event_name`*的事件。该事件立即停止活动，并从服务器完全删除。

如果事件不存在，则会出现错误 ERROR 1517 (HY000): 未知事件 '*`event_name`*'。您可以使用 `IF EXISTS` 来覆盖此错误，并使语句对不存在的事件生成警告。

这个语句需要对要删除事件所属模式的`EVENT`权限。
