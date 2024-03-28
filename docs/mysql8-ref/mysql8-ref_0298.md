> 原文：[`dev.mysql.com/doc/refman/8.0/en/rewriter-query-rewrite-plugin-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/rewriter-query-rewrite-plugin-reference.html)

#### 7.6.4.3 `Rewriter`查询重写插件参考

以下讨论作为与`Rewriter`查询重写插件相关的这些元素的参考：

+   `query_rewrite`数据库中的`Rewriter`规则表

+   `Rewriter`过程和函数

+   `Rewriter`系统和状态变量

##### 7.6.4.3.1 `Rewriter`查询重写插件规则表

`query_rewrite`数据库中的`rewrite_rules`表为`Rewriter`插件提供了规则的持久存储，用于决定是否重写语句。

用户通过修改存储在此表中的规则集与插件进行通信。插件通过设置表的`message`列向用户传递信息。

注意

通过`flush_rewrite_rules`存储过程将规则表加载到插件中。除非在最近的表修改后调用了该过程，否则表内容不一定对应插件正在使用的规则集。

`rewrite_rules`表具有以下列：

+   `id`

    规则 ID。此列是表的主键。您可以使用 ID 唯一标识任何规则。

+   `pattern`

    指示规则匹配语句模式的模板。使用`?`表示匹配数据值的参数标记。

+   `pattern_database`

    用于匹配语句中未限定表名的数据库。语句中的限定表名仅在默认数据库与`pattern_database`相同且表名相同时，才与模式中的限定名称匹配。语句中的未限定表名仅在默认数据库与`pattern_database`相同且表名相同时，才与模式中的未限定名称匹配。

+   `replacement`

    指示如何重写与`pattern`列值匹配的语句的模板。使用`?`表示匹配数据值的参数标记。在重写的语句中，插件使用`replacement`中的`?`参数标记，使用与`pattern`中相应标记匹配的数据值进行替换。

+   `enabled`

    是否启用规则。加载操作（通过调用`flush_rewrite_rules()`存储过程执行）仅在此列为`YES`时，将规则从表加载到`Rewriter`内存缓存中。

    此列使得可以在不移除规则的情况下停用规则：将列设置为非`YES`的值，并重新加载表到插件中。

+   `message`

    插件使用此列与用户进行通信。如果在将规则表加载到内存时没有发生错误，则插件将`message`列设置为`NULL`。非`NULL`值表示错误，列内容为错误消息。错误可能发生在以下情况下：

    +   要么模式，要么替换是产生语法错误的不正确 SQL 语句。

    +   替换包含比模式更多的`?`参数标记。

    如果发生加载错误，插件还会将`Rewriter_reload_error`状态变量设置为`ON`。

+   `pattern_digest`

    此列用于调试和诊断。如果在将规则表加载到内存时存在该列，则插件将使用模式摘要更新它。如果您试图确定某个语句未能重写的原因，此列可能会有用。

+   `normalized_pattern`

    此列用于调试和诊断。如果在将规则表加载到内存时存在该列，则插件将使用模式的规范形式更新它。如果您试图确定某个语句未能重写的原因，此列可能会有用。

##### 7.6.4.3.2 Rewriter 查询重写插件的过程和函数

`Rewriter`插件操作使用一个存储过程，将规则表加载到其内存缓存中，并使用一个辅助可加载函数。在正常操作下，用户只调用存储过程。该函数旨在由存储过程调用，而不是直接由用户调用。

+   `flush_rewrite_rules()`

    此存储过程使用`load_rewrite_rules()`函数将`rewrite_rules`表的内容加载到`Rewriter`内存缓存中。

    调用`flush_rewrite_rules()`意味着`COMMIT`。

    在修改规则表后调用此存储过程，使插件从新表内容更新其缓存。如果发生任何错误，插件将为表中适当规则行设置`message`列，并将`Rewriter_reload_error`状态变量设置为`ON`。

+   `load_rewrite_rules()`

    此函数是由`flush_rewrite_rules()`存储过程使用的辅助程序。

##### 7.6.4.3.3 Rewriter 查询重写插件系统变量

`Rewriter`查询重写插件支持以下系统变量。仅当安装了插件时才可用这些变量（请参阅第 7.6.4.1 节，“安装或卸载 Rewriter 查询重写插件”）。

+   `rewriter_enabled`

    | 系统变量 | `rewriter_enabled` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |
    | 有效值 | `OFF` |

    是否启用了`Rewriter`查询重写插件。

+   `rewriter_enabled_for_threads_without_privilege_checks`

    | 引入版本 | 8.0.31 |
    | --- | --- |
    | 系统变量 | `rewriter_enabled_for_threads_without_privilege_checks` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |
    | 有效值 | `OFF` |

    是否应用于执行时禁用权限检查的复制线程的重写。如果设置为`OFF`，则跳过此类重写。需要`SYSTEM_VARIABLES_ADMIN`权限或`SUPER`权限来设置。

    如果`rewriter_enabled`为`OFF`，则此变量无效。

+   `rewriter_verbose`

    | 系统变量 | `rewriter_verbose` |
    | --- | --- |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |

    供内部使用。

##### 7.6.4.3.4 Rewriter 查询重写插件状态变量

`Rewriter`查询重写插件支持以下状态变量。仅当插件已安装时才可用（请参阅第 7.6.4.1 节，“安装或卸载 Rewriter 查询重写插件”）。

+   `Rewriter_number_loaded_rules`

    成功从`rewrite_rules`表加载到内存中供`Rewriter`插件使用的重写插件重写规则数量。

+   `Rewriter_number_reloads`

    `rewrite_rules`表被加载到`Rewriter`插件使用的内存缓存中的次数。

+   `Rewriter_number_rewritten_queries`

    `Rewriter`查询重写插件自加载以来重写的查询次数。

+   `Rewriter_reload_error`

    上次将`rewrite_rules`表加载到`Rewriter`插件使用的内存缓存中时是否发生错误。如果值为`OFF`，则未发生错误。如果值为`ON`，则发生了错误；请检查`rewriter_rules`表的`message`列以获取错误消息。
