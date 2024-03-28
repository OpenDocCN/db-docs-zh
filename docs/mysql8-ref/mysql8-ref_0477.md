# 8.7.4 SELinux 文件上下文

> 原文：[`dev.mysql.com/doc/refman/8.0/en/selinux-file-context.html`](https://dev.mysql.com/doc/refman/8.0/en/selinux-file-context.html)

MySQL 服务器从许多文件中读取和写入。如果这些文件的 SELinux 上下文未正确设置，可能会拒绝对文件的访问。

接下来的说明使用 `semanage` 二进制文件来管理文件上下文；在 RHEL 上，它是 `policycoreutils-python-utils` 包的一部分：

```sql
yum install -y policycoreutils-python-utils
```

安装 `semanage` 二进制文件后，您可以使用 `semanage` 和 `fcontext` 选项列出 MySQL 文件上下文。

```sql
semanage fcontext -l | grep -i mysql
```

#### 设置 MySQL 数据目录上下文

默认数据目录位置是 `/var/lib/mysql/`；使用的 SELinux 上下文是 `mysqld_db_t`。

如果您编辑配置文件以使用不同位置作为数据目录，或者用于数据目录中通常的任何文件（例如二进制日志），则可能需要为新位置设置上下文。例如：

```sql
semanage fcontext -a -t mysqld_db_t "/path/to/my/custom/datadir(/.*)?"
restorecon -Rv /path/to/my/custom/datadir

semanage fcontext -a -t mysqld_db_t "/path/to/my/custom/logdir(/.*)?"
restorecon -Rv /path/to/my/custom/logdir
```

#### 设置 MySQL 错误日志文件上下文

RedHat RPM 的默认位置是 `/var/log/mysqld.log`；使用的 SELinux 上下文类型是 `mysqld_log_t`。

如果您编辑配置文件以使用不同位置，则可能需要为新位置设置上下文。例如：

```sql
semanage fcontext -a -t mysqld_log_t "/path/to/my/custom/error.log"
restorecon -Rv /path/to/my/custom/error.log
```

#### 设置 PID 文件上下文

PID 文件的默认位置是 `/var/run/mysqld/mysqld.pid`；使用的 SELinux 上下文类型是 `mysqld_var_run_t`。

如果您编辑配置文件以使用不同位置，则可能需要为新位置设置上下文。例如：

```sql
semanage fcontext -a -t mysqld_var_run_t "/path/to/my/custom/pidfile/directory/.*?"
restorecon -Rv /path/to/my/custom/pidfile/directory
```

#### 设置 Unix 域套接字上下文

Unix 域套接字的默认位置是 `/var/lib/mysql/mysql.sock`；使用的 SELinux 上下文类型是 `mysqld_var_run_t`。

如果您编辑配置文件以使用不同位置，则可能需要为新位置设置上下文。例如：

```sql
semanage fcontext -a -t mysqld_var_run_t "/path/to/my/custom/mysql\.sock"
restorecon -Rv /path/to/my/custom/mysql.sock
```

#### 设置 secure_file_priv 目录上下文

对于自 MySQL 5.6.34、5.7.16 和 8.0.11 版本以来的 MySQL 版本。

安装 MySQL Server RPM 会创建一个 `/var/lib/mysql-files/` 目录，但不会为其设置 SELinux 上下文。`/var/lib/mysql-files/` 目录旨在用于诸如 `SELECT ... INTO OUTFILE` 等操作。

如果您通过设置 `secure_file_priv` 启用了对此目录的使用，则可能需要这样设置上下文：

```sql
semanage fcontext -a -t mysqld_db_t "/var/lib/mysql-files/(/.*)?"
restorecon -Rv /var/lib/mysql-files
```

如果您使用了不同位置，请编辑此路径。出于安全目的，此目录永远不应在数据目录内。

有关此变量的更多信息，请参阅`secure_file_priv` 文档。
