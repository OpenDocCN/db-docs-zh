> 译文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-tde-setup.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-tde-setup.html)

#### 25.6.14.1 NDB 文件系统加密设置和使用

*文件系统加密*：要启用先前未加密的文件系统的加密，需要执行以下步骤：

1.  在`config.ini`文件的`[ndbd default]`部分中设置所需的数据节点参数，如下所示：

    ```sql
    [ndbd default]
    EncryptedFileSystem= 1
    ```

    这些参数必须如所有数据节点所示设置。

1.  使用`--initial`或`--reload`启动管理服务器，以使其读取更新后的配置文件。

1.  执行所有数据节点的滚动初始启动（或重新启动）（参见第 25.6.5 节，“执行 NDB 集群的滚动重启”启动每个数据节点；此外，对每个数据节点进程提供`--filesystem-password`或`--filesystem-password-from-stdin`中的任一选项，以及密码。当您在命令行上提供密码时，会显示警告，类似于这样：

    ```sql
    > ndbmtd -c 127.0.0.1 --filesystem-password=ndbsecret
    ndbmtd: [Warning] Using a password on the command line interface can be insecure.
    2022-08-22 16:17:58 [ndbd] INFO     -- Angel connected to '127.0.0.1:1186'
    2022-08-22 16:17:58 [ndbd] INFO     -- Angel allocated nodeid: 5
    ```

    `--filesystem-password`可以接受来自文件、`tty`或`stdin`的密码；`--filesystem-password-from-stdin`仅接受来自`stdin`的密码。后者保护密码免受在进程命令行或文件系统中暴露，并允许从另一个安全应用程序传递密码的可能性。

    您还可以将密码放在一个`my.cnf`文件中，该文件可以被数据节点进程读取，但不能被系统的其他用户读取。使用与前面示例中相同的密码，文件的相关部分应如下所示：

    ```sql
    [ndbd]

    filesystem-password=ndbsecret
    ```

    您还可以在`my.cnf`文件中使用`--filesystem-password-from-stdin`选项提示启动数据节点进程的用户在启动时提供加密密码，如下所示：

    ```sql
    [ndbd]

    filesystem-password-from-stdin
    ```

    在这种情况下，当启动数据节点进程时，用户会被提示输入密码，如下所示：

    ```sql
    > ndbmtd -c 127.0.0.1 
    Enter filesystem password: *********
    2022-08-22 16:36:00 [ndbd] INFO     -- Angel connected to '127.0.0.1:1186'
    2022-08-22 16:36:00 [ndbd] INFO     -- Angel allocated nodeid: 5
    >
    ```

    无论使用何种方法，加密密码的格式与用于加密备份密码的格式相同（参见第 25.6.8.2 节，“使用 NDB 集群管理客户端创建备份”或`--filesystem-password-from-stdin`。

*文件系统解密*：要从加密文件系统中删除加密，请执行以下操作：

1.  在`config.ini`文件的`[ndbd default]`部分中，将`EncryptedFileSystem = OFF`设置为关闭。

1.  使用`--initial`或`--reload`重新启动管理服务器。

1.  执行数据节点的滚动初始重启。在重新启动节点二进制文件时，*不要*使用任何与密码相关的选项。

    重新启动时，每个数据节点都会清除其磁盘上的状态，并以未加密形式重建。

要查看文件系统加密是否正确配置，可以使用针对`ndbinfo` `config_values`和`config_params`表的查询，类似于这样：

```sql
mysql> SELECT v.node_id AS Node, p.param_name AS Parameter, v.config_value AS Value
 ->    FROM ndbinfo.config_values v
 ->  JOIN ndbinfo.config_params p      
 ->    ON v.config_param=p.param_number
 ->  WHERE p.param_name='EncryptedFileSystem';
+------+----------------------+-------+
| Node | Parameter            | Value |
+------+----------------------+-------+
|    5 | EncryptedFileSystem  | 1     |
|    6 | EncryptedFileSystem  | 1     |
|    7 | EncryptedFileSystem  | 1     |
|    8 | EncryptedFileSystem  | 1     |
+------+----------------------+-------+
4 rows in set (0.10 sec)
```

在这里，`EncryptedFileSystem`在所有数据节点上都等于`1`，这意味着文件系统加密已启用。
