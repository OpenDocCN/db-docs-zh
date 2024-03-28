> 原文：[`dev.mysql.com/doc/refman/8.0/en/validate-password-transitioning.html`](https://dev.mysql.com/doc/refman/8.0/en/validate-password-transitioning.html)

#### 8.4.3.3 过渡到密码验证组件

注意

在 MySQL 8.0 中，`validate_password`插件被重新实现为`validate_password`组件。`validate_password`插件已被弃用；预计将在 MySQL 的未来版本中删除。

目前使用`validate_password`插件的 MySQL 安装应该过渡到使用`validate_password`组件。为此，请使用以下过程。该过程在卸载插件之前安装组件，以避免出现没有密码验证发生的时间窗口。（组件和插件可以同时安装。在这种情况下，服务器尝试使用组件，如果组件不可用，则退回到插件。）

1.  安装`validate_password`组件：

    ```sql
    INSTALL COMPONENT 'file://component_validate_password';
    ```

1.  测试`validate_password`组件以确保其按预期工作。如果需要设置任何`validate_password.*`xxx`*`系统变量，可以使用`SET GLOBAL`在运行时执行。（必须进行的任何选项文件更改在下一步中执行。）

1.  调整任何引用插件系统和状态变量的引用，以引用相应的组件系统和状态变量。假设以前您使用类似于以下选项文件在启动时配置插件：

    ```sql
    [mysqld]
    validate-password=FORCE_PLUS_PERMANENT
    validate_password_dictionary_file=/usr/share/dict/words
    validate_password_length=10
    validate_password_number_count=2
    ```

    那些设置适用于插件，但必须修改以适用于组件。要调整选项文件，请省略`--validate-password`选项（仅适用于插件，而不是组件），并将系统变量引用从适用于插件的无点名称修改为适用于组件的有点名称：

    ```sql
    [mysqld]
    validate_password.dictionary_file=/usr/share/dict/words
    validate_password.length=10
    validate_password.number_count=2
    ```

    对于在运行时引用`validate_password`插件系统和状态变量的应用程序，需要进行类似的调整。将无点插件变量名称更改为相应的有点组件变量名称。

1.  卸载`validate_password`插件：

    ```sql
    UNINSTALL PLUGIN validate_password;
    ```

    如果在服务器启动时使用`--plugin-load`或`--plugin-load-add`选项加载`validate_password`插件，请从服务器启动过程中省略该选项。例如，如果该选项在服务器选项文件中列出，请从文件中删除。

1.  重新启动服务器。
