> 原文：[`dev.mysql.com/doc/refman/8.0/en/update-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/update-optimization.html)

#### 10.2.5.2 优化 UPDATE 语句

更新语句被优化为像`SELECT`查询一样，但额外增加了写入的开销。写入的速度取决于被更新的数据量和被更新的索引数量。未更改的索引不会被更新。

另一种获得快速更新的方法是延迟更新，然后稍后连续进行多次更新。如果锁定表，一次性执行多次更新比逐个执行要快得多。

对于使用动态行格式的`MyISAM`表，将行更新为更长的总长度可能会导致行拆分。如果经常这样做，偶尔使用`OPTIMIZE TABLE`非常重要。参见 Section 15.7.3.4, “OPTIMIZE TABLE Statement”。
