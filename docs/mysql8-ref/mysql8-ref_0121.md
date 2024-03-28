# 5.2 输入查询

> 原文：[`dev.mysql.com/doc/refman/8.0/en/entering-queries.html`](https://dev.mysql.com/doc/refman/8.0/en/entering-queries.html)

确保您已连接到服务器，如前一节所述。这本身并不选择要使用的任何数据库，但没关系。此时，更重要的是了解如何发出查询的基本原则，而不是立即开始创建表，将数据加载到其中并从中检索数据。本节描述了输入查询的基本原则，使用几个您可以尝试以熟悉**mysql**工作方式的查询。

这是一个简单的查询，要求服务器告诉您其版本号和当前日期。按照这里显示的方式在`mysql>`提示后键入并按 Enter：

```sql
mysql> SELECT VERSION(), CURRENT_DATE;
+-----------+--------------+
| VERSION() | CURRENT_DATE |
+-----------+--------------+
| 5.8.0-m17 | 2015-12-21   |
+-----------+--------------+
1 row in set (0.02 sec)
mysql>
```

此查询说明了关于**mysql**的几个方面：

+   查询通常由一个 SQL 语句后跟一个分号组成。（有一些例外情况，其中分号可以省略。`QUIT`，前面提到的，是其中之一。我们稍后会介绍其他情况。）

+   当您发出查询时，**mysql**将其发送到服务器执行并显示结果，然后打印另一个`mysql>`提示，表示它已准备好接受另一个查询。

+   **mysql**以表格形式（行和列）显示查询输出。第一行包含列的标签。随后的行是查询结果。通常，列标签是您从数据库表中提取的列的名称。如果您检索的是表列的值而不是表列（如刚刚显示的示例中），**mysql**使用表达式本身标记列。

+   **mysql**显示了返回的行数以及查询执行所需的时间，这给出了服务器性能的大致概念。这些值不精确，因为它们代表挂钟时间（而不是 CPU 或机器时间），并且受到服务器负载和网络延迟等因素的影响。（为简洁起见，本章剩余示例中有时不显示“结果集中的行”行。）

关键字可以以任何大小写形式输入。以下查询是等效的：

```sql
mysql> SELECT VERSION(), CURRENT_DATE;
mysql> select version(), current_date;
mysql> SeLeCt vErSiOn(), current_DATE;
```

这是另一个查询。它演示了您可以将**mysql**用作简单的计算器：

```sql
mysql> SELECT SIN(PI()/4), (4+1)*5;
+------------------+---------+
| SIN(PI()/4)      | (4+1)*5 |
+------------------+---------+
| 0.70710678118655 |      25 |
+------------------+---------+
1 row in set (0.02 sec)
```

到目前为止显示的查询相对较短，是单行语句。您甚至可以在一行上输入多个语句。只需用分号结束每个语句：

```sql
mysql> SELECT VERSION(); SELECT NOW();
+-----------+
| VERSION() |
+-----------+
| 8.0.13    |
+-----------+
1 row in set (0.00 sec)

+---------------------+
| NOW()               |
+---------------------+
| 2018-08-24 00:56:40 |
+---------------------+
1 row in set (0.00 sec)
```

查询不必全部在一行上给出，因此需要多行的长查询不是问题。**mysql**通过查找终止分号来确定语句的结束位置，而不是查找输入行的结尾。（换句话说，**mysql**接受自由格式的输入：它收集输入行但直到看到分号才执行它们。）

这是一个简单的多行语句：

```sql
mysql> SELECT
 -> USER()
 -> ,
 -> CURRENT_DATE;
+---------------+--------------+
| USER()        | CURRENT_DATE |
+---------------+--------------+
| jon@localhost | 2018-08-24   |
+---------------+--------------+
```

在这个例子中，请注意在输入多行查询的第一行后，提示符从`mysql>`变为`->`。这是**mysql**表示它尚未看到完整语句并正在等待其余部分的方式。提示符是你的朋友，因为它提供了宝贵的反馈。如果你利用这个反馈，你就可以始终了解**mysql**正在等待什么。

如果您决定不想执行正在输入过程中的查询，请键入`\c`取消它：

```sql
mysql> SELECT
 -> USER()
 -> \c
mysql>
```

在这里，也请注意提示符。在键入`\c`后，它会切换回`mysql>`，提供反馈以指示**mysql**已准备好进行新查询。

以下表显示了您可能看到的每个提示符以及它们对**mysql**所处状态的含义。

| 提示符 | 含义 |
| --- | --- |
| `mysql>` | 准备新查询 |
| `->` | 等待多行查询的下一行 |
| `'>` | 等待下一行，等待完成以单引号（`'`）开始的字符串 |
| `">` | 等待下一行，等待完成以双引号（`"`）开始的字符串 |

| ``>` | 等待下一行，等待完成以反引号开始的标识符（```sql) |
| `/*>` | Waiting for next line, waiting for completion of a comment that began with `/*` |

Multiple-line statements commonly occur by accident when you intend to issue a query on a single line, but forget the terminating semicolon. In this case, **mysql** waits for more input:

```

mysql> SELECT USER()

->

```sql

If this happens to you (you think you've entered a statement but the only response is a `->` prompt), most likely **mysql** is waiting for the semicolon. If you don't notice what the prompt is telling you, you might sit there for a while before realizing what you need to do. Enter a semicolon to complete the statement, and **mysql** executes it:

```

mysql> SELECT USER()

-> ;

+---------------+

| USER()        |
| --- |

+---------------+

| jon@localhost |
| --- |

+---------------+

```sql

The `'>` and `">` prompts occur during string collection (another way of saying that MySQL is waiting for completion of a string). In MySQL, you can write strings surrounded by either `'` or `"` characters (for example, `'hello'` or `"goodbye"`), and **mysql** lets you enter strings that span multiple lines. When you see a `'>` or `">` prompt, it means that you have entered a line containing a string that begins with a `'` or `"` quote character, but have not yet entered the matching quote that terminates the string. This often indicates that you have inadvertently left out a quote character. For example:

```

mysql> SELECT * FROM my_table WHERE name = 'Smith AND age < 30;

    '>

```sql

If you enter this `SELECT` statement, then press **Enter** and wait for the result, nothing happens. Instead of wondering why this query takes so long, notice the clue provided by the `'>` prompt. It tells you that **mysql** expects to see the rest of an unterminated string. (Do you see the error in the statement? The string `'Smith` is missing the second single quotation mark.)

At this point, what do you do? The simplest thing is to cancel the query. However, you cannot just type `\c` in this case, because **mysql** interprets it as part of the string that it is collecting. Instead, enter the closing quote character (so **mysql** knows you've finished the string), then type `\c`:

```

mysql> SELECT * FROM my_table WHERE name = 'Smith AND age < 30;

    '> '\c

mysql>

```

提示符改回`mysql>`，表示**mysql**已准备好进行新查询。

``>`提示符类似于`'>`和`">`提示符，但表示您已经开始但尚未完成反引号引用的标识符。

了解`'>`，`">`和``>`提示符的含义很重要，因为如果您错误地输入了一个未终止的字符串，您输入的任何进一步行似乎都会被**mysql**忽略，包括包含`QUIT`的行。这可能会令人困惑，特别是如果您不知道在取消当前查询之前需要提供终止引号。

注意

从这一点开始，多行语句将不再带有次要提示（`->`或其他），以便更容易复制并粘贴语句以供自己尝试。
