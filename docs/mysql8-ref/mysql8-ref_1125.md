> 原文：[`dev.mysql.com/doc/refman/8.0/en/reset.html`](https://dev.mysql.com/doc/refman/8.0/en/reset.html)

#### 15.7.8.6 **RESET** 语句

```sql
RESET *reset_option* [, *reset_option*] ...

*reset_option*: {
    MASTER
  | REPLICA
  | SLAVE
}
```

`RESET` 语句用于清除各种服务器操作的状态。您必须具有 `RELOAD` 权限才能执行 `RESET`。

有关删除持久化全局系统变量的 `RESET PERSIST` 语句的信息，请参见 第 15.7.8.7 节，“RESET PERSIST Statement”。

`RESET` 作为 `FLUSH` 语句的更强版本。参见 第 15.7.8.3 节，“FLUSH Statement”。

`RESET` 语句会导致隐式提交。参见 第 15.3.3 节，“导致隐式提交的语句”。

以下列表描述了允许的 `RESET` 语句 *`reset_option`* 值：

+   `RESET MASTER`

    删除索引文件中列出的所有二进制日志，将二进制日志索引文件重置为空，并创建一个新的二进制日志文件。

+   `RESET REPLICA`

    使复制忘记其在源二进制日志中的复制位置。还通过删除任何现有的中继日志文件并开始一个新的中继日志来重置中继日志。从 MySQL 8.0.22 开始，请使用 `RESET REPLICA` 替代 `RESET SLAVE`。
