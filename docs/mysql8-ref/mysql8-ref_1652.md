# 25.3.5 带有表和数据的 NDB Cluster 示例

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-example-data.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-example-data.html)

注意

本节内容适用于在 Unix 和 Windows 平台上运行的 NDB Cluster。

与在标准 MySQL 中操作数据库表和数据并没有太大区别。有两个关键点需要记住：

+   要在集群中复制表，必须使用`NDBCLUSTER`存储引擎。在创建表时，使用`ENGINE=NDBCLUSTER`或`ENGINE=NDB`选项来指定：

    ```sql
    CREATE TABLE *tbl_name* (*col_name* *column_definitions*) ENGINE=NDBCLUSTER;
    ```

    或者，对于使用不同存储引擎的现有表，可以使用`ALTER TABLE`将表更改为使用`NDBCLUSTER`：

    ```sql
    ALTER TABLE *tbl_name* ENGINE=NDBCLUSTER;
    ```

+   每个`NDBCLUSTER`表都有一个主键。如果用户在创建表时没有定义主键，`NDBCLUSTER`存储引擎会自动生成一个隐藏的主键。这样的主键占用空间，就像任何其他表索引一样。（由于内存不足以容纳这些自动生成的索引，遇到问题并不罕见。）

如果您正在使用**mysqldump**的输出从现有数据库导入表，您可以在文本编辑器中打开 SQL 脚本，并为任何表创建语句添加`ENGINE`选项，或替换任何现有的`ENGINE`选项。假设您在另一个不支持 NDB Cluster 的 MySQL 服务器上有`world`示例数据库，并且想要导出`City`表：

```sql
$> mysqldump --add-drop-table world City > city_table.sql
```

生成的`city_table.sql`文件包含此表创建语句（以及导入表数据所需的`INSERT`语句）:

```sql
DROP TABLE IF EXISTS `City`;
CREATE TABLE `City` (
  `ID` int(11) NOT NULL auto_increment,
  `Name` char(35) NOT NULL default '',
  `CountryCode` char(3) NOT NULL default '',
  `District` char(20) NOT NULL default '',
  `Population` int(11) NOT NULL default '0',
  PRIMARY KEY  (`ID`)
) ENGINE=MyISAM;

INSERT INTO `City` VALUES (1,'Kabul','AFG','Kabol',1780000);
INSERT INTO `City` VALUES (2,'Qandahar','AFG','Qandahar',237500);
INSERT INTO `City` VALUES (3,'Herat','AFG','Herat',186800);
*(remaining INSERT statements omitted)*
```

你需要确保 MySQL 为这个表使用`NDBCLUSTER`存储引擎。有两种方法可以实现这一点。其中一种是在将表导入到集群数据库*之前*修改表定义。以`City`表为例，修改定义的`ENGINE`选项如下：

```sql
DROP TABLE IF EXISTS `City`;
CREATE TABLE `City` (
  `ID` int(11) NOT NULL auto_increment,
  `Name` char(35) NOT NULL default '',
  `CountryCode` char(3) NOT NULL default '',
  `District` char(20) NOT NULL default '',
  `Population` int(11) NOT NULL default '0',
  PRIMARY KEY  (`ID`)
) ENGINE=NDBCLUSTER;

INSERT INTO `City` VALUES (1,'Kabul','AFG','Kabol',1780000);
INSERT INTO `City` VALUES (2,'Qandahar','AFG','Qandahar',237500);
INSERT INTO `City` VALUES (3,'Herat','AFG','Herat',186800);
*(remaining INSERT statements omitted)*
```

必须为要成为集群数据库一部分的每个表的定义执行此操作。实现这一点的最简单方法是在包含定义的文件上执行搜索和替换，并将所有`TYPE=*`engine_name`*`或`ENGINE=*`engine_name`*`的实例替换为`ENGINE=NDBCLUSTER`。如果您不想修改文件，可以使用未修改的文件创建表，然后使用`ALTER TABLE`来更改它们的存储引擎。具体细节稍后在本节中给出。

假设您已经在集群的 SQL 节点上创建了名为`world`的数据库，然后您可以使用**mysql**命令行客户端读取`city_table.sql`，并按照通常的方式创建和填充相应的表：

```sql
$> mysql world < city_table.sql
```

非常重要的一点是要记住，前述命令必须在运行 SQL 节点的主机上执行（在本例中，在具有 IP 地址`198.51.100.20`的机器上）。

