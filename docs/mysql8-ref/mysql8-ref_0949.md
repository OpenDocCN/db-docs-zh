# 15.2.10 LOAD XML Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/load-xml.html`](https://dev.mysql.com/doc/refman/8.0/en/load-xml.html)

```sql
LOAD XML
    [LOW_PRIORITY | CONCURRENT] [LOCAL]
    INFILE '*file_name*'
    [REPLACE | IGNORE]
    INTO TABLE [*db_name*.]*tbl_name*
    [CHARACTER SET *charset_name*]
    [ROWS IDENTIFIED BY '<*tagname*>']
    [IGNORE *number* {LINES | ROWS}]
    [(*field_name_or_user_var*
        [, *field_name_or_user_var*] ...)]
    [SET *col_name*={*expr* | DEFAULT}
        [, *col_name*={*expr* | DEFAULT}] ...]
```

`LOAD XML`语句将数据从 XML 文件读取到表中。*`file_name`*必须作为文字字符串给出。可选的`ROWS IDENTIFIED BY`子句中的*`tagname`*也必须作为文字字符串给出，并且必须用尖括号（`<`和`>`）括起来。

`LOAD XML`作为运行**mysql**客户端以 XML 输出模式（即使用`--xml`选项启动客户端）的补充。要将表中数据写入 XML 文件，可以从系统 shell 中调用带有`--xml`和`-e`选项的**mysql**客户端，如下所示：

```sql
$> mysql --xml -e 'SELECT * FROM mydb.mytable' > file.xml
```

要将文件读取回表中，请使用`LOAD XML`。默认情况下，`<row>`元素被视为数据库表行的等价物；可以使用`ROWS IDENTIFIED BY`子句进行更改。

此语句支持三种不同的 XML 格式：

+   列名作为属性，列值作为属性值：

    ```sql
    <*row* *column1*="*value1*" *column2*="*value2*" .../>
    ```

+   列名作为标签，列值作为这些标签的内容：

    ```sql
    <*row*>
      <*column1*>*value1*</*column1*>
      <*column2*>*value2*</*column2*>
    </*row*>
    ```

+   列名是`<field>`标签的`name`属性，而值是这些标签的内容：

    ```sql
    <row>
      <field name='*column1*'>*value1*</field>
      <field name='*column2*'>*value2*</field>
    </row>
    ```

    这是其他 MySQL 工具（如**mysqldump**）使用的格式。

所有三种格式可以在同一个 XML 文件中使用；导入程序会自动检测每一行的格式并正确解释。标签是根据标签或属性名和列名进行匹配的。

在 MySQL 8.0.21 之前，`LOAD XML`不支持源 XML 中的`CDATA`部分。（Bug #30753708，Bug #98199）

以下子句对于`LOAD XML`与对于`LOAD DATA`基本上是相同的方式工作：

+   `LOW_PRIORITY`或`CONCURRENT`

+   `LOCAL`

+   `REPLACE`或`IGNORE`

+   `CHARACTER SET`

+   `SET`

更多关于这些子句的信息，请参见第 15.2.9 节，“LOAD DATA Statement”。

`(*`field_name_or_user_var`*, ...)`是一个由一个或多个逗号分隔的 XML 字段或用户变量列表。用于此目的的用户变量的名称必须与 XML 文件中的字段名称匹配，并以`@`为前缀。您可以使用字段名称仅选择所需的字段。用户变量可用于存储相应的字段值以供后续重用。

`IGNORE *`number`* LINES`或`IGNORE *`number`* ROWS`子句导致 XML 文件中的前*`number`*行被跳过。这类似于`LOAD DATA`语句的`IGNORE ... LINES`子句。

假设我们有一个名为`person`的表，如下所示创建：

```sql
USE test;

CREATE TABLE person (
    person_id INT NOT NULL PRIMARY KEY,
    fname VARCHAR(40) NULL,
    lname VARCHAR(40) NULL,
    created TIMESTAMP
);
```

进一步假设此表最初为空。

现在假设我们有一个简单的 XML 文件`person.xml`，其内容如下所示：

