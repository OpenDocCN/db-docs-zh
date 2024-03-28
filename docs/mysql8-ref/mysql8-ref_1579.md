# 22.3.4 关系表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-javascript-relational-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-javascript-relational-tables.html)

22.3.4.1 向表中插入记录

22.3.4.2 选择表

22.3.4.3 更新表

22.3.4.4 删除表

您还可以使用 X DevAPI 来处理关系表。在 MySQL 中，每个关系表都与特定的存储引擎相关联。本节中的示例使用 `world_x` 模式中的 `InnoDB` 表。

#### 确认模式

要显示分配给 `db` 全局变量的模式，请发出 `db`。

```sql
mysql-js> db
<Schema:world_x>
```

如果返回值不是`Schema:world_x`，则将`db`变量设置如下：

```sql
mysql-js> \use world_x
Schema `world_x` accessible through db.
```

#### 显示所有表

要显示 `world_x` 模式中的所有关系表，请在 `db` 对象上使用 `getTables()` 方法。

```sql
mysql-js> db.getTables()
{
    "city": <Table:city>,
    "country": <Table:country>,
    "countrylanguage": <Table:countrylanguage>
}
```

#### 基本表操作

按表范围的基本操作包括：

| 操作形式 | 描述 |
| --- | --- |
| `db.*`name`*.insert()` | insert() 方法向指定表中插入一个或多个记录。 |
| `db.*`name`*.select()` | select() 方法返回指定表中的一些或所有记录。 |
| `db.*`name`*.update()` | update() 方法更新指定表中的记录。 |
| `db.*`name`*.delete()` | delete() 方法从指定表中删除一个或多个记录。 |

#### 相关信息

+   有关处理关系表的更多信息，请参阅 处理关系表。

+   CRUD EBNF 定义 提供了操作的完整列表。

+   请参阅 第 22.3.2 节“下载并导入 world_x 数据库” 以获取设置 `world_x` 模式示例的说明。
