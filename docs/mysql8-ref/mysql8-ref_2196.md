> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-disable-instrument.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-disable-instrument.html)

#### 30.4.4.6 ps_setup_disable_instrument() 过程

禁用包含参数的性能模式仪器名称。生成一个结果集，指示禁用了多少个仪器。已禁用的仪器不计入其中。

##### 参数

+   `in_pattern VARCHAR(128)`: 用于匹配仪器名称的值，通过在`LIKE`模式匹配中使用`%in_pattern%`作为操作数来识别。

    `''`的值匹配所有仪器。

##### 示例

禁用特定仪器：

```sql
mysql> CALL sys.ps_setup_disable_instrument('wait/lock/metadata/sql/mdl');
+-----------------------+
| summary               |
+-----------------------+
| Disabled 1 instrument |
+-----------------------+
```

禁用所有互斥仪器：

```sql
mysql> CALL sys.ps_setup_disable_instrument('mutex');
+--------------------------+
| summary                  |
+--------------------------+
| Disabled 177 instruments |
+--------------------------+
```
