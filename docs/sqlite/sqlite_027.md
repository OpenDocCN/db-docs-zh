# 1\. 概述

> 原文：[`sqlite.com/json1.html`](https://sqlite.com/json1.html)

默认情况下，SQLite 支持三十个函数和两个运算符以处理 JSON 值。还有两个表值函数，用于分解 JSON 字符串。

共有二十六个标量函数和运算符：

1.  json(*json*)

1.  jsonb(*json*)

1.  json_array(*value1*,*value2*,...)

1.  jsonb_array(*value1*,*value2*,...)

1.  json_array_length(*json*)

    json_array_length(*json*,*path*)

1.  json_error_position(*json*)

1.  json_extract(*json*,*path*,...)

1.  jsonb_extract(*json*,*path*,...)

1.  *json* -> *path*

1.  *json* ->> *path*

1.  json_insert(*json*,*path*,*value*,...)

1.  jsonb_insert(*json*,*path*,*value*,...)

1.  json_object(*label1*,*value1*,...)

1.  jsonb_object(*label1*,*value1*,...)

1.  json_patch(*json*1,json2)

1.  jsonb_patch(*json*1,json2)

1.  json_pretty(*json*)

1.  json_remove(*json*,*path*,...)

1.  jsonb_remove(*json*,*path*,...)

1.  json_replace(*json*,*path*,*value*,...)

1.  jsonb_replace(*json*,*path*,*value*,...)

1.  json_set(*json*,*path*,*value*,...)

1.  jsonb_set(*json*,*path*,*value*,...)

1.  json_type(*json*)

    json_type(*json*,*path*)

1.  json_valid(*json*)

    json_valid(*json*,flags)

1.  json_quote(*value*)

共有四个聚合 SQL 函数：

1.  json_group_array(*value*)

1.  jsonb_group_array(*value*)

1.  json_group_object(*label*,*value*)

1.  jsonb_group_object(name,*value*)

两个表值函数包括：

1.  json_each(*json*)

    json_each(*json*,*path*)

1.  json_tree(*json*)

    json_tree(*json*,*path*)

# 2\. 编译 JSON 支持

从 SQLite 版本 3.38.0（2022-02-22）起，默认情况下内置 JSON 函数。可以通过添加 -DSQLITE_OMIT_JSON 编译选项来省略它们。在版本 3.38.0 之前，JSON 函数是一个扩展，只有在包含 -DSQLITE_ENABLE_JSON1 编译选项的构建中才会包括它们。换句话说，JSON 函数从 SQLite 版本 3.37.2 及更早版本的选择性支持变为 SQLite 版本 3.38.0 及更高版本的默认支持。

# 3\. 接口概览

SQLite 将 JSON 存储为普通文本。由于向后兼容性约束，SQLite 只能存储 NULL、整数、浮点数、文本和 BLOB 值。不可能添加新的“JSON”类型。

## 3.1\. JSON 参数

对于将 JSON 作为其第一个参数接受的函数，该参数可以是 JSON 对象、数组、数字、字符串或 null。SQLite 数字值和 NULL 值分别被解释为 JSON 数字和 null。SQLite 文本值可以理解为 JSON 对象、数组或字符串。如果将不是格式良好的 JSON 对象、数组或字符串的 SQLite 文本值传递给 JSON 函数，则该函数通常会抛出错误。（此规则的例外包括 json_valid()、json_quote() 和 json_error_position()。）

这些例程理解所有 [rfc-8259 JSON 语法](https://www.rfc-editor.org/rfc/rfc8259.txt)，以及 [JSON5 扩展](https://spec.json5.org/)。这些例程生成的 JSON 文本始终严格遵循 [canonical JSON 定义](https://json.org)，不包含任何 JSON5 或其他扩展。在版本 3.42.0（2023-05-16）中增加了读取和理解 JSON5 的功能。SQLite 的早期版本只能读取规范 JSON。

## 3.2\. JSONB

从版本 3.45.0（2024-01-15）开始，SQLite 允许将其内部的 JSON 的 "解析树" 表示存储在磁盘上，作为 BLOB，以一种我们称之为 "JSONB" 的格式。通过直接在数据库中存储 SQLite 的内部二进制表示的 JSON，应用程序可以在读取和更新 JSON 值时绕过解析和渲染 JSON 的开销。内部的 JSONB 格式在磁盘空间上也比文本 JSON 少一些。

任何接受文本 JSON 作为输入的 SQL 函数参数也将接受 JSONB 格式的 BLOB。无论输入是 JSON 还是 JSONB，函数的操作都相同，只是在输入为 JSONB 时速度更快，因为无需运行 JSON 解析器。

大多数返回 JSON 文本的 SQL 函数都有对应的返回等效 JSONB 的函数。返回文本格式 JSON 的函数以 "`json_`" 开头，返回二进制 JSONB 格式的函数以 "`jsonb_`" 开头。

### 3.2.1\. JSONB 格式

JSONB 是 SQLite 使用的 JSON 的二进制表示，仅供 SQLite 内部使用。应用程序不应在 SQLite 外部使用 JSONB，也不应尝试逆向工程 JSONB 格式。

“JSONB”这个名称灵感来自于[PostgreSQL](https://postgresql.org)，但 SQLite 的 JSONB 的磁盘格式与 PostgreSQL 的格式不同。这两种格式有相同的名称，但二进制不兼容。PostgreSQL 的 JSONB 格式声称可以在对象和数组中以 O(1)的时间复杂度查找元素。SQLite 的 JSONB 格式没有这样的声明。在 SQLite 中，SQLite 的 JSONB 在大多数操作中具有 O(N)的时间复杂度，就像文本 JSON 一样。SQLite 的 JSONB 比文本 JSON 更小更快，潜在的速度可能快几倍。在磁盘上的 JSONB 格式中有空间来添加增强功能，未来的 SQLite 版本可能包括提供 JSONB 中元素 O(1)查找的选项，但目前没有这样的能力。

### 3.2.2\. 处理格式错误的 JSONB

SQLite 生成的 JSONB 将始终是格式良好的。如果按照建议的做法将 JSONB 视为不透明的 BLOB，那么您将不会遇到任何问题。但是 JSONB 只是一个 BLOB，因此一个恶意的程序员可能会设计类似于 JSONB 但技术上不正确的 BLOB。当将格式不正确的 JSONB 输入 JSON 函数时，可能会发生以下情况之一：

+   SQL 语句可能会因为“格式错误的 JSON”而中止。

+   如果 JSONB blob 的格式错误的部分不影响答案，可能会返回正确的答案。

+   可能会返回一个荒谬或不合理的答案。

SQLite 处理无效的 JSONB 的方式可能会从一个版本变化到下一个版本。系统遵循垃圾进/垃圾出的原则：如果您向 JSON 函数提供无效的 JSONB，将返回一个无效的答案。如果您对 JSONB 的有效性存有疑问，请使用 json_valid()函数来验证它。

我们确实做出这样一个承诺：格式错误的 JSONB 不会引起内存错误或类似问题，可能导致漏洞。无效的 JSONB 可能会导致不合理的答案，或者可能会导致查询中止，但不会导致崩溃。

## 3.3\. PATH 参数

对于接受 PATH 参数的函数，该路径必须格式良好，否则函数将抛出错误。格式良好的 PATH 是以一个'$'字符开头，后面跟随零个或多个实例的“.*objectlabel*”或“[*arrayindex*]”文本值。

*arrayindex*通常是非负整数*N*。在这种情况下，所选的数组元素是数组的第*N*个元素，从左边开始从零开始计数。*arrayindex*也可以是形式为“**#-***N*”，在这种情况下，所选的元素是从右边开始的第*N*个元素。数组的最后一个元素是“**#-1**”。把“#”字符看作是“数组中的元素数量”。然后表达式“#-1”评估为对应于数组中最后一个条目的整数。当追加值到现有 JSON 数组时，数组索引仅为“#”字符可能是有用的：

+   json_set('[0,1,2]','$[#]','new') → '[0,1,2,"new"]'

## 3.4\. VALUE 参数

对于接受 "*value*" 参数的函数（也显示为 "*value1*" 和 "*value2*"），这些参数通常被理解为被引用的字面字符串，并在结果中成为 JSON 字符串值。即使输入的 *value* 字符串看起来像格式良好的 JSON，它们在结果中仍然被解释为字面字符串。

但是，如果 *value* 参数直接来自另一个 JSON 函数的结果或者来自 -> 运算符（但不是 ->> 运算符），那么该参数被理解为实际的 JSON，完整的 JSON 将被插入而不是一个带引号的字符串。

例如，在以下调用 json_object() 中，*value* 参数看起来像一个格式良好的 JSON 数组。然而，因为它只是普通的 SQL 文本，它被解释为字面字符串并作为带引号的字符串添加到结果中：

+   json_object('ex','[52,3.14159]') → '{"ex":"[52,3.14159]"}'

+   json_object('ex',('[52,3.14159]'->>'$')) → '{"ex":"[52,3.14159]"}'

但是，如果外部 json_object() 调用中的 *value* 参数是另一个 JSON 函数（如 json() 或 json_array()）的结果，那么该值被理解为实际的 JSON，并作为这样插入：

+   json_object('ex',json('[52,3.14159]')) → '{"ex":[52,3.14159]}'

+   json_object('ex',json_array(52,3.14159)) → '{"ex":[52,3.14159]}'

+   json_object('ex','[52,3.14159]'->'$') → '{"ex":[52,3.14159]}'

明确一点：无论 *json* 参数的值来自何处，它们始终被解释为 JSON。但是，*value* 参数仅在该参数直接来自另一个 JSON 函数或 -> 运算符 时才被解释为 JSON。

在被解释为 JSON 字符串的 JSON 值参数中，Unicode 转义序列不被视为等同于所表示的字符或转义的控制字符。这些转义序列不被翻译或特殊处理；它们被 SQLite 的 JSON 函数视为普通文本。

## 3.5\. 兼容性

这个 JSON 库的当前实现使用递归下降解析器。为了避免使用过多的堆栈空间，任何具有超过 1000 层嵌套的 JSON 输入都被视为无效。JSON 嵌套深度的限制允许兼容的实现按照 [RFC-8259 第九部分](https://tools.ietf.org/html/rfc8259#section-9) 进行。

## 3.6\. JSON5 扩展

从版本 3.42.0（2023-05-16）开始，这些例程将读取并解释包含 [JSON5](https://spec.json5.org/) 扩展的输入 JSON 文本。然而，这些例程生成的 JSON 文本始终严格符合 [JSON 的规范定义](https://json.org)。

这里是 JSON5 扩展的简介（改编自 [JSON5 规范](https://spec.json5.org/#introduction)）：

+   对象键可以是未引用的标识符。

+   对象可能有单个尾随逗号。

+   数组可能有单个尾随逗号。

+   字符串可以用单引号括起来。

+   字符串可以通过转义换行符跨越多行。

+   字符串可以包含新的字符转义序列。

+   数字可以是十六进制。

+   数字可能具有前导或尾随小数点。

+   数字可以是"Infinity"、"-Infinity"和"NaN"。

+   数字可以以显式加号开头。

+   可以使用单行（//...）和多行（/*...*/）注释。

+   允许额外的空白字符。

要将 JSON5 中的字符串 X 转换为规范 JSON，请调用"json(X)"。"json()"函数的输出将始终是规范 JSON，无论输入中是否存在任何 JSON5 扩展。为了向后兼容，不带"flags"参数的 json_valid(X)函数仍然对非规范 JSON 的输入报告 false，即使该函数能够理解 JSON5。要确定输入字符串是否有效的 JSON5，请在 json_valid 的"flags"参数中包含 0x02 位："`json_valid(X,2)`"。

这些例程理解所有的 JSON5，并且稍微扩展。SQLite 在以下两个方面扩展了 JSON5 语法：

1.  严格的 JSON5 要求未引用的对象键必须是 ECMAScript 5.1 标识符名。但是，为了确定键是否为 ECMAScript 5.1 标识符名，需要大量的 Unicode 表和大量的代码。因此，SQLite 允许对象键包含任何大于 U+007f 且不是空白字符的 Unicode 字符。这种放宽对“标识符”的定义大大简化了实现，并允许 JSON 解析器更小、运行更快。

1.  JSON5 允许用"`Infinity`"、"`-Infinity`"或"`+Infinity`"表示浮点无穷大，这种情况下，“I”要大写，其他字符要小写。SQLite 还允许缩写"`Inf`"代替"`Infinity`"，并且允许这两个关键词以任何大小写组合出现。类似地，JSON5 允许使用"NaN"表示非数值。SQLite 将其扩展到还允许"QNaN"和"SNaN"以任何大小写形式出现。注意，SQLite 将 NaN、QNaN 和 SNaN 解释为"null"的替代拼写形式。这一扩展是因为（据说）在野外存在许多包含这些非标准表示的 JSON。

## 3.7\. 性能考虑

大多数 JSON 函数使用 JSONB 进行内部处理。因此，如果输入是文本，则它们首先将输入文本转换为 JSONB。如果输入已经是 JSONB 格式，则不需要转换，可以跳过该步骤，从而提高性能。

因此，当一个 JSON 函数的参数由另一个 JSON 函数提供时，通常更高效的方法是使用作为参数使用的"`jsonb_`"变体。

+   `... json_insert(A,'$.b',json(C)) ...`   ← 效率较低。

+   `... json_insert(A,'$.b',jsonb(C)) ...`   ← 效率更高。

聚合 JSON SQL 函数 是此规则的一个例外。这些函数都使用文本而不是 JSONB 进行处理。因此，对于聚合 JSON SQL 函数，使用 "`json_`" 函数而不是 "`jsonb_`" 函数来提供参数更为有效。

+   `... json_group_array(json(A))) ...`   ← 效率更高。

+   `... json_group_array(jsonb(A))) ...`   ← 效率较低。

## 3.8\. JSON BLOB 输入 Bug

如果一个 JSON 输入是一个不是 JSONB 类型的 BLOB，并且在转换为文本时看起来像文本 JSON，那么它将被接受为文本 JSON。这实际上是原始实现中的一个长期存在的 bug，SQLite 的开发人员对此并不知情。文档中声明，将 BLOB 输入到 JSON 函数应该引发错误。但实际上的实现中，只要 BLOB 内容在数据库的文本编码中是一个有效的 JSON 字符串，就会被接受。

当 JSON 例程在 3.45.0 版本（2024-01-15）重新实现时，意外修复了此 JSON BLOB 输入 bug。这导致依赖旧行为的应用程序出现了故障。（为了那些应用程序辩护：它们经常被诱导使用 BLOBs 作为 JSON，因为在 CLI 中可用的 readfile() SQL 函数可以从磁盘文件中读取 JSON。但 readfile() 返回一个 BLOB，并且对它们有效，那么为什么不这样做呢？）

为了向后兼容，如果没有其他解释适用，则现在记录的（先前不正确的）旧行为解释 BLOBs 为文本 JSON，并且在版本 3.45.1（2024-01-30）及其所有后续版本中官方支持。

# 4\. 函数详情

下面的部分提供了关于各种 JSON 函数和操作的详细信息：

## 4.1\. json() 函数

json(X) 函数验证其参数 X 是否是有效的 JSON 字符串或 JSONB BLOB，并返回该 JSON 字符串的缩减版本，去除所有不必要的空白。如果 X 不是格式良好的 JSON 字符串或 JSONB BLOB，则此例程会抛出错误。

如果输入是 JSON5 文本，则在返回之前将其转换为规范的 RFC-8259 文本。

如果 json(X) 的参数 X 包含具有重复标签的 JSON 对象，则未定义是否保留重复项。当前实现保留重复项。但是，此例程的未来增强可能会选择静默删除重复项。

示例：

+   json(' { "this" : "is", "a": [ "test" ] } ') → '{"this":"is","a":["test"]}'

## 4.2\. jsonb() 函数

jsonb(X) 函数返回作为参数 X 提供的 JSON 的二进制 JSONB 表示。如果 X 是不具有有效 JSON 语法的 TEXT，则会引发错误。

如果 X 是 BLOB 类型并且看起来是 JSONB，则此例程简单地返回 X 的副本。然而，只检查 JSONB 输入的最外层元素。不验证 JSONB 的深层结构。

## 4.3\. json_array() 函数

json_array() SQL 函数接受零个或多个参数，并返回一个由这些参数组成的格式良好的 JSON 数组。如果 json_array() 的任何参数是 BLOB 类型，则会抛出错误。

SQL 类型为 TEXT 的参数通常会被转换为带引号的 JSON 字符串。但是，如果参数是另一个 json1 函数的输出，则会以 JSON 格式存储。这允许嵌套调用 json_array() 和 json_object()。json() 函数也可以用来强制字符串被识别为 JSON。

示例:

+   json_array(1,2,'3',4) → '[1,2,"3",4]'

+   json_array('[1,2]') → '["[1,2]"]'

+   json_array(json_array(1,2)) → '[[1,2]]'

+   json_array(1,null,'3','[4,5]','{"six":7.7}') → '[1,null,"3","[4,5]","{\"six\":7.7}"]'

+   json_array(1,null,'3',json('[4,5]'),json('{"six":7.7}')) → '[1,null,"3",[4,5],{"six":7.7}]'

## 4.4\. jsonb_array() 函数

jsonb_array() SQL 函数的工作方式与 json_array() 函数相同，只是它返回 SQLite 的私有 JSONB 格式构造的 JSON 数组，而不是标准的 RFC 8259 文本格式。

## 4.5\. json_array_length() 函数

json_array_length(X) 函数返回 JSON 数组 X 中的元素数量，如果 X 是除数组之外的某种 JSON 值，则返回 0。json_array_length(X,P) 在 X 中定位路径 P 处的数组，并返回该数组的长度，如果路径 P 定位到 X 中非 JSON 数组的元素，则返回 0，如果路径 P 没有定位到 X 的任何元素，则返回 NULL。如果 X 不是格式良好的 JSON 或者 P 不是格式良好的路径，则会抛出错误。

示例:

+   json_array_length('[1,2,3,4]') → 4

+   json_array_length('[1,2,3,4]', '$') → 4

+   json_array_length('[1,2,3,4]', '$[2]') → 0

+   json_array_length('{"one":[1,2,3]}') → 0

+   json_array_length('{"one":[1,2,3]}', '$.one') → 3

+   json_array_length('{"one":[1,2,3]}', '$.two') → NULL

## 4.6\. json_error_position() 函数

json_error_position(X) 函数如果输入 X 是格式良好的 JSON 或 JSON5 字符串，则返回 0。如果输入 X 包含一个或多个语法错误，则该函数返回第一个语法错误的字符位置。最左边的字符位置是 1。

如果输入 X 是 BLOB，则此例程返回 0 如果 X 是格式良好的 JSONB BLOB。如果返回值为正数，则表示检测到的第一个错误的*大致*基于 1 的位置。

json_error_position() 函数是在 SQLite 版本 3.42.0 (2023-05-16) 中添加的。

## 4.7\. json_extract() 函数

json_extract(X,P1,P2,...) 函数从 X 中提取并返回一个或多个值，这些值是形式良好的 JSON。如果只提供了单个路径 P1，则结果的 SQL 数据类型是 JSON null 时为 NULL，JSON 数值时为 INTEGER 或 REAL，JSON false 时为 INTEGER 0，JSON true 时为 INTEGER 1，JSON 字符串值的去引号文本，以及 JSON 对象和数组值的文本表示。如果有多个路径参数（P1、P2 等），则此函数返回一个 SQLite 文本，其中包含各个值的形式良好的 JSON 数组。

示例：

+   json_extract('{"a":2,"c":[4,5,{"f":7}]}', '$') → '{"a":2,"c":[4,5,{"f":7}]}'

+   json_extract('{"a":2,"c":[4,5,{"f":7}]}', '$.c') → '[4,5,{"f":7}]'

+   json_extract('{"a":2,"c":[4,5,{"f":7}]}', '$.c[2]') → '{"f":7}'

+   json_extract('{"a":2,"c":[4,5,{"f":7}]}', '$.c[2].f') → 7

+   json_extract('{"a":2,"c":[4,5],"f":7}','$.c','$.a') → '[[4,5],2]'

+   json_extract('{"a":2,"c":[4,5],"f":7}','$.c[#-1]') → 5

+   json_extract('{"a":2,"c":[4,5,{"f":7}]}', '$.x') → NULL

+   json_extract('{"a":2,"c":[4,5,{"f":7}]}', '$.x', '$.a') → '[null,2]'

+   json_extract('{"a":"xyz"}', '$.a') → 'xyz'

+   json_extract('{"a":null}', '$.a') → NULL

SQLite 中的 json_extract() 函数与 MySQL 中的 json_extract() 函数之间存在微妙的不兼容性。MySQL 版本的 json_extract() 总是返回 JSON。SQLite 版本的 json_extract() 只有在有两个或更多 PATH 参数时才返回 JSON（因为结果是一个 JSON 数组），或者如果单个 PATH 参数引用了数组或对象。在 SQLite 中，如果 json_extract() 只有一个单独的 PATH 参数，并且该 PATH 引用了 JSON 的 null、字符串或数值，则 json_extract() 返回相应的 SQL NULL、TEXT、INTEGER 或 REAL 值。

MySQL json_extract() 和 SQLite json_extract() 之间的区别只有在访问 JSON 中的字符串或 NULL 值时才显著。以下表格展示了这种差异：

| Operation | SQLite Result | MySQL Result |
| --- | --- | --- |
| json_extract('{"a":null,"b":"xyz"}','$.a') | NULL | 'null' |
| json_extract('{"a":null,"b":"xyz"}','$.b') | 'xyz' | '"xyz"' |

## 4.8\. The jsonb_extract() function

jsonb_extract() 函数与 json_extract() 函数的工作方式相同，除了在 json_extract() 通常返回文本 JSON 数组对象的情况下，此函数以 JSONB 格式返回数组或对象。对于返回文本、数值、null 或布尔值的常见情况，此函数与 json_extract() 完全相同。

## 4.9\. The -> and ->> operators

自 SQLite 版本 3.38.0（2022-02-22）起，**->** 和 **->>** 操作符可用于提取 JSON 的子组件。SQLite 的 **->** 和 **->>** 实现力求与 MySQL 和 PostgreSQL 兼容。**->** 和 **->>** 操作符以 JSON 字符串或 JSONB blob 作为其左操作数，并以 PATH 表达式或对象字段标签或数组索引作为其右操作数。**->** 操作符返回所选子组件的文本 JSON 表示或 NULL（如果该子组件不存在）。**->>** 操作符返回表示所选子组件的 SQL TEXT、INTEGER、REAL 或 NULL 值，如果子组件不存在，则返回 NULL。

**->** 和 **->>** 操作符选择其左侧 JSON 的相同子组件。不同之处在于 **->** 始终返回该子组件的 JSON 表示，而 **->>** 操作符始终返回该子组件的 SQL 表示。因此，这些操作符与双参数 json_extract() 函数调用略有不同。调用 json_extract() 时，如果子组件是 JSON 数组或对象，则返回其 JSON 表示，否则返回其 SQL 表示（如果子组件是 JSON null、字符串或数值）。

当 **->** 操作符返回 JSON 时，它始终返回该 JSON 的 RFC 8565 文本表示，而不是 JSONB。如果需要 JSONB 格式中的子组件，请使用 jsonb_extract() 函数。

**->** 和 **->>** 操作符的右操作数可以是形成良好的 JSON 路径表达式。这是 MySQL 使用的形式。为了与 PostgreSQL 兼容，**->** 和 **->>** 操作符还接受右操作数作为文本对象标签或整数数组索引。如果右操作数是文本标签 X，则其解释为 JSON 路径 '$.X'。如果右操作数是整数值 N，则其解释为 JSON 路径 '$[N]'。

示例：

+   '{"a":2,"c":[4,5,{"f":7}]}' -> '$' → '{"a":2,"c":[4,5,{"f":7}]}'

+   '{"a":2,"c":[4,5,{"f":7}]}' -> '$.c' → '[4,5,{"f":7}]'

+   '{"a":2,"c":[4,5,{"f":7}]}' -> 'c' → '[4,5,{"f":7}]'

+   '{"a":2,"c":[4,5,{"f":7}]}' -> '$.c[2]' → '{"f":7}'

+   '{"a":2,"c":[4,5,{"f":7}]}' -> '$.c[2].f' → '7'

+   '{"a":2,"c":[4,5,{"f":7}]}' ->> '$.c[2].f' → 7

+   '{"a":2,"c":[4,5,{"f":7}]}' -> 'c' -> 2 ->> 'f' → 7

+   '{"a":2,"c":[4,5],"f":7}' -> '$.c[#-1]' → '5'

+   '{"a":2,"c":[4,5,{"f":7}]}' -> '$.x' → NULL

+   '[11,22,33,44]' -> 3 → '44'

+   '[11,22,33,44]' ->> 3 → 44

+   '{"a":"xyz"}' -> '$.a' → '"xyz"'

+   '{"a":"xyz"}' ->> '$.a' → 'xyz'

+   '{"a":null}' -> '$.a' → 'null'

+   '{"a":null}' ->> '$.a' → NULL

## 4.10\. json_insert()、json_replace 和 json_set() 函数

json_insert()、json_replace 和 json_set()函数的第一个参数始终是原始 JSON，后面跟随零个或多个路径和值参数对，返回通过路径/值对更新输入 JSON 形成的新 JSON 字符串。这些函数在处理创建新值和覆盖现有值时略有不同。

| Function | 如果已存在则覆盖？ | 如果不存在则创建？ |
| --- | --- | --- |
| json_insert() | 否 | 是 |
| json_replace() | 是 | 否 |
| json_set() | 是 | 是 |

json_insert()、json_replace()和 json_set()函数始终接受奇数个参数。第一个参数始终是要编辑的原始 JSON。随后的参数成对出现，每对的第一个元素是路径，第二个元素是要在该路径上插入、替换或设置的值。

编辑按从左到右的顺序依次发生。先前的编辑引起的更改可能会影响后续编辑的路径搜索。

如果路径/值对的值是 SQLite TEXT 值，则通常会作为带引号的 JSON 字符串插入，即使该字符串看起来像有效的 JSON。但是，如果值是另一个 json 函数（例如 json()或 json_array()或 json_object()的结果，或者它是->运算符的结果，则将其解释为 JSON 并插入为保留其所有子结构的 JSON。使用->>运算符的值始终被解释为 TEXT，并且即使它看起来像有效的 JSON，也被插入为 JSON 字符串。

这些例程如果第一个 JSON 参数格式不正确或任何路径参数格式不正确或任何参数是 BLOB，则会抛出错误。

要将元素追加到数组的末尾，可以使用带有数组索引"#的 json_insert()。例如：

+   json_insert('[1,2,3,4]','$[#]',99) → '[1,2,3,4,99]'

+   json_insert('[1,[2,3],4]','$[1][#]',99) → '[1,[2,3,99],4]'

其他例子：

+   json_insert('{"a":2,"c":4}', '$.a', 99) → '{"a":2,"c":4}'

+   json_insert('{"a":2,"c":4}', '$.e', 99) → '{"a":2,"c":4,"e":99}'

+   json_replace('{"a":2,"c":4}', '$.a', 99) → '{"a":99,"c":4}'

+   json_replace('{"a":2,"c":4}', '$.e', 99) → '{"a":2,"c":4}'

+   json_set('{"a":2,"c":4}', '$.a', 99) → '{"a":99,"c":4}'

+   json_set('{"a":2,"c":4}', '$.e', 99) → '{"a":2,"c":4,"e":99}'

+   json_set('{"a":2,"c":4}', '$.c', '[97,96]') → '{"a":2,"c":"[97,96]"}'

+   json_set('{"a":2,"c":4}', '$.c', json('[97,96]')) → '{"a":2,"c":[97,96]}'

+   json_set('{"a":2,"c":4}', '$.c', json_array(97,96)) → '{"a":2,"c":[97,96]}'

## 4.11\. The jsonb_insert(), jsonb_replace, and jsonb_set() functions

jsonb_insert()、jsonb_replace()和 jsonb_set()函数与它们对应的 json_insert()、json_replace()和 json_set()函数相同，唯一的区别在于"`jsonb_`"版本以二进制 JSONB 格式返回其结果。

## 4.12\. The json_object() function

json_object() SQL 函数接受零个或多个参数对，并返回由这些参数组成的格式正确的 JSON 对象。每一对参数的第一个参数是标签，第二个参数是值。如果 json_object() 的任何参数是 BLOB，则会抛出错误。

json_object() 函数当前允许重复的标签而不报错，但这可能在将来的增强版本中更改。

一个带有 SQL 类型 TEXT 的参数通常会被转换为带引号的 JSON 字符串，即使输入文本是格式良好的 JSON。但是，如果参数是来自另一个 JSON 函数或 -> 操作符 的直接结果（但不包括 ->> 操作符），则将其视为 JSON 并保留其所有的 JSON 类型信息和子结构。这允许嵌套调用 json_object() 和 json_array()。也可以使用 json() 函数将字符串强制识别为 JSON。

示例：

+   json_object('a',2,'c',4) → '{"a":2,"c":4}'

+   json_object('a',2,'c','{e:5}') → '{"a":2,"c":"{e:5}"}'

+   json_object('a',2,'c',json_object('e',5)) → '{"a":2,"c":{"e":5}}'

## 4.13\. jsonb_object() 函数

jsonb_object() 函数与 json_object() 函数的工作方式完全相同，只是生成的对象以二进制 JSONB 格式返回。

## 4.14\. json_patch() 函数

json_patch(T,P) SQL 函数运行 [RFC-7396](https://tools.ietf.org/html/rfc7396) MergePatch 算法，对输入 T 应用修补 P。返回修补后的 T 的副本。

MergePatch 可以添加、修改或删除 JSON 对象的元素，因此对于 JSON 对象，json_patch() 例程是 json_set() 和 json_remove() 的通用替代。但是，MergePatch 将 JSON 数组对象视为原子。MergePatch 无法向数组追加或修改单个数组元素。它只能插入、替换或删除整个数组作为单个单位。因此，在处理包含数组的 JSON 时，特别是包含大量子结构的数组时，json_patch() 并不那么有用。

示例：

+   json_patch('{"a":1,"b":2}','{"c":3,"d":4}') → '{"a":1,"b":2,"c":3,"d":4}'

+   json_patch('{"a":[1,2],"b":2}','{"a":9}') → '{"a":9,"b":2}'

+   json_patch('{"a":[1,2],"b":2}','{"a":null}') → '{"b":2}'

+   json_patch('{"a":1,"b":2}','{"a":9,"b":null,"c":8}') → '{"a":9,"c":8}'

+   json_patch('{"a":{"x":1,"y":2},"b":3}','{"a":{"y":9},"c":8}') → '{"a":{"x":1,"y":9},"b":3,"c":8}'

## 4.15\. jsonb_patch() 函数

jsonb_patch() 函数与 json_patch() 函数的工作方式完全相同，只是返回的修补 JSON 是二进制 JSONB 格式。

## 4.16\. json_pretty() 函数

json_pretty()函数的工作方式类似于 json()，但它添加额外的空格以使 JSON 结果更易于人类阅读。第一个参数是要漂亮打印的 JSON 或 JSONB。可选的第二个参数是用于缩进的文本字符串。如果省略第二个参数或其为 NULL，则每级缩进为四个空格。

json_pretty()函数是在 SQLite 版本 3.46.0（2024-05-23）中添加的。

## 4.17\. json_remove()函数

json_remove(X,P,...)函数以单个 JSON 值作为其第一个参数，后跟零个或多个路径参数。json_remove(X,P,...)函数返回 X 参数的副本，其中移除了所有由路径参数标识的元素。选择 X 中不存在的元素的路径会被静默忽略。

删除操作从左到右依次进行。之前的删除会影响后续参数的路径搜索。

如果调用 json_remove(X)函数时没有路径参数，则返回重新格式化的输入 X，删除多余的空白。

json_remove()函数如果第一个参数不是格式良好的 JSON，或者任何后续参数不是格式良好的路径，则会抛出错误。

Examples:

+   json_remove('[0,1,2,3,4]','$[2]') → '[0,1,3,4]'

+   json_remove('[0,1,2,3,4]','$[2]','$[0]') → '[1,3,4]'

+   json_remove('[0,1,2,3,4]','$[0]','$[2]') → '[1,2,4]'

+   json_remove('[0,1,2,3,4]','$[#-1]','$[0]') → '[1,2,3]'

+   json_remove('{"x":25,"y":42}') → '{"x":25,"y":42}'

+   json_remove('{"x":25,"y":42}','$.z') → '{"x":25,"y":42}'

+   json_remove('{"x":25,"y":42}','$.y') → '{"x":25}'

+   json_remove('{"x":25,"y":42}','$') → NULL

## 4.18\. jsonb_remove()函数

jsonb_remove()函数与 json_remove()函数的工作方式完全相同，只是编辑后的 JSON 结果以二进制 JSONB 格式返回。

## 4.19\. json_type()函数

json_type(X)函数返回 X 的最外层元素的“类型”。json_type(X,P)函数返回路径 P 选择的 X 中的元素的“类型”。json_type()返回的“类型”是以下 SQL 文本值之一：'null'、'true'、'false'、'integer'、'real'、'text'、'array'或'object'。如果 json_type(X,P)中的路径 P 选择了 X 中不存在的元素，则该函数返回 NULL。

json_type()函数如果其第一个参数不是格式良好的 JSON 或 JSONB，或者第二个参数不是格式良好的 JSON 路径，则会抛出错误。

Examples:

+   json_type('{"a":[2,3.5,true,false,null,"x"]}') → 'object'

+   json_type('{"a":[2,3.5,true,false,null,"x"]}','$') → 'object'

+   json_type('{"a":[2,3.5,true,false,null,"x"]}','$.a') → 'array'

+   json_type('{"a":[2,3.5,true,false,null,"x"]}','$.a[0]') → 'integer'

+   json_type('{"a":[2,3.5,true,false,null,"x"]}','$.a[1]') → 'real'

+   json_type('{"a":[2,3.5,true,false,null,"x"]}','$.a[2]') → 'true'

+   json_type('{"a":[2,3.5,true,false,null,"x"]}','$.a[3]') → 'false'

+   json_type('{"a":[2,3.5,true,false,null,"x"]}','$.a[4]') → 'null'

+   json_type('{"a":[2,3.5,true,false,null,"x"]}','$.a[5]') → 'text'

+   json_type('{"a":[2,3.5,true,false,null,"x"]}','$.a[6]') → NULL

## 4.20\. json_valid() 函数

json_valid(X,Y) 函数如果参数 X 是格式良好的 JSON，则返回 1，如果 X 不符合规范，则返回 0。Y 参数是一个整数位掩码，定义了何为“格式良好”。当前定义的 Y 位如下：

+   **0x01** → 输入是严格遵循 RFC-8259 规范的文本，没有任何扩展。

+   **0x02** → 输入是具有上述 JSON5 扩展的 JSON 文本。

+   **0x04** → 输入是一个看起来表面上像 JSONB 的 BLOB。

+   **0x08** → 输入是严格符合内部 JSONB 格式的 BLOB。

通过组合位，可以得出以下 Y 的有用值：

+   **1** → X 是 RFC-8259 JSON 文本

+   **2** → X 是 JSON5 文本

+   **4** → X 可能是 JSONB

+   **5** → X 是 RFC-8259 JSON 文本或 JSONB

+   **6** → X 是 JSON5 文本或 JSONB ← *这可能是您想要的值*

+   **8** → X 是严格符合 JSONB

+   **9** → X 是 RFC-8259 或严格符合 JSONB

+   **10** → X 是 JSON5 或严格符合 JSONB

Y 参数是可选的。如果省略，则默认为 1，这意味着默认行为是仅当输入 X 严格符合 RFC-8259 JSON 文本且没有任何扩展时返回 true。这使得 json_valid() 的单参数版本与 SQLite 的旧版本兼容，在支持 JSON5 和 JSONB 之前。

**0x04** → Y 参数中的 0x04 位仅检查 BLOB 的外包装，看起来是否表面上像是 JSONB。对于大多数情况来说这是足够的，且速度非常快。0x08 位则彻底检查 BLOB 的所有内部细节，这会随着输入 X 的大小线性增长而变慢。推荐大多数情况使用 0x04 位。

如果您只想知道一个值是否是其他 JSON 函数的合理输入，那么 Y 值为 6 可能是您想要使用的。

对于最新版本的 json_valid()，任何小于 1 或大于 15 的 Y 值会引发错误。然而，未来版本的 json_valid() 可能会增强以接受超出此范围的标志值，具有我们尚未考虑的新含义。

如果 json_valid() 的 X 或 Y 输入为 NULL，则函数返回 NULL。

示例：

+   json_valid('{"x":35}') → 1

+   json_valid('{x:35}') → 0

+   json_valid('{x:35}',6) → 1

+   json_valid('{"x":35') → 0

+   json_valid(NULL) → NULL

## 4.21\. json_quote() 函数

`json_quote(X)` 函数将 SQL 值 X（数字或字符串）转换为其对应的 JSON 表示。如果 X 是另一个 JSON 函数返回的 JSON 值，则此函数不起作用。

示例：

+   `json_quote(3.14159)` → 3.14159

+   json_quote('verdant') → '"verdant"'

+   json_quote('[1]') → '"[1]"'

+   json_quote(json('[1]')) → '[1]'

+   json_quote('1,') → '"[1,"'

## 4.22\. 数组和对象的聚合函数

`json_group_array(X)` 函数是一个[聚合 SQL 函数，返回聚合中所有 X 值组成的 JSON 数组。类似地，`json_group_object(NAME,VALUE)` 函数返回聚合中所有 NAME/VALUE 对组成的 JSON 对象。"`jsonb_`" 变体与它们相同，只是它们以二进制 JSONB 格式返回其结果。

## 4.23\. `json_each()` 和 `json_tree()` 表值函数

`json_each(X)` 和 `json_tree(X)` 表值函数遍历其第一个参数提供的 JSON 值，并为每个元素返回一行。`json_each(X)` 函数仅遍历顶级数组或对象的直接子元素，或者如果顶级元素本身是原始值，则仅遍历顶级元素本身。`json_tree(X)` 函数递归遍历从顶级元素开始的 JSON 子结构。

`json_each(X,P)` 和 `json_tree(X,P)` 函数的工作方式与它们的单参数版本相同，除了它们将由路径 P 标识的元素视为顶级元素。

`json_each()` 和 `json_tree()` 返回的表的模式如下：

> ```sql
> CREATE TABLE json_tree(
>     key ANY,             -- key for current element relative to its parent
>     value ANY,           -- value for the current element
>     type TEXT,           -- 'object','array','string','integer', etc.
>     atom ANY,            -- value for primitive types, null for array & object
>     id INTEGER,          -- integer ID for this element
>     parent INTEGER,      -- integer ID for the parent of this element
>     fullkey TEXT,        -- full path describing the current element
>     path TEXT,           -- path to the container of the current row
>     json JSON HIDDEN,    -- 1st input parameter: the raw JSON
>     root TEXT HIDDEN     -- 2nd input parameter: the PATH at which to start
> );
> 
> ```

"key" 列是 JSON 数组元素的整数数组索引和 JSON 对象元素的文本标签。在所有其他情况下，键列为 NULL。

"atom" 列是对应于原始元素的 SQL 值 - JSON 数组和对象以外的元素。对于 JSON 数组或对象，"atom" 列为 NULL。"value" 列对于原始 JSON 元素与 "atom" 列相同，但对于数组和对象，它采用文本 JSON 值。

"type" 列是从（'null', 'true', 'false', 'integer', 'real', 'text', 'array', 'object'）中选取的 SQL 文本值，根据当前 JSON 元素的类型而定。

"id" 列是标识完整 JSON 字符串中特定 JSON 元素的整数。"id" 整数是一个内部管理编号，其计算可能会在未来版本中更改。唯一的保证是每一行的 "id" 列都不同。

"parent" 列对于 `json_each()` 总是 NULL。对于 `json_tree()`，"parent" 列是当前元素的父元素的 "id" 整数，或者对于顶级 JSON 元素或第二个参数中根路径标识的元素为 NULL。

"fullkey" 列是唯一标识原始 JSON 字符串中当前行元素的文本路径。即使通过 "root" 参数提供了替代起始点，也会返回到真正顶级元素的完整键。

"path" 列是指向包含当前行的数组或对象容器的路径，或者在迭代从基本类型开始且仅提供单行输出的情况下，是指向当前行的路径。

### 4.23.1\. 使用 json_each() 和 json_tree() 的示例

假设表 "CREATE TABLE user(name,phone)" 将零个或多个电话号码存储为用户的 JSON 数组对象。要找到所有具有任何 704 区号电话号码的用户：

> ```sql
> SELECT DISTINCT user.name
>   FROM user, json_each(user.phone)
>  WHERE json_each.value LIKE '704-%';
> 
> ```

现在假设用户的 phone 字段包含纯文本（如果用户只有一个电话号码）和 JSON 数组（如果用户有多个电话号码）。提出同样的问题："哪些用户的电话号码在 704 区号内？"但现在 json_each() 函数只能对那些拥有两个或更多电话号码的用户调用，因为 json_each() 要求其第一个参数是格式良好的 JSON：

> ```sql
> SELECT name FROM user WHERE phone LIKE '704-%'
> UNION
> SELECT user.name
>   FROM user, json_each(user.phone)
>  WHERE json_valid(user.phone)
>    AND json_each.value LIKE '704-%';
> 
> ```

考虑一个不同的数据库，有 "CREATE TABLE big(json JSON)"。要查看数据的完整逐行分解：

> ```sql
> SELECT big.rowid, fullkey, value
>   FROM big, json_tree(big.json)
>  WHERE json_tree.type NOT IN ('object','array');
> 
> ```

在先前的例子中，WHERE 子句中的 "type NOT IN ('object','array')" 条件抑制了容器，只允许叶子元素通过。可以通过以下方式实现相同效果：

> ```sql
> SELECT big.rowid, fullkey, atom
>   FROM big, json_tree(big.json)
>  WHERE atom IS NOT NULL;
> 
> ```

假设 BIG 表中的每个条目都是一个 JSON 对象，其中包含一个 '$.id' 字段作为唯一标识符，以及一个 '$.partlist' 字段，可以是深度嵌套的对象。您希望找到包含其 '$.partlist' 中任何位置对 uuid '6fa5181e-5721-11e5-a04e-57f3d7b32808' 的引用的每个条目的 id。

> ```sql
> SELECT DISTINCT json_extract(big.json,'$.id')
>   FROM big, json_tree(big.json, '$.partlist')
>  WHERE json_tree.key='uuid'
>    AND json_tree.value='6fa5181e-5721-11e5-a04e-57f3d7b32808';
> 
> ```
