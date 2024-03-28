> 原文：[`dev.mysql.com/doc/refman/8.0/en/validate-password-options-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/validate-password-options-variables.html)

#### 8.4.3.2 密码验证选项和变量

本节描述了`validate_password`提供的系统和状态变量，以便配置和监视其操作。

+   密码验证组件系统变量

+   密码验证组件状态变量

+   密码验证插件选项

+   密码验证插件系统变量

+   密码验证插件状态变量

##### 密码验证组件系统变量

如果启用了`validate_password`组件，则会公开几个系统变量，以便配置密码检查：

```sql
mysql> SHOW VARIABLES LIKE 'validate_password.%';
+-------------------------------------------------+--------+
| Variable_name                                   | Value  |
+-------------------------------------------------+--------+
| validate_password.changed_characters_percentage | 0      |
| validate_password.check_user_name               | ON     |
| validate_password.dictionary_file               |        |
| validate_password.length                        | 8      |
| validate_password.mixed_case_count              | 1      |
| validate_password.number_count                  | 1      |
| validate_password.policy                        | MEDIUM |
| validate_password.special_char_count            | 1      |
+-------------------------------------------------+--------+
```

要更改密码检查的方式，您可以在服务器启动时或运行时设置这些系统变量。以下列表描述了每个变量的含义。

+   `validate_password.changed_characters_percentage`

    | 命令行格式 | `--validate-password.changed-characters-percentage[=value]` |
    | --- | --- |
    | 引入版本 | 8.0.34 |
    | 系统变量 | `validate_password.changed_characters_percentage` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `100` |

    表示用户在更改现有密码时必须更改的密码中的字符数的最小数量，作为所有字符的百分比，以便`validate_password`接受用户自己帐户的新密码。这仅适用于更改现有密码时，在设置用户帐户的初始密码时没有影响。

    除非安装了`validate_password`，否则此变量不可用。

    默认情况下，`validate_password.changed_characters_percentage` 允许在新密码中重复使用当前密码中的所有字符。有效百分比范围为 0 到 100。如果设置为 100%，则拒绝使用当前密码中的所有字符，不考虑大小写。字符 '`abc`' 和 '`ABC`' 被视为相同字符。如果 `validate_password` 拒绝新密码，则会报告一个错误，指示必须有多少个字符不同。

    如果 `ALTER USER` 语句未在 `REPLACE` 子句中提供现有密码，则不会强制执行此变量。是否需要 `REPLACE` 子句取决于密码验证策略适用于特定帐户。有关策略概述，请参阅 密码验证-必需策略。

+   `validate_password.check_user_name`

    | 命令行格式 | `--validate-password.check-user-name[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `validate_password.check_user_name` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    `validate_password` 是否将密码与当前会话的有效用户帐户的用户名部分进行比较，并在匹配时拒绝密码。此变量仅在安装了 `validate_password` 时才可用。

    默认情况下，`validate_password.check_user_name` 已启用。该变量控制用户名称匹配，与 `validate_password.policy` 的值无关。

    当启用 `validate_password.check_user_name` 时，会产生以下影响：

    +   检查发生在调用 `validate_password` 的所有上下文中，包括使用诸如 `ALTER USER` 或 `SET PASSWORD` 来更改当前用户密码的语句，以及调用诸如 `VALIDATE_PASSWORD_STRENGTH()` 的函数。

    +   用于比较的用户名取自当前会话的 `USER()` 和 `CURRENT_USER()` 函数的值。一个含义是，具有足够权限设置另一个用户密码的用户可以将密码设置为该用户的名称，并且不能将该用户的密码设置为执行语句的用户的名称。例如，`'root'@'localhost'` 可以将 `'jeffrey'@'localhost'` 的密码设置为 `'jeffrey'`，但不能将密码设置为 `'root`。

    +   仅使用 `USER()` 和 `CURRENT_USER()` 函数值的用户名部分，而不使用主机名部分。如果用户名为空，则不进行比较。

    +   如果密码与用户名相同或其反转，则会发生匹配，密码将被拒绝。

    +   用户名匹配区分大小写。密码和用户名值按字节逐字节比较为二进制字符串。

    +   如果密码与用户名匹配，`VALIDATE_PASSWORD_STRENGTH()` 返回 0，无论如何设置其他 `validate_password` 系统变量。

+   `validate_password.dictionary_file`

    | 命令行格式 | `--validate-password.dictionary-file=file_name` |
    | --- | --- |
    | 系统变量 | `validate_password.dictionary_file` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |

    字典文件的路径名，`validate_password` 用于检查密码。除非安装了 `validate_password`，否则此变量不可用。

    默认情况下，此变量的值为空，不执行字典检查。要执行字典检查，变量值必须非空。如果文件命名为相对路径，则解释为相对于服务器数据目录。文件内容应为小写，每行一个单词。内容被视为具有字符集 `utf8mb3`。允许的最大文件大小为 1MB。

    为了在检查密码期间使用字典文件，密码策略必须设置为 2 (`STRONG`)；请参阅 `validate_password.policy` 系统变量的描述。假设是真的，密码的每个长度为 4 到 100 的子字符串将与字典文件中的单词进行比较。任何匹配都会导致密码被拒绝。比较不区分大小写。

    对于`VALIDATE_PASSWORD_STRENGTH()`，密码将根据所有策略进行检查，包括`STRONG`，因此强度评估将包括字典检查，无论`validate_password.policy`的值如何。

    `validate_password.dictionary_file` 可以在运行时设置，分配一个值会导致读取命名文件而无需重新启动服务器。

+   `validate_password.length`

    | 命令行格式 | `--validate-password.length=#` |
    | --- | --- |
    | 系统��量 | `validate_password.length` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `8` |
    | 最小值 | `0` |

    `validate_password`要求密码中包含的最小字符数。除非安装了`validate_password`，否则此变量不可用。

    `validate_password.length` 的最小值是几个其他相关系统变量的函数。该值不能设置为小于以下表达式的值：

    ```sql
    validate_password.number_count
    + validate_password.special_char_count
    + (2 * validate_password.mixed_case_count)
    ```

    如果`validate_password`由于前面的约束调整了`validate_password.length`的值，则会将消息写入错误日志。

+   `validate_password.mixed_case_count`

    | 命令行格式 | `--validate-password.mixed-case-count=#` |
    | --- | --- |
    | 系统变量 | `validate_password.mixed_case_count` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `0` |

    如果密码策略为`MEDIUM`或更强，`validate_password`要求密码中包含的小写和大写字符的最小数量。除非安装了`validate_password`，否则此变量不可用。

    对于给定的`validate_password.mixed_case_count`值，密码必须具有相同数量的小写字符和大写字符。

+   `validate_password.number_count`

    | 命令行格式 | `--validate-password.number-count=#` |
    | --- | --- |
    | 系统变量 | `validate_password.number_count` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `0` |

    `validate_password`要求密码具有的数字（数字）字符的最小数量，如果密码策略为`MEDIUM`或更强。除非安装了`validate_password`，否则此变量不可用。

+   `validate_password.policy`

    | 命令行格式 | `--validate-password.policy=value` |
    | --- | --- |
    | 系统变量 | `validate_password.policy` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `1` |
    | 有效值 | `0``1``2` |

    由`validate_password`强制执行的密码策略。除非安装了`validate_password`，否则此变量不可用。

    `validate_password.policy`影响`validate_password`如何使用其其他设置策略的系统变量，除了检查密码是否与用户名匹配，这是由`validate_password.check_user_name`独立控制的。

    可以使用数值 0、1、2 或相应的符号值`LOW`、`MEDIUM`、`STRONG`来指定`validate_password.policy`的值。以下表格描述了每个策略执行的测试。对于长度测试，所需长度是`validate_password.length`系统变量的值。类似地，其他测试的所需值由其他`validate_password.*`xxx`*`变量给出。

    | 策略 | 执行的测试 |
    | --- | --- |
    | `0` 或 `LOW` | 长度 |
    | `1` 或 `MEDIUM` | 长度；数字、小写/大写字母和特殊字符 |
    | `2` 或 `STRONG` | 长度；数字、小写/大写字母和特殊字符；字典文件 |

+   `validate_password.special_char_count`

    | 命令行格式 | `--validate-password.special-char-count=#` |
    | --- | --- |
    | 系统变量 | `validate_password.special_char_count` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `0` |

    如果密码策略为`MEDIUM`或更强时，`validate_password`要求密码具有的最小非字母数字字符数。除非安装了`validate_password`，否则此变量不可用。

##### 密码验证组件状态变量

如果启用了`validate_password`组件，则会公开提供操作信息的状态变量：

```sql
mysql> SHOW STATUS LIKE 'validate_password.%';
+-----------------------------------------------+---------------------+
| Variable_name                                 | Value               |
+-----------------------------------------------+---------------------+
| validate_password.dictionary_file_last_parsed | 2019-10-03 08:33:49 |
| validate_password.dictionary_file_words_count | 1902                |
+-----------------------------------------------+---------------------+
```

以下列表描述了每个状态变量的含义。

+   `validate_password.dictionary_file_last_parsed`

    上次解析字典文件的时间。除非安装了`validate_password`，否则此变量不可用。

+   `validate_password.dictionary_file_words_count`

    从字典文件中读取的单词数量。除非安装了`validate_password`，否则此变量不可用。

##### 密码验证插件选项

注意

在 MySQL 8.0 中，`validate_password`插件被重新实现为`validate_password`组件。`validate_password`插件已被弃用；预计在未来的 MySQL 版本中将其移除。因此，其选项也已被弃用，您应该预期它们也将被移除。使用插件的 MySQL 安装应该过渡到使用组件。参见第 8.4.3.3 节，“过渡到密码验证组件”。

使用此选项来控制`validate_password`插件的激活：

+   [`--validate-password[=*`value`*]`](validate-password-options-variables.html#option_mysqld_validate-password)

    | 命令行格式 | `--validate-password[=value]` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `ON` |
    | 有效数值 | `ON``OFF``FORCE``FORCE_PLUS_PERMANENT` |

    此选项控制服务器在启动时如何加载已弃用的`validate_password`插件。该值应为插件加载选项中可用的值之一，如第 7.6.1 节，“安装和卸载插件”中所述。例如，`--validate-password=FORCE_PLUS_PERMANENT`告诉服务器在启动时加载插件，并防止在服务器运行时将其移除。

    只有在之前使用`INSTALL PLUGIN`注册了`validate_password`插件或者使用`--plugin-load-add`加载了该插件时，此选项才可用。参见第 8.4.3.1 节，“密码验证组件的安装和卸载”。

##### 密码验证插件系统变量

注意

在 MySQL 8.0 中，`validate_password`插件被重新实现为`validate_password`组件。`validate_password`插件已被弃用；预计在 MySQL 的未来版本中将被移除。因此，其系统变量也已被弃用，您应该预期它们也将被移除。改用`validate_password`组件的相应系统变量；请参阅密码验证组件系统变量。使用插件的 MySQL 安装应该过渡到使用组件。请参阅第 8.4.3.3 节，“过渡到密码验证组件”。

+   `validate_password_check_user_name`

    | 命令行格式 | `--validate-password-check-user-name[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `validate_password_check_user_name` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔 |
    | 默认值 | `ON` |

    此`validate_password`插件系统变量已被弃用；预计在 MySQL 的未来版本中将被移除。请改用`validate_password`组件的相应`validate_password.check_user_name`系统变量。

+   `validate_password_dictionary_file`

    | 命令行格式 | `--validate-password-dictionary-file=file_name` |
    | --- | --- |
    | 系统变量 | `validate_password_dictionary_file` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |

    此`validate_password`插件系统变量已被弃用；预计在 MySQL 的未来版本中将被移除。请改用`validate_password`组件的相应`validate_password.dictionary_file`系统变量。

+   `validate_password_length`

    | 命令行格式 | `--validate-password-length=#` |
    | --- | --- |
    | 系统变量 | `validate_password_length` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `8` |
    | 最小值 | `0` |

    此 `validate_password` 插件系统变量已被弃用；预计在 MySQL 的未来版本中将被移除。请改用 `validate_password` 组件的相应 `validate_password.length` 系统变量。

+   `validate_password_mixed_case_count`

    | 命令行格式 | `--validate-password-mixed-case-count=#` |
    | --- | --- |
    | 系统变量 | `validate_password_mixed_case_count` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `0` |

    此 `validate_password` 插件系统变量已被弃用；预计在 MySQL 的未来版本中将被移除。请改用 `validate_password` 组件的相应 `validate_password.mixed_case_count` 系统变量。

+   `validate_password_number_count`

    | 命令行格式 | `--validate-password-number-count=#` |
    | --- | --- |
    | 系统变量 | `validate_password_number_count` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `0` |

    此 `validate_password` 插件系统变量已被弃用；预计在 MySQL 的未来版本中将被移除。请改用 `validate_password` 组件的相应 `validate_password.number_count` 系统变量。

+   `validate_password_policy`

    | 命令行格式 | `--validate-password-policy=value` |
    | --- | --- |
    | 系统变量 | `validate_password_policy` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `1` |
    | 有效值 | `0``1``2` |

    此 `validate_password` 插件系统变量已被弃用；预计在 MySQL 的未来版本中将被移除。请改用 `validate_password` 组件的相应 `validate_password.policy` 系统变量。

+   `validate_password_special_char_count`

    | 命令行格式 | `--validate-password-special-char-count=#` |
    | --- | --- |
    | 系统变量 | `validate_password_special_char_count` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `0` |

    此`validate_password`插件系统变量已被弃用；预计将在 MySQL 的未来版本中移除。请改用`validate_password`组件的相应`validate_password.special_char_count`系统变量。

##### 密码验证插件状态变量

注意

在 MySQL 8.0 中，`validate_password`插件被重新实现为`validate_password`组件。`validate_password`插件已被弃用；预计将在 MySQL 的未来版本中移除。因此，其状态变量也已被弃用；预计将被移除。请使用`validate_password`组件的相应状态变量；请参阅密码验证组件状态变量。使用插件的 MySQL 安装应该过渡到使用组件。请参阅第 8.4.3.3 节，“过渡到密码验证组件”。

+   `validate_password_dictionary_file_last_parsed`

    此`validate_password`插件状态变量已被弃用；预计将在 MySQL 的未来版本中移除。请改用`validate_password`组件的相应`validate_password.dictionary_file_last_parsed`状态变量。

+   `validate_password_dictionary_file_words_count`

    此`validate_password`插件状态变量已被弃用；预计将在 MySQL 的未来版本中移除。请改用`validate_password`组件的相应`validate_password.dictionary_file_words_count`状态变量。
