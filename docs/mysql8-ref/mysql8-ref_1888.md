# 27.6 存储对象访问控制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/stored-objects-security.html`](https://dev.mysql.com/doc/refman/8.0/en/stored-objects-security.html)

存储程序（过程、函数、触发器和事件）和视图在使用之前被定义，并且在引用时，在确定其权限的安全上下文中执行。适用于执行存储对象的权限由其`DEFINER`属性和`SQL SECURITY`特性控制。

+   DEFINER 属性

+   SQL 安全特性

+   示例

+   孤立存储对象

+   风险最小化指南

### DEFINER 属性

存储对象定义可以包括一个`DEFINER`属性，用于指定一个 MySQL 账户。如果定义省略了`DEFINER`属性，那么默认的对象定义者是创建它的用户。

下列规则确定了你可以指定为存储对象`DEFINER`属性的账户：

+   如果你拥有`SET_USER_ID`权限（或已弃用的`SUPER`权限），你可以指定任何账户作为`DEFINER`属性。如果该账户不存在，将生成一个警告。此外，要将存储对象的`DEFINER`属性设置为具有`SYSTEM_USER`权限的账户，你必须拥有`SYSTEM_USER`权限。

+   否则，唯一允许的账户是你自己，可以明确指定为`CURRENT_USER`或`CURRENT_USER()`。你不能将定义者设置为其他账户。

使用不存在的`DEFINER`账户创建存储对象会创建一个孤立对象，可能会产生负面后果；参见孤立存储对象。

### SQL 安全特性

对于存储例程（过程和函数）和视图，对象定义可以包括一个`SQL SECURITY`特性，其值为`DEFINER`或`INVOKER`，以指定对象是在定义者还是调用者上下文中执行。如果定义省略了`SQL SECURITY`特性，则默认为定义者上下文。

触发器和事件没有`SQL SECURITY`特性，始终在定义者上下文中执行。服务器根据需要自动调用这些对象，因此没有调用用户。

定义者和调用者安全上下文的区别如下：

+   在定义者安全上下文中执行的存储对象将以其`DEFINER`属性命名的帐户的特权执行。这些特权可能与调用用户的特权完全不同。调用者必须具有适当的特权来引用对象（例如，`EXECUTE`来调用存储过程或`SELECT`来从视图中选择），但在对象执行期间，调用者的特权将被忽略，只有`DEFINER`帐户的特权才重要。如果`DEFINER`帐户特权较少，则对象可以执行的操作也相应受限。如果`DEFINER`帐户具有高特权（例如管理帐户），则对象可以执行强大的操作*无论谁调用它*。

+   在调用者安全上下文中执行的存储过程或视图只能执行调用者具有特权的操作。`DEFINER`属性对对象执行没有影响。

### 例子

考虑以下存储过程，它声明为使用`SQL SECURITY DEFINER`在定义者安全上下文中执行：

```sql
CREATE DEFINER = 'admin'@'localhost' PROCEDURE p1()
SQL SECURITY DEFINER
BEGIN
  UPDATE t1 SET counter = counter + 1;
END;
```

任何具有`p1`的`EXECUTE`特权的用户都可以使用`CALL`语句调用它。但是，当`p1`执行时，它将在定义者安全上下文中执行，因此以其`DEFINER`属性命名的帐户`'admin'@'localhost'`的特权执行。此帐户必须对`p1`具有`EXECUTE`特权以及对对象体内引用的表`t1`具有`UPDATE`特权。否则，该过程将失败。

现在考虑这个存储过程，它与`p1`完全相同，只是其`SQL SECURITY`特性为`INVOKER`：

```sql
CREATE DEFINER = 'admin'@'localhost' PROCEDURE p2()
SQL SECURITY INVOKER
BEGIN
  UPDATE t1 SET counter = counter + 1;
END;
```

与`p1`不同，`p2`在调用者安全上下文中执行，因此以调用者的特权执行，而不管`DEFINER`属性值如何。如果调用者缺少`p2`的`EXECUTE`特权或表`t1`的`UPDATE`特权，则`p2`将失败。

### 孤立的存储对象

孤立的存储对象是指其`DEFINER`属性命名了一个不存在的帐户：

