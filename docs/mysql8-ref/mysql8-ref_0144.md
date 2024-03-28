# 5.6.6 使用外键

> 原文：[`dev.mysql.com/doc/refman/8.0/en/example-foreign-keys.html`](https://dev.mysql.com/doc/refman/8.0/en/example-foreign-keys.html)

MySQL 支持外键，允许跨表引用相关数据，并支持外键约束，有助于保持相关数据的一致性。

外键关系涉及一个持有初始列值的父表，以及一个引用父列值的子表。外键约束定义在子表上。

以下示例通过单列外键关联`parent`和`child`表，并展示了外键约束如何强制执行引用完整性。

使用以下 SQL 语句创建父表和子表：

```sql
CREATE TABLE parent (
    id INT NOT NULL,
    PRIMARY KEY (id)
) ENGINE=INNODB;

CREATE TABLE child (
    id INT,
    parent_id INT,
    INDEX par_ind (parent_id),
    FOREIGN KEY (parent_id)
        REFERENCES parent(id)
) ENGINE=INNODB;
```

插入一行到父表中，如下所示：

```sql
mysql> INSERT INTO parent (id) VALUES ROW(1);
```

验证数据是否已插入。你可以通过简单地选择所有`parent`表中的行来做到这一点，如下所示：

```sql
mysql> TABLE parent;
+----+
| id |
+----+
|  1 |
+----+
```

使用以下 SQL 语句向子表中插入一行：

```sql
mysql> INSERT INTO child (id,parent_id) VALUES ROW(1,1);
```

插入操作成功是因为`parent_id` 1 存在于父表中。

尝试将具有在父表中不存在的`parent_id`值的行插入到子表中会被拒绝，并显示错误，如下所示：

```sql
mysql> INSERT INTO child (id,parent_id) VALUES ROW(2,2);
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails 
(`test`.`child`, CONSTRAINT `child_ibfk_1` FOREIGN KEY (`parent_id`) 
REFERENCES `parent` (`id`))
```

这个操作失败是因为指定的`parent_id`值在父表中不存在。

尝试删除先前插入到父表中的行也会失败，如下所示：

```sql
mysql> DELETE FROM parent WHERE id = 1;
ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails 
(`test`.`child`, CONSTRAINT `child_ibfk_1` FOREIGN KEY (`parent_id`) 
REFERENCES `parent` (`id`))
```

这个操作失败是因为子表中的记录包含了引用的 id（`parent_id`）值。

当一个操作影响到父表中具有匹配行的键值时，结果取决于`FOREIGN KEY`子句的`ON UPDATE`和`ON DELETE`子句指定的引用动作。省略`ON DELETE`和`ON UPDATE`子句（如当前子表定义中）等同于指定`RESTRICT`选项，它拒绝影响父表中具有匹配行的键值的操作。

为了演示`ON DELETE`和`ON UPDATE`引用动作，删除子表并重新创建它以包括带有`CASCADE`选项的`ON UPDATE`和`ON DELETE`子句。`CASCADE`选项在删除或更新父表中的行时，会自动删除或更新子表中匹配的行。

```sql
DROP TABLE child;

CREATE TABLE child (
    id INT,
    parent_id INT,
    INDEX par_ind (parent_id),
    FOREIGN KEY (parent_id)
        REFERENCES parent(id)
        ON UPDATE CASCADE
        ON DELETE CASCADE
) ENGINE=INNODB;
```

使用下面显示的语句向子表中插入一些行：

```sql
mysql> INSERT INTO child (id,parent_id) VALUES ROW(1,1), ROW(2,1), ROW(3,1);
```

验证数据是否已插入，如下所示：

```sql
mysql> TABLE child;
+------+-----------+
| id   | parent_id |
+------+-----------+
|    1 |         1 |
|    2 |         1 |
|    3 |         1 |
+------+-----------+
```

更新父表中的 ID，将其从 1 更改为 2，使用下面显示的 SQL 语句：

```sql
mysql> UPDATE parent SET id = 2 WHERE id = 1;
```

通过选择所有父表中的行来验证更新是否成功，如下所示：

```sql
mysql> TABLE parent;
+----+
| id |
+----+
|  2 |
+----+
```

验证`ON UPDATE CASCADE`引用动作是否已更新子表，如下所示：

```sql
mysql> TABLE child;
+------+-----------+
| id   | parent_id |
+------+-----------+
|    1 |         2 |
|    2 |         2 |
|    3 |         2 |
+------+-----------+
```

为了演示`ON DELETE CASCADE`引用动作，删除父表中`parent_id = 2`的记录；这将删除父表中的所有记录。

```sql
mysql> DELETE FROM parent WHERE id = 2;
```

因为子表中的所有记录都与`parent_id = 2`相关联，所以`ON DELETE CASCADE`参照操作会从子表中删除所有记录，如下所示：

```sql
mysql> TABLE child;
Empty set (0.00 sec)
```

关于外键约束的更多信息，请参见第 15.1.20.5 节，“外键约束”。
