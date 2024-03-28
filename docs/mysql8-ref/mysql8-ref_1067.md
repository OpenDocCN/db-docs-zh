> 原文：[`dev.mysql.com/doc/refman/8.0/en/install-component.html`](https://dev.mysql.com/doc/refman/8.0/en/install-component.html)

#### 15.7.4.3 INSTALL COMPONENT 语句

```sql
INSTALL COMPONENT *component_name*  [, *component_name* ...
     [SET *variable* = *expr* [, *variable* = *expr*] ...] 

  *variable*: {
    {GLOBAL | @@GLOBAL.} [*component_prefix*.]*system_var_name*
  | {PERSIST | @@PERSIST.} [*component_prefix*.]*system_var_name*
}
```

此语句安装一个或多个组件，这些组件立即生效。组件提供服务器和其他组件可用的服务；请参阅第 7.5 节，“MySQL 组件”。`INSTALL COMPONENT`需要对`mysql.component`系统表具有`INSERT`权限，因为它向该表添加一行以注册组件。

示例：

```sql
INSTALL COMPONENT 'file://component1', 'file://component2';
```

组件使用以`file://`开头的 URN 命名，指示实现组件的库文件的基本名称，位于由`plugin_dir`系统变量命名的目录中。组件名称不包括任何平台相关的文件名后缀，如`.so`或`.dll`。（这些命名细节可能会发生变化，因为组件名称的解释本身是由一个服务执行的，并且组件基础设施使得可以用替代实现替换默认服务实现。）

`INSTALL COMPONENT`（从 8.0.33 版本开始）允许在安装一个或多个组件时设置组件系统变量的值。`SET`子句使您能够在需要时精确指定变量值，而不会受到其他形式赋值的不便或限制。具体来说，您还可以使用以下替代方法设置组件变量：

+   在服务器启动时使用命令行选项或选项文件，但这样做需要重新启动服务器。在安装组件之前，这些值不会生效。您可以在命令行上为组件指定一个无效的变量名而不会触发错误。

+   在服务器运行时通过`SET`语句动态设置，这使您可以修改服务器的操作而无需停止和重新启动。不允许设置只读变量。

可选的`SET`子句仅将一个值或多个值应用于`INSTALL COMPONENT`语句中指定的组件，而不是应用于该组件的所有后续安装。`SET GLOBAL|PERSIST`适用于所有类型的变量，包括只读变量，而无需重新启动服务器。使用`INSTALL COMPONENT`设置的组件系统变量优先于来自命令行或选项文件的任何冲突值。

示例：

```sql
INSTALL COMPONENT 'file://component1', 'file://component2' 
    SET GLOBAL component1.var1 = 12 + 3, PERSIST component2.var2 = 'strings';
```

省略`PERSIST`或`GLOBAL`等同于指定`GLOBAL`。

在 `SET` 中为任何变量指定 `PERSIST` 会在 `INSTALL COMPONENT` 加载组件后立即执行 `SET PERSIST_ONLY`，但在更新 `mysql.component` 表之前。如果 `SET PERSIST_ONLY` 失败，则服务器会卸载所有先前加载的新组件，而不会将任何内容持久化到 `mysql.component`。

`SET` 子句仅接受正在安装的组件的有效变量名称，并对所有无效名称发出错误消息。子查询、存储函数和聚合函数不允许作为值表达式的一部分。如果安装单个组件，则不需要使用组件名称作为变量名称的前缀。

注意

使用 `SET` 子句指定变量值与命令行类似——在变量注册时立即可用——但 `SET` 子句在处理布尔变量的 *无效数值* 时有明显差异。例如，如果将布尔变量设置为 11（`component1.boolvar = 11`），您会看到以下行为：

+   `SET` 子句返回 true

+   命令行返回 false（11 既不是 ON 也不是 1）

如果发生任何错误，语句将失败且不会产生任何效果。例如，如果组件名称错误，命名组件不存在或已安装，或组件初始化失败，则会发生这种情况。

加载服务处理组件加载，包括将已安装的组件添加到作为注册表的 `mysql.component` 系统表。对于后续的服务器重启，`mysql.component` 中列出的任何组件都将在启动序列期间由加载服务加载。即使服务器使用 `--skip-grant-tables` 选项启动也会发生这种情况。

如果一个组件依赖于注册表中不存在的服务，并且您尝试安装该组件而没有安装提供所依赖服务的组件或组件，则会发生错误：

```sql
ERROR 3527 (HY000): Cannot satisfy dependency for service 'component_a'
required by component 'component_b'.
```

要避免此问题，要么在同一语句中安装所有组件，要么在安装任何依赖的组件之后安装依赖组件。

注意

对于密钥环组件，请勿使用 `INSTALL COMPONENT`。而是使用清单文件配置密钥环组件加载。参见 Section 8.4.4.2, “Keyring Component Installation”。