+   可以通过在创建对象时指定一个不存在的`DEFINER`帐户来创建孤立的存储对象。

+   通过执行`DROP USER`语句删除对象`DEFINER`帐户，或通过执行`RENAME USER`语句重命名对象`DEFINER`帐户，现有的存储对象可能变为孤立状态。

孤立的存储对象可能存在以下问题：

+   因为`DEFINER`帐户不存在，如果在定义者安全上下文中执行对象，则该对象可能无法按预期工作：

    +   对于存储过程，如果`SQL SECURITY`值为`DEFINER`但定义者帐户不存在，则在例程执行时会出现错误。

    +   对于触发器，直到帐户实际存在之前触发器激活并不是一个好主意。否则，关于权限检查的行为是未定义的。

    +   对于事件，如果帐户不存在，则在事件执行时会出现错误。

    +   对于视图，如果`SQL SECURITY`值为`DEFINER`但定义者帐户不存在，则在引用视图时会出现错误。

+   如果不存在的`DEFINER`帐户随后被重新创建用于与对象无关的目的，则该对象可能存在安全风险。在这种情况下，该帐户“接管”了对象，并且在具有适当权限的情况下，即使不打算如此，也能执行它。

从 MySQL 8.0.22 开始，服务器实施了额外的帐户管理安全检查，旨在防止（可能无意中）导致存储对象变为孤立或导致接管当前孤立的存储对象的操作：

+   `DROP USER`如果要删除的任何帐户被命名为任何存储对象的`DEFINER`属性，则会出现错误。（也就是说，如果删除帐户会导致存储对象变成孤立状态，则该语句将失败。）

+   `RENAME USER`如果要重命名的任何帐户被命名为任何存储对象的`DEFINER`属性，则会出现错误。（也就是说，如果重命名帐户会导致存储对象变成孤立状态，则该语句将失败。）

+   `CREATE USER`如果要创建的任何帐户被命名为任何存储对象的`DEFINER`属性，则会出现错误。（也就是说，如果创建帐户会导致帐户接管当前孤立的存储对象，则该语句将失败。）

在某些情况下，可能需要故意执行那些帐户管理语句，即使它们本来会失败。为了实现这一点，如果用户具有`SET_USER_ID`权限，则该权限将覆盖孤立对象安全检查，并且语句将成功并显示警告，而不是失败并显示错误。

要获取有关在 MySQL 安装中用作存储对象定义者的帐户的信息，请查询`INFORMATION_SCHEMA`。

此查询标识了哪些`INFORMATION_SCHEMA`表描述具有`DEFINER`属性的对象：

```sql
mysql> SELECT TABLE_SCHEMA, TABLE_NAME FROM INFORMATION_SCHEMA.COLUMNS
       WHERE COLUMN_NAME = 'DEFINER';
+--------------------+------------+
| TABLE_SCHEMA       | TABLE_NAME |
+--------------------+------------+
| information_schema | EVENTS     |
| information_schema | ROUTINES   |
| information_schema | TRIGGERS   |
| information_schema | VIEWS      |
+--------------------+------------+
```

结果告诉您要查询哪些表以发现哪些存储对象`DEFINER`值存在以及哪些对象具有特定的`DEFINER`值：

+   要确定每个表中存在哪些`DEFINER`值，请使用以下查询：

    ```sql
    SELECT DISTINCT DEFINER FROM INFORMATION_SCHEMA.EVENTS;
    SELECT DISTINCT DEFINER FROM INFORMATION_SCHEMA.ROUTINES;
    SELECT DISTINCT DEFINER FROM INFORMATION_SCHEMA.TRIGGERS;
    SELECT DISTINCT DEFINER FROM INFORMATION_SCHEMA.VIEWS;
    ```

    查询结果对于任何显示为以下内容的帐户都很重要：

    +   如果账户存在，则删除或重命名它会导致存储对象变为孤立。如果计划删除或重命名账户，请首先考虑删除其关联的存储对象或重新定义它们以具有不同的定义者。

    +   如果账户不存在，则创建它会导致它接管当前孤立的存储对象。如果计划创建账户，请考虑是否应将孤立的对象与之关联。如果不需要，请重新定义它们以具有不同的定义者。

    要重新定义具有不同定义者的对象，可以使用`ALTER EVENT`或`ALTER VIEW`直接修改事件和视图的`DEFINER`账户。对于存储过程和函数以及触发器，必须删除对象并重新创建以分配不同的`DEFINER`账户。

