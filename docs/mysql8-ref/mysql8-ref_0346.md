# 7.9.3 LOCK_ORDER 工具

> 原文：[`dev.mysql.com/doc/refman/8.0/en/lock-order-tool.html`](https://dev.mysql.com/doc/refman/8.0/en/lock-order-tool.html)

MySQL 服务器是一个使用许多内部锁定和与锁相关的原语的多线程应用程序，例如互斥锁，rwlocks（包括 prlocks 和 sxlocks），条件和文件。在服务器内部，锁相关对象的集合随着实现新功能和代码重构以提高性能而改变。与使用锁定原语的任何多线程应用程序一样，当同时持有多个锁时，在执行期间遇到死锁的风险始终存在。对于 MySQL，死锁的影响是灾难性的，会导致完全丧失服务。

从 MySQL 8.0.17 开始，为了启用锁获取死锁检测和确保运行时执行不包含死锁的强制执行，MySQL 支持`LOCK_ORDER`工具。这使得可以定义锁顺序依赖图作为服务器设计的一部分，并且服务器运行时检查以确保锁获取是无环的，并且执行路径符合图形。

本节提供有关使用`LOCK_ORDER`工具的信息，但仅在基本水平上。有关完整详情，请参阅 MySQL Server Doxygen 文档中的 Lock Order 部分，网址为`dev.mysql.com/doc/index-other.html`。

`LOCK_ORDER`工具旨在用于调试服务器，而非生产使用。

要使用`LOCK_ORDER`工具，请按照以下步骤进行：

1.  从源代码构建 MySQL，并使用`-DWITH_LOCK_ORDER=ON` **CMake**选项进行配置，以便构建包含`LOCK_ORDER`工具的内容。

    注意

    启用`WITH_LOCK_ORDER`选项后，MySQL 构建需要**flex**程序。

1.  要在启用`LOCK_ORDER`工具的情况下运行服务器，请在服务器启动时启用`lock_order`系统变量。还有其他用于`LOCK_ORDER`配置的系统变量可用。

1.  对于 MySQL 测试套件操作，**mysql-test-run.pl**具有一个`--lock-order`选项，用于控制在测试用例执行期间是否启用`LOCK_ORDER`工具。

以下描述的系统变量配置`LOCK_ORDER`工具的操作，假设 MySQL 已构建包含`LOCK_ORDER`工具。主要变量是`lock_order`，指示是否在运行时启用`LOCK_ORDER`工具：

+   如果`lock_order`被禁用（默认情况下），则其他`LOCK_ORDER`系统变量不会产生任何影响。

+   如果启用了`lock_order`，则其他系统变量配置哪些`LOCK_ORDER`功能要启用。

注意

通常情况下，应通过使用 `--lock-order` 选项执行 **mysql-test-run.pl** 来配置 `LOCK_ORDER` 工具，并让 **mysql-test-run.pl** 将 `LOCK_ORDER` 系统变量设置为适当的值。

所有 `LOCK_ORDER` 系统变量必须在服务器启动时设置。在运行时，它们的值是可见的，但不能更改。

一些系统变量存在成对出现，例如 `lock_order_debug_loop` 和 `lock_order_trace_loop`。对于这样的成对变量，在发生与其关联的条件时，这些变量的区别如下：

+   如果启用了 `_debug_` 变量，则会引发调试断言。

+   如果启用了 `_trace_` 变量，则会将错误打印到日志中。

**表 7.8 LOCK_ORDER 系统变量摘要**

| 变量名称 | 变量类型 | 变量范围 |
| --- | --- | --- |
| lock_order | 布尔值 | 全局 |
| lock_order_debug_loop | 布尔值 | 全局 |
| lock_order_debug_missing_arc | 布尔值 | 全局 |
| lock_order_debug_missing_key | 布尔值 | 全局 |
| lock_order_debug_missing_unlock | 布尔值 | 全局 |
| lock_order_dependencies | 文件名 | 全局 |
| lock_order_extra_dependencies | 文件名 | 全局 |
| lock_order_output_directory | 目录名 | 全局 |
| lock_order_print_txt | 布尔值 | 全局 |
| lock_order_trace_loop | 布尔值 | 全局 |
| lock_order_trace_missing_arc | 布尔值 | 全局 |
| lock_order_trace_missing_key | 布尔值 | 全局 |
| lock_order_trace_missing_unlock | 布尔值 | 全局 |
| 变量名称 | 变量类型 | 变量范围 |

+   `lock_order`

    | 命令行格式 | `--lock-order[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.17 |
    | 系统变量 | `lock_order` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    是否在运行时启用`LOCK_ORDER`工具。如果禁用了`lock_order`，则不会影响任何其他`LOCK_ORDER`系统变量。如果启用了`lock_order`，则其他系统变量配置哪些`LOCK_ORDER`功能启用。

    如果启用了`lock_order`，则如果服务器遇到未在锁定顺序图中声明的锁定获取序列，将引发错误。

+   `lock_order_debug_loop`

    | 命令行格式 | `--lock-order-debug-loop[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.17 |
    | 系统变量 | `lock_order_debug_loop` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    当`LOCK_ORDER`工具遇到在锁定顺序图中标记为循环的依赖关系时，是否导致调试断言失败。

+   `lock_order_debug_missing_arc`

    | 命令行格式 | `--lock-order-debug-missing-arc[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.17 |
    | 系统变量 | `lock_order_debug_missing_arc` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    当`LOCK_ORDER`工具遇到在锁定顺序图中未声明的依赖关系时，是否导致调试断言失败。

+   `lock_order_debug_missing_key`

    | 命令行格式 | `--lock-order-debug-missing-key[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.17 |
    | 系统变量 | `lock_order_debug_missing_key` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    当`LOCK_ORDER`工具遇到未正确使用性能模式仪器的对象时，是否导致调试断言失败。

+   `lock_order_debug_missing_unlock`

    | 命令行格式 | `--lock-order-debug-missing-unlock[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.17 |
    | 系统变量 | `lock_order_debug_missing_unlock` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    `LOCK_ORDER`工具在遇到仍在持有的锁被销毁时是否导致调试断言失败。

+   `lock_order_dependencies`

    | 命令行格式 | `--lock-order-dependencies=file_name` |
    | --- | --- |
    | 引入版本 | 8.0.17 |
    | 系统变量 | `lock_order_dependencies` |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `空字符串` |

    `lock_order_dependencies.txt`文件的路径，定义了服务器锁定顺序依赖图。

    可以指定不使用任何依赖项。在这种情况下，将使用空依赖图。

+   `lock_order_extra_dependencies`

    | 命令行格式 | `--lock-order-extra-dependencies=file_name` |
    | --- | --- |
    | 引入版本 | 8.0.17 |
    | 系统变量 | `lock_order_extra_dependencies` |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `空字符串` |

    包含额外依赖项的文件路径，用于修改锁定顺序依赖图的主要服务器依赖图。这对于使用额外依赖项描述第三方代码行为很有用，而不是修改`lock_order_dependencies.txt`文件本身（这是不鼓励的替代方法）。

    如果未设置此变量，则不使用辅助文件。

+   `lock_order_output_directory`

    | 命令行格式 | `--lock-order-output-directory=dir_name` |
    | --- | --- |
    | ���入版本 | 8.0.17 |
    | 系统变量 | `lock_order_output_directory` |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 目录名称 |
    | 默认值 | `空字符串` |

    `LOCK_ORDER`工具写入其日志的目录。如果未设置此变量，则默认为当前目录。

+   `lock_order_print_txt`

    | 命令行格式 | `--lock-order-print-txt[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.17 |
    | 系统变量 | `lock_order_print_txt` |
    | 作用域 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    `LOCK_ORDER`工具是否执行锁定顺序图分析并打印文本报告。报告包括检测到的任何锁获取循环。

+   `lock_order_trace_loop`

    | 命令行格式 | `--lock-order-trace-loop[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.17 |
    | 系统变量 | `lock_order_trace_loop` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    `LOCK_ORDER` 工具在遇到在锁序图中标记为循环的依赖关系时是否在日志文件中打印跟踪信息。

+   `lock_order_trace_missing_arc`

    | 命令行格式 | `--lock-order-trace-missing-arc[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.17 |
    | 系统变量 | `lock_order_trace_missing_arc` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    `LOCK_ORDER` 工具在遇到在锁序图中未声明的依赖关系时是否在日志文件中打印跟踪信息。

+   `lock_order_trace_missing_key`

    | 命令行格式 | `--lock-order-trace-missing-key[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.17 |
    | 系统变量 | `lock_order_trace_missing_key` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    `LOCK_ORDER` 工具在遇到未正确使用性能模式进行仪表化的对象时是否在日志文件中打印跟踪信息。

+   `lock_order_trace_missing_unlock` 

    | 命令行格式 | `--lock-order-trace-missing-unlock[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.17 |
    | 系统变量 | `lock_order_trace_missing_unlock` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    `LOCK_ORDER` 工具在遇到一个被销毁但仍被持有的锁时是否在日志文件中打印跟踪信息。