要在 SQL 节点上创建整个`world`数据库的副本，请在非集群服务器上使用**mysqldump**将数据库导出到名为`world.sql`的文件中（例如，在`/tmp`目录中）。然后按照刚才描述的方式修改表定义，并将文件导入到集群的 SQL 节点中：

```sql
$> mysql world < /tmp/world.sql
```

如果您将文件保存到不同位置，请相应调整前述说明。

在 SQL 节点上运行`SELECT`查询与在任何其他 MySQL 服务器实例上运行它们没有任何区别。要从命令行运行查询，您首先需要以通常的方式登录到 MySQL Monitor（在`Enter password:`提示处指定`root`密码）：

```sql
$> mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1 to server version: 8.0.35-ndb-8.0.35

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql>
```

我们只使用 MySQL 服务器的`root`帐户，并假设您已经遵循了安装 MySQL 服务器的标准安全预防措施，包括设置强大的`root`密码。有关更多信息，请参见 Section 2.9.4, “Securing the Initial MySQL Account”。

值得注意的是，NDB Cluster 节点在相互访问时*不*使用 MySQL 权限系统。设置或更改 MySQL 用户帐户（包括`root`帐户）仅影响访问 SQL 节点的应用程序，而不影响节点之间的交互。有关更多信息，请参见 Section 25.6.20.2, “NDB Cluster and MySQL Privileges”。

如果在导入 SQL 脚本之前未修改表定义中的`ENGINE`子句，则应在此时运行以下语句：

```sql
mysql> USE world;
mysql> ALTER TABLE City ENGINE=NDBCLUSTER;
mysql> ALTER TABLE Country ENGINE=NDBCLUSTER;
mysql> ALTER TABLE CountryLanguage ENGINE=NDBCLUSTER;
```

选择数据库并针对该数据库中的表运行**SELECT**查询也是按照通常的方式完成的，退出 MySQL Monitor 也是一样：

```sql
mysql> USE world;
mysql> SELECT Name, Population FROM City ORDER BY Population DESC LIMIT 5;
+-----------+------------+
| Name      | Population |
+-----------+------------+
| Bombay    |   10500000 |
| Seoul     |    9981619 |
| São Paulo |    9968485 |
| Shanghai  |    9696300 |
| Jakarta   |    9604900 |
+-----------+------------+
5 rows in set (0.34 sec)

mysql> \q
Bye

$>
```

使用 MySQL 的应用程序可以使用标准 API 来访问`NDB`表。重要的是要记住，您的应用程序必须访问 SQL 节点，而不是管理或数据节点。这个简短的示例展示了我们如何通过在网络上的其他地方运行的 PHP 5.X `mysqli`扩展来执行刚刚显示的`SELECT`语句：

```sql
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
  "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
  <meta http-equiv="Content-Type"
           content="text/html; charset=iso-8859-1">
  <title>SIMPLE mysqli SELECT</title>
</head>
<body>
<?php
  # connect to SQL node:
  $link = new mysqli('198.51.100.20', 'root', '*root_password*', 'world');
  # parameters for mysqli constructor are:
  #   host, user, password, database

  if( mysqli_connect_errno() )
    die("Connect failed: " . mysqli_connect_error());

  $query = "SELECT Name, Population
            FROM City
            ORDER BY Population DESC
            LIMIT 5";

  # if no errors...
  if( $result = $link->query($query) )
  {
?>
<table border="1" width="40%" cellpadding="4" cellspacing ="1">
  <tbody>
  <tr>
    <th width="10%">City</th>
    <th>Population</th>
  </tr>
<?
    # then display the results...
    while($row = $result->fetch_object())
      printf("<tr>\n  <td align=\"center\">%s</td><td>%d</td>\n</tr>\n",
              $row->Name, $row->Population);
?>
  </tbody
</table>
<?
  # ...and verify the number of rows that were retrieved
    printf("<p>Affected rows: %d</p>\n", $link->affected_rows);
  }
  else
    # otherwise, tell us what went wrong
    echo mysqli_error();

  # free the result set and the mysqli connection object
  $result->close();
  $link->close();
?>
</body>
</html>
```

我们假设运行在 Web 服务器上的进程可以访问 SQL 节点的 IP 地址。

以类似的方式，您可以使用 MySQL C API、Perl-DBI、Python-mysql 或 MySQL Connectors 来执行数据定义和操作任务，就像您通常使用 MySQL 一样。
