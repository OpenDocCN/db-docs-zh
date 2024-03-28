> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-floatvalues.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-floatvalues.html)

#### 19.5.1.12 复制和浮点值

在基于语句的复制中，值从十进制转换为二进制。由于十进制和二进制表示之间的转换可能是近似的，涉及浮点值的比较是不精确的。这对明确使用浮点值的操作，或者隐式转换为浮点的值都是适用的。由于计算机架构、用于构建 MySQL 的编译器等方面的差异，源服务器和副本服务器上对浮点值的比较可能产生不同的结果。参见第 14.3 节，“表达式评估中的类型转换”，以及第 B.3.4.8 节，“浮点值的问题”。
