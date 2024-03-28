> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-enable-consumer.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-enable-consumer.html)

#### 30.4.4.9 ps_setup_enable_consumer() 过程

启用包含参数的性能模式消费者名称。生成一个结果集，指示启用了多少个消费者。已经启用的消费者不计入其中。

##### 参数

+   `consumer VARCHAR(128)`: 用于匹配消费者名称的值，通过在`LIKE`模式匹配中使用`%consumer%`作为操作数来识别。

    值为`''`匹配所有消费者。

##### 示例

启用所有语句消费者：

```sql
mysql> CALL sys.ps_setup_enable_consumer('statement');
+---------------------+
| summary             |
+---------------------+
| Enabled 4 consumers |
+---------------------+
```
