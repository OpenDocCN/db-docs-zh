> 原文：[`dev.mysql.com/doc/refman/8.0/en/validate-password-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/validate-password-installation.html)

#### 8.4.3.1 密码验证组件的安装和卸载

本节描述了如何安装和卸载 `validate_password` 密码验证组件。有关安装和卸载组件的一般信息，请参阅第 7.5 节，“MySQL 组件”。

注意

如果使用[MySQL Yum 仓库](https://dev.mysql.com/downloads/repo/yum/)、[MySQL SLES 仓库](https://dev.mysql.com/downloads/repo/suse/)或由 Oracle 提供的 RPM 软件包安装 MySQL 8.0，则在首次启动 MySQL 服务器后，默认启用 `validate_password` 组件。

从使用 Yum 或 RPM 软件包升级到 MySQL 8.0 时，`validate_password` 插件会保留在原位。要从 `validate_password` 插件过渡到 `validate_password` 组件，请参阅第 8.4.3.3 节，“过渡到密码验证组件”。

要使组件库文件可被服务器使用，必须将其放置在 MySQL 插件目录中（由`plugin_dir`系统变量命名的目录）。如有必要，在服务器启动时通过设置`plugin_dir`的值来配置插件目录位置。

要安装 `validate_password` 组件，请使用以下语句：

```sql
INSTALL COMPONENT 'file://component_validate_password';
```

组件安装是一次性操作，不需要每次服务器启动都执行。`INSTALL COMPONENT`加载组件，并在 `mysql.component` 系统表中注册它，以便在随后的服务器启动期间加载它。

要卸载 `validate_password` 组件，请使用以下语句：

```sql
UNINSTALL COMPONENT 'file://component_validate_password';
```

`UNINSTALL COMPONENT`卸载组件，并从 `mysql.component` 系统表中注销它，以使其在随后的服务器启动期间不加载。
