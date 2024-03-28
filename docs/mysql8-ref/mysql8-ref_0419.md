> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring-file-component.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-file-component.html)

#### 8.4.4.4 使用基于文件的密钥环组件

`component_keyring_file`密钥环组件将密钥环数据存储在服务器主机本地的文件中。

警告

对于加密密钥管理，`component_keyring_file`和`component_keyring_encrypted_file`组件，以及`keyring_file`和`keyring_encrypted_file`插件并不是用作符合法规的解决方案。PCI、FIPS 等安全标准要求使用密钥管理系统来保护、管理和保护密钥库或硬件安全模块（HSM）中的加密密钥。

要使用`component_keyring_file`进行密钥库管理，您必须：

1.  按照 8.4.4.2 节“密钥环组件安装”中描述的方式编写一个清单，告诉服务器加载`component_keyring_file`。

1.  按照这里的描述为`component_keyring_file`编写一个配置文件。

当初始化时，`component_keyring_file`会读取全局配置文件，或者全局配置文件与本地配置文件配对：

+   该组件尝试从组件库文件安装的目录（即服务器插件目录）中读取其全局配置文件。

+   如果全局配置文件指示使用本地配置文件，组件将尝试从数据目录中读取其本地配置文件。

+   尽管全局和本地配置文件位于不同的目录中，但文件名在两个位置都是`component_keyring_file.cnf`。

+   如果不存在配置文件，则无法初始化`component_keyring_file`。没有有效配置是一个错误。

本地配置文件允许设置多个服务器实例使用`component_keyring_file`，以便每个服务器实例的组件配置特定于给定的数据目录实例。这使得相同的密钥环组件可以与每个实例的不同数据文件一起使用。

`component_keyring_file`配置文件具有以下属性：

+   配置文件必须是有效的 JSON 格式。

+   配置文件允许设置这些配置项：

    +   `"read_local_config"`：此项仅允许在全局配置文件中。如果该项不存在，则组件仅使用全局配置文件。如果该项存在，则其值为`true`或`false`，指示组件是否应从本地配置文件中读取配置信息。

        如果全局配置文件中存在`"read_local_config"`项以及其他项，则组件首先检查`"read_local_config"`项的值：

        +   如果值为`false`，组件将处理全局配置文件中的其他项并忽略本地配置文件。

        +   如果数值为`true`，组件将忽略全局配置文件中的其他项，并尝试读取本地配置文件。

    +   `"path"`：项目值是一个字符串，用于命名用于存储密钥环数据的文件。文件应使用绝对路径而不是相对路径命名。此项目在配置中是强制性的。如果未指定，则`component_keyring_file`初始化失败。

    +   `"read_only"`：项目值指示密钥环数据文件是否为只读。项目值为`true`（只读）或`false`（读/写）。此项目在配置中是强制性的。如果未指定，则`component_keyring_file`初始化失败。

+   数据库管理员有责任创建要使用的任何配置文件，并确保其内容正确。如果发生错误，服务器启动将失败，并且管理员必须根据服务器错误日志中的诊断信息纠正任何问题。

鉴于前述配置文件属性，要配置`component_keyring_file`，请在安装了`component_keyring_file`库文件的目录中创建一个名为`component_keyring_file.cnf`的全局配置文件，并在数据目录中创建一个名为`component_keyring_file.cnf`的本地配置文件（可选）。以下说明假定要以读/写方式使用名为`/usr/local/mysql/keyring/component_keyring_file`的密钥环数据文件。

+   要仅使用全局配置文件，文件内容如下所示：

    ```sql
    {
      "path": "/usr/local/mysql/keyring/component_keyring_file",
      "read_only": false
    }
    ```

    在安装了`component_keyring_file`库文件的目录中创建此文件。

+   或者，要使用全局和本地配置文件对，全局文件如下所示：

    ```sql
    {
      "read_local_config": true
    }
    ```

    在安装了`component_keyring_file`库文件的目录中创建此文件。

    本地文件如下所示：

    ```sql
    {
      "path": "/usr/local/mysql/keyring/component_keyring_file",
      "read_only": false
    }
    ```

    在数据目录中创建此文件。

密钥环操作是事务性的：`component_keyring_file`在写操作期间使用备份文件，以确保如果操作失败，可以回滚到原始文件。备份文件与数据文件具有相同的名称，后缀为`.backup`。

`component_keyring_file`支持组成标准 MySQL 密钥环服务接口的函数。这些函数执行的密钥环操作可在两个级别访问：

+   SQL 接口：在 SQL 语句中，调用第 8.4.4.15 节，“通用密钥环密钥管理函数”中描述的函数。

+   C 接口：在 C 语言代码中，调用第 7.6.9.2 节，“密钥环服务”中描述的密钥环服务函数。

示例（使用 SQL 接口）：

```sql
SELECT keyring_key_generate('MyKey', 'AES', 32);
SELECT keyring_key_remove('MyKey');
```

有关`component_keyring_file`允许的关键值特征的信息，请参阅第 8.4.4.13 节，“支持的钥匙环键类型和长度”。
