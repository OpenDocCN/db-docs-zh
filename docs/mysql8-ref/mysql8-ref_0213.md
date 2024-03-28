# 6.6.10 mysqldumpslow — 汇总慢查询日志文件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqldumpslow.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqldumpslow.html)

MySQL 慢查询日志包含执行时间长的查询的信息（参见 Section 7.4.5, “The Slow Query Log”）。**mysqldumpslow** 解析 MySQL 慢查询日志文件并汇总其内容。

通常，**mysqldumpslow** 对于除了数字和字符串数据值的特定值之外相似的查询进行分组。在显示汇总输出时，它将这些值“抽象化”为 `N` 和 `'S'`。要修改值抽象化行为，请使用 `-a` 和 `-n` 选项。

像这样调用 **mysqldumpslow**：

```sql
mysqldumpslow [*options*] [*log_file* ...]
```

未提供任何选项的示例输出：

```sql
Reading mysql slow query log from /usr/local/mysql/data/mysqld80-slow.log
Count: 1  Time=4.32s (4s)  Lock=0.00s (0s)  Rows=0.0 (0), root[root]@localhost
 insert into t2 select * from t1

Count: 3  Time=2.53s (7s)  Lock=0.00s (0s)  Rows=0.0 (0), root[root]@localhost
 insert into t2 select * from t1 limit N

Count: 3  Time=2.13s (6s)  Lock=0.00s (0s)  Rows=0.0 (0), root[root]@localhost
 insert into t1 select * from t1
```

**mysqldumpslow** 支持以下选项。

**表 6.24 mysqldumpslow 选项**

| 选项名称 | 描述 |
| --- | --- |
| -a | 不要将所有数字抽象为 N 和字符串抽象为 'S' |
| -n | 至少具有指定位数的数字抽象 |
| --debug | 写入调试信息 |
| -g | 仅考虑与模式匹配的语句 |
| --help | 显示帮助信息并退出 |
| -h | 日志文件名中服务器的主机名 |
| -i | 服务器实例的名称 |
| -l | 不要从总时间中减去锁定时间 |
| -r | 反转排序顺序 |
| -s | 输出排序方式 |
| -t | 仅显示前 num 个查询 |
| --verbose | 详细模式 |
| 选项名称 | 描述 |

+   `--help`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助信息并退出。

+   `-a`

    不要将所有数字抽象为 `N`，字符串抽象为 `'S'`。

+   `--debug`, `-d`

    | 命令行格式 | `--debug` |
    | --- | --- |

    以调试模式运行。

    只有在使用 `WITH_DEBUG` 构建 MySQL 时才可用此选项。由 Oracle 提供的 MySQL 发行版二进制文件 *不* 使用此选项构建。

+   `-g *`pattern`*`

    | 类型 | 字符串 |
    | --- | --- |

    仅考虑与（grep 风格）模式匹配的查询。

+   `-h *`host_name`*`

    | 类型 | 字符串 |
    | --- | --- |
    | 默认值 | `*` |

    MySQL 服务器的主机名，用于 `*-slow.log` 文件名。该值可以包含通配符。默认值为 `*`（匹配所有）。

+   `-i *`name`*`

    | 类型 | 字符串 |
    | --- | --- |

    服务器实例的名称（如果使用 **mysql.server** 启动脚本）。

+   `-l`

    不要从总时间中减去锁定时间。

+   `-n *`N`*`

    | 类型 | 数字 |
    | --- | --- |

    在名称中至少有 *`N`* 位数字的抽象数字。

+   `-r`

    反转排序顺序。

+   `-s *`sort_type`*`

    | 类型 | 字符串 |
    | --- | --- |
    | 默认值 | `at` |

    如何对输出进行排序。*`sort_type`* 的值应从以下列表中选择：

    +   `t`, `at`: 按查询时间或平均查询时间排序

    +   `l`, `al`: 按锁定时间或平均锁定时间排序

    +   `r`, `ar`: 按发送的行数或平均发送的行数排序

    +   `c`: 按计数排序

    默认情况下，**mysqldumpslow** 按平均查询时间排序（相当于 `-s at`）。

+   `-t *`N`*`

    | 类型 | 数字 |
    | --- | --- |

    仅显示输出中的前 *`N`* 个查询。

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    详细模式。打印程序执行的更多信息。