```sql
<list>
  <person person_id="1" fname="Kapek" lname="Sainnouine"/>
  <person person_id="2" fname="Sajon" lname="Rondela"/>
  <person person_id="3"><fname>Likame</fname><lname>Örrtmons</lname></person>
  <person person_id="4"><fname>Slar</fname><lname>Manlanth</lname></person>
  <person><field name="person_id">5</field><field name="fname">Stoma</field>
    <field name="lname">Milu</field></person>
  <person><field name="person_id">6</field><field name="fname">Nirtam</field>
    <field name="lname">Sklöd</field></person>
  <person person_id="7"><fname>Sungam</fname><lname>Dulbåd</lname></person>
  <person person_id="8" fname="Sraref" lname="Encmelt"/>
</list>
```

在此示例文件中表示了先前讨论的每种允许的 XML 格式。

要将`person.xml`中的数据导入`person`表中，可以使用以下语句：

```sql
mysql> LOAD XML LOCAL INFILE 'person.xml'
 ->   INTO TABLE person
 ->   ROWS IDENTIFIED BY '<person>';

Query OK, 8 rows affected (0.00 sec)
Records: 8  Deleted: 0  Skipped: 0  Warnings: 0
```

在这里，我们假设`person.xml`位于 MySQL 数据目录中。如果找不到文件，将出现以下错误：

```sql
ERROR 2 (HY000): File '/person.xml' not found (Errcode: 2)
```

`ROWS IDENTIFIED BY '<person>'`子句意味着 XML 文件中的每个`<person>`元素被视为要导入数据的表中的一行。在这种情况下，这是`test`数据库中的`person`表。

从服务器的响应可以看出，有 8 行被导入到`test.person`表中。可以通过简单的`SELECT`语句进行验证：

```sql
mysql> SELECT * FROM person;
+-----------+--------+------------+---------------------+
| person_id | fname  | lname      | created             |
+-----------+--------+------------+---------------------+
|         1 | Kapek  | Sainnouine | 2007-07-13 16:18:47 |
|         2 | Sajon  | Rondela    | 2007-07-13 16:18:47 |
|         3 | Likame | Örrtmons   | 2007-07-13 16:18:47 |
|         4 | Slar   | Manlanth   | 2007-07-13 16:18:47 |
|         5 | Stoma  | Nilu       | 2007-07-13 16:18:47 |
|         6 | Nirtam | Sklöd      | 2007-07-13 16:18:47 |
|         7 | Sungam | Dulbåd     | 2007-07-13 16:18:47 |
|         8 | Sreraf | Encmelt    | 2007-07-13 16:18:47 |
+-----------+--------+------------+---------------------+
8 rows in set (0.00 sec)
```

正如本节前面所述，任何或所有 3 种允许的 XML 格式都可以出现在单个文件中，并且可以使用`LOAD XML`进行读取。

刚刚展示的导入操作的反向操作——即将 MySQL 表数据转储到 XML 文件中——可以使用系统 shell 中的**mysql**客户端来完成，如下所示：

```sql
$> mysql --xml -e "SELECT * FROM test.person" > person-dump.xml
$> cat person-dump.xml
<?xml version="1.0"?>

<resultset statement="SELECT * FROM test.person" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <row>
	<field name="person_id">1</field>
	<field name="fname">Kapek</field>
	<field name="lname">Sainnouine</field>
  </row>

  <row>
	<field name="person_id">2</field>
	<field name="fname">Sajon</field>
	<field name="lname">Rondela</field>
  </row>

  <row>
	<field name="person_id">3</field>
	<field name="fname">Likema</field>
	<field name="lname">Örrtmons</field>
  </row>

  <row>
	<field name="person_id">4</field>
	<field name="fname">Slar</field>
	<field name="lname">Manlanth</field>
  </row>

  <row>
	<field name="person_id">5</field>
	<field name="fname">Stoma</field>
	<field name="lname">Nilu</field>
  </row>

  <row>
	<field name="person_id">6</field>
	<field name="fname">Nirtam</field>
	<field name="lname">Sklöd</field>
  </row>

  <row>
	<field name="person_id">7</field>
	<field name="fname">Sungam</field>
	<field name="lname">Dulbåd</field>
  </row>

  <row>
	<field name="person_id">8</field>
	<field name="fname">Sreraf</field>
	<field name="lname">Encmelt</field>
  </row>
</resultset>
```

