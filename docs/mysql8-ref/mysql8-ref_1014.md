> 原文：[`dev.mysql.com/doc/refman/8.0/en/local-variable-scope.html`](https://dev.mysql.com/doc/refman/8.0/en/local-variable-scope.html)

#### 15.6.4.2 本地变量作用域和解析

本地变量的作用域是其声明的 `BEGIN ... END` 块。该变量可以在声明块内嵌套的块中引用，除了那些声明具有相同名称变量的块。

因为本地变量仅在存储过程执行期间处于作用域内，所以不允许在存储过程内创建的准备语句中引用它们。准备语句的作用域是当前会话，而不是存储过程，因此该语句可能在程序结束后执行，此时变量将不再处于作用域内。例如，`SELECT ... INTO *`local_var`*` 不能作为准备语句使用。此限制也适用于存储过程和函数参数。参见 Section 15.5.1, “PREPARE Statement”。

本地变量不应与表列具有相同的名称。如果 SQL 语句（例如 `SELECT ... INTO` 语句）包含对具有相同名称的列和声明的本地变量的引用，MySQL 目前将该引用解释为变量的名称。考虑以下过程定义：

```sql
CREATE PROCEDURE sp1 (x VARCHAR(5))
BEGIN
  DECLARE xname VARCHAR(5) DEFAULT 'bob';
  DECLARE newname VARCHAR(5);
  DECLARE xid INT;

  SELECT xname, id INTO newname, xid
    FROM table1 WHERE xname = xname;
  SELECT newname;
END;
```

MySQL 在 `SELECT` 语句中将 `xname` 解释为 `xname` *变量*的引用，而不是 `xname` *列*的引用。因此，当调用过程 `sp1()` 时，`newname` 变量返回值 `'bob'`，而不管 `table1.xname` 列的值如何。

类似地，以下过程中的游标定义包含一个引用 `xname` 的 `SELECT` 语句。MySQL 将其解释为对该名称变量的引用，而不是列的引用。

```sql
CREATE PROCEDURE sp2 (x VARCHAR(5))
BEGIN
  DECLARE xname VARCHAR(5) DEFAULT 'bob';
  DECLARE newname VARCHAR(5);
  DECLARE xid INT;
  DECLARE done TINYINT DEFAULT 0;
  DECLARE cur1 CURSOR FOR SELECT xname, id FROM table1;
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

  OPEN cur1;
  read_loop: LOOP
    FETCH FROM cur1 INTO newname, xid;
    IF done THEN LEAVE read_loop; END IF;
    SELECT newname;
  END LOOP;
  CLOSE cur1;
END;
```

另请参见 Section 27.8, “Restrictions on Stored Programs”。
