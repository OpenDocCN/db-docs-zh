# 2.8.5 使用开发源树安装 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/installing-development-tree.html`](https://dev.mysql.com/doc/refman/8.0/en/installing-development-tree.html)

本节描述了如何从托管在 [GitHub](https://github.com/) 上的最新开发源代码安装 MySQL。要从这个仓库托管服务获取 MySQL Server 源代码，您可以设置一个本地 MySQL Git 仓库。

在 [GitHub](https://github.com/) 上，MySQL Server 和其他 MySQL 项目可以在 [MySQL](https://github.com/mysql) 页面找到。MySQL Server 项目是一个包含多个 MySQL 系列分支的单个仓库。

+   从开发源安装的先决条件

+   设置 MySQL Git 仓库

#### 从开发源安装的先决条件

要从开发源树安装 MySQL，您的系统必须满足第 2.8.2 节，“源码安装先决条件”中列出的工具要求。

#### 设置 MySQL Git 仓库

要在您的计算机上设置一个 MySQL Git 仓库：

1.  将 MySQL Git 仓库克隆到您的计算机上。以下命令将 MySQL Git 仓库克隆到一个名为 `mysql-server` 的目录中。初始下载可能需要一些时间才能完成，这取决于您的连接速度。

    ```sql
    $> git clone https://github.com/mysql/mysql-server.git
    Cloning into 'mysql-server'...
    remote: Counting objects: 1198513, done.
    remote: Total 1198513 (delta 0), reused 0 (delta 0), pack-reused 1198513
    Receiving objects: 100% (1198513/1198513), 1.01 GiB | 7.44 MiB/s, done.
    Resolving deltas: 100% (993200/993200), done.
    Checking connectivity... done.
    Checking out files: 100% (25510/25510), done.
    ```

1.  克隆操作完成后，您本地的 MySQL Git 仓库的内容会类似于以下内容：

    ```sql
    ~> cd mysql-server
    ~/mysql-server> ls
    client             extra                mysys              storage
    cmake              include              packaging          strings
    CMakeLists.txt     INSTALL              plugin             support-files
    components         libbinlogevents      README             testclients
    config.h.cmake     libchangestreams     router             unittest
    configure.cmake    libmysql             run_doxygen.cmake  utilities
    Docs               libservices          scripts            VERSION
    Doxyfile-ignored   LICENSE              share              vio
    Doxyfile.in        man                  sql                win
    doxygen_resources  mysql-test           sql-common
    ```

1.  使用 **git branch -r** 命令查看 MySQL 仓库的远程跟踪分支。

    ```sql
    ~/mysql-server> git branch -r
      origin/5.7
      origin/8.0
      origin/HEAD -> origin/trunk
      origin/cluster-7.4
      origin/cluster-7.5
      origin/cluster-7.6
      origin/trunk
    ```

1.  要查看本地仓库中检出的分支，请发出 **git branch** 命令。当您克隆 MySQL Git 仓库时，最新的 MySQL 分支会自动检出。星号标识活动分支。

    ```sql
    ~/mysql-server$ git branch
    * trunk
    ```

1.  要检出较早的 MySQL 分支，请运行 **git checkout** 命令，并指定分支名称。例如，要检出 MySQL 5.7 分支：

    ```sql
    ~/mysql-server$ git checkout 5.7
    Checking out files: 100% (9600/9600), done.
    Branch 5.7 set up to track remote branch 5.7 from origin.
    Switched to a new branch '5.7'
    ```

1.  要获取在初始设置 MySQL Git 仓库后进行的更改，请切换到要更新的分支并发出 **git pull** 命令：

    ```sql
    ~/mysql-server$ git checkout 8.0
    ~/mysql-server$ git pull
    ```

    要查看提交历史，请使用 **git log** 命令：

    ```sql
    ~/mysql-server$ git log
    ```

    您还可以在 GitHub 的 [MySQL](https://github.com/mysql) 网站上浏览提交历史和源代码。

    如果您看到有变化或代码有疑问的地方，请在 [MySQL Community Slack](https://mysqlcommunity.slack.com/) 上提问。

1.  在克隆了 MySQL Git 仓库并检出了要构建的分支后，您可以从源代码构建 MySQL 服务器。有关说明，请参阅第 2.8.4 节，“使用标准源分发安装 MySQL”，不过您可以跳过获取和解压分发的部分。

    在生产机器上小心安装来自分发源代码树的构建。安装命令可能会覆盖您的实时发布安装。如果您已经安装了 MySQL 并且不想覆盖它，请使用与生产服务器不同的值运行**CMake**，分别为`CMAKE_INSTALL_PREFIX`、`MYSQL_TCP_PORT`和`MYSQL_UNIX_ADDR`选项。有关防止多个服务器相互干扰的其他信息，请参见第 7.8 节，“在一台机器上运行多个 MySQL 实例”。

    通过您的新安装来进行严格测试。例如，尝试让新功能崩溃。首先运行**make test**。请参阅 MySQL 测试套件。
