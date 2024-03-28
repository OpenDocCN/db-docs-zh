> 原文：[`dev.mysql.com/doc/refman/8.0/en/rewriter-query-rewrite-plugin-usage.html`](https://dev.mysql.com/doc/refman/8.0/en/rewriter-query-rewrite-plugin-usage.html)

#### 7.6.4.2 使用 Rewriter 查询重写插件

要启用或禁用插件，请启用或禁用`rewriter_enabled`系统变量。默认情况下，安装插件时`Rewriter`插件是启用的（参见第 7.6.4.1 节，“安装或卸载 Rewriter 查询重写插件”）。要显式设置初始插件状态，可以在服务器启动时设置该变量。例如，要在选项文件中启用插件，请使用以下行：

```sql
[mysqld]
rewriter_enabled=ON
```

也可以在运行时启用或禁用插件：

```sql
SET GLOBAL rewriter_enabled = ON;
SET GLOBAL rewriter_enabled = OFF;
```

假设`Rewriter`插件已启用，它会检查并可能修改服务器接收到的每个可重写语句。插件根据其内存中的重写规则缓存来决定是否重写语句，这些规则从`query_rewrite`数据库中的`rewrite_rules`表中加载。

这些语句会被重写：

+   截至 MySQL 8.0.12：`SELECT`、`INSERT`、`REPLACE`、`UPDATE`和`DELETE`。

+   在 MySQL 8.0.12 之前：仅`SELECT`。

独立语句和准备语句会被重写。在视图定义或存储程序中出现的语句不会被重写。

从 MySQL 8.0.31 开始，具有`SKIP_QUERY_REWRITE`权限的用户运行的语句不会被重写，前提是`rewriter_enabled_for_threads_without_privilege_checks`系统变量设置为`OFF`（默认为`ON`）。这可用于控制语句和应该保持不变的语句，例如来自由`CHANGE REPLICATION SOURCE TO`指定的`SOURCE_USER`的语句。对于由 MySQL 客户端程序执行的语句，包括**mysqlbinlog**、**mysqladmin**、**mysqldump**和**mysqlpump**；因此，您应该授予`SKIP_QUERY_REWRITE`给这些实用程序用于连接到 MySQL 的用户帐户或帐户。

+   添加重写规则

+   语句匹配工作原理

+   重写准备语句

+   重写插件操作信息

+   字符集的重写插件使用

##### 添加重写规则

要为`Rewriter`插件添加规则，请向`rewrite_rules`表添加行，然后调用`flush_rewrite_rules()`存储过程将规则从表中加载到插件中。以下示例创建了一个简单的规则，用于匹配选择单个文字值的语句：

```sql
INSERT INTO query_rewrite.rewrite_rules (pattern, replacement)
VALUES('SELECT ?', 'SELECT ? + 1');
```

结果表内容如下所示：

```sql
mysql> SELECT * FROM query_rewrite.rewrite_rules\G
*************************** 1\. row ***************************
                id: 1
           pattern: SELECT ?
  pattern_database: NULL
       replacement: SELECT ? + 1
           enabled: YES
           message: NULL
    pattern_digest: NULL
normalized_pattern: NULL
```

规则指定了一个模式模板，指示要匹配哪些`SELECT`语句，并指定了一个替换模板，指示如何重写匹配的语句。但是，将规则添加到`rewrite_rules`表中并不足以使`Rewriter`插件使用该规则。您必须调用`flush_rewrite_rules()`将表内容加载到插件的内存缓存中：

```sql
mysql> CALL query_rewrite.flush_rewrite_rules();
```

提示

如果您的重写规则似乎无法正常工作，请确保通过调用`flush_rewrite_rules()`重新加载规则表。

当插件从规则表中读取每个规则时，它会从模式计算出一个规范化（语句摘要）形式和一个摘要哈希值，并使用它们来更新`normalized_pattern`和`pattern_digest`列：

```sql
mysql> SELECT * FROM query_rewrite.rewrite_rules\G
*************************** 1\. row ***************************
                id: 1
           pattern: SELECT ?
  pattern_database: NULL
       replacement: SELECT ? + 1
           enabled: YES
           message: NULL
    pattern_digest: d1b44b0c19af710b5a679907e284acd2ddc285201794bc69a2389d77baedddae
normalized_pattern: select ?
```

有关语句摘要、规范化语句和摘要哈希值的信息，请参见第 29.10 节，“性能模式语句摘要和采样”。

如果由于某些错误而无法加载��则，则调用`flush_rewrite_rules()`会产生一个错误：

```sql
mysql> CALL query_rewrite.flush_rewrite_rules();
ERROR 1644 (45000): Loading of some rule(s) failed.
```

当发生这种情况时，插件会将错误消息写入规则行的`message`列，以传达问题。检查`rewrite_rules`表，查看具有非`NULL` `message`列值的行，以查看存在哪些问题。

模式使用与准备语句相同的语法（参见第 15.5.1 节，“PREPARE 语句”）。在模式模板中，`?`字符充当匹配数据值的参数标记。`?`字符不应包含在引号内。参数标记仅可用于数据值应出现的位置，不能用于 SQL 关键字、标识符、函数等。插件解析语句以识别文本值（如第 11.1 节，“文本值”中定义的那样），因此您可以在任何文本值的位置放置参数标记。

像模式一样，替换内容可以包含`?`字符。对于与模式模板匹配的语句，插件会重写它，使用数据值替换替换中的`?`参数标记，这些数据值由模式中相应标记匹配的数据值确定。结果是一个完整的语句字符串。插件要求服务器解析它，并将重写后的语句表示返回给服务器。

添加并加载规则后，请检查是否根据语句是否与规则模式匹配而进行重写：

```sql
mysql> SELECT PI();
+----------+
| PI()     |
+----------+
| 3.141593 |
+----------+
1 row in set (0.01 sec)

mysql> SELECT 10;
+--------+
| 10 + 1 |
+--------+
|     11 |
+--------+
1 row in set, 1 warning (0.00 sec)
```

第一个`SELECT`语句不会进行重写，但第二个会。第二个语句说明了当`Rewriter`插件重写语句时，会生成警告消息。要查看消息，请使用`SHOW WARNINGS`：

```sql
mysql> SHOW WARNINGS\G
*************************** 1\. row ***************************
  Level: Note
   Code: 1105
Message: Query 'SELECT 10' rewritten to 'SELECT 10 + 1' by a query rewrite plugin
```

语句不必重写为相同类型的语句。以下示例加载一个将`DELETE`语句重写为`UPDATE`语句的规则：

```sql
INSERT INTO query_rewrite.rewrite_rules (pattern, replacement)
VALUES('DELETE FROM db1.t1 WHERE col = ?',
       'UPDATE db1.t1 SET col = NULL WHERE col = ?');
CALL query_rewrite.flush_rewrite_rules();
```

要启用或禁用现有规则，请修改其`enabled`列并重新加载表到插件中。要禁用规则 1：

```sql
UPDATE query_rewrite.rewrite_rules SET enabled = 'NO' WHERE id = 1;
CALL query_rewrite.flush_rewrite_rules();
```

这使您可以停用规则而无需将其从表中删除。

要重新启用规则 1：

```sql
UPDATE query_rewrite.rewrite_rules SET enabled = 'YES' WHERE id = 1;
CALL query_rewrite.flush_rewrite_rules();
```

`rewrite_rules`表包含一个`pattern_database`列，`Rewriter`用于匹配未使用数据库名称限定的表名：

+   语句中的限定表名仅在相应数据库和表名相同的情况下与模式中的限定名称匹配。

+   语句中的未限定表名仅在默认数据库与`pattern_database`相同且表名相同的情况下与模式中的未限定名称匹配。

假设一个名为`appdb.users`的表有一个名为`id`的列，并且应用程序预期使用以下形式之一的查询从表中选择行，第二种形式可以在`appdb`是默认数据库时使用：

```sql
SELECT * FROM users WHERE appdb.id = *id_value*;
SELECT * FROM users WHERE id = *id_value*;
```

还假设`id`列被重命名为`user_id`（也许必须修改表以添加另一种 ID，并且有必要更明确地指示`id`列代表什么类型的 ID）。

更改意味着应用程序必须在`WHERE`子句中引用`user_id`而不是`id`，但无法更新的旧应用程序将不再正常工作。`Rewriter`插件可以通过匹配和重写有问题的语句来解决此问题。要将语句`SELECT * FROM appdb.users WHERE id = *`value`*`匹配并重写为`SELECT * FROM appdb.users WHERE user_id = *`value`*`，您可以向重写规则表中插入代表替换规则的行。如果还想使用未限定表名匹配此`SELECT`，还需要添加一个显式规则。使用`?`作为值占位符，需要的两个`INSERT`语句如下：

```sql
INSERT INTO query_rewrite.rewrite_rules
    (pattern, replacement) VALUES(
    'SELECT * FROM appdb.users WHERE id = ?',
    'SELECT * FROM appdb.users WHERE user_id = ?'
    );
INSERT INTO query_rewrite.rewrite_rules
    (pattern, replacement, pattern_database) VALUES(
    'SELECT * FROM users WHERE id = ?',
    'SELECT * FROM users WHERE user_id = ?',
    'appdb'
    );
```

添加两个新规则后，执行以下语句使其生效：

```sql
CALL query_rewrite.flush_rewrite_rules();
```

`Rewriter`使用第一条规则匹配使用限定表名的语句，第二条规则匹配使用未限定名称的语句。只有当`appdb`是默认数据库时，第二条规则才有效。

##### 语句匹配原理

`Rewriter`插件使用语句摘要和摘要哈希值来在各个阶段匹配传入语句与重写规则。`max_digest_length`系统变量确定用于计算语句摘要的缓冲区大小。较大的值可以计算出区分较长语句的摘要。较小的值使用更少的内存，但增加了较长语句与相同摘要值发生冲突的可能性。

该插件将每个语句与重写规则进行匹配：

1.  计算语句摘要哈希值并将其与规则摘要哈希值进行比较。这可能会产生误报，但可以作为快速拒绝测试。

1.  如果语句摘要哈希值与任何模式摘要哈希值匹配，则将语句的规范形式（语句摘要）与匹配规则模式的规范形式进行匹配。

1.  如果规范语句与规则匹配，则比较语句和模式中的文字值。模式中的`?`字符匹配语句中的任何文字值。如果语句准备了一个语句，则模式中的`?`也匹配语句中的`?`。否则，相应的文字值必须相同。

如果多个规则匹配一个语句，则插件使用哪个规则重写语句是不确定的。

如果模式包含的标记比替换多，插件会丢弃多余的数据值。如果模式包含的标记比替换少，这将是一个错误。当加载规则表时，插件会注意到这一点，将错误消息写入规则行的`message`列以传达问题，并将`Rewriter_reload_error`状态变量设置为`ON`。

##### 重写预处理语句

预处理语句在解析时（即在准备时）重写，而不是在稍后执行时。

预处理语句与非预处理语句的区别在于，它们可能包含`?`字符作为参数标记。要匹配预处理语句中的`?`，`Rewriter`模式必须在相同位置包含`?`。假设重写规则具有以下模式：

```sql
SELECT ?, 3
```

以下表格显示了几个预处理`SELECT`语句以及规则模式是否匹配它们。

| 预处理语句 | 匹配语句的模式是否匹配 |
| --- | --- |
| `PREPARE s AS 'SELECT 3, 3'` | 是 |
| `PREPARE s AS 'SELECT ?, 3'` | 是 |
| `PREPARE s AS 'SELECT 3, ?'` | 否 |
| `PREPARE s AS 'SELECT ?, ?'` | 否 |

##### 重写插件操作信息

`Rewriter`插件通过几个状态变量提供有关其操作的信息：

```sql
mysql> SHOW GLOBAL STATUS LIKE 'Rewriter%';
+-----------------------------------+-------+
| Variable_name                     | Value |
+-----------------------------------+-------+
| Rewriter_number_loaded_rules      | 1     |
| Rewriter_number_reloads           | 5     |
| Rewriter_number_rewritten_queries | 1     |
| Rewriter_reload_error             | ON    |
+-----------------------------------+-------+
```

有关这些变量的描述，请参阅 Section 7.6.4.3.4, “Rewriter Query Rewrite Plugin Status Variables”。

当通过调用`flush_rewrite_rules()`存储过程加载规则表时，如果某个规则出现错误，则`CALL`语句会产生错误，并且插件将`Rewriter_reload_error`状态变量设置为`ON`：

```sql
mysql> CALL query_rewrite.flush_rewrite_rules();
ERROR 1644 (45000): Loading of some rule(s) failed. 
mysql> SHOW GLOBAL STATUS LIKE 'Rewriter_reload_error';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| Rewriter_reload_error | ON    |
+-----------------------+-------+
```

在这种情况下，检查`rewrite_rules`表中具有非`NULL` `message`列值的行，以查看存在哪些问题。

##### 重写插件使用字符集

当`rewrite_rules`表加载到`Rewriter`插件中时，插件使用当前全局值的`character_set_client`系统变量解释语句。如果随后更改了全局`character_set_client`值，则必须重新加载规则表。

客户端必须具有与加载规则表时全局值相同的会话`character_set_client`值，否则该客户端的规则匹配将无法工作。
