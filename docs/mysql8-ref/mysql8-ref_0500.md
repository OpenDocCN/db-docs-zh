> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqldump-upgrade-testing.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqldump-upgrade-testing.html)

#### 9.4.5.5 使用 mysqldump 进行升级不兼容性测试

在考虑升级 MySQL 时，明智的做法是将新版本单独安装在当前生产版本之外。然后，您可以从生产服务器中转储数据库和数据库对象定义，然后加载到新服务器中以验证它们是否被正确处理。（这也对降级测试很有用。）

在生产服务器上：

```sql
$> mysqldump --all-databases --no-data --routines --events > dump-defs.sql
```

在升级后的服务器上：

```sql
$> mysql < dump-defs.sql
```

因为转储文件不包含表数据，所以可以快速处理。这使您能够在等待长时间的数据加载操作时发现潜在的不兼容性。在处理转储文件时查找警告或错误。

在确认定义已经正确处理后，转储数据并尝试加载到升级后的服务器中。

在生产服务器上：

```sql
$> mysqldump --all-databases --no-create-info > dump-data.sql
```

在升级后的服务器上：

```sql
$> mysql < dump-data.sql
```

现在检查表内容并运行一些测试查询。
