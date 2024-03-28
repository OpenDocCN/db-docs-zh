> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-launching.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-launching.html)

#### 20.2.1.4 启动 Group Replication

首先需要确保在服务器 s1 上安装了 Group Replication 插件。如果在选项文件中使用了 `plugin_load_add='group_replication.so'`，那么 Group Replication 插件已经安装好了，您可以继续下一步操作。否则，您必须手动安装该插件；要做到这一点，使用 **mysql** 客户端连接到服务器，并执行这里显示的 SQL 语句：

```sql
mysql> INSTALL PLUGIN group_replication SONAME 'group_replication.so';
```

重要提示

在加载 Group Replication 之前，`mysql.session` 用户必须存在。`mysql.session` 是在 MySQL 版本 8.0.2 中添加的。如果您的数据字典是使用早期版本初始化的，则必须执行 MySQL 升级过程（参见 第三章，*升级 MySQL*）。如果未运行升级，则 Group Replication 在启动时会出现错误消息，指出尝试使用用户 mysql.session@localhost 访问服务器时出错。确保用户存在于服务器中，并且在服务器更新后运行了 mysql_upgrade。

要检查插件是否成功安装，请执行 `SHOW PLUGINS;` 并检查输出。应该显示类似于以下内容：

```sql
mysql> SHOW PLUGINS;
+----------------------------+----------+--------------------+----------------------+-------------+
| Name                       | Status   | Type               | Library              | License     |
+----------------------------+----------+--------------------+----------------------+-------------+
| binlog                     | ACTIVE   | STORAGE ENGINE     | NULL                 | PROPRIETARY |

(...)

| group_replication          | ACTIVE   | GROUP REPLICATION  | group_replication.so | PROPRIETARY |
+----------------------------+----------+--------------------+----------------------+-------------+
```
