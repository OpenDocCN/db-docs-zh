# 14.17.2 创建 JSON 值的函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/json-creation-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/json-creation-functions.html)

此部分列出的函数从组件元素组成 JSON 值。

+   [`JSON_ARRAY([*`值`*[, *`值`*] ...])`](json-creation-functions.html#function_json-array)

    评估（可能为空）的值列表，并返回包含这些值的 JSON 数组。

    ```sql
    mysql> SELECT JSON_ARRAY(1, "abc", NULL, TRUE, CURTIME());
    +---------------------------------------------+
    | JSON_ARRAY(1, "abc", NULL, TRUE, CURTIME()) |
    +---------------------------------------------+
    | [1, "abc", null, true, "11:30:24.000000"]   |
    +---------------------------------------------+
    ```

+   [`JSON_OBJECT([*`键`*, *`值`*[, *`键`*, *`值`*] ...])`](json-creation-functions.html#function_json-object)

    评估（可能为空）的键值对列表，并返回包含这些对的 JSON 对象。如果任何键名为`NULL`或参数数量为奇数，则会发生错误。

    ```sql
    mysql> SELECT JSON_OBJECT('id', 87, 'name', 'carrot');
    +-----------------------------------------+
    | JSON_OBJECT('id', 87, 'name', 'carrot') |
    +-----------------------------------------+
    | {"id": 87, "name": "carrot"}            |
    +-----------------------------------------+
    ```

+   `JSON_QUOTE(*`字符串`*)`

    通过用双引号字符包装字符串并转义内部引号和其他字符，将字符串引用为 JSON 值，然后将结果作为`utf8mb4`字符串返回。如果参数为`NULL`，则返回`NULL`。

    此函数通常用于生成 JSON 文档中的有效 JSON 字符串文字。

    某些特殊字符使用反斜杠进行转义，具体转义序列请参见表 14.23，“JSON_UNQUOTE()特殊字符转义序列”特殊字符转义序列")。

    ```sql
    mysql> SELECT JSON_QUOTE('null'), JSON_QUOTE('"null"');
    +--------------------+----------------------+
    | JSON_QUOTE('null') | JSON_QUOTE('"null"') |
    +--------------------+----------------------+
    | "null"             | "\"null\""           |
    +--------------------+----------------------+
    mysql> SELECT JSON_QUOTE('[1, 2, 3]');
    +-------------------------+
    | JSON_QUOTE('[1, 2, 3]') |
    +-------------------------+
    | "[1, 2, 3]"             |
    +-------------------------+
    ```

你也可以通过将其他类型的值转换为`JSON`类型并使用`CAST(*`值`* AS JSON)`来获取 JSON 值；更多信息请参见在 JSON 和非 JSON 值之间转换。

有两个生成 JSON 值的聚合函数可用。`JSON_ARRAYAGG()`将结果集作为单个 JSON 数组返回，而`JSON_OBJECTAGG()`将结果集作为单个 JSON 对象返回。更多信息，请参见第 14.19 节，“聚合函数”。
