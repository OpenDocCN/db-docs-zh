> 原文：[`dev.mysql.com/doc/refman/8.0/en/creating-many-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/creating-many-tables.html)

#### 10.4.3.2 在同一数据库中创建许多表的缺点

如果在同一个数据库目录中有许多`MyISAM`表，那么打开、关闭和创建操作会很慢。如果在许多不同的表上执行`SELECT`语句，当表缓存已满时会有一些开销，因为每次打开一个表时，都必须关闭另一个表。您可以通过增加表缓存中允许的条目数来减少这种开销。
