# 28.3.30 INFORMATION_SCHEMA ROUTINES 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-routines-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-routines-table.html)

`ROUTINES` 表提供有关存储例程（存储过程和存储函数）的信息。`ROUTINES` 表不包括内置（本地）函数或可加载函数。

`ROUTINES` 表包含以下列：

+   `SPECIFIC_NAME`

    例程的名称。

+   `ROUTINE_CATALOG`

    例程所属的目录名称。该值始终为`def`。

+   `ROUTINE_SCHEMA`

    例程所属的模式（数据库）的名称。

+   `ROUTINE_NAME`

    例程的名称。

+   `ROUTINE_TYPE`

    对于存储过程，`PROCEDURE`；对于存储函数，`FUNCTION`。

+   `DATA_TYPE`

    如果例程是存储函数，则返回值数据类型。如果例程是存储过程，则该值为空。

    `DATA_TYPE` 值仅为类型名称，没有其他信息。`DTD_IDENTIFIER` 值包含类型名称和可能的其他信息，如精度或长度。

+   `CHARACTER_MAXIMUM_LENGTH`

    对于存储函数的字符串返回值，最大长度（以字符计）。如果例程是存储过程，则该值为`NULL`。

+   `CHARACTER_OCTET_LENGTH`

    对于存储函数的字符串返回值，最大长度（以字节计）。如果例程是存储过程，则该值为`NULL`。

+   `NUMERIC_PRECISION`

    对于存储函数的数值返回值，数值精度。如果例程是存储过程，则该值为`NULL`。

+   `NUMERIC_SCALE`

    对于存储函数的数值返回值，数值精度。如果例程是存储过程，则该值为`NULL`。

+   `DATETIME_PRECISION`

    对于存储函数的时间返回值，小数秒精度。如果例程是存储过程，则该值为`NULL`。

+   `CHARACTER_SET_NAME`

    对于存储函数的字符字符串返回值，字符集名称。如果例程是存储过程，则该值为`NULL`。

+   `COLLATION_NAME`

    对于存储函数的字符字符串返回值，排序规则名称。如果例程是存储过程，则该值为`NULL`。

+   `DTD_IDENTIFIER`

    如果例程是存储函数，则返回值数据类型。如果例程是存储过程，则该值为空。

    `DATA_TYPE` 值仅为类型名称，没有其他信息。`DTD_IDENTIFIER` 值包含类型名称和可能的其他信息，如精度或长度。

+   `ROUTINE_BODY`

    用于例程定义的语言。该值始终为`SQL`。

+   `ROUTINE_DEFINITION`

    例程执行的 SQL 语句文本。

+   `EXTERNAL_NAME`

    该值始终为`NULL`。

+   `EXTERNAL_LANGUAGE`

    存储例程的语言。该值从`mysql.routines`数据字典表的`external_language`列中读取。

+   `PARAMETER_STYLE`

    此值始终为`SQL`。

+   `IS_DETERMINISTIC`

    根据例程是否定义了`DETERMINISTIC`特性，为`YES`或`NO`。

+   `SQL_DATA_ACCESS`

    例程的数据访问特性。该值为`CONTAINS SQL`、`NO SQL`、`READS SQL DATA`或`MODIFIES SQL DATA`之一。

+   `SQL_PATH`

    此值始终为`NULL`。

+   `SECURITY_TYPE`

    例程的`SQL SECURITY`特性。该值为`DEFINER`或`INVOKER`之一。

+   `CREATED`

    创建例程的日期和时间。这是一个`TIMESTAMP`值。

+   `LAST_ALTERED`

    上次修改例程的日期和时间。这是一个`TIMESTAMP`值。如果自创建以来未修改例程，则此值与`CREATED`值相同。

+   `SQL_MODE`

    创建或更改例程时生效的 SQL 模式，以及例程执行时的模式。有关允许的值，请参见第 7.1.11 节，“服务器 SQL 模式”。

+   `ROUTINE_COMMENT`

    如果例程有注释，则为注释的文本。如果没有，则此值为空。

+   `DEFINER`

    `DEFINER`子句中命名的帐户（通常是创建例程的用户），格式为`'*`user_name`*'@'*`host_name`*'`。

+   `CHARACTER_SET_CLIENT`

    创建例程时的`character_set_client`系统变量的会话值。

+   `COLLATION_CONNECTION`

    创建例程时的`collation_connection`系统变量的会话值。

+   `DATABASE_COLLATION`

    与例程关联的数据库的排序规则。

#### 注意事项

+   要查看有关例程的信息，您必须是例程`DEFINER`命名的用户，具有`SHOW_ROUTINE`权限，在全局级别具有`SELECT`权限，或者在包括例程的范围内被授予`CREATE ROUTINE`、`ALTER ROUTINE`或`EXECUTE`权限。如果您只有`CREATE ROUTINE`、`ALTER ROUTINE`或`EXECUTE`权限，则`ROUTINE_DEFINITION`列为`NULL`。

+   存储函数返回值的信息也可以在`PARAMETERS`表中找到。存储函数的返回值行可以通过具有`ORDINAL_POSITION`值为 0 的行来识别。