+   要识别具有特定`DEFINER`账户的对象，请使用以下查询，将感兴趣的账户替换为`*`user_name`*@*`host_name`*`：

    ```sql
    SELECT EVENT_SCHEMA, EVENT_NAME FROM INFORMATION_SCHEMA.EVENTS
    WHERE DEFINER = '*user_name*@*host_name*';
    SELECT ROUTINE_SCHEMA, ROUTINE_NAME, ROUTINE_TYPE
    FROM INFORMATION_SCHEMA.ROUTINES
    WHERE DEFINER = '*user_name*@*host_name*';
    SELECT TRIGGER_SCHEMA, TRIGGER_NAME FROM INFORMATION_SCHEMA.TRIGGERS
    WHERE DEFINER = '*user_name*@*host_name*';
    SELECT TABLE_SCHEMA, TABLE_NAME FROM INFORMATION_SCHEMA.VIEWS
    WHERE DEFINER = '*user_name*@*host_name*';
    ```

    对于`ROUTINES`表，查询包括`ROUTINE_TYPE`列，以便输出行区分`DEFINER`是存储过程还是存储函数。

    如果您正在搜索的账户不存在，则这些查询显示的任何对象都是孤立对象。

### 风险最小化准则

为了最大限度地减少存储对象创建和使用的风险潜力，请遵循以下准则：

+   不要创建孤立的存储对象；也就是说，`DEFINER`属性命名不存在的账户的对象。不要通过删除或重命名`DEFINER`属性命名的任何现有对象的账户来导致存储对象变为孤立。

+   对于存储过程或视图，在对象定义中尽可能使用`SQL SECURITY INVOKER`，以便它只能被具有适合对象执行操作权限的用户使用。

+   如果在具有`SET_USER_ID`权限（或已弃用的`SUPER`权限）的账户下创建定义者上下文存储对象，请指定一个显式的`DEFINER`属性，命名一个仅具有对象执行所需权限的账户。仅在绝对必要时指定高权限的`DEFINER`账户。

+   管理员可以通过不授予他们`SET_USER_ID`权限（或已弃用的`SUPER`权限）来防止用户创建指定高权限`DEFINER`账户的存储对象。

+   定义者上下文对象应该编写时考虑到它们可能能够访问调用用户没有权限的数据。在某些情况下，您可以通过不授予未经授权的用户特定权限来防止对这些对象的引用：

    +   未授予`EXECUTE`权限的用户无法引用存储过程。

    +   未授予适当权限的用户无法引用视图（需要`SELECT`从中选择，`INSERT`插入等）。

    然而，对于触发器和事件，不存在这样的控制，因为它们始终在定义者上下文中执行。服务器根据需要自动调用这些对象，用户不直接引用它们：

    +   触发器通过访问与其关联的表而被激活，即使是普通用户也可以访问没有特殊权限的表。

    +   事件由服务器定期执行。

    在这两种情况下，如果`DEFINER`账户权限很高，对象可能能够执行敏感或危险的操作。即使从创建对象所需的权限中撤销了创建者账户的权限，这仍然成立。管理员在授予用户对象创建权限时应格外小心。

+   默认情况下，当具有`SQL SECURITY DEFINER`特性的存储过程被执行时，MySQL 服务器不会为`DEFINER`子句中命名的 MySQL 账户设置任何活动角色，只有默认角色。例外情况是如果启用了`activate_all_roles_on_login`系统变量，此时 MySQL 服务器会设置所有授予`DEFINER`用户的角色，包括强制角色。因此，默认情况下，在发出`CREATE PROCEDURE`或`CREATE FUNCTION`语句时，不会检查通过角色授予的任何权限。对于存储程序，如果执行应该使用与默认不同的角色，则程序体可以执行`SET ROLE`来激活所需的角色。这必须谨慎进行，因为分配给角色的权限可能会更改。
