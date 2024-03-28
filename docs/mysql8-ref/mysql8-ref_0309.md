> 原文：[`dev.mysql.com/doc/refman/8.0/en/clone-plugin-local.html`](https://dev.mysql.com/doc/refman/8.0/en/clone-plugin-local.html)

#### 7.6.7.2 本地克隆数据

克隆插件支持以下语法用于本地克隆数据；即从本地 MySQL 数据目录克隆数据到同一服务器或节点上的另一个目录：

```sql
CLONE LOCAL DATA DIRECTORY [=] '*clone_dir*';
```

要使用`CLONE`语法，必须安装克隆插件。有关安装说明，请参见第 7.6.7.1 节，“安装克隆插件”。

执行`CLONE LOCAL DATA DIRECTORY`语句需要*`BACKUP_ADMIN`*权限。

```sql
mysql> GRANT BACKUP_ADMIN ON *.* TO '*clone_user*';
```

其中`*`clone_user`*`是执行克隆操作的 MySQL 用户。您选择执行克隆操作的用户可以是具有*`BACKUP_ADMIN`*权限的任何 MySQL 用户。

以下示例演示了本地克隆数据的操作：

```sql
mysql> CLONE LOCAL DATA DIRECTORY = '*/path/to/clone_dir*';
```

其中*`/path/to/clone_dir`*是数据被克隆到的本地目录的完整路径。需要绝对路径，并且指定的目录（“*`clone_dir`*”）不得存在，但指定路径必须是一个已存在的路径。MySQL 服务器必须具有必要的写访问权限以创建目录。

注意

本地克隆操作不支持克隆存储在数据目录之外的用户创建的表或表空间。尝试克隆这样的表或表空间会导致以下错误：ERROR 1086 (HY000): File '*`/path/to/tablespace_name.ibd`*' already exists. 尝试克隆具有与源表空间相同路径的表空间会导致冲突，因此被禁止。

所有其他用户创建的`InnoDB`表和表空间，`InnoDB`系统表空间，重做日志和撤销表空间都会被克隆到指定目录。

如果需要，您可以在克隆操作完成后在克隆目录上启动 MySQL 服务器。

```sql
$> mysqld_safe --datadir=*clone_dir*
```

其中*`clone_dir`*是数据被克隆到的目录。

有关监视克隆操作状态和进度的信息，请参见第 7.6.7.10 节，“监视克隆操作”。
