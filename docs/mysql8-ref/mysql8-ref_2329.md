> 原文：[`dev.mysql.com/doc/refman/8.0/en/deleting-from-related-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/deleting-from-related-tables.html)

#### B.3.4.6 删除相关表中的行

如果`related_table`的`DELETE`语句的总长度超过了`max_allowed_packet`系统变量的默认值，您应该将其拆分为较小的部分，并执行多个`DELETE`语句。如果`related_column`被索引，您可能通过每个语句仅指定 100 到 1,000 个`related_column`值来获得最快的`DELETE`。如果`related_column`没有被索引，则速度与`IN`子句中的参数数量无关。
