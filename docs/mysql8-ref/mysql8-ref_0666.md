# 11.6 查询属性

> 原文：[`dev.mysql.com/doc/refman/8.0/en/query-attributes.html`](https://dev.mysql.com/doc/refman/8.0/en/query-attributes.html)

SQL 语句最显著的部分是语句的文本。从 MySQL 8.0.23 开始，客户端还可以定义适用于发送到服务器执行的下一个语句的查询属性：

+   在发送语句之前定义属性。

+   属性存在直到语句执行结束，此时属性集被清除。

+   在属性存在期间，它们可以在服务器端访问。

查询属性可能被用于的示例方式：

+   一个 Web 应用程序生成页面，生成数据库查询，并且对于每个查询必须跟踪生成它的页面的 URL。

+   应用程序在每个查询中传递额外的处理信息，供插件（如审计插件或查询重写插件）使用。

MySQL 支持这些功能，无需使用诸如在查询字符串中包含特殊格式的注释之类的变通方法。本节的其余部分描述了如何使用查询属性支持，包括必须满足的先决条件。

+   定义和访问查询属性

+   使用查询属性的先决条件

+   查询属性可加载函数

### 定义和访问查询属性

使用 MySQL C API 的应用程序通过调用`mysql_bind_param()`函数定义查询属性。参见 mysql_bind_param()。其他 MySQL 连接器也可能提供查询属性支持。请参阅各个连接器的文档。

**mysql**客户端具有`query_attributes`命令，可以定义最多 32 对属性名称和值。参见 Section 6.5.1.2, “mysql 客户端命令”。

查询属性名称使用由`character_set_client`系统变量指示的字符集传输。

要在已定义属性的 SQL 语句中访问查询属性，请按照使用查询属性的先决条件中描述的方式安装`query_attributes`组件。该组件实现了一个`mysql_query_attribute_string()`可加载函数，该函数接受属性名称参数并将属性值作为字符串返回，如果属性不存在，则返回`NULL`。参见查询属性可加载函数。

以下示例使用**mysql**客户端的`query_attributes`命令来定义属性名称/值对，并使用`mysql_query_attribute_string()`函数通过名称访问属性值。

此示例定义了两个名为`n1`和`n2`的属性。第一个`SELECT`显示了如何检索这些属性，并演示了检索不存在属性（`n3`）会返回`NULL`。第二个`SELECT`显示了属性在语句之间不会持久存在。

```sql
mysql> query_attributes n1 v1 n2 v2;
mysql> SELECT
         mysql_query_attribute_string('n1') AS 'attr 1',
         mysql_query_attribute_string('n2') AS 'attr 2',
         mysql_query_attribute_string('n3') AS 'attr 3';
+--------+--------+--------+
| attr 1 | attr 2 | attr 3 |
+--------+--------+--------+
| v1     | v2     | NULL   |
+--------+--------+--------+

mysql> SELECT
         mysql_query_attribute_string('n1') AS 'attr 1',
         mysql_query_attribute_string('n2') AS 'attr 2';
+--------+--------+
| attr 1 | attr 2 |
+--------+--------+
| NULL   | NULL   |
+--------+--------+
```

如第二个`SELECT`语句所示，在给定语句之前定义的属性仅对该语句可用，并在语句执行后被清除。要在多个语句中使用属性值，请将其分配给变量。以下示例显示了如何执行此操作，并说明了属性值通过变量在后续语句中可用，但不能通过调用`mysql_query_attribute_string()`来获取：

```sql
mysql> query_attributes n1 v1 n2 v2;
mysql> SET
         @attr1 = mysql_query_attribute_string('n1'),
         @attr2 = mysql_query_attribute_string('n2');

mysql> SELECT
         @attr1, mysql_query_attribute_string('n1') AS 'attr 1',
         @attr2, mysql_query_attribute_string('n2') AS 'attr 2';
+--------+--------+--------+--------+
| @attr1 | attr 1 | @attr2 | attr 2 |
+--------+--------+--------+--------+
| v1     | NULL   | v2     | NULL   |
+--------+--------+--------+--------+
```

属性也可以通过将它们存储在表中以供以后使用：

```sql
mysql> CREATE TABLE t1 (c1 CHAR(20), c2 CHAR(20));

mysql> query_attributes n1 v1 n2 v2;
mysql> INSERT INTO t1 (c1, c2) VALUES(
         mysql_query_attribute_string('n1'),
         mysql_query_attribute_string('n2')
       );

mysql> SELECT * FROM t1;
+------+------+
| c1   | c2   |
+------+------+
| v1   | v2   |
+------+------+
```

查询属性受到这些限制和限制：

+   如果在将语句发送到服务器执行之前发生多个属性定义操作，则最近的定义操作适用并替换了之前操作中定义的属性。

+   如果使用相同名称定义了多个属性，则尝试检索属性值会产生未定义的结果。

+   使用空名称定义的属性无法通过名称检索。

+   属性对使用`PREPARE`准备的语句不可用。

+   `mysql_query_attribute_string()`函数不能在 DDL 语句中使用。

+   属性不会被复制。调用`mysql_query_attribute_string()`函数的语句在所有服务器上不会得到相同的值。

### 使用查询属性的先决条件

要在已定义属性的 SQL 语句中访问查询属性，必须安装 `query_attributes` 组件。请使用以下语句执行此操作：

```sql
INSTALL COMPONENT "file://component_query_attributes";
```

组件安装是一次性操作，不需要在每次服务器启动时执行。`INSTALL COMPONENT` 加载组件，并在 `mysql.component` 系统表中注册它，以使其在后续服务器启动期间加载。

`query_attributes` 组件访问查询属性以实现 `mysql_query_attribute_string()` 函数。参见 第 7.5.4 节，“查询属性组件”。

要卸载 `query_attributes` 组件，请使用以下语句：

```sql
UNINSTALL COMPONENT "file://component_query_attributes";
```

`UNINSTALL COMPONENT` 卸载组件，并从 `mysql.component` 系统表中注销它，以使其在后续服务器启动期间不被加载。

因为安装和卸载 `query_attributes` 组件会安装和卸载组件实现的 `mysql_query_attribute_string()` 函数，所以不需要使用 `CREATE FUNCTION` 或 `DROP FUNCTION` 来执行此操作。

### 查询属性可加载函数

+   `mysql_query_attribute_string(*`name`*)`

    应用程序可以定义应用于发送到服务器的下一个查询的属性。`mysql_query_attribute_string()` 函数自 MySQL 8.0.23 起可用，根据属性名称返回属性值作为字符串。此函数使查询能够访问和合并适用于它的属性值。

    通过安装 `query_attributes` 组件来安装 `mysql_query_attribute_string()`。参见 第 11.6 节，“查询属性”，该节还讨论了查询属性的目的和用途。

    参数：

    +   *`name`*: 属性名称。

    返回值：

    返回成功的属性值作为字符串，如果属性不存在则返回 `NULL`。

    示例：

    以下示例使用 **mysql** 客户端 `query_attributes` 命令来定义可以被 `mysql_query_attribute_string()` 检索的查询属性。`SELECT` 显示检索不存在属性（`n3`）返回 `NULL`。

    ```sql
    mysql> query_attributes n1 v1 n2 v2;
    mysql> SELECT
     ->   mysql_query_attribute_string('n1') AS 'attr 1',
     ->   mysql_query_attribute_string('n2') AS 'attr 2',
     ->   mysql_query_attribute_string('n3') AS 'attr 3';
    +--------+--------+--------+
    | attr 1 | attr 2 | attr 3 |
    +--------+--------+--------+
    | v1     | v2     | NULL   |
    +--------+--------+--------+
    ```
