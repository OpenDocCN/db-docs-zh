# 14.17.5 返回 JSON 值属性的函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/json-attribute-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/json-attribute-functions.html)

本节中的函数返回 JSON 值的属性。

+   `JSON_DEPTH(*`json_doc`*)`

    返回 JSON 文档的最大深度。如果参数为 `NULL`，则返回 `NULL`。如果参数不是有效的 JSON 文档，则会出现错误。

    一个空数组、空对象或标量值的深度为 1。一个只包含深度为 1 的元素的非空数组，或者只包含深度为 1 的成员值的非空对象，其深度为 2。否则，JSON 文档的深度大于 2。

    ```sql
    mysql> SELECT JSON_DEPTH('{}'), JSON_DEPTH('[]'), JSON_DEPTH('true');
    +------------------+------------------+--------------------+
    | JSON_DEPTH('{}') | JSON_DEPTH('[]') | JSON_DEPTH('true') |
    +------------------+------------------+--------------------+
    |                1 |                1 |                  1 |
    +------------------+------------------+--------------------+
    mysql> SELECT JSON_DEPTH('[10, 20]'), JSON_DEPTH('[[], {}]');
    +------------------------+------------------------+
    | JSON_DEPTH('[10, 20]') | JSON_DEPTH('[[], {}]') |
    +------------------------+------------------------+
    |                      2 |                      2 |
    +------------------------+------------------------+
    mysql> SELECT JSON_DEPTH('[10, {"a": 20}]');
    +-------------------------------+
    | JSON_DEPTH('[10, {"a": 20}]') |
    +-------------------------------+
    |                             3 |
    +-------------------------------+
    ```

+   [`JSON_LENGTH(*`json_doc`*[, *`path`*])`](json-attribute-functions.html#function_json-length)

    返回 JSON 文档的长度，或者如果给定了 *`path`* 参数，则返回标识路径中的值的文档内值的长度。如果任何参数为 `NULL` 或 *`path`* 参数在文档中没有标识值，则返回 `NULL`。如果 *`json_doc`* 参数不是有效的 JSON 文档，或者 *`path`* 参数不是有效的路径��达式，则会出现错误。在 MySQL 8.0.26 之前，如果路径表达式包含 `*` 或 `**` 通配符，还会引发错误。

    文档的长度如下确定：

    +   标量的长度为 1。

    +   一个数组的长度是数组元素的数量。

    +   一个对象的长度是对象成员的数量。

    +   长度不包括嵌套数组或对象的长度。

    ```sql
    mysql> SELECT JSON_LENGTH('[1, 2, {"a": 3}]');
    +---------------------------------+
    | JSON_LENGTH('[1, 2, {"a": 3}]') |
    +---------------------------------+
    |                               3 |
    +---------------------------------+
    mysql> SELECT JSON_LENGTH('{"a": 1, "b": {"c": 30}}');
    +-----------------------------------------+
    | JSON_LENGTH('{"a": 1, "b": {"c": 30}}') |
    +-----------------------------------------+
    |                                       2 |
    +-----------------------------------------+
    mysql> SELECT JSON_LENGTH('{"a": 1, "b": {"c": 30}}', '$.b');
    +------------------------------------------------+
    | JSON_LENGTH('{"a": 1, "b": {"c": 30}}', '$.b') |
    +------------------------------------------------+
    |                                              1 |
    +------------------------------------------------+
    ```

+   `JSON_TYPE(*`json_val`*)`

    返回一个`utf8mb4`字符串，指示 JSON 值的类型。这可以是一个对象、一个数组，或者一个标量类型，如下所示：

    ```sql
    mysql> SET @j = '{"a": [10, true]}';
    mysql> SELECT JSON_TYPE(@j);
    +---------------+
    | JSON_TYPE(@j) |
    +---------------+
    | OBJECT        |
    +---------------+
    mysql> SELECT JSON_TYPE(JSON_EXTRACT(@j, '$.a'));
    +------------------------------------+
    | JSON_TYPE(JSON_EXTRACT(@j, '$.a')) |
    +------------------------------------+
    | ARRAY                              |
    +------------------------------------+
    mysql> SELECT JSON_TYPE(JSON_EXTRACT(@j, '$.a[0]'));
    +---------------------------------------+
    | JSON_TYPE(JSON_EXTRACT(@j, '$.a[0]')) |
    +---------------------------------------+
    | INTEGER                               |
    +---------------------------------------+
    mysql> SELECT JSON_TYPE(JSON_EXTRACT(@j, '$.a[1]'));
    +---------------------------------------+
    | JSON_TYPE(JSON_EXTRACT(@j, '$.a[1]')) |
    +---------------------------------------+
    | BOOLEAN                               |
    +---------------------------------------+
    ```

    `JSON_TYPE()` 如果参数为 `NULL`，则返回 `NULL`：

    ```sql
    mysql> SELECT JSON_TYPE(NULL);
    +-----------------+
    | JSON_TYPE(NULL) |
    +-----------------+
    | NULL            |
    +-----------------+
    ```

    如果参数不是有效的 JSON 值，则会出现错误：

    ```sql
    mysql> SELECT JSON_TYPE(1);
    ERROR 3146 (22032): Invalid data type for JSON data in argument 1
    to function json_type; a JSON string or JSON type is required.
    ```

    对于非`NULL`、非错误结果，以下列表描述了可能的 `JSON_TYPE()` 返回值：

    +   纯粹的 JSON 类型：

        +   `OBJECT`: JSON 对象

        +   `ARRAY`: JSON 数组

        +   `BOOLEAN`: JSON 中的 true 和 false 字面值

        +   `NULL`: JSON 中的 null 字面值

    +   数值类型：

        +   `INTEGER`: MySQL `TINYINT`, `SMALLINT`, `MEDIUMINT` 和 `INT` 和 `BIGINT` 标量

        +   `DOUBLE`: MySQL `DOUBLE` - FLOAT, DOUBLE") 和 `FLOAT` - FLOAT, DOUBLE") 标量

        +   `DECIMAL`: MySQL `DECIMAL` - DECIMAL, NUMERIC") 和 `NUMERIC` - DECIMAL, NUMERIC") 标量

    +   时间类型:

        +   `DATETIME`: MySQL `DATETIME` 和 `TIMESTAMP` 标量

        +   `DATE`: MySQL `DATE` 标量

        +   `TIME`: MySQL `TIME` 标量

    +   字符串类型:

        +   `STRING`: MySQL `utf8mb3` 字符类型标量: `CHAR`、`VARCHAR`、`TEXT`、`ENUM` 和 `SET`

    +   二进制类型:

        +   `BLOB`: MySQL 二进制类型标量，包括 `BINARY`、`VARBINARY`、`BLOB` 和 `BIT`

    +   所有其他类型:

        +   `OPAQUE`（原始位）

+   `JSON_VALID(*`val`*)`

    返回 0 或 1 表示值是否为有效 JSON。如果参数为 `NULL`，则返回 `NULL`。

    ```sql
    mysql> SELECT JSON_VALID('{"a": 1}');
    +------------------------+
    | JSON_VALID('{"a": 1}') |
    +------------------------+
    |                      1 |
    +------------------------+
    mysql> SELECT JSON_VALID('hello'), JSON_VALID('"hello"');
    +---------------------+-----------------------+
    | JSON_VALID('hello') | JSON_VALID('"hello"') |
    +---------------------+-----------------------+
    |                   0 |                     1 |
    +---------------------+-----------------------+
    ```
