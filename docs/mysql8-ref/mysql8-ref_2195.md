> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-disable-consumer.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-disable-consumer.html)

#### 30.4.4.5 ps_setup_disable_consumer() 存储过程

禁用包含指定参数的 Performance Schema 消费者。返回一个结果集，指示禁用了多少个消费者。已经禁用的消费者不计入其中。

##### 参数

+   `consumer VARCHAR(128)`: 用于匹配消费者名称的值，可以使用 `%consumer%` 作为 `LIKE` 模式匹配的操作数来识别消费者名称。

    值为 `''` 匹配所有消费者。

##### 示例

禁用所有语句消费者：

```sql
mysql> CALL sys.ps_setup_disable_consumer('statement');
+----------------------+
| summary              |
+----------------------+
| Disabled 4 consumers |
+----------------------+
```
