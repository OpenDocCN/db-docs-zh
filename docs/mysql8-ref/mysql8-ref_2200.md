> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-enable-instrument.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-enable-instrument.html)

#### 30.4.4.10 `ps_setup_enable_instrument()`过程

启用包含参数的性能模式仪器的名称。生成一个结果集，指示启用了多少个仪器。已启用的仪器不计入其中。

##### 参数

+   `in_pattern VARCHAR(128)`: 用于匹配仪器名称的值，通过在`LIKE`模式匹配中使用`%in_pattern%`作为操作数来识别。

    一个值为`''`匹配所有仪器。

##### 示例

启用特定仪器：

```sql
mysql> CALL sys.ps_setup_enable_instrument('wait/lock/metadata/sql/mdl');
+----------------------+
| summary              |
+----------------------+
| Enabled 1 instrument |
+----------------------+
```

启用所有互斥仪器：

```sql
mysql> CALL sys.ps_setup_enable_instrument('mutex');
+-------------------------+
| summary                 |
+-------------------------+
| Enabled 177 instruments |
+-------------------------+
```