注意

`--xml`选项使**mysql**客户端使用 XML 格式进行输出；`-e`选项使客户端执行紧随选项后的 SQL 语句。参见第 6.5.1 节，“mysql — The MySQL Command-Line Client”。

您可以通过创建`person`表的副本并将转储文件导入新表来验证转储是否有效，如下所示：

```sql
mysql> USE test;
mysql> CREATE TABLE person2 LIKE person;
Query OK, 0 rows affected (0.00 sec)

mysql> LOAD XML LOCAL INFILE 'person-dump.xml'
 ->   INTO TABLE person2;
Query OK, 8 rows affected (0.01 sec)
Records: 8  Deleted: 0  Skipped: 0  Warnings: 0

mysql> SELECT * FROM person2;
+-----------+--------+------------+---------------------+
| person_id | fname  | lname      | created             |
+-----------+--------+------------+---------------------+
|         1 | Kapek  | Sainnouine | 2007-07-13 16:18:47 |
|         2 | Sajon  | Rondela    | 2007-07-13 16:18:47 |
|         3 | Likema | Örrtmons   | 2007-07-13 16:18:47 |
|         4 | Slar   | Manlanth   | 2007-07-13 16:18:47 |
|         5 | Stoma  | Nilu       | 2007-07-13 16:18:47 |
|         6 | Nirtam | Sklöd      | 2007-07-13 16:18:47 |
|         7 | Sungam | Dulbåd     | 2007-07-13 16:18:47 |
|         8 | Sreraf | Encmelt    | 2007-07-13 16:18:47 |
+-----------+--------+------------+---------------------+
8 rows in set (0.00 sec)
```

XML 文件中的每个字段都不必与相应表中的列匹配。没有相应列的字段将被跳过。您可以通过首先清空`person2`表并删除`created`列，然后使用我们之前使用的相同的`LOAD XML`语句来查看这一点，如下所示：

```sql
mysql> TRUNCATE person2;
Query OK, 8 rows affected (0.26 sec)

mysql> ALTER TABLE person2 DROP COLUMN created;
Query OK, 0 rows affected (0.52 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW CREATE TABLE person2\G
*************************** 1\. row ***************************
       Table: person2
Create Table: CREATE TABLE `person2` (
  `person_id` int NOT NULL,
  `fname` varchar(40) DEFAULT NULL,
  `lname` varchar(40) DEFAULT NULL,
  PRIMARY KEY (`person_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 1 row in set (0.00 sec)

mysql> LOAD XML LOCAL INFILE 'person-dump.xml'
 ->   INTO TABLE person2;
Query OK, 8 rows affected (0.01 sec)
Records: 8  Deleted: 0  Skipped: 0  Warnings: 0

mysql> SELECT * FROM person2;
+-----------+--------+------------+
| person_id | fname  | lname      |
+-----------+--------+------------+
|         1 | Kapek  | Sainnouine |
|         2 | Sajon  | Rondela    |
|         3 | Likema | Örrtmons   |
|         4 | Slar   | Manlanth   |
|         5 | Stoma  | Nilu       |
|         6 | Nirtam | Sklöd      |
|         7 | Sungam | Dulbåd     |
|         8 | Sreraf | Encmelt    |
+-----------+--------+------------+
8 rows in set (0.00 sec)
```

XML 文件中每行中给定字段的顺序不会影响`LOAD XML`的操作；字段顺序可以在行与行之间变化，并且不需要与表中相应列的顺序相同。

如前所述，你可以使用一个`(*`field_name_or_user_var`*, ...)`列表选择一个或多个 XML 字段（仅选择所需字段）或用户变量（存储相应字段值以供以后使用）。当你想要将数据从 XML 文件插入到表列中的名称与 XML 字段不匹配时，用户变量尤其有用。为了看到这是如何工作的，我们首先创建一个名为`individual`的表，其结构与`person`表相匹配，但列名不同：

```sql
mysql> CREATE TABLE individual (
 ->     individual_id INT NOT NULL PRIMARY KEY,
 ->     name1 VARCHAR(40) NULL,
 ->     name2 VARCHAR(40) NULL,
 ->     made TIMESTAMP
 -> );
