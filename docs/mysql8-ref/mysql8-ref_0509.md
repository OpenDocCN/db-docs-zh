# 9.6.5 设置 MyISAM 表维护计划

> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisam-maintenance-schedule.html`](https://dev.mysql.com/doc/refman/8.0/en/myisam-maintenance-schedule.html)

定期执行表检查是一个好主意，而不是等到问题发生时再处理。检查和修复`MyISAM`表的一种方法是使用`CHECK TABLE`和`REPAIR TABLE`语句。参见第 15.7.3 节，“表维护语句”。

另一种检查表的方法是使用**myisamchk**。为了维护目的，您可以使用**myisamchk -s**。`-s` 选项（简写为`--silent`）会导致**myisamchk**以静默模式运行，仅在发生错误时打印消息。

启用自动的`MyISAM`表检查也是一个好主意。例如，每当机器在更新过程中重新启动时，通常需要在进一步使用之前检查可能受影响的每个表。（这些是“预期崩溃的表”）。要使服务器自动检查`MyISAM`表，请使用设置`myisam_recover_options`系统变量来启动。参见第 7.1.8 节，“服务器系统变量”。

在正常系统运行期间，您还应定期检查您的表。例如，您可以运行一个**cron**作业，每周检查一次重要的表，可以在`crontab`文件中使用以下行：

```sql
35 0 * * 0 */path/to/myisamchk* --fast --silent */path/to/datadir*/*/*.MYI
```

这将打印出有关崩溃表的信息，以便您可以根据需要检查和修复它们。

首先，在所有在过去 24 小时内更新过的表上每晚执行**myisamchk -s**。当您发现问题很少发生时，您可以将检查频率减少到每周一次或类似频率。

通常，MySQL 表需要很少的维护。如果您对具有动态大小行（具有`VARCHAR`、`BLOB`或`TEXT`列的`MyISAM`表进行了许多更新，或者有许多已删除行的表，您可能希望不时地对表进行碎片整理/回收空间。您可以通过对相关表使用`OPTIMIZE TABLE`来实现这一点。或者，如果您可以暂停**mysqld**服务器一段时间，切换到数据目录并在服务器停止时使用以下命令：

```sql
$> myisamchk -r -s --sort-index --myisam_sort_buffer_size=16M */*.MYI
```
