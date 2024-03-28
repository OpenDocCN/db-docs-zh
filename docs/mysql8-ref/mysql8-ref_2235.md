> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-quote-identifier.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-quote-identifier.html)

#### 30.4.5.18 quote_identifier()函数

给定一个字符串参数，此函数生成一个适合包含在 SQL 语句中的带引号标识符。当要用作标识符的值是保留字或包含反引号时，这将非常有用。

mysql> SELECT sys.quote_identifier('plain');

+-------------------------------+

| sys.quote_identifier('plain') |
| --- |

+-------------------------------+

| `plain`                       |
| --- |

+-------------------------------+

mysql> SELECT sys.quote_identifier('trick`ier');

+-----------------------------------+

| sys.quote_identifier('trick`ier') |
| --- |

+-----------------------------------+

| `trick``ier`                      |
| --- |

+-----------------------------------+

mysql> SELECT sys.quote_identifier('integer');

+---------------------------------+

| sys.quote_identifier('integer') |
| --- |

+---------------------------------+

| `integer`                       |
| --- |

+---------------------------------+

```