Query OK, 0 rows affected (0.42 sec)
```

在这种情况下，你不能简单地直接将 XML 文件加载到表中，因为字段和列名不匹配：

```sql
mysql> LOAD XML INFILE '../bin/person-dump.xml' INTO TABLE test.individual;
ERROR 1263 (22004): Column set to default value; NULL supplied to NOT NULL column 'individual_id' at row 1
```

这是因为 MySQL 服务器会查找与目标表的列名匹配的字段名。你可以通过将字段值选择到用户变量中，然后使用`SET`将目标表的列设置为这些变量的值来解决这个问题。你可以在单个语句中执行这两个操作，如下所示：

```sql
mysql> LOAD XML INFILE '../bin/person-dump.xml'
 ->     INTO TABLE test.individual (@person_id, @fname, @lname, @created)
 ->     SET individual_id=@person_id, name1=@fname, name2=@lname, made=@created;
Query OK, 8 rows affected (0.05 sec)
Records: 8  Deleted: 0  Skipped: 0  Warnings: 0

mysql> SELECT * FROM individual;
+---------------+--------+------------+---------------------+
| individual_id | name1  | name2      | made                |
+---------------+--------+------------+---------------------+
|             1 | Kapek  | Sainnouine | 2007-07-13 16:18:47 |
|             2 | Sajon  | Rondela    | 2007-07-13 16:18:47 |
|             3 | Likema | Örrtmons   | 2007-07-13 16:18:47 |
|             4 | Slar   | Manlanth   | 2007-07-13 16:18:47 |
|             5 | Stoma  | Nilu       | 2007-07-13 16:18:47 |
|             6 | Nirtam | Sklöd      | 2007-07-13 16:18:47 |
|             7 | Sungam | Dulbåd     | 2007-07-13 16:18:47 |
|             8 | Srraf  | Encmelt    | 2007-07-13 16:18:47 |
+---------------+--------+------------+---------------------+
8 rows in set (0.00 sec)
```

用户变量的名称*必须*与 XML 文件中相应字段的名称匹配，并添加必需的`@`前缀以指示它们是变量。用户变量不需要按照相应字段的顺序列出或分配。

使用`ROWS IDENTIFIED BY '<*`tagname`*>'`子句，可以将来自相同 XML 文件的数据导入到具有不同定义的数据库表中。例如，假设你有一个名为`address.xml`的文件，其中包含以下 XML：

```sql
<?xml version="1.0"?>

<list>
  <person person_id="1">
    <fname>Robert</fname>
    <lname>Jones</lname>
    <address address_id="1" street="Mill Creek Road" zip="45365" city="Sidney"/>
    <address address_id="2" street="Main Street" zip="28681" city="Taylorsville"/>
  </person>

  <person person_id="2">
    <fname>Mary</fname>
    <lname>Smith</lname>
    <address address_id="3" street="River Road" zip="80239" city="Denver"/>
    <!-- <address address_id="4" street="North Street" zip="37920" city="Knoxville"/> -->
  </person>

</list>
```

在清空表中的所有现有记录并显示其结构后，你可以再次使用本节中先前定义的`test.person`表，如下所示：

```sql
mysql< TRUNCATE person;
Query OK, 0 rows affected (0.04 sec)

mysql< SHOW CREATE TABLE person\G
*************************** 1\. row ***************************
       Table: person
