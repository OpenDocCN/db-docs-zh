# 14.9.3 使用查询扩展进行全文搜索

> 原文：[`dev.mysql.com/doc/refman/8.0/en/fulltext-query-expansion.html`](https://dev.mysql.com/doc/refman/8.0/en/fulltext-query-expansion.html)

全文搜索支持查询扩展（特别是其变体“盲目的查询扩展”）。当搜索短语太短时，这通常是很有用的，这意味着用户依赖于全文搜索引擎缺乏的暗示知识。例如，一个搜索“数据库”的用户可能真正想要匹配“数据库”并返回“MySQL”、“Oracle”、“DB2”和“RDBMS”等短语。这是暗示的知识。

通过在搜索短语后添加`WITH QUERY EXPANSION`或`IN NATURAL LANGUAGE MODE WITH QUERY EXPANSION`来启用盲目的查询扩展（也称为自动相关反馈）。它通过执行两次搜索来工作，第二次搜索的搜索短语是原始搜索短语与第一次搜索中最相关的几篇文档连接在一起。因此，如果这些文档中有一个包含“数据库”和“MySQL”这两个词，第二次搜索将找到包含“MySQL”这个词的文档，即使它们不包含“数据库”这个词。以下示例展示了这种差异：

```sql
mysql> SELECT * FROM articles
    WHERE MATCH (title,body)
    AGAINST ('database' IN NATURAL LANGUAGE MODE);
+----+-------------------+------------------------------------------+
| id | title             | body                                     |
+----+-------------------+------------------------------------------+
|  1 | MySQL Tutorial    | DBMS stands for DataBase ...             |
|  5 | MySQL vs. YourSQL | In the following database comparison ... |
+----+-------------------+------------------------------------------+
2 rows in set (0.00 sec)

mysql> SELECT * FROM articles
    WHERE MATCH (title,body)
    AGAINST ('database' WITH QUERY EXPANSION);
+----+-----------------------+------------------------------------------+
| id | title                 | body                                     |
+----+-----------------------+------------------------------------------+
|  5 | MySQL vs. YourSQL     | In the following database comparison ... |
|  1 | MySQL Tutorial        | DBMS stands for DataBase ...             |
|  3 | Optimizing MySQL      | In this tutorial we show ...             |
|  6 | MySQL Security        | When configured properly, MySQL ...      |
|  2 | How To Use MySQL Well | After you went through a ...             |
|  4 | 1001 MySQL Tricks     | 1\. Never run mysqld as root. 2\. ...      |
+----+-----------------------+------------------------------------------+
6 rows in set (0.00 sec)
```

另一个例子可能是搜索乔治·西梅诺关于梅格雷的书籍，当用户不确定如何拼写“梅格雷”时。在没有查询扩展的情况下，搜索“Megre and the reluctant witnesses”只会找到“Maigret and the Reluctant Witnesses”。而使用查询扩展的搜索会在第二次搜索时找到所有包含“Maigret”这个词的书籍。

注意

盲目的查询扩展往往会通过返回不相关的文档显著增加噪音，因此只有在搜索短语很短的情况下才使用它。
