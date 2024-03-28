> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-table-update.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-table-update.html)

#### 22.4.4.3 更新表格

您可以使用`update()`方法修改表中的一个或多个记录。`update()`方法通过过滤查询以仅包括要更新的记录，然后对这些记录应用您指定的操作来工作。

要在城市表中替换城市名称，请将新城市名称传递给`set()`方法。然后，将要定位和替换的城市名称传递给`where()`方法。以下示例将城市 Peking 替换为 Beijing。

```sql
mysql-py> db.city.update().set("Name", "Beijing").where("Name = 'Peking'")
```

使用`select()`方法验证更改。

```sql
mysql-py> db.city.select(["ID", "Name", "CountryCode", "District", "Info"]).where("Name = 'Beijing'")
+------+-----------+-------------+----------+-----------------------------+
| ID   | Name      | CountryCode | District | Info                        |
+------+-----------+-------------+----------+-----------------------------+
| 1891 | Beijing   | CHN         | Peking   | {"Population": 7472000}     |
+------+-----------+-------------+----------+-----------------------------+
1 row in set (0.00 sec)
```

##### 相关信息

+   查看 TableUpdateFunction 以获取完整的语法定义。
