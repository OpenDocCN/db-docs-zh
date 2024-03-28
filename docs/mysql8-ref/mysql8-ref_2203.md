> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-reset-to-default.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-reset-to-default.html)

#### 30.4.4.13 `ps_setup_reset_to_default()` 过程

将性能模式配置重置为默认设置。

##### 参数

+   `in_verbose BOOLEAN`: 是否在过程执行期间显示有关每个设置阶段的信息。这包括执行的 SQL 语句。

##### 示例

```sql
mysql> CALL sys.ps_setup_reset_to_default(TRUE)\G
*************************** 1\. row ***************************
status: Resetting: setup_actors
DELETE
FROM performance_schema.setup_actors
WHERE NOT (HOST = '%' AND USER = '%' AND ROLE = '%')

*************************** 1\. row ***************************
status: Resetting: setup_actors
INSERT IGNORE INTO performance_schema.setup_actors
VALUES ('%', '%', '%')

...
```
