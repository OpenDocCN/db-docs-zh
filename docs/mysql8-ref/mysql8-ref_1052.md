> 原文：[`dev.mysql.com/doc/refman/8.0/en/set-role.html`](https://dev.mysql.com/doc/refman/8.0/en/set-role.html)

#### 15.7.1.11 设置角色语句

```sql
SET ROLE {
    DEFAULT
  | NONE
  | ALL
  | ALL EXCEPT *role* [, *role* ] ...
  | *role* [, *role* ] ...
}
```

`SET ROLE`通过指定哪些授予的角色是活动的，修改当前用户在当前会话中的有效特权。授予的角色包括明确授予用户的角色和在`mandatory_roles`系统变量值中命名的角色。

示例：

```sql
SET ROLE DEFAULT;
SET ROLE 'role1', 'role2';
SET ROLE ALL;
SET ROLE ALL EXCEPT 'role1', 'role2';
```

每个角色名称使用第 8.2.5 节，“指定角色名称”中描述的格式。如果省略角色名称的主机名部分，则默认为`'%'`。

用户直接授予的特权（而不是通过角色）不受活动角色的更改影响。

该语句允许这些角色说明符：

+   `DEFAULT`: 激活账户的默认角色。默认角色是使用`SET DEFAULT ROLE`指定的角色。

    当用户连接到服务器并成功验证时，服务器确定要激活的默认角色。如果启用了`activate_all_roles_on_login`系统变量，则服务器激活所有授予的角色。否则，服务器隐式执行`SET ROLE DEFAULT`。服务器仅激活可以激活的默认角色。服务器会将警告写入其错误日志，对于无法激活的默认角色，但客户端不会收到警告。

    如果用户在会话期间执行`SET ROLE DEFAULT`，则如果任何默认角色无法激活（例如，如果不存在或未授予给用户），则会发生错误。在这种情况下，当前活动角色不会更改。

+   `NONE`: 将活动角色设置为`NONE`（无活动角色）。

+   `ALL`: 激活授予账户的所有角色。

+   `ALL EXCEPT *`role`* [, *`role`* ] ...`: 激活授予账户的所有角色，除了指定的角色。指定的角色不需要存在或被授予给账户。

+   `*`role`* [, *`role`* ] ...`: 激活命名的角色，这些角色必须授予给账户。

注意

`SET DEFAULT ROLE`和`SET ROLE DEFAULT`是不同的语句：

+   `SET DEFAULT ROLE`定义了默认情况下在账户会话中激活的账户角色。

+   `SET ROLE DEFAULT`将当前会话中的活动角色设置为当前账户的默认角色。

有关角色使用示例，请参见第 8.2.10 节，“使用角色”。
