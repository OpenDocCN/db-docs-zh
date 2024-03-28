> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-character-set.html`](https://dev.mysql.com/doc/refman/8.0/en/show-character-set.html)

#### 15.7.7.3 SHOW CHARACTER SET Statement

```sql
SHOW {CHARACTER SET | CHARSET}
    [LIKE '*pattern*' | WHERE *expr*]
```

`SHOW CHARACTER SET`语句显示所有可用的字符集。如果有`LIKE`子句，则表示要匹配的字符集名称。可以使用`WHERE`子句选择使用更一般条件的行，如第 28.8 节“SHOW 语句的扩展”中所讨论的那样。例如：

```sql
mysql> SHOW CHARACTER SET LIKE 'latin%';
+---------+-----------------------------+-------------------+--------+
| Charset | Description                 | Default collation | Maxlen |
+---------+-----------------------------+-------------------+--------+
| latin1  | cp1252 West European        | latin1_swedish_ci |      1 |
| latin2  | ISO 8859-2 Central European | latin2_general_ci |      1 |
| latin5  | ISO 8859-9 Turkish          | latin5_turkish_ci |      1 |
| latin7  | ISO 8859-13 Baltic          | latin7_general_ci |      1 |
+---------+-----------------------------+-------------------+--------+
```

`SHOW CHARACTER SET`输出具有以下列：

+   `Charset`

    字符集名称。

+   `Description`

    字符集的描述。

+   `Default collation`

    字符集的默认排序规则。

+   `Maxlen`

    存储一个字符所需的最大字节数。

`filename`字符集仅供内部使用；因此，`SHOW CHARACTER SET`不会显示它。

字符集信息也可以从`INFORMATION_SCHEMA` `CHARACTER_SETS`表中获取。
