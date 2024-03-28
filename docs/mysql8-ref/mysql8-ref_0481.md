# 8.7.6 故障排除 SELinux

> 原文：[`dev.mysql.com/doc/refman/8.0/en/selinux-troubleshooting.html`](https://dev.mysql.com/doc/refman/8.0/en/selinux-troubleshooting.html)

诊断 SELinux 通常涉及将 SELinux 置于宽容模式，重新运行有问题的操作，在 SELinux 审计日志中检查访问拒绝消息，并在问题解决后将 SELinux 放回强制模式。

为了避免使用 **setenforce** 将整个系统置于宽容模式，您可以通过使用 **semanage** 命令将其 SELinux 域（`mysqld_t`）置于宽容模式，只允许 MySQL 服务以宽容模式运行：

```sql
semanage permissive -a mysqld_t
```

在完成故障排除后，请使用以下命令将 `mysqld_t` 域放回强制模式：

```sql
semanage permissive -d mysqld_t
```

SELinux 将拒绝操作的日志写入 `/var/log/audit/audit.log`。您可以通过搜索“denied”消息来检查拒绝。

```sql
grep "denied" /var/log/audit/audit.log
```

以下部分描述了可能遇到与 SELinux 相关问题的几个常见领域。

#### 文件上下文

如果 MySQL 目录或文件具有不正确的 SELinux 上下文，则可能会拒绝访问。如果 MySQL 配置为从非默认目录或文件读取或写入，则可能会出现此问题。例如，如果您配置 MySQL 使用非默认数据目录，则该目录可能没有预期的 SELinux 上下文。

尝试在具有无效 SELinux 上下文的非默认数据目录上启动 MySQL 服务会导致以下启动失败。

```sql
$> systemctl start mysql.service
Job for mysqld.service failed because the control process exited with error code.
See "systemctl status mysqld.service" and "journalctl -xe" for details.
```

在这种情况下，将“拒绝”消息记录到 `/var/log/audit/audit.log` 中：

```sql
$> grep "denied" /var/log/audit/audit.log
type=AVC msg=audit(1587133719.786:194): avc:  denied  { write } for  pid=7133 comm="mysqld"
name="mysql" dev="dm-0" ino=51347078 scontext=system_u:system_r:mysqld_t:s0
tcontext=unconfined_u:object_r:default_t:s0 tclass=dir permissive=0
```

有关为 MySQL 目录和文件设置正确的 SELinux 上下文的信息，请参见 第 8.7.4 节，“SELinux 文件上下文”。

#### 端口访问

SELinux 期望诸如 MySQL 服务器之类的服务使用特定端口。更改端口而不更新 SELinux 策略可能会导致服务失败。

`mysqld_port_t` 端口类型定义了 MySQL 监听的端口。如果将 MySQL 服务器配置为使用非默认端口，例如端口 3307，并且不更新策略以反映更改，则 MySQL 服务无法启动：

```sql
$> systemctl start mysqld.service
Job for mysqld.service failed because the control process exited with error code.
See "systemctl status mysqld.service" and "journalctl -xe" for details.
```

在这种情况下，拒绝消息被记录到 `/var/log/audit/audit.log` 中：

```sql
$> grep "denied" /var/log/audit/audit.log
type=AVC msg=audit(1587134375.845:198): avc:  denied  { name_bind } for  pid=7340
comm="mysqld" src=3307 scontext=system_u:system_r:mysqld_t:s0
tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0
```

有关为 MySQL 设置正确的 SELinux 端口上下文的信息，请参见 第 8.7.5 节，“SELinux TCP 端口上下文”。启用使用未定义所需上下文的端口的 MySQL 功能时，可能会出现类似的端口访问问题。有关更多信息，请参见 第 8.7.5.2 节，“为 MySQL 功能设置 TCP 端口上下文”。

#### 应用程序更改

SELinux 可能不知道应用程序的更改。例如，新版本、应用程序扩展或新功能可能以 SELinux 不允许的方式访问系统资源，导致访问被拒绝。在这种情况下，您可以使用**audit2allow**实用程序创建自定义策略以允许必要的访问。创建自定义策略的典型方法是将 SELinux 模式更改为宽容模式，识别 SELinux 审计日志中的访问拒绝消息，并使用**audit2allow**实用程序创建自定义策略以允许访问。

有关使用**audit2allow**实用程序的信息，请参考您发行版的 SELinux 文档。

如果您遇到 MySQL 的访问问题，认为应该由标准 MySQL SELinux 策略模块处理，请在您发行版的错误跟踪系统中提交错误报告。
