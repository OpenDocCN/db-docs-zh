> 原文：[`dev.mysql.com/doc/refman/8.0/en/clone-plugin-directories.html`](https://dev.mysql.com/doc/refman/8.0/en/clone-plugin-directories.html)

#### 7.6.7.8 克隆操作期间创建的目录和文件

当数据被克隆时，以下目录和文件会被创建用于内部使用。不应该被修改。

+   `#clone`：包含克隆操作使用的内部克隆文件。在数据被克隆到的目录中创建。

+   `#ib_archive`：包含在克隆操作期间在捐赠者上归档的内部归档日志文件。

+   `*.#clone` 文件：在接收端创建的临时数据文件，当数据从接收端数据目录中移除并在远程克隆操作期间克隆新数据时创建。
