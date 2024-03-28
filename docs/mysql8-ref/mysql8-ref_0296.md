> 原文：[`dev.mysql.com/doc/refman/8.0/en/rewriter-query-rewrite-plugin-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/rewriter-query-rewrite-plugin-installation.html)

#### 7.6.4.1 安装或卸载 Rewriter 查询重写插件

注意

如果安装了`Rewriter`插件，即使禁用了也会带来一些开销。为避免这种开销，除非打算使用该插件，否则不要安装它。

要安装或卸载`Rewriter`查询重写插件，请选择位于 MySQL 安装的`share`目录中的适当脚本：

+   `install_rewriter.sql`：选择此脚本以安装`Rewriter`插件及其相关元素。

+   `uninstall_rewriter.sql`：选择此脚本以卸载`Rewriter`插件及其相关元素。

按照以下方式运行所选脚本：

```sql
$> mysql -u root -p < install_rewriter.sql
Enter password: *(enter root password here)*
```

此处示例使用`install_rewriter.sql`安装脚本。如果要卸载插件，请替换为`uninstall_rewriter.sql`。

运行安装脚本应该会安装并启用插件。要验证，请连接到服务器并执行以下语句：

```sql
mysql> SHOW GLOBAL VARIABLES LIKE 'rewriter_enabled';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| rewriter_enabled | ON    |
+------------------+-------+
```

有关使用说明，请参见第 7.6.4.2 节，“使用 Rewriter 查询重写插件”。有关参考信息，请参见第 7.6.4.3 节，“Rewriter 查询重写插件参考”。
