# 15.6.6 游标

> 原文：[`dev.mysql.com/doc/refman/8.0/en/cursors.html`](https://dev.mysql.com/doc/refman/8.0/en/cursors.html)

15.6.6.1 游标关闭语句

15.6.6.2 游标声明语句

15.6.6.3 游标获取语句

15.6.6.4 游标打开语句

15.6.6.5 服务器端游标的限制

MySQL 支持存储程序内的游标。语法与嵌入式 SQL 相同。游标具有以下属性：

+   Asensitive：服务器可能会或可能不会复制其结果表

+   只读：不可更新

+   Nonscrollable：只能单向遍历，不能跳过行

游标声明必须出现在处理程序声明之前，并且在变量和条件声明之后。

示例：

```sql
CREATE PROCEDURE curdemo()
BEGIN
  DECLARE done INT DEFAULT FALSE;
  DECLARE a CHAR(16);
  DECLARE b, c INT;
  DECLARE cur1 CURSOR FOR SELECT id,data FROM test.t1;
  DECLARE cur2 CURSOR FOR SELECT i FROM test.t2;
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

  OPEN cur1;
  OPEN cur2;

  read_loop: LOOP
    FETCH cur1 INTO a, b;
    FETCH cur2 INTO c;
    IF done THEN
      LEAVE read_loop;
    END IF;
    IF b < c THEN
      INSERT INTO test.t3 VALUES (a,b);
    ELSE
      INSERT INTO test.t3 VALUES (a,c);
    END IF;
  END LOOP;

  CLOSE cur1;
  CLOSE cur2;
END;
```
