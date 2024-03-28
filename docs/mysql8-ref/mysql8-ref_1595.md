# 22.4.4 关系表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-relational-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python-relational-tables.html)

22.4.4.1 向表中插入记录

22.4.4.2 选择表

22.4.4.3 更新表

22.4.4.4 删除表

您也可以使用 X DevAPI 来处理关系表。在 MySQL 中，每个关系表都与特定的存储引擎相关联。本节中的示例使用`InnoDB`表在`world_x`模式中。

#### 确认模式

要显示分配给`db`全局变量的模式，请发出`db`。

```sql
mysql-py> db
<Schema:world_x>
```

如果返回的值不是`Schema:world_x`，请将`db`变量设置如下：

```sql
mysql-py> \use world_x
Schema `world_x` accessible through db.
```

#### 显示所有表

要显示`world_x`模式中的所有关系表，请在`db`对象上使用`get_tables()`方法。

```sql
mysql-py> db.get_tables()
[
    <Table:city>,
    <Table:country>,
    <Table:countrylanguage>
]
```

#### 基本表操作

由表限定的基本操作包括：

| 操作形式 | 描述 |
| --- | --- |
| `db.*`name`*.insert()` | insert() 方法将一个或多个记录插入到指定的表中。 |
| `db.*`name`*.select()` | select() 方法返回指定表中的一些或所有记录。 |
| `db.*`name`*.update()` | update() 方法更新指定表中的记录。 |
| `db.*`name`*.delete()` | delete() 方法从指定的表中删除一个或多个记录。 |

#### 相关信息

+   有关更多信息，请参阅处理关系表。

+   CRUD EBNF 定义 提供了操作的完整列表。

+   请参阅第 22.4.2 节，“下载和导入 world_x 数据库”，了解设置`world_x`模式示例的说明。
