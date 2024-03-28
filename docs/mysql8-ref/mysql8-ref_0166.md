# 6.2.9 设置环境变量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/setting-environment-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/setting-environment-variables.html)

环境变量可以在命令提示符下设置以影响当前命令处理器的当前调用，或永久设置以影响将来的调用。要永久设置变量，可以将其设置在启动文件中或使用系统为此目的提供的界面。请查阅您的命令解释器的文档以获取具体细节。第 6.9 节，“环境变量”列出了影响 MySQL 程序操作的所有环境变量。

要为环境变量指定值，请使用适合您的命令处理器的语法。例如，在 Windows 上，您可以设置`USER`变量以指定您的 MySQL 帐户名。要这样做，请使用以下语法：

```sql
SET USER=*your_name*
```

Unix 上的语法取决于您的 shell。假设您想使用`MYSQL_TCP_PORT`变量指定 TCP/IP 端口号。典型的语法（例如对于**sh**，**ksh**，**bash**，**zsh**等）如下：

```sql
MYSQL_TCP_PORT=3306
export MYSQL_TCP_PORT
```

第一个命令设置变量，`export`命令将变量导出到 shell 环境，使其值对 MySQL 和其他进程可访问。

对于**csh**和**tcsh**，使用**setenv**使 shell 变量可用于环境：

```sql
setenv MYSQL_TCP_PORT 3306
```

设置环境变量的命令可以在命令提示符下立即生效，但设置仅在注销前有效。要使设置在每次登录时生效，请使用系统提供的界面或将适当的命令或命令放置在启动文件中，以便您的命令解释器每次启动时都会读取。

在 Windows 上，您可以使用系统控制面板（在高级选项下）设置环境变量。

在 Unix 上，典型的 shell 启动文件是`.bashrc`或`.bash_profile`对于**bash**，或`.tcshrc`对于**tcsh**。

假设您的 MySQL 程序安装在`/usr/local/mysql/bin`中，并且您希望轻松调用这些程序。为此，请将`PATH`环境变量的值设置为包含该目录。例如，如果您的 shell 是**bash**，请将以下行添加到您的`.bashrc`文件中：

```sql
PATH=${PATH}:/usr/local/mysql/bin
```

**bash** 对登录和非登录 shell 使用不同的启动文件，因此您可能希望为登录 shell 添加设置到`.bashrc`，而对于非登录 shell 添加到`.bash_profile`，以确保`PATH`被设置。

如果您的 shell 是**tcsh**，请将以下行添加到您的`.tcshrc`文件中：

```sql
setenv PATH ${PATH}:/usr/local/mysql/bin
```

如果您的家目录中不存在适当的启动文件，请使用文本编辑器创建它。

修改`PATH`设置后，在 Windows 上打开一个新的控制台窗口，或在 Unix 上重新登录，以使设置生效。
