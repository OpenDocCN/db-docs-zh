> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-masking-components-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/data-masking-components-installation.html)

#### 8.5.2.1 MySQL 企业数据脱敏和去标识化组件安装

从 MySQL 8.0.33 开始，组件提供了访问 MySQL 企业数据脱敏和去标识化功能的功能。以前，MySQL 将脱敏和去标识化功能实现为包含插件和几个可加载函数的插件库文件。在开始组件安装之前，请删除`data_masking`插件及其所有可加载函数，以避免冲突。有关说明，请参见第 8.5.3.1 节，“MySQL 企业数据脱敏和去标识化插件安装”。

MySQL 企业数据脱敏和去标识化数据库表和组件包括：

+   `masking_dictionaries` 表

    目的：`mysql`系统模式中的表，提供字典和术语的持久存储。

+   `component_masking` 组件

    目的：该组件实现了脱敏功能的核心，并将其作为服务公开。

    URN：`file://component_masking`

+   `component_masking_functions` 组件

    目的：该组件将`component_masking`组件的所有功能作为可加载函数公开。其中一些函数需要`MASKING_DICTIONARIES_ADMIN`动态权限。

    URN：`file://component_masking_functions`

要设置 MySQL 企业数据脱敏和去标识化，请执行以下操作：

1.  创建`masking_dictionaries`表。

    ```sql
    CREATE TABLE IF NOT EXISTS
    mysql.masking_dictionaries(
        Dictionary VARCHAR(256) NOT NULL,
        Term VARCHAR(256) NOT NULL,
        UNIQUE INDEX dictionary_term_idx (Dictionary, Term),
        INDEX dictionary_idx (Dictionary)
    ) ENGINE = InnoDB DEFAULT CHARSET=utf8mb4;
    ```

1.  使用`INSTALL COMPONENT` SQL 语句来安装数据脱敏组件。

    ```sql
    INSTALL COMPONENT 'file://component_masking';
    INSTALL COMPONENT 'file://component_masking_functions';
    ```

    如果在复制源服务器上使用组件和函数，请在所有副本服务器上安装它们，以避免复制问题。在加载组件时，有关它们的信息可如第 7.5.2 节，“获取组件信息”所述获得。

要移除 MySQL 企业数据脱敏和去标识化，请执行以下操作：

1.  使用`UNINSTALL COMPONENT` SQL 语句来卸载数据脱敏组件。

    ```sql
    UNINSTALL COMPONENT 'file://component_masking';
    UNINSTALL COMPONENT 'file://component_masking_functions';
    ```

1.  删除`masking_dictionaries`表。

    ```sql
    DROP TABLE mysql.masking_dictionaries;
    ```

`component_masking_functions` 会自动安装所有相关的可加载函数。同样，当卸载该组件时，也会自动卸载这些函数。有关安装或卸载组件的一般信息，请参见第 7.5.1 节，“安装和卸载组件”。
