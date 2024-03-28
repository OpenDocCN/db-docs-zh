# 5.3.3 将数据加载到表中

> 原文：[`dev.mysql.com/doc/refman/8.0/en/loading-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/loading-tables.html)

创建表后，你需要填充它。`LOAD DATA` 和 `INSERT` 语句对此很有用。

假设你的宠物记录可以如下所示描述。（注意 MySQL 期望日期以 `'*`YYYY-MM-DD`*'` 格式；这可能与你习惯的格式不同。）

| 名字 | 主人 | 种类 | 性别 | 出生 | 死亡 |
| --- | --- | --- | --- | --- | --- |
| Fluffy | Harold | 猫 | f | 1993-02-04 |  |
| Claws | Gwen | 猫 | m | 1994-03-17 |  |
| Buffy | Harold | 狗 | f | 1989-05-13 |  |
| Fang | Benny | 狗 | m | 1990-08-27 |  |
| Bowser | Diane | 狗 | m | 1979-08-31 | 1995-07-29 |
| Chirpy | Gwen | 鸟 | f | 1998-09-11 |  |
| Whistler | Gwen | 鸟 |  | 1997-12-09 |  |
| Slim | Benny | 蛇 | m | 1996-04-29 |  |

因为你从一个空表开始，一个简单的方法是创建一个包含每只动物一行的文本文件，然后用一条语句将文件内容加载到表中。

你可以创建一个文本文件 `pet.txt`，每行包含一条记录，值之间用制表符分隔，并按照 `CREATE TABLE` 语句中列出的顺序给出。对于缺失的值（例如未知性别或仍然活着的动物的死亡日期），你可以使用 `NULL` 值。在文本文件中表示这些值，使用 `\N`（反斜杠，大写 N）。例如，鸟 Whistler 的记录将如下所示（值之间的空格是一个制表符字符）：

```sql
Whistler        Gwen    bird    \N      1997-12-09      \N
```

要将文本文件 `pet.txt` 加载到 `pet` 表中，请使用以下语句：

```sql
mysql> LOAD DATA LOCAL INFILE '/path/pet.txt' INTO TABLE pet;
```

如果你在 Windows 上使用以 `\r\n` 作为行终止符的编辑器创建文件，你应该使用这条语句：

```sql
mysql> LOAD DATA LOCAL INFILE '/path/pet.txt' INTO TABLE pet
       LINES TERMINATED BY '\r\n';
```

(在运行 macOS 的 Apple 机器上，你可能想使用 `LINES TERMINATED BY '\r'`。)

如果你愿意，可以在 `LOAD DATA` 语句中明确指定列值分隔符和行结束标记，但默认值是制表符和换行符。这对于语句正确读取文件 `pet.txt` 是足够的。

如果语句失败，很可能是因为你的 MySQL 安装默认没有启用本地文件功能。请参阅 Section 8.1.6, “LOAD DATA LOCAL 安全注意事项”，了解如何更改此设置。

当您想逐个添加新记录时，`INSERT`语句很有用。在其最简单的形式中，您按照`CREATE TABLE`语句中列出列的顺序为每一列提供值。假设黛安获得了一只名为“Puffball”的新仓鼠。您可以使用类似于以下的`INSERT`语句添加新记录：

```sql
mysql> INSERT INTO pet
       VALUES ('Puffball','Diane','hamster','f','1999-03-30',NULL);
```

字符串和日期值在此处被指定为带引号的字符串。此外，使用`INSERT`，您可以直接插入`NULL`表示缺失值。您不像在`LOAD DATA`中使用`\N`。

从这个例子中，您应该能够看到，使用多个`INSERT`语句来最初加载您的记录将涉及更多的输入，而不是使用单个`LOAD DATA`语句。
