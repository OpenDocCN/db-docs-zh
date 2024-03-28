> 原文：[`dev.mysql.com/doc/refman/8.0/en/ansi-diff-foreign-keys.html`](https://dev.mysql.com/doc/refman/8.0/en/ansi-diff-foreign-keys.html)

#### 1.6.2.3 外键约束的差异

MySQL 对外键约束的实现与 SQL 标准在以下关键方面有所不同：

+   如果父表中有多行具有相同的引用键值，`InnoDB`会像其他具有相同键值的父行不存在一样执行外键检查。例如，如果定义了`RESTRICT`类型的约束，并且有一个子行具有多个父行，`InnoDB`不允许删除任何父行。

+   如果`ON UPDATE CASCADE`或`ON UPDATE SET NULL`递归更新*相同表*，它会像`RESTRICT`一样操作。这意味着不能使用自引用的`ON UPDATE CASCADE`或`ON UPDATE SET NULL`操作。这是为了防止由级联更新导致的无限循环。另一方面，自引用的`ON DELETE SET NULL`是可能的，就像自引用的`ON DELETE CASCADE`一样。级联操作不能嵌套超过 15 层。

+   在插入、删除或更新多行的 SQL 语句中，外键约束（如唯一约束）会逐行检查。在执行外键检查时，`InnoDB`会在必须检查的子记录或父记录上设置共享的行级锁。MySQL 会立即检查外键约束；检查不会延迟到事务提交。根据 SQL 标准，默认行为应该是延迟检查。也就是说，只有在*整个 SQL 语句*被处理完之后才会检查约束。这意味着不可能使用外键删除引用自身的行。

+   没有存储引擎，包括`InnoDB`，识别或执行引用完整性约束定义中使用的`MATCH`子句。使用显式的`MATCH`子句不会产生指定的效果，并且会导致`ON DELETE`和`ON UPDATE`子句被忽略。应避免指定`MATCH`。

    SQL 标准中的`MATCH`子句控制如何处理复合（多列）外键中的`NULL`值，当与引用表中的主键进行比较时。MySQL 基本上实现了`MATCH SIMPLE`定义的语义，允许外键全部或部分为`NULL`。在这种情况下，包含这种外键的（子表）行可以被插入，即使它与引用（父表）中的任何行都不匹配。（可以使用触发器实现其他语义。）

+   出于性能原因，MySQL 要求引用的列被索引。然而，MySQL 不强制要求引用的列是`UNIQUE`或声明为`NOT NULL`。

    引用非`UNIQUE`键的`FOREIGN KEY`约束不是标准 SQL，而是`InnoDB`的扩展。另一方面，`NDB`存储引擎要求在任何作为外键引用的列上显式唯一键（或主键）。

    对于包含非唯一键或包含`NULL`值的外键引用的处理对于诸如`UPDATE`或`DELETE CASCADE`等操作并不明确定义。建议您使用仅引用`UNIQUE`（包括`PRIMARY`）和`NOT NULL`键的外键。

+   对于不支持外键的存储引擎（如`MyISAM`)，MySQL 服务器解析并忽略外键规范。

+   MySQL 解析但忽略“内联`REFERENCES`规范”（如 SQL 标准中定义的），其中引用是作为列规范的一部分定义的。MySQL 仅在作为单独的`FOREIGN KEY`规范的一部分指定时才接受`REFERENCES`子句。

    定义一个列使用`REFERENCES *tbl_name*(col_name)`子句实际上没有任何效果，*仅仅作为一个备忘录或注释，告诉您当前正在定义的列意图引用另一个表中的列*。在使用这种语法时，重要的是要意识到：

    +   MySQL 不执行任何检查以确保*col_name*实际存在于*tbl_name*中（甚至*tbl_name*本身是否存在）。

    +   MySQL 不对*tbl_name*执行任何操作，例如根据您定义的表中的行所采取的操作删除行；换句话说，这种语法根本不引起任何`ON DELETE`或`ON UPDATE`行为。（尽管您可以将`ON DELETE`或`ON UPDATE`子句编写为`REFERENCES`子句的一部分，但它也会被忽略。）

    +   这种语法创建了一个*列*；它**不**创建任何索引或键。

    您可以将创建的列用作连接列，如下所示：

    ```sql
    CREATE TABLE person (
        id SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT,
        name CHAR(60) NOT NULL,
        PRIMARY KEY (id)
    );

    CREATE TABLE shirt (
        id SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT,
        style ENUM('t-shirt', 'polo', 'dress') NOT NULL,
        color ENUM('red', 'blue', 'orange', 'white', 'black') NOT NULL,
        owner SMALLINT UNSIGNED NOT NULL REFERENCES person(id),
        PRIMARY KEY (id)
    );

    INSERT INTO person VALUES (NULL, 'Antonio Paz');

    SELECT @last := LAST_INSERT_ID();

    INSERT INTO shirt VALUES
    (NULL, 'polo', 'blue', @last),
    (NULL, 'dress', 'white', @last),
    (NULL, 't-shirt', 'blue', @last);

    INSERT INTO person VALUES (NULL, 'Lilliana Angelovska');

    SELECT @last := LAST_INSERT_ID();

    INSERT INTO shirt VALUES
    (NULL, 'dress', 'orange', @last),
    (NULL, 'polo', 'red', @last),
    (NULL, 'dress', 'blue', @last),
    (NULL, 't-shirt', 'white', @last);

    SELECT * FROM person;
    +----+---------------------+
    | id | name                |
    +----+---------------------+
    |  1 | Antonio Paz         |
    |  2 | Lilliana Angelovska |
    +----+---------------------+

    SELECT * FROM shirt;
    +----+---------+--------+-------+
    | id | style   | color  | owner |
    +----+---------+--------+-------+
    |  1 | polo    | blue   |     1 |
    |  2 | dress   | white  |     1 |
    |  3 | t-shirt | blue   |     1 |
    |  4 | dress   | orange |     2 |
    |  5 | polo    | red    |     2 |
    |  6 | dress   | blue   |     2 |
    |  7 | t-shirt | white  |     2 |
    +----+---------+--------+-------+

    SELECT s.* FROM person p INNER JOIN shirt s
       ON s.owner = p.id
     WHERE p.name LIKE 'Lilliana%'
       AND s.color <> 'white';

    +----+-------+--------+-------+
    | id | style | color  | owner |
    +----+-------+--------+-------+
    |  4 | dress | orange |     2 |
    |  5 | polo  | red    |     2 |
    |  6 | dress | blue   |     2 |
    +----+-------+--------+-------+
    ```

    当以这种方式使用时，`REFERENCES`子句不会显示在`SHOW CREATE TABLE`或`DESCRIBE`的输出中：

    ```sql
    SHOW CREATE TABLE shirt\G
    *************************** 1\. row ***************************
    Table: shirt
    Create Table: CREATE TABLE `shirt` (
    `id` smallint(5) unsigned NOT NULL auto_increment,
    `style` enum('t-shirt','polo','dress') NOT NULL,
    `color` enum('red','blue','orange','white','black') NOT NULL,
    `owner` smallint(5) unsigned NOT NULL,
    PRIMARY KEY  (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
    ```

有关外键约束的信息，请参阅第 15.1.20.5 节，“FOREIGN KEY Constraints”。
