# 14.17.1 JSON 函数参考

> 原文：[`dev.mysql.com/doc/refman/8.0/en/json-function-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/json-function-reference.html)

**表格 14.22 JSON 函数**

| 名称 | 描述 | 引入版本 | 弃用版本 |
| --- | --- | --- | --- |
| `->` | 在评估路径后从 JSON 列返回值；等同于 JSON_EXTRACT()。 |  |  |
| `->>` | 在评估路径并取消引用结果后从 JSON 列返回值；等同于 JSON_UNQUOTE(JSON_EXTRACT())。 |  |  |
| `JSON_ARRAY()` | 创建 JSON 数组 |  |  |
| `JSON_ARRAY_APPEND()` | 向 JSON 文档追加数据 |  |  |
| `JSON_ARRAY_INSERT()` | 插入 JSON 数组 |  |  |
| `JSON_CONTAINS()` | 判断 JSON 文档是否包含特定路径的对象 |  |  |
| `JSON_CONTAINS_PATH()` | 判断 JSON 文档是否在路径上包含任何数据 |  |  |
| `JSON_DEPTH()` | JSON 文档的最大深度 |  |  |
| `JSON_EXTRACT()` | 从 JSON 文档返回数据 |  |  |
| `JSON_INSERT()` | 向 JSON 文档插入数据 |  |  |
| `JSON_KEYS()` | 从 JSON 文档中获取键数组 |  |  |
| `JSON_LENGTH()` | JSON 文档中的元素数量 |  |  |
| `JSON_MERGE()` | 合并 JSON 文档，保留重复的键。已弃用，JSON_MERGE_PRESERVE() 的同义词 |  | 是 |
| `JSON_MERGE_PATCH()` | 合并 JSON 文档，替换重复键的值 |  |  |
| `JSON_MERGE_PRESERVE()` | 合并 JSON 文档，保留重复的键 |  |  |
| `JSON_OBJECT()` | 创建 JSON 对象 |  |  |
| `JSON_OVERLAPS()` | 比较两个 JSON 文档，如果有任何键值对或数组元素相同则返回 TRUE (1)，否则返回 FALSE (0) | 8.0.17 |  |
| `JSON_PRETTY()` | 以人类可读的格式打印 JSON 文档 |  |  |
| `JSON_QUOTE()` | 引用 JSON 文档 |  |  |
| `JSON_REMOVE()` | 从 JSON 文档中移除数据 |  |  |
| `JSON_REPLACE()` | 替换 JSON 文档中的值 |  |  |
| `JSON_SCHEMA_VALID()` | 验证 JSON 文档是否符合 JSON 模式；如果文档符合模式，则返回 TRUE/1，否则返回 FALSE/0 | 8.0.17 |  |
| `JSON_SCHEMA_VALIDATION_REPORT()` | 针对 JSON 模式验证 JSON 文档；以 JSON 格式返回验证结果报告，包括成功或失败以及失败原因 | 8.0.17 |  |
| `JSON_SEARCH()` | JSON 文档中值的路径 |  |  |
| `JSON_SET()` | 向 JSON 文档中插入数据 |  |  |
| `JSON_STORAGE_FREE()` | 部分更新后 JSON 列值的二进制表示中释放的空间 |  |  |
| `JSON_STORAGE_SIZE()` | 存储 JSON 文档的二进制表示所使用的空间 |  |  |
| `JSON_TABLE()` | 将 JSON 表达式的数据作为关系表返回 |  |  |
| `JSON_TYPE()` | JSON 值的类型 |  |  |
| `JSON_UNQUOTE()` | 取消 JSON 值的引号 |  |  |
| `JSON_VALID()` | JSON 值是否有效 |  |  |
| `JSON_VALUE()` | 在路径指定的位置从 JSON 文档中提取值；将此值作为 VARCHAR(512) 或指定类型返回 | 8.0.21 |  |
| `MEMBER OF()` | 如果第一个操作数与作为第二个操作数传递的 JSON 数组的任何元素匹配，则返回 true (1)，否则返回 false (0) | 8.0.17 |  |
| 名称 | 描述 | 引入版本 | 废弃版本 |

MySQL 支持两个聚合 JSON 函数 `JSON_ARRAYAGG()` 和 `JSON_OBJECTAGG()`。有关这些函数的描述，请参见 第 14.19 节，“聚合函数”。

MySQL 还支持以易于阅读的格式“漂亮打印”JSON 值，使用`JSON_PRETTY()`函数。您可以通过`JSON_STORAGE_SIZE()`和`JSON_STORAGE_FREE()`函数，分别查看给定 JSON 值占用的存储空间以及剩余的存储空间。有关这些函数的完整描述，请参见第 14.17.8 节，“JSON 实用函数”。
