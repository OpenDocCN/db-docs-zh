# 3.13 升级故障排除

> 原文：[`dev.mysql.com/doc/refman/8.0/en/upgrade-troubleshooting.html`](https://dev.mysql.com/doc/refman/8.0/en/upgrade-troubleshooting.html)

+   在 MySQL 5.7 实例中，表的`.frm`文件与`InnoDB`数据字典之间的模式不匹配可能导致升级到 MySQL 8.0 失败。这种不匹配可能是由于`.frm`文件损坏造成的。要解决此问题，请在再次尝试升级之前转储和恢复受影响的表。

+   如果出现问题，比如新的**mysqld**服务器无法启动，请验证您是否有来自先前安装的旧`my.cnf`文件。您可以使用`--print-defaults`选项来检查（例如，**mysqld --print-defaults**）。如果此命令显示除程序名称之外的任何内容，则您有一个影响服务器或客户端操作的活动`my.cnf`文件。

+   如果在升级后，您遇到编译的客户端程序出现问题，比如`Commands out of sync`或意外的核心转储，那么您可能在编译程序时使用了旧的头文件或库文件。在这种情况下，请检查您的`mysql.h`文件和`libmysqlclient.a`库文件的日期，以验证它们是否来自新的 MySQL 发行版。如果不是，请使用新的头文件和库文件重新编译您的程序。如果共享客户端库的主要版本号已更改（例如，从`libmysqlclient.so.20`到`libmysqlclient.so.21`），则可能还需要重新编译针对共享客户端库编译的程序。

+   如果您已经创建了一个具有特定名称的可加载函数，并将 MySQL 升级到实现具有相同名称的新内置函数的版本，则该可加载函数将变得无法访问。要纠正此问题，请使用`DROP FUNCTION`删除可加载函数，然后使用`CREATE FUNCTION`使用不冲突的不同名称重新创建可加载函数。如果新版本的 MySQL 实现了与现有存储函数同名的内置函数，则情况也是如此。有关服务器如何解释对不同类型函数的引用的规则，请参见 Section 11.2.5, “Function Name Parsing and Resolution”。

+   如果由于第 3.6 节“准备升级安装”中概述的任何问题而导致升级到 MySQL 8.0 失败，则服务器会将所有更改恢复到数据目录。在这种情况下，删除所有重做日志文件并在现有数据目录上重新启动 MySQL 5.7 服务器以解决错误。重做日志文件（`ib_logfile*`）默认位于 MySQL 数据目录中。在错误修复后，在尝试再次升级之前执行慢关闭（通过设置`innodb_fast_shutdown=0`）。