Create Table: CREATE TABLE `person` (
  `person_id` int(11) NOT NULL,
  `fname` varchar(40) DEFAULT NULL,
  `lname` varchar(40) DEFAULT NULL,
  `created` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`person_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 1 row in set (0.00 sec)
```

现在在`test`数据库中使用以下`CREATE TABLE`语句创建一个`address`表：

```sql
CREATE TABLE address (
    address_id INT NOT NULL PRIMARY KEY,
    person_id INT NULL,
    street VARCHAR(40) NULL,
    zip INT NULL,
    city VARCHAR(40) NULL,
    created TIMESTAMP
);
```

要将 XML 文件中的数据导入到`person`表中，请执行以下`LOAD XML`语句，该语句指定了行应由`<person>`元素指定，如下所示；

```sql
mysql> LOAD XML LOCAL INFILE 'address.xml'
 ->   INTO TABLE person
 ->   ROWS IDENTIFIED BY '<person>';
Query OK, 2 rows affected (0.00 sec)
Records: 2  Deleted: 0  Skipped: 0  Warnings: 0
```

你可以通过`SELECT`语句验证记录是否已导入：

```sql
mysql> SELECT * FROM person;
+-----------+--------+-------+---------------------+
| person_id | fname  | lname | created             |
+-----------+--------+-------+---------------------+
|         1 | Robert | Jones | 2007-07-24 17:37:06 |
|         2 | Mary   | Smith | 2007-07-24 17:37:06 |
+-----------+--------+-------+---------------------+
2 rows in set (0.00 sec)
```

由于 XML 文件中的`<address>`元素在`person`表中没有对应的列，因此它们被跳过。

要将`<address>`元素中的数据导入到`address`表中，请使用以下显示的`LOAD XML`语句：

```sql
mysql> LOAD XML LOCAL INFILE 'address.xml'
 ->   INTO TABLE address
 ->   ROWS IDENTIFIED BY '<address>';
Query OK, 3 rows affected (0.00 sec)
Records: 3  Deleted: 0  Skipped: 0  Warnings: 0
```

你可以通过类似于这样的`SELECT`语句查看导入的数据：

```sql
mysql> SELECT * FROM address;
+------------+-----------+-----------------+-------+--------------+---------------------+
| address_id | person_id | street          | zip   | city         | created             |
+------------+-----------+-----------------+-------+--------------+---------------------+
|          1 |         1 | Mill Creek Road | 45365 | Sidney       | 2007-07-24 17:37:37 |
|          2 |         1 | Main Street     | 28681 | Taylorsville | 2007-07-24 17:37:37 |
|          3 |         2 | River Road      | 80239 | Denver       | 2007-07-24 17:37:37 |
+------------+-----------+-----------------+-------+--------------+---------------------+
3 rows in set (0.00 sec)
```

被 XML 注释包围的`<address>`元素中的数据不会被导入。然而，由于`address`表中有一个`person_id`列，因此每个`<address>`元素的父级`<person>`元素的`person_id`属性值*会*被导入到`address`表中。

**安全注意事项。** 与 `LOAD DATA` 语句一样，从客户端主机到服务器主机的 XML 文件传输是由 MySQL 服务器发起的。理论上，可以构建一个经过修补的服务器，告诉客户端程序传输服务器选择的文件，而不是客户端在 `LOAD XML` 语句中命名的文件。这样的服务器可以访问客户端主机上客户端用户具有读取权限的任何文件。

在 Web 环境中，客户端通常从 Web 服务器连接到 MySQL。可以运行任何命令来针对 MySQL 服务器的用户可以使用 `LOAD XML LOCAL` 读取 Web 服务器进程具有读取权限的任何文件。在这种环境中，相对于 MySQL 服务器的客户端实际上是 Web 服务器，而不是连接到 Web 服务器的用户运行的远程程序。

您可以通过使用 `--local-infile=0` 或 `--local-infile=OFF` 启动服务器来禁用客户端加载 XML 文件。在启动 **mysql** 客户端时，也可以使用此选项禁用客户端会话期间的 `LOAD XML`。

要阻止客户端从服务器加载 XML 文件，不要向相应的 MySQL 用户帐户授予 `FILE` 权限，或者如果客户端用户帐户已经具有该权限，则撤销此权限。

重要

撤销 `FILE` 权限（或者一开始不授予它）只会阻止用户执行 `LOAD XML` 语句（以及 `LOAD_FILE()` 函数；它 *不* 阻止用户执行 `LOAD XML LOCAL`。要禁止此语句，必须使用 `--local-infile=OFF` 启动服务器或客户端。

换句话说，`FILE` 权限仅影响客户端是否可以读取服务器上的文件；它与客户端是否可以读取本地文件系统上的文件无关。
