> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-enable-background-threads.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-enable-background-threads.html)

#### 30.4.4.8 **ps_setup_enable_background_threads()** 过程

启用性能模式对所有后台线程进行仪器化。生成一个结果集，指示启用了多少个后台线程。已经启用的线程不计入其中。

##### 参数

无。

##### 示例

```sql
mysql> CALL sys.ps_setup_enable_background_threads();
+-------------------------------+
| summary                       |
+-------------------------------+
| Enabled 24 background threads |
+-------------------------------+
```
