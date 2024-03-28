# 14.17.7 JSON 模式验证函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/json-validation-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/json-validation-functions.html)

从 MySQL 8.0.17 开始，MySQL 支持根据[JSON Schema 规范的 Draft 4](https://json-schema.org/specification-links.html#draft-4)对 JSON 文档进行验证。可以使用本节详细介绍的两个函数之一来实现，这两个函数都接受两个参数，一个是 JSON 模式，另一个是根据模式进行验证的 JSON 文档。`JSON_SCHEMA_VALID()` 如果文档符合模式，则返回 true，否则返回 false；`JSON_SCHEMA_VALIDATION_REPORT()` 以 JSON 格式提供验证报告。

两个函数处理空值或无效输入如下：

+   如果至少一个参数为 `NULL`，则函数返回 `NULL`。

+   如果至少一个参数不是有效的 JSON，则函数会引发错误（`ER_INVALID_TYPE_FOR_JSON`）

+   如果模式不是有效的 JSON 对象，则该函数返回`ER_INVALID_JSON_TYPE`。

MySQL 支持 JSON 模式中的 `required` 属性，以强制包含必需属性（请参阅函数描述中的示例）。

MySQL 支持 JSON 模式中的 `id`、`$schema`、`description` 和 `type` 属性，但不要求其中任何一个。

MySQL 不支持 JSON 模式中的外部资源；使用 `$ref` 关键字会导致 `JSON_SCHEMA_VALID()` 失败，并显示`ER_NOT_SUPPORTED_YET`。

注意

MySQL 支持 JSON 模式中的正则表达式模式，支持但会静默忽略无效模式（请参阅 `JSON_SCHEMA_VALID()` 的描述以获取示例）。

这些函数在以下列表中有详细描述：

+   `JSON_SCHEMA_VALID(*`schema`*,*`document`*)`

    验证 JSON *`文档`* 是否符合 JSON *`模式`*。两者都是必需的。模式必须是有效的 JSON 对象；文档必须是有效的 JSON 文档。只要满足这些条件：如果文档符合模式，则函数返回 true（1）；否则返回 false（0）。

    在此示例中，我们将用户变量 `@schema` 设置为地理坐标的 JSON 模式的值，另一个变量 `@document` 设置为包含一个这样的坐标的 JSON 文档的值。然后，我们通过将它们用作 `JSON_SCHEMA_VALID()` 的参数来验证 `@document` 是否符合 `@schema`：

    ```sql
    mysql> SET @schema = '{
        '>  "id": "http://json-schema.org/geo",
        '> "$schema": "http://json-schema.org/draft-04/schema#",
        '> "description": "A geographical coordinate",
        '> "type": "object",
        '> "properties": {
        '>   "latitude": {
        '>     "type": "number",
        '>     "minimum": -90,
        '>     "maximum": 90
        '>   },
        '>   "longitude": {
        '>     "type": "number",
        '>     "minimum": -180,
        '>     "maximum": 180
        '>   }
        '> },
        '> "required": ["latitude", "longitude"]
        '>}';
    Query OK, 0 rows affected (0.01 sec)

    mysql> SET @document = '{
        '> "latitude": 63.444697,
        '> "longitude": 10.445118
        '>}';
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT JSON_SCHEMA_VALID(@schema, @document);
    +---------------------------------------+
    | JSON_SCHEMA_VALID(@schema, @document) |
    +---------------------------------------+
    |                                     1 |
    +---------------------------------------+
    1 row in set (0.00 sec)
    ```

    由于`@schema`包含`required`属性，我们可以将`@document`设置为一个在其他方面有效但不包含所需属性的值，然后对其进行与`@schema`的测试，如下所示：

    ```sql
    mysql> SET @document = '{}';
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT JSON_SCHEMA_VALID(@schema, @document);
    +---------------------------------------+
    | JSON_SCHEMA_VALID(@schema, @document) |
    +---------------------------------------+
    |                                     0 |
    +---------------------------------------+
    1 row in set (0.00 sec)
    ```

    如果现在将`@schema`的值设置为相同的 JSON 模式，但不包含`required`属性，`@document`会验证通过，因为它是有效的 JSON 对象，即使不包含任何属性，如下所示：

    ```sql
    mysql> SET @schema = '{
        '> "id": "http://json-schema.org/geo",
        '> "$schema": "http://json-schema.org/draft-04/schema#",
        '> "description": "A geographical coordinate",
        '> "type": "object",
        '> "properties": {
        '>   "latitude": {
        '>     "type": "number",
        '>     "minimum": -90,
        '>     "maximum": 90
        '>   },
        '>   "longitude": {
        '>     "type": "number",
        '>     "minimum": -180,
        '>     "maximum": 180
        '>   }
        '> }
        '>}';
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT JSON_SCHEMA_VALID(@schema, @document);
    +---------------------------------------+
    | JSON_SCHEMA_VALID(@schema, @document) |
    +---------------------------------------+
    |                                     1 |
    +---------------------------------------+
    1 row in set (0.00 sec)
    ```

    **JSON_SCHEMA_VALID() 和 CHECK 约束。** `JSON_SCHEMA_VALID()` 也可用于强制执行`CHECK`约束。

    考虑在此处创建的表`geo`，其中包含表示地图上纬度和经度点的 JSON 列`coordinate`，由作为`JSON_SCHEMA_VALID()`调用参数使用的 JSON 模式控制，该模式作为此表上的`CHECK`约束的表达式传递：

    ```sql
    mysql> CREATE TABLE geo (
     ->     coordinate JSON,
     ->     CHECK(
     ->         JSON_SCHEMA_VALID(
     ->             '{
        '>                 "type":"object",
        '>                 "properties":{
        '>                       "latitude":{"type":"number", "minimum":-90, "maximum":90},
        '>                       "longitude":{"type":"number", "minimum":-180, "maximum":180}
        '>                 },
        '>                 "required": ["latitude", "longitude"]
        '>             }',
     ->             coordinate
     ->         )
     ->     )
     -> );
    Query OK, 0 rows affected (0.45 sec)
    ```

    注意

    因为 MySQL 的`CHECK`约束不能包含对变量的引用，所以在为表指定此类约束时，必须在使用`JSON_SCHEMA_VALID()`时内联传递 JSON 模式。

    我们将表示坐标的 JSON 值分配给三个变量，如下所示：

    ```sql
    mysql> SET @point1 = '{"latitude":59, "longitude":18}';
    Query OK, 0 rows affected (0.00 sec)

    mysql> SET @point2 = '{"latitude":91, "longitude":0}';
    Query OK, 0 rows affected (0.00 sec)

    mysql> SET @point3 = '{"longitude":120}';
    Query OK, 0 rows affected (0.00 sec)
    ```

    这些值中的第一个是有效的，如下面的`INSERT`语句中所示：

    ```sql
    mysql> INSERT INTO geo VALUES(@point1);
    Query OK, 1 row affected (0.05 sec)
    ```

    第二个 JSON 值是无效的，因此未通过约束，如下所示：

    ```sql
    mysql> INSERT INTO geo VALUES(@point2);
    ERROR 3819 (HY000): Check constraint 'geo_chk_1' is violated.
    ```

    在 MySQL 8.0.19 及更高版本中，您可以通过发出`SHOW WARNINGS`语句来获取有关失败性质的精确信息，例如，`latitude`值超过模式中定义的最大值：

    ```sql
    mysql> SHOW WARNINGS\G
    *************************** 1\. row ***************************
      Level: Error
       Code: 3934
    Message: The JSON document location '#/latitude' failed requirement 'maximum' at
    JSON Schema location '#/properties/latitude'.
    *************************** 2\. row ***************************
      Level: Error
       Code: 3819
    Message: Check constraint 'geo_chk_1' is violated. 2 rows in set (0.00 sec)
    ```

    上述定义的第三个坐标值也是无效的，因为缺少必需的`latitude`属性。与之前一样，您可以通过尝试将该值插入`geo`表中，然后在之后发出`SHOW WARNINGS`来看到这一点：

    ```sql
    mysql> INSERT INTO geo VALUES(@point3);
    ERROR 3819 (HY000): Check constraint 'geo_chk_1' is violated.
    mysql> SHOW WARNINGS\G
    *************************** 1\. row ***************************
      Level: Error
       Code: 3934
    Message: The JSON document location '#' failed requirement 'required' at JSON
    Schema location '#'.
    *************************** 2\. row ***************************
      Level: Error
       Code: 3819
    Message: Check constraint 'geo_chk_1' is violated. 2 rows in set (0.00 sec)
    ```

    有关更多信息，请参见 Section 15.1.20.6, “CHECK Constraints”。

    JSON Schema 支持为字符串指定正则表达式模式，但 MySQL 使用的实现会默默忽略无效模式。这意味着即使正则表达式模式无效，`JSON_SCHEMA_VALID()`也可能返回 true，如下所示：

    ```sql
    mysql> SELECT JSON_SCHEMA_VALID('{"type":"string","pattern":"("}', '"abc"');
    +---------------------------------------------------------------+
    | JSON_SCHEMA_VALID('{"type":"string","pattern":"("}', '"abc"') |
    +---------------------------------------------------------------+
    |                                                             1 |
    +---------------------------------------------------------------+
    1 row in set (0.04 sec)
    ```

+   `JSON_SCHEMA_VALIDATION_REPORT(*`schema`*,*`document`*)`

    针对 JSON *`document`* 对 JSON *`schema`* 进行验证。两者都是必需的。与 JSON_VALID_SCHEMA() 一样，模式必须是有效的 JSON 对象，文档必须是有效的 JSON 文档。只要满足这些条件，函数就会返回一个报告，作为 JSON 文档，关于验证结果的情况。如果根据 JSON Schema 认为 JSON 文档有效，函数将返回一个具有一个属性`valid`且值为"true"的 JSON 对象。如果 JSON 文档未通过验证，函数将返回一个包含以下属性的 JSON 对象：

    +   `valid`：对于失败的模式验证始终为"false"

    +   `reason`：包含失败原因的人类可读字符串

    +   `schema-location`：指示验证失败的 JSON 模式中的位置的 JSON 指针 URI 片段标识符（请参见此列表后面的注释）

    +   `document-location`：指示验证失败的 JSON 文档中的位置的 JSON 指针 URI 片段标识符（请参见此列表后面的注释）

    +   `schema-failed-keyword`：包含在违反 JSON 模式的关键字或属性的字符串

    注

    JSON 指针 URI 片段标识符在[RFC 6901 - JavaScript Object Notation (JSON) Pointer](https://tools.ietf.org/html/rfc6901#page-5)中定义。（这些与`JSON_EXTRACT()`和其他 MySQL JSON 函数使用的 JSON 路径表示法不同。）在这种表示法中，`#`代表整个文档，`#/myprop`代表包含在名为`myprop`的顶级属性中的文档部分。有关更多信息，请参阅刚才引用的规范以及本节后面显示的示例。

    在此示例中，我们将一个用户变量`@schema`设置为地理坐标的 JSON 模式的值，另一个变量`@document`设置为包含一个这样的坐标的 JSON 文档的值。然后，通过将它们用作`JSON_SCHEMA_VALIDATION_REORT()`的参数来验证`@document`是否符合`@schema`：

    ```sql
    mysql> SET @schema = '{
        '>  "id": "http://json-schema.org/geo",
        '> "$schema": "http://json-schema.org/draft-04/schema#",
        '> "description": "A geographical coordinate",
        '> "type": "object",
        '> "properties": {
        '>   "latitude": {
        '>     "type": "number",
        '>     "minimum": -90,
        '>     "maximum": 90
        '>   },
        '>   "longitude": {
        '>     "type": "number",
        '>     "minimum": -180,
        '>     "maximum": 180
        '>   }
        '> },
        '> "required": ["latitude", "longitude"]
        '>}';
    Query OK, 0 rows affected (0.01 sec)

    mysql> SET @document = '{
        '> "latitude": 63.444697,
        '> "longitude": 10.445118
        '>}';
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT JSON_SCHEMA_VALIDATION_REPORT(@schema, @document);
    +---------------------------------------------------+
    | JSON_SCHEMA_VALIDATION_REPORT(@schema, @document) |
    +---------------------------------------------------+
    | {"valid": true}                                   |
    +---------------------------------------------------+
    1 row in set (0.00 sec)
    ```

    现在我们设置`@document`，使其指定其中一个属性的非法值，如下所示：

    ```sql
    mysql> SET @document = '{
        '> "latitude": 63.444697,
        '> "longitude": 310.445118
        '> }';
    ```

    现在，当使用`JSON_SCHEMA_VALIDATION_REPORT()`测试`@document`时，验证失败。函数调用的输出包含有关失败的详细信息（使用`JSON_PRETTY()`包装函数以提供更好的格式化），如下所示：

    ```sql
    mysql> SELECT JSON_PRETTY(JSON_SCHEMA_VALIDATION_REPORT(@schema, @document))\G
    *************************** 1\. row ***************************
    JSON_PRETTY(JSON_SCHEMA_VALIDATION_REPORT(@schema, @document)): {
      "valid": false,
      "reason": "The JSON document location '#/longitude' failed requirement 'maximum' at JSON Schema location '#/properties/longitude'",
      "schema-location": "#/properties/longitude",
      "document-location": "#/longitude",
      "schema-failed-keyword": "maximum"
    } 1 row in set (0.00 sec)
    ```

    由于`@schema`包含`required`属性，我们可以将`@document`设置为否则有效但不包含所需属性的值，然后对其进行与`@schema`的测试。`JSON_SCHEMA_VALIDATION_REPORT()`的输出显示验证失败，因为缺少必需元素，如下所示：

    ```sql
    mysql> SET @document = '{}';
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT JSON_PRETTY(JSON_SCHEMA_VALIDATION_REPORT(@schema, @document))\G
    *************************** 1\. row ***************************
    JSON_PRETTY(JSON_SCHEMA_VALIDATION_REPORT(@schema, @document)): {
      "valid": false,
      "reason": "The JSON document location '#' failed requirement 'required' at JSON Schema location '#'",
      "schema-location": "#",
      "document-location": "#",
      "schema-failed-keyword": "required"
    } 1 row in set (0.00 sec)
    ```

    如果我们现在将`@schema`的值设置为相同的 JSON 模式，但没有`required`属性，`@document`会通过验证，因为它是一个有效的 JSON 对象，即使它不包含任何属性，如下所示：

    ```sql
    mysql> SET @schema = '{
        '> "id": "http://json-schema.org/geo",
        '> "$schema": "http://json-schema.org/draft-04/schema#",
        '> "description": "A geographical coordinate",
        '> "type": "object",
        '> "properties": {
        '>   "latitude": {
        '>     "type": "number",
        '>     "minimum": -90,
        '>     "maximum": 90
        '>   },
        '>   "longitude": {
        '>     "type": "number",
        '>     "minimum": -180,
        '>     "maximum": 180
        '>   }
        '> }
        '>}';
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT JSON_SCHEMA_VALIDATION_REPORT(@schema, @document);
    +---------------------------------------------------+
    | JSON_SCHEMA_VALIDATION_REPORT(@schema, @document) |
    +---------------------------------------------------+
    | {"valid": true}                                   |
    +---------------------------------------------------+
    1 row in set (0.00 sec)
    ```
