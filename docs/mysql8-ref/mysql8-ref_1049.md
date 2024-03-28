> 原文：[`dev.mysql.com/doc/refman/8.0/en/revoke.html`](https://dev.mysql.com/doc/refman/8.0/en/revoke.html)

#### 15.7.1.8 REVOKE Statement

```sql
REVOKE [IF EXISTS]
    *priv_type* [(*column_list*)]
      [, *priv_type* [(*column_list*)]] ...
    ON [*object_type*] *priv_level*
    FROM *user_or_role* [, *user_or_role*] ...
    [IGNORE UNKNOWN USER]

REVOKE [IF EXISTS] ALL [PRIVILEGES], GRANT OPTION
    FROM *user_or_role* [, *user_or_role*] ...
    [IGNORE UNKNOWN USER]

REVOKE [IF EXISTS] PROXY ON *user_or_role*
    FROM *user_or_role* [, *user_or_role*] ...
    [IGNORE UNKNOWN USER]

REVOKE [IF EXISTS] *role* [, *role* ] ...
    FROM *user_or_role* [, *user_or_role* ] ...
    [IGNORE UNKNOWN USER]

*user_or_role*: {
    *user* (see Section 8.2.4, “Specifying Account Names”)
  | *role* (see Section 8.2.5, “Specifying Role Names”
}
```

`REVOKE`语句使系统管理员能够撤销用户帐户和角色的权限和角色。

有关权限存在的级别、允许的*`priv_type`*、*`priv_level`*和*`object_type`*值，以及指定用户和密码的语法的详细信息，请参见第 15.7.1.6 节，“GRANT Statement”。

有关角色的信息，请参见第 8.2.10 节，“使用角色”。

当启用`read_only`系统变量时，`REVOKE`需要`CONNECTION_ADMIN`或权限（或已弃用的`SUPER`权限），以及以下讨论中描述的任何其他所需权限。

从 MySQL 8.0.30 开始，所有`REVOKE`显示的形式都支持`IF EXISTS`选项以及`IGNORE UNKNOWN USER`选项。如果没有这两个修改，`REVOKE`对所有命名用户和角色都成功，或者如果发生任何错误则回滚并且没有效果；如果对所有命名用户和角色都成功，则该语句仅写入二进制日志。`IF EXISTS` 和 `IGNORE UNKNOWN USER` 的确切效果将在本节后面讨论。

每个帐户名称使用第 8.2.4 节，“指定帐户名称”中描述的格式。每个角色名称使用第 8.2.5 节，“指定角色名称”中描述的格式。例如：

```sql
REVOKE INSERT ON *.* FROM 'jeffrey'@'localhost';
REVOKE 'role1', 'role2' FROM 'user1'@'localhost', 'user2'@'localhost';
REVOKE SELECT ON world.* FROM 'role3';
```

帐户或角色名称的主机名部分，如果省略，默认为`'%'`。

要使用第一个`REVOKE`语法，您必须具有`GRANT OPTION`权限，并且必须具有您要撤销的权限。

要撤销所有权限，请使用第二种语法，该语法会为指定的用户或角色删除所有全局、数据库、表、列和例程权限。

```sql
REVOKE ALL PRIVILEGES, GRANT OPTION
  FROM *user_or_role* [, *user_or_role*] ...
```

`REVOKE ALL PRIVILEGES, GRANT OPTION` 不会撤销任何角色。

要使用此`REVOKE`语法，您必须具有全局`CREATE USER`权限，或者对`mysql`系统模式具有`UPDATE`权限。

后跟一个或多个角���名称的`REVOKE`关键字的语法需要一个`FROM`子句，指示要从中撤销角色的一个或多个用户或角色。

`IF EXISTS` 和 `IGNORE UNKNOWN USER` 选项（MySQL 8.0.30 及更高版本）具有以下列出的效果：

+   `IF EXISTS` 意味着，如果目标用户或角色存在，但由于任何原因未分配给目标，找不到这样的权限或角色，则会引发警告，而不是错误；如果语句中命名的权限或角色未分配给目标，语句没有（其他）效果。否则，`REVOKE` 正常执行；如果用户不存在，则语句会引发错误。

    *示例*：给定数据库 `test` 中的表 `t1`，我们执行以下语句，并显示结果。

    ```sql
    mysql> CREATE USER jerry@localhost;
    Query OK, 0 rows affected (0.01 sec)

    mysql> REVOKE SELECT ON test.t1 FROM jerry@localhost;
    ERROR 1147 (42000): There is no such grant defined for user 'jerry' on host
    'localhost' on table 't1' 
    mysql> REVOKE IF EXISTS SELECT ON test.t1 FROM jerry@localhost;
    Query OK, 0 rows affected, 1 warning (0.00 sec)

    mysql> SHOW WARNINGS\G
    *************************** 1\. row ***************************
      Level: Warning
       Code: 1147
    Message: There is no such grant defined for user 'jerry' on host 'localhost' on
    table 't1' 1 row in set (0.00 sec)
    ```

    如果 `REVOKE` 语句包括 `IF EXISTS`，即使命名的权限或角色不存在，或者语句尝试在错误的级别分配它，也会将错误降级为警告。

+   如果 `REVOKE` 语句包括 `IGNORE UNKNOWN USER`，则对于语句中命名但未找到的任何目标用户或角色，语句会引发警告；如果语句中没有存在的目标，`REVOKE` 成功但没有实际效果。否则，语句会像往常一样执行，并且尝试撤销由于任何原因未分配给目标的权限会引发错误，如预期的那样。

    *示例*（继续上一个示例）：

    ```sql
    mysql> DROP USER IF EXISTS jerry@localhost;
    Query OK, 0 rows affected (0.01 sec)

    mysql> REVOKE SELECT ON test.t1 FROM jerry@localhost;
    ERROR 1147 (42000): There is no such grant defined for user 'jerry' on host
    'localhost' on table 't1' 
    mysql> REVOKE SELECT ON test.t1 FROM jerry@localhost IGNORE UNKNOWN USER;
    Query OK, 0 rows affected, 1 warning (0.01 sec)

    mysql> SHOW WARNINGS\G
    *************************** 1\. row ***************************
      Level: Warning
       Code: 3162
    Message: Authorization ID jerry does not exist. 1 row in set (0.00 sec)
    ```

+   `IF EXISTS` 和 `IGNORE UNKNOWN USER` 的组合意味着 `REVOKE` 永远不会因为未知的目标用户或角色或未分配或不可用的权限而引发错误，在这种情况下，整个语句成功；只要可能，现有目标用户或角色将被移除角色或权限，并且任何无法撤销的撤销将引发警告并执行为 `NOOP`。

    *示例*（继续上一项中的示例）：

    ```sql
    # No such user, no such role
    mysql> DROP ROLE IF EXISTS Bogus;
    Query OK, 0 rows affected, 1 warning (0.02 sec)

    mysql> SHOW WARNINGS;
    +-------+------+----------------------------------------------+
    | Level | Code | Message                                      |
    +-------+------+----------------------------------------------+
    | Note  | 3162 | Authorization ID 'Bogus'@'%' does not exist. |
    +-------+------+----------------------------------------------+
    1 row in set (0.00 sec)

    # This statement attempts to revoke a nonexistent role from a nonexistent user
    mysql> REVOKE Bogus ON test FROM jerry@localhost;
    ERROR 3619 (HY000): Illegal privilege level specified for test

    # The same, with IF EXISTS
    mysql> REVOKE IF EXISTS Bogus ON test FROM jerry@localhost;
    ERROR 1147 (42000): There is no such grant defined for user 'jerry' on host
    'localhost' on table 'test' 

    # The same, with IGNORE UNKNOWN USER
    mysql> REVOKE Bogus ON test FROM jerry@localhost IGNORE UNKNOWN USER;
    ERROR 3619 (HY000): Illegal privilege level specified for test

    # The same, with both options
    mysql> REVOKE IF EXISTS Bogus ON test FROM jerry@localhost IGNORE UNKNOWN USER;
    Query OK, 0 rows affected, 2 warnings (0.01 sec)

    mysql> SHOW WARNINGS;
    +---------+------+--------------------------------------------+
    | Level   | Code | Message                                    |
    +---------+------+--------------------------------------------+
    | Warning | 3619 | Illegal privilege level specified for test |
    | Warning | 3162 | Authorization ID jerry does not exist.     |
    +---------+------+--------------------------------------------+
    2 rows in set (0.00 sec)
    ```

在 `mandatory_roles` 系统变量值中命名的角色无法被撤销。当在尝试移除强制权限的语句中同时使用 `IF EXISTS` 和 `IGNORE UNKNOWN USER` 时，通常由于尝试这样做而引发的错误会降级为警告；语句成功执行，但不会进行任何更改。

撤销的角色立即影响被撤销的任何用户账户，因此在账户的任何当前会话中，其权限将在执行下一条语句时进行调整。

撤销角色会撤销角色本身，而不是它代表的权限。假设一个账户被授予一个包含给定权限的角色，并且还明确授予该权限或包含该权限的另一个角色。在这种情况下，如果撤销第一个角色，则账户仍然拥有该权限。例如，如果一个账户被授予两个都包含 `SELECT` 的角色，那么在撤销任一角色后，该账户仍然可以进行选择。

`REVOKE ALL ON *.*`（在全局级别）撤销所有授予的静态全局权限和所有授予的动态权限。

服务器不知道的已授予但未知的已撤销权限会带有警告被撤销。这种情况可能发生在动态权限上。例如，动态权限可以在安装注册它的组件时授予，但如果随后卸载该组件，则权限变为未注册，尽管拥有该权限的账户仍然拥有它，并且可以从他们那里撤销。

`REVOKE`会移除权限，但不会从`mysql.user`系统表中删除行。要完全删除用户账户，请使用`DROP USER`。参见 Section 15.7.1.5, “DROP USER Statement”。

如果授权表中包含包含大小写混合的数据库或表名的权限行，并且`lower_case_table_names`系统变量设置为非零值，则无法使用`REVOKE`来撤销这些权限。在这种情况下，必须直接操作授权表。(`GRANT`在设置`lower_case_table_names`时不会创建这样的行，但在设置变量之前可能已创建这样的行。只能在初始化服务器时配置`lower_case_table_names`设置。)

当成功从**mysql**程序执行时，`REVOKE`会回应`Query OK, 0 rows affected`。要确定操作后剩余的权限，使用`SHOW GRANTS`。参见 Section 15.7.7.21, “SHOW GRANTS Statement”。
