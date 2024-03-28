# 7.1.3 服务器配置验证

> 原文：[`dev.mysql.com/doc/refman/8.0/en/server-configuration-validation.html`](https://dev.mysql.com/doc/refman/8.0/en/server-configuration-validation.html)

截至 MySQL 8.0.16，MySQL 服务器支持一个 `--validate-config` 选项，该选项允许在不以正常运行模式运行服务器的情况下检查启动配置是否存在问题：

```sql
mysqld --validate-config
```

如果未发现错误，则服务器以退出代码 0 终止。如果发现错误，则服务器会显示诊断消息并以退出代码 1 终止。例如：

```sql
$> mysqld --validate-config --no-such-option
2018-11-05T17:50:12.738919Z 0 [ERROR] [MY-000068] [Server] unknown
option '--no-such-option'.
2018-11-05T17:50:12.738962Z 0 [ERROR] [MY-010119] [Server] Aborting
```

一旦发现任何错误，服务器就会终止。要进行额外的检查，需要纠正初始问题，然后再次使用 `--validate-config` 运行服务器。

对于前面的示例，使用 `--validate-config` 导致显示错误消息时，服务器退出代码为 1。根据 `log_error_verbosity` 的值，也可能显示警告和信息消息，但不会立即终止验证或退出代码为 1。例如，此命令会产生多个警告，两者都会显示。但不会发生错误，因此退出代码为 0：

```sql
$> mysqld --validate-config --log_error_verbosity=2
         --read-only=s --transaction_read_only=s
2018-11-05T15:43:18.445863Z 0 [Warning] [MY-000076] [Server] option
'read_only': boolean value 's' was not recognized. Set to OFF.
2018-11-05T15:43:18.445882Z 0 [Warning] [MY-000076] [Server] option
'transaction-read-only': boolean value 's' was not recognized. Set to OFF.
```

此命令产生相同的警告，但也会有一个错误，因此错误消息会显示在警告和退出代码为 1 的情况下：

```sql
$> mysqld --validate-config --log_error_verbosity=2
         --no-such-option --read-only=s --transaction_read_only=s
2018-11-05T15:43:53.152886Z 0 [Warning] [MY-000076] [Server] option
'read_only': boolean value 's' was not recognized. Set to OFF.
2018-11-05T15:43:53.152913Z 0 [Warning] [MY-000076] [Server] option
'transaction-read-only': boolean value 's' was not recognized. Set to OFF.
2018-11-05T15:43:53.164889Z 0 [ERROR] [MY-000068] [Server] unknown
option '--no-such-option'.
2018-11-05T15:43:53.165053Z 0 [ERROR] [MY-010119] [Server] Aborting
```

`--validate-config` 选项的范围仅限于服务器可以在不进行正常启动过程的情况下执行的配置检查。因此，配置检查不会初始化存储引擎和其他插件、组件等，并且不会验证与这些未初始化子系统相关联的选项。

`--validate-config` 可以随时使用，但在升级后特别有用，用于检查旧服务器先前使用的任何选项是否被升级后的服务器视为已弃用或过时。例如，`tx_read_only` 系统变量在 MySQL 5.7 中已弃用，并在 8.0 中移除。假设一个 MySQL 5.7 服务器在其 `my.cnf` 文件中使用了该系统变量，然后升级到 MySQL 8.0。使用 `--validate-config` 运行升级后的服务器以检查配置会产生以下结果：

```sql
$> mysqld --validate-config
2018-11-05T10:40:02.712141Z 0 [ERROR] [MY-000067] [Server] unknown variable
'tx_read_only=ON'.
2018-11-05T10:40:02.712178Z 0 [ERROR] [MY-010119] [Server] Aborting
```

`--validate-config` 可以与 `--defaults-file` 选项一起使用，仅验证特定文件中的选项：

```sql
$> mysqld --defaults-file=./my.cnf-test --validate-config
2018-11-05T10:40:02.712141Z 0 [ERROR] [MY-000067] [Server] unknown variable
'tx_read_only=ON'.
2018-11-05T10:40:02.712178Z 0 [ERROR] [MY-010119] [Server] Aborting
```

请记住，如果指定了`--defaults-file`，它必须是命令行上的第一个选项。（以相反顺序执行上述示例会产生一个消息，指出`--defaults-file`本身是未知的。）
