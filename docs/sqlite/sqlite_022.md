# 1\. 引言

> 原文：[`sqlite.com/gencol.html`](https://sqlite.com/gencol.html)

生成列（有时也称为"计算列"）是表中值是同一行中其他列函数的列。生成列可以读取，但它们的值不能直接写入。更改生成列值的唯一方法是修改用于计算生成列的其他列的值。

# 2\. 语法

从语法上讲，使用"GENERATED ALWAYS" column-constraint 来指定生成列。例如：

```sql
CREATE TABLE t1(
   a INTEGER PRIMARY KEY,
   b INT,
   c TEXT,
   d INT GENERATED ALWAYS AS (a*abs(b)) VIRTUAL,
   e TEXT GENERATED ALWAYS AS (substr(c,b,b+1)) STORED
);

```

上述语句有三个普通列，"a"（主键），"b"和"c"，以及两个生成列"d"和"e"。

约束开始处的"GENERATED ALWAYS"关键字和末尾的"VIRTUAL"或"STORED"关键字都是可选的。只有"AS"关键字和括号内的表达式是必需的。如果省略末尾的"VIRTUAL"或"STORED"关键字，则 VIRTUAL 是默认的。因此，上述示例语句可以简化为：

```sql
CREATE TABLE t1(
   a INTEGER PRIMARY KEY,
   b INT,
   c TEXT,
   d INT AS (a*abs(b)),
   e TEXT AS (substr(c,b,b+1)) STORED
);

```

## 2.1\. VIRTUAL 与 STORED 列

生成列可以是 VIRTUAL 或 STORED。读取 VIRTUAL 列时计算其值，而写入行时计算 STORED 列的值。STORED 列在数据库文件中占用空间，而 VIRTUAL 列在读取时使用更多 CPU 周期。

从 SQL 的角度来看，STORED 和 VIRTUAL 列几乎完全相同。针对任一类生成列的查询产生相同的结果。唯一的功能区别是不能使用 ALTER TABLE ADD COLUMN 命令添加新的 STORED 列。只能使用 ALTER TABLE 添加 VIRTUAL 列。

## 2.2\. 能力

1.  生成列可以具有数据类型。SQLite 尝试使用与普通列相同的亲和性规则将生成表达式的结果转换为该数据类型。

1.  生成列可能具有 NOT NULL、CHECK 和 UNIQUE 约束，以及外键约束，就像普通列一样。

1.  生成列可以参与索引，就像普通列一样。

1.  生成列的表达式可以引用表中声明的任何其他列，包括其他生成列，只要该表达式不直接或间接地引用自身。

1.  生成列可以出现在表定义的任何位置。生成列可以与普通列交错。不需要将生成列放在表定义列列表的末尾，就像上面的示例所示。

## 2.3\. 限制

1.  生成列可能没有默认值（它们不能使用"DEFAULT"子句）。生成列的值始终是跟在"AS"关键字后面的表达式指定的值。

1.  生成列不能作为 PRIMARY KEY 的一部分使用。（SQLite 的未来版本可能会放宽对 STORED 列的此约束。）

1.  生成列的表达式只能引用常量文字和同一行内的列，并且只能使用标量 deterministic functions。该表达式不能使用子查询、聚合函数、窗口函数或表值函数。

1.  生成列的表达式可以引用同一行中的其他生成列，但任何生成列都不能直接或间接依赖于自身。

1.  生成列的表达式不能直接引用 ROWID，尽管可以引用 INTEGER PRIMARY KEY 列，这两者通常是相同的。

1.  每个表必须至少有一个非生成列。

1.  不可能使用 ALTER TABLE ADD COLUMN 添加一个存储列。但可以添加一个虚拟列。

1.  生成列的数据类型和排序顺序仅由列定义上的数据类型和 COLLATE 子句决定。GENERATED ALWAYS AS 表达式的数据类型和排序顺序对列本身的数据类型和排序顺序没有影响。

1.  生成列不包括在 PRAGMA table_info 语句提供的列列表中。但它们包括在较新的 PRAGMA table_xinfo 语句的输出中。

# 3\. 兼容性

生成列支持从 SQLite 版本 3.31.0（2020-01-22）开始添加。如果早期版本的 SQLite 尝试读取包含生成列的数据库文件，那么该早期版本将认为生成列语法是错误的，并报告数据库模式损坏。

为了澄清：SQLite 版本 3.31.0 能够读取和写入任何由任何早期版本的 SQLite 创建的数据库，从 SQLite 3.0.0（2004-06-18）开始。而早期版本的 SQLite，在 3.31.0 之前，能够读取和写入由 SQLite 版本 3.31.0 或更新版本创建的数据库，只要数据库模式不包含早期版本不理解的生成列等功能。只有当您使用 SQLite 版本 3.31.0 或更新版本创建一个包含生成列的新数据库，并尝试使用早期版本的 SQLite 读取或写入该数据库文件时，才会出现问题。
