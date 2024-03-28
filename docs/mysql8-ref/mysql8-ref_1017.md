> 原文：[`dev.mysql.com/doc/refman/8.0/en/if.html`](https://dev.mysql.com/doc/refman/8.0/en/if.html)

#### 15.6.5.2 IF Statement

```sql
IF *search_condition* THEN *statement_list*
    [ELSEIF *search_condition* THEN *statement_list*] ...
    [ELSE *statement_list*]
END IF
```

用于存储程序的`IF`语句实现了一个基本的条件构造。

注意

还有一个`IF()` *函数*，它与此处描述的`IF` *语句*不同。请参阅第 14.5 节，“流程控制函数”。`IF`语句可以有`THEN`、`ELSE`和`ELSEIF`子句，并以`END IF`结束。

如果给定的*`search_condition`*评估为 true，则相应的`THEN`或`ELSEIF`子句*`statement_list`*执行。如果没有*`search_condition`*匹配，则`ELSE`子句*`statement_list`*执行。

每个*`statement_list`*由一个或多个 SQL 语句组成；不允许空的*`statement_list`*。

一个`IF ... END IF`块，就像存储程序中使用的所有其他流程控制块一样，必须以分号结束，如本例所示：

```sql
DELIMITER //

CREATE FUNCTION SimpleCompare(n INT, m INT)
  RETURNS VARCHAR(20)

  BEGIN
    DECLARE s VARCHAR(20);

    IF n > m THEN SET s = '>';
    ELSEIF n = m THEN SET s = '=';
    ELSE SET s = '<';
    END IF;

    SET s = CONCAT(n, ' ', s, ' ', m);

    RETURN s;
  END //

DELIMITER ;
```

与其他流程控制结构一样，`IF ... END IF`块可以嵌套在其他流程控制结构中，包括其他`IF`语句。每个`IF`必须由其自己的`END IF`后跟一个分号来终止。您可以使用缩进使嵌套的流程控制块更容易被人类阅读（尽管 MySQL 不要求），如下所示：

```sql
DELIMITER //

CREATE FUNCTION VerboseCompare (n INT, m INT)
  RETURNS VARCHAR(50)

  BEGIN
    DECLARE s VARCHAR(50);

    IF n = m THEN SET s = 'equals';
    ELSE
      IF n > m THEN SET s = 'greater';
      ELSE SET s = 'less';
      END IF;

      SET s = CONCAT('is ', s, ' than');
    END IF;

    SET s = CONCAT(n, ' ', s, ' ', m, '.');

    RETURN s;
  END //

DELIMITER ;
```

在这个例子中，只有当`n`不等于`m`时，内部的`IF`才会被评估。
