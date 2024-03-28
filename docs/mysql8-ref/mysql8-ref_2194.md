> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-disable-background-threads.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-disable-background-threads.html)

#### 30.4.4.4 `ps_setup_disable_background_threads()` 过程

禁用性能模式对所有后台线程的仪器化。生成一个结果集，指示禁用了多少个后台线程。已经禁用的线程不计入其中。

##### 参数

无。

##### 示例

```sql
mysql> CALL sys.ps_setup_disable_background_threads();
+--------------------------------+
| summary                        |
+--------------------------------+
| Disabled 24 background threads |
+--------------------------------+
```
