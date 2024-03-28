# 14.17 JSON 函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/json-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/json-functions.html)

14.17.1 JSON 函数参考

14.17.2 创建 JSON 值的函数

14.17.3 搜索 JSON 值的函数

14.17.4 修改 JSON 值的函数

14.17.5 返回 JSON 值属性的函数

14.17.6 JSON 表函数

14.17.7 JSON 模式验证函数

14.17.8 JSON 实用函数

本节描述的函数对 JSON 值执行操作。有关`JSON`数据类型的讨论以及显示如何使用这些函数的其他示例，请参见第 13.5 节，“JSON 数据类型”。

对于接受 JSON 参数的函数，如果参数不是有效的 JSON 值，则会发生错误。以 JSON 解析的参数由*`json_doc`*表示；由*`val`*表示的参数未解析。

返回 JSON 值的函数始终对这些值进行规范化（参见 JSON 值的规范化、合并和自动包装。
