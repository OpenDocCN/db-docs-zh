# 22.5.1 检查 X 插件安装情况

> 原文：[`dev.mysql.com/doc/refman/8.0/en/x-plugin-checking-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/x-plugin-checking-installation.html)

MySQL 8 默认启用了 X 插件，因此安装或升级到 MySQL 8 将使插件可用。您可以使用 `SHOW plugins` 语句查看插件列表来验证 X 插件是否已安装在 MySQL 服务器的实例上。

使用 MySQL Shell 来验证 X 插件是否已安装，请执行：

```sql
$> mysqlsh -u *user* --sqlc -P 3306 -e "SHOW plugins"
```

使用 MySQL Client 来验证 X 插件是否已安装，请执行：

```sql
$> mysql -u *user* -p -e "SHOW plugins"
```

如果 X 插件已安装，则示例结果如下所示：

```sql
+----------------------------+----------+--------------------+---------+---------+
| Name                       | Status   | Type               | Library | License |
+----------------------------+----------+--------------------+---------+---------+

...

| mysqlx                     | ACTIVE   | DAEMON             | NULL    | GPL     |

...

+----------------------------+----------+--------------------+---------+---------+
```
