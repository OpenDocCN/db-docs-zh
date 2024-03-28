# 5.6.9 使用 AUTO_INCREMENT

> 原文：[`dev.mysql.com/doc/refman/8.0/en/example-auto-increment.html`](https://dev.mysql.com/doc/refman/8.0/en/example-auto-increment.html)

`AUTO_INCREMENT`属性可用于为新行生成唯一标识：

```sql
CREATE TABLE animals (
     id MEDIUMINT NOT NULL AUTO_INCREMENT,
     name CHAR(30) NOT NULL,
     PRIMARY KEY (id)
);

INSERT INTO animals (name) VALUES
    ('dog'),('cat'),('penguin'),
    ('lax'),('whale'),('ostrich');

SELECT * FROM animals;
```

返回：

```sql
+----+---------+
| id | name    |
+----+---------+
|  1 | dog     |
|  2 | cat     |
|  3 | penguin |
|  4 | lax     |
|  5 | whale   |
|  6 | ostrich |
+----+---------+
```

未为`AUTO_INCREMENT`列指定值，因此 MySQL 会自动分配序列号。您也可以显式地将 0 分配给该列以生成序列号，除非启用了`NO_AUTO_VALUE_ON_ZERO` SQL 模式。例如：

```sql
INSERT INTO animals (id,name) VALUES(0,'groundhog');
```

如果列声明为`NOT NULL`，也可以将`NULL`分配给该列以生成序列号。例如：

```sql
INSERT INTO animals (id,name) VALUES(NULL,'squirrel');
```

当您向`AUTO_INCREMENT`列插入任何其他值时，该列将设置为该值，并且序列将被重置，以便下一个自动生成的值从最大列值顺序生成。例如：

```sql
INSERT INTO animals (id,name) VALUES(100,'rabbit');
INSERT INTO animals (id,name) VALUES(NULL,'mouse');
SELECT * FROM animals;
+-----+-----------+
| id  | name      |
+-----+-----------+
|   1 | dog       |
|   2 | cat       |
|   3 | penguin   |
|   4 | lax       |
|   5 | whale     |
|   6 | ostrich   |
|   7 | groundhog |
|   8 | squirrel  |
| 100 | rabbit    |
| 101 | mouse     |
+-----+-----------+
```

更新现有的`AUTO_INCREMENT`列值也会重置`AUTO_INCREMENT`序列。

您可以使用`LAST_INSERT_ID()` SQL 函数或`mysql_insert_id()` C API 函数检索最近自动生成的`AUTO_INCREMENT`值。这些函数是特定于连接的，因此它们的返回值不受另一个执行插入操作的连接的影响。

对于`AUTO_INCREMENT`列，请使用足够大以容纳所需最大序列值的最小整数数据类型。当列达到数据类型的上限时，下一次尝试生成序列号将失败。如果可能的话，请使用`UNSIGNED`属性以允许更大的范围。例如，如果使用`TINYINT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")，则最大允许的序列号为 127。对于`TINYINT UNSIGNED` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")，最大值为 255。请参阅第 13.1.2 节，“整数类型（精确值） - INTEGER、INT、SMALLINT、TINYINT、MEDIUMINT、BIGINT” - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")以获取所有整数类型的范围。

注意

对于多行插入，`LAST_INSERT_ID()` 和 `mysql_insert_id()` 实际上返回插入行中*第一个*的`AUTO_INCREMENT`键。这使得可以在复制设置中的其他服务器上正确重现多行插入。

要从 1 开始的`AUTO_INCREMENT`值，可以使用`CREATE TABLE` 或 `ALTER TABLE` 设置该值，如下所示：

```sql
mysql> ALTER TABLE tbl AUTO_INCREMENT = 100;
```

#### InnoDB 注意事项

有关特定于`InnoDB`的`AUTO_INCREMENT`用法的信息，请参阅第 17.6.1.6 节，“InnoDB 中的 AUTO_INCREMENT 处理”。

#### MyISAM 注意事项

+   对于`MyISAM`表，您可以在多列索引中的辅助列上指定`AUTO_INCREMENT`。在这种情况下，`AUTO_INCREMENT`列的生成值计算为`MAX(*auto_increment_column*) + 1 WHERE prefix=*given-prefix*`。当您想要将数据放入有序组时，这是很有用的。

    ```sql
    CREATE TABLE animals (
        grp ENUM('fish','mammal','bird') NOT NULL,
        id MEDIUMINT NOT NULL AUTO_INCREMENT,
        name CHAR(30) NOT NULL,
        PRIMARY KEY (grp,id)
    ) ENGINE=MyISAM;

    INSERT INTO animals (grp,name) VALUES
        ('mammal','dog'),('mammal','cat'),
        ('bird','penguin'),('fish','lax'),('mammal','whale'),
        ('bird','ostrich');

    SELECT * FROM animals ORDER BY grp,id;
    ```

    返回：

    ```sql
    +--------+----+---------+
    | grp    | id | name    |
    +--------+----+---------+
    | fish   |  1 | lax     |
    | mammal |  1 | dog     |
    | mammal |  2 | cat     |
    | mammal |  3 | whale   |
    | bird   |  1 | penguin |
    | bird   |  2 | ostrich |
    +--------+----+---------+
    ```

    在这种情况下（当`AUTO_INCREMENT`列是多列索引的一部分时），如果删除任何组中具有最大`AUTO_INCREMENT`值的行，则`AUTO_INCREMENT`值将被重用。即使对于通常不会重用`AUTO_INCREMENT`值的`MyISAM`表也是如此。

+   如果`AUTO_INCREMENT`列是多个索引的一部分，MySQL 会使用以`AUTO_INCREMENT`列开头的索引生成序列值（如果有的话）。例如，如果`animals`表包含索引`PRIMARY KEY (grp, id)`和`INDEX (id)`，MySQL 会忽略`PRIMARY KEY`来生成序列值。因此，表中将包含一个单一序列，而不是每个`grp`值一个序列。

#### 进一步阅读

有关`AUTO_INCREMENT`的更多信息，请参阅此��：

+   如何将`AUTO_INCREMENT`属性分配给列：第 15.1.20 节，“CREATE TABLE 语句”和第 15.1.9 节，“ALTER TABLE 语句”。

+   根据`NO_AUTO_VALUE_ON_ZERO` SQL 模式，`AUTO_INCREMENT`的行为如何：第 7.1.11 节，“服务器 SQL 模式”。

+   如何使用`LAST_INSERT_ID()`函数找到包含最新`AUTO_INCREMENT`值的行：第 14.15 节，“信息函数”。

+   设置要使用的`AUTO_INCREMENT`值：第 7.1.8 节，“服务器系统变量”。

+   第 17.6.1.6 节，“InnoDB 中的 AUTO_INCREMENT 处理”

+   `AUTO_INCREMENT`和复制：第 19.5.1.1 节，“复制和 AUTO_INCREMENT”。

+   与`AUTO_INCREMENT`相关的服务器系统变量（`auto_increment_increment`和`auto_increment_offset`）可用于复制：第 7.1.8 节，“服务器系统变量”。
