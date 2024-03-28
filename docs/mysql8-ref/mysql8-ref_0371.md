# 8.2.11 账户类别

> 原文：[`dev.mysql.com/doc/refman/8.0/en/account-categories.html`](https://dev.mysql.com/doc/refman/8.0/en/account-categories.html)

从 MySQL 8.0.16 开始，MySQL 引入了基于 `SYSTEM_USER` 权限的用户账户类别概念。

+   系统和常规账户

+   SYSTEM_USER 权限影响的操作

+   系统和常规会话

+   保护系统账户免受常规账户篡改

#### 系统和常规账户

MySQL 引入了用户账户类别的概念，根据是否拥有 `SYSTEM_USER` 权限来区分系统用户和常规用户：

+   拥有 `SYSTEM_USER` 权限的用户是系统用户。

+   没有 `SYSTEM_USER` 权限的用户是常规用户。

`SYSTEM_USER` 权限对用户可以应用其它权限的账户产生影响，以及用户是否受到其他账户的保护：

+   系统用户可以修改系统和常规账户。也就是说，拥有适当权限在常规账户上执行特定操作的用户，通过拥有 `SYSTEM_USER` 权限也能在系统账户上执行该操作。系统账户只能被拥有适当权限的系统用户修改，而不能被常规用户修改。

+   拥有适当权限的常规用户可以修改常规账户，但不能修改系统账户。系统账户可以被拥有适当权限的系统用户和常规用户修改。

如果用户有适当权限在常规账户上执行特定操作，`SYSTEM_USER` 使用户也能在系统账户上执行该操作。`SYSTEM_USER` 不意味着任何其他权限，因此执行特定账户操作的能力仍取决于是否拥有任何其他所需权限。例如，如果用户可以授予常规账户 `SELECT` 和 `UPDATE` 权限，那么拥有 `SYSTEM_USER` 的用户也可以授予系统账户 `SELECT` 和 `UPDATE` 权限。

系统账户和普通账户之间的区别通过保护具有`SYSTEM_USER`特权的账户免受没有该特权的账户的影响，从而更好地控制某些账户管理问题。例如，`CREATE USER`特权不仅允许创建新账户，还允许修改和删除现有账户。没有系统用户概念，拥有`CREATE USER`特权的用户可以修改或删除任何现有账户，包括`root`账户。系统用户概念使得限制对`root`账户（本身是系统账户）的修改只能由系统用户进行。拥有`CREATE USER`特权的普通用户仍然可以修改或删除现有账户，但只能是普通账户。

#### 受`SYSTEM_USER`特权影响的操作

`SYSTEM_USER`特权影响以下操作：

+   账户操作。

    账户操作包括创建和删除账户，授予和撤销特权，更改账户认证特性（如凭证或认证插件），以及更改其他账户特性，如密码过期策略。

    使用账户管理语句（如`CREATE USER`和`GRANT`"）操纵系统账户需要`SYSTEM_USER`特权。为防止账户以这种方式修改系统账户，将其设为普通账户，不授予`SYSTEM_USER`特权。（然而，要完全保护系统账户免受普通账户的修改，还必须对普通账户限制对`mysql`系统模式的修改特权。请参见保护系统账户免受普通账户操纵。）

+   终止当前会话和其中执行的语句。

    要终止使用`SYSTEM_USER`特权执行的会话或语句，您自己的会话必须具有`SYSTEM_USER`特权，以及任何其他所需特权（`CONNECTION_ADMIN`或已弃用的`SUPER`特权）。

    从 MySQL 8.0.30 开始，如果将服务器置于脱机模式的用户没有`SYSTEM_USER`权限，则具有`SYSTEM_USER`权限的连接客户端用户也不会被断开连接。但是，这些用户在服务器处于脱机模式时无法启动新连接，除非他们还具有`CONNECTION_ADMIN`或`SUPER`权限。仅仅是他们现有的连接不会被终止，因为需要`SYSTEM_USER`权限才能执行该操作。

    在 MySQL 8.0.16 之前，`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限）足以终止任何会话或语句。

+   为存储对象设置`DEFINER`属性。

    要将存储对象的`DEFINER`属性设置为具有`SYSTEM_USER`权限的帐户，您必须具有`SYSTEM_USER`权限，以及任何其他所需权限（`SET_USER_ID`或已弃用的`SUPER`权限）。

    在 MySQL 8.0.16 之前，`SET_USER_ID`权限（或已弃用的`SUPER`权限）足以为存储对象指定任何`DEFINER`值。

+   指定强制角色。

    具有`SYSTEM_USER`权限的角色不能在`mandatory_roles`系统变量的值中列出。

    在 MySQL 8.0.16 之前，任何角色都可以列在`mandatory_roles`中。

+   覆盖 MySQL Enterprise Audit 的审计日志过滤器中的“中止”项目。

    从 MySQL 8.0.28 开始，具有`SYSTEM_USER`权限的帐户会自动被分配`AUDIT_ABORT_EXEMPT`权限，因此即使审计日志过滤器中的“中止”项目会阻止它们，该帐户的查询也会始终执行。具有`SYSTEM_USER`权限的帐户因此可用于在审计配置错误后恢复对系统的访问。参见第 8.4.5 节，“MySQL Enterprise Audit”。

#### 系统和常规会话

在服务器内执行的会��被区分为系统会话或常规会话，类似于系统和常规用户之间的区别：

+   拥有`SYSTEM_USER`权限的会话是系统会话。

+   不具有`SYSTEM_USER`权限的会话是普通会话。

普通会话只能执行普通用户允许的操作。系统会话还能执行仅允许系统用户的操作。

会话拥有的权限包括直接授予其基础帐户的权限，以及授予会话内所有当前活动角色的权限。因此，会话可能是系统会话，因为其帐户直接被授予`SYSTEM_USER`权限，或者因为会话已激活具有`SYSTEM_USER`权限的角色。授予帐户但在会话中未激活的角色不会影响会话权限。

因为激活和停用角色可以改变会话拥有的权限，会话可能从普通会话变为系统会话，反之亦然。如果会话激活或停用具有`SYSTEM_USER`权限的角色，则会话之间的普通和系统会话之间的适当变化立即发生，仅适用于该会话：

+   如果普通会话激活具有`SYSTEM_USER`权限的角色，则该会话将变为系统会话。

+   如果系统会话停用具有`SYSTEM_USER`权限的角色，则该会话将变为普通会话，除非仍有其他具有`SYSTEM_USER`权限的角色处于活动状态。

这些操作不会影响现有会话：

+   如果向帐户授予或撤销`SYSTEM_USER`权限，则帐户的现有会话在普通会话和系统会话之间不会发生变化。授予或撤销操作仅影响帐户后续连接的会话。

+   在会话中调用的存储对象执行的语句将以父会话的系统或普通状态执行，即使对象的`DEFINER`属性命名了系统帐户。

因为角色激活仅影响会话而不影响帐户，向普通帐户授予具有`SYSTEM_USER`权限的角色不会保护该帐户免受普通用户的攻击。该角色仅保护已激活该角色的帐户的会话，并仅保护会话免受普通会话的终止。

#### 保护系统帐户免受普通帐户的操纵

帐户操作包括创建和删除帐户，授予和撤销权限，更改帐户身份验证特性，如凭据或身份验证插件，以及更改其他帐户特性，如密码过期策略。

帐户操作可以通过两种方式完成：

+   通过使用诸如`CREATE USER`和`GRANT`等帐户管理语句。这是首选方法。

+   通过使用诸如`INSERT`和`UPDATE`等语句直接修改授权表。虽然不鼓励这种方法，但对于具有`mysql`系统模式上适当特权的用户是可能的。

要完全保护系统帐户免受特定帐户的修改，将其设置为常规帐户，并不授予`mysql`模式的修改特权：

+   使用帐户管理语句操作系统帐户需要`SYSTEM_USER`特权。为防止帐户以这种方式修改系统帐户，将其设置为常规帐户，不授予`SYSTEM_USER`。这包括不授予任何角色授予的帐户`SYSTEM_USER`。

+   `mysql`模式的特权允许通过直接修改授权表来操作系统帐户，即使修改帐户是常规帐户。为了防止常规帐户通过直接修改未经授权地修改系统帐户，请不要向帐户（或任何授予帐户的角色）授予`mysql`模式的修改特权。如果常规帐户必须具有适用于所有模式的全局特权，则可以使用部分撤销施加的特权限制来阻止`mysql`模式的修改。参见 Section 8.2.12, “Privilege Restriction Using Partial Revokes”。

注意

与不授予`SYSTEM_USER`特权不同，阻止帐户修改系统帐户但不是常规帐户，不授予`mysql`模式特权会阻止帐户修改系统帐户以及常规帐户。这不应该是一个问题，因为如前所述，不鼓励直接修改授权表。

假设您想要创建一个名为`u1`的用户，该用户对所有模式具有所有特权，但`u1`应该是一个不能修改系统帐户的常规用户。假设`partial_revokes`系统变量已启用，配置`u1`如下：

```sql
CREATE USER u1 IDENTIFIED BY '*password*';

GRANT ALL ON *.* TO u1 WITH GRANT OPTION;
-- GRANT ALL includes SYSTEM_USER, so at this point
-- u1 can manipulate system or regular accounts

REVOKE SYSTEM_USER ON *.* FROM u1;
-- Revoking SYSTEM_USER makes u1 a regular user;
-- now u1 can use account-management statements
-- to manipulate only regular accounts

REVOKE ALL ON mysql.* FROM u1;
-- This partial revoke prevents u1 from directly
-- modifying grant tables to manipulate accounts
```

为防止帐户访问所有`mysql`系统模式，撤销其在`mysql`模式上的所有特权，如上所示。也可以允许部分`mysql`模式访问，例如只读访问。以下示例创建一个帐户，该帐户对所有模式具有全局的`SELECT`、`INSERT`、`UPDATE`和`DELETE`特权，但对`mysql`模式仅具有`SELECT`特权：

```sql
CREATE USER u2 IDENTIFIED BY '*password*';
GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO u2;
REVOKE INSERT, UPDATE, DELETE ON mysql.* FROM u2;
```

另一种可能性是撤销所有`mysql`模式的权限，但授予对特定`mysql`表或列的访问权限。即使对`mysql`进行部分撤销，也可以完成此操作。以下语句允许在`mysql`模式内对`u1`进行只读访问，但仅限于`db`表以及`user`表的`Host`和`User`列：

```sql
CREATE USER u3 IDENTIFIED BY '*password*';
GRANT ALL ON *.* TO u3;
REVOKE ALL ON mysql.* FROM u3;
GRANT SELECT ON mysql.db TO u3;
GRANT SELECT(Host,User) ON mysql.user TO u3;
```
