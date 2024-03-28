> 原文：[`dev.mysql.com/doc/refman/8.0/en/insert-delayed.html`](https://dev.mysql.com/doc/refman/8.0/en/insert-delayed.html)

#### 15.2.7.3 INSERT DELAYED Statement

```sql
INSERT DELAYED ...
```

`DELAYED`选项用于`INSERT`语句，是 MySQL 对标准 SQL 的扩展。在 MySQL 的早期版本中，它可以用于某些类型的表（如`MyISAM`），这样当客户端使用`INSERT DELAYED`时，客户端立即收到服务器的确认，并且该行被排队等待在表不被其他线程使用时插入。

在 MySQL 5.6 中，`DELAYED`插入和替换已被弃用。在 MySQL 8.0 中，不再支持`DELAYED`。服务器会识别但忽略`DELAYED`关键字，将插入处理为非延迟插入，并生成一个`ER_WARN_LEGACY_SYNTAX_CONVERTED`警告：INSERT DELAYED 不再受支持。该语句已转换为 INSERT。`DELAYED`关键字计划在将来的版本中移除。
