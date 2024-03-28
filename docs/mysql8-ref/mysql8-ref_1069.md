> 原文：[`dev.mysql.com/doc/refman/8.0/en/uninstall-component.html`](https://dev.mysql.com/doc/refman/8.0/en/uninstall-component.html)

#### 15.7.4.5 UNINSTALL COMPONENT Statement

```sql
UNINSTALL COMPONENT *component_name* [, *component_name* ] ...
```

此语句停用并卸载一个或多个组件。组件提供服务器和其他组件可用的服务；请参阅 Section 7.5, “MySQL Components”。`UNINSTALL COMPONENT`是`INSTALL COMPONENT`的补充。它需要对`mysql.component`系统表具有`DELETE`权限，因为它会从注册组件的表中删除行。`UNINSTALL COMPONENT`不会撤消已持久化的变量，包括使用`INSTALL COMPONENT ... SET PERSIST`持久化的变量。

示例：

```sql
UNINSTALL COMPONENT 'file://component1', 'file://component2';
```

有关组件命名的信息，请参阅 Section 15.7.4.3, “INSTALL COMPONENT Statement”。

如果发生任何错误，该语句将失败且不起作用。例如，如果组件名称错误，未安装命名组件或无法卸载因为其他已安装的组件依赖于它。

一个加载程序服务处理组件卸载，包括从作为注册表的`mysql.component`系统表中删除已卸载的组件。因此，在后续服务器重新启动的启动序列中不会加载已卸载的组件。

注意

此语句对于使用清单文件加载的密钥环组件没有效果，并且无法卸载。请参阅 Section 8.4.4.2, “Keyring Component Installation”。
