> 原文：[`dev.mysql.com/doc/refman/8.0/en/docker-mysql-more-topics.html`](https://dev.mysql.com/doc/refman/8.0/en/docker-mysql-more-topics.html)

#### 2.5.6.2 使用 Docker 部署 MySQL 服务器的更多主题

注意

以下大部分示例命令使用 `container-registry.oracle.com/mysql/community-server` 作为 Docker 镜像（比如 **docker pull** 和 **docker run** 命令）；如果你的镜像来自其他仓库，请进行更改，例如，用 `container-registry.oracle.com/mysql/enterprise-server` 替换从 Oracle Container Registry (OCR) 下载的 MySQL Enterprise Edition 镜像，或者用 `mysql/enterprise-server` 替换从 [My Oracle Support](https://support.oracle.com/) 下载的 MySQL Enterprise Edition 镜像。

+   为 Docker 优化的 MySQL 安装

+   配置 MySQL 服务器

+   持久化数据和配置更改

+   运行额外的初始化脚本

+   从另一个 Docker 容器中的应用连接到 MySQL

+   服务器错误日志

+   在 Docker 中使用 MySQL Enterprise Backup

+   在 Docker 中使用 mysqldump

+   已知问题

+   Docker 环境变量

##### 为 Docker 优化的 MySQL 安装

MySQL 的 Docker 镜像经过了代码大小的优化，这意味着它们只包含了对于在 Docker 容器中运行 MySQL 实例的大多数用户来说是相关的关键组件。MySQL Docker 安装与常规的非 Docker 安装在以下方面有所不同：

+   只包含有限数量的二进制文件。

+   所有二进制文件都经过了剥离；它们不包含调试信息。

警告

用户对 Docker 容器执行的任何软件更新或安装（包括 MySQL 组件）可能与 Docker 镜像创建的优化 MySQL 安装发生冲突。Oracle 不支持在这种被更改的容器中运行的 MySQL 产品，或者从被更改的 Docker 镜像创建的容器。

##### 配置 MySQL 服务器

当您启动 MySQL Docker 容器时，可以通过 **docker run** 命令向服务器传递配置选项。例如：

```sql
docker run --name mysql1 -d container-registry.oracle.com/mysql/community-server:*tag* --character-set-server=utf8mb4 --collation-server=utf8mb4_col
```

该命令以 `utf8mb4` 作为默认字符集和 `utf8mb4_col` 作为数据库的默认排序规则启动 MySQL 服务器。

配置 MySQL 服务器的另一种方法是准备一个配置文件，并将其挂载到容器内部服务器配置文件的位置。详细信息请参见 持久化数据和配置更改。

##### 持久化数据和配置更改

Docker 容器原则上是短暂的，如果容器被删除或损坏，则预计任何数据或配置都将丢失（请参见[此处](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)的讨论）。[Docker 卷](https://docs.docker.com/engine/admin/volumes/volumes/) 提供了一种机制来持久化在 Docker 容器内创建的数据。在初始化时，MySQL 服务器容器为服务器数据目录创建一个 Docker 卷。容器上的 **docker inspect** 命令的 JSON 输出包含一个 `Mount` 键，其值提供有关数据目录卷的信息：

```sql
$> docker inspect mysql1
...
 "Mounts": [
 {
                "Type": "volume",
                "Name": "4f2d463cfc4bdd4baebcb098c97d7da3337195ed2c6572bc0b89f7e845d27652",
                "Source": "/var/lib/docker/volumes/4f2d463cfc4bdd4baebcb098c97d7da3337195ed2c6572bc0b89f7e845d27652/_data",
                "Destination": "/var/lib/mysql",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
...
```

输出显示，源目录 `/var/lib/docker/volumes/4f2d463cfc4bdd4baebcb098c97d7da3337195ed2c6572bc0b89f7e845d27652/_data`，其中数据在主机上持久化，已挂载到容器内部的 `/var/lib/mysql`，即服务器数据目录。

保留数据的另一种方法是在创建容器时使用 `--mount` 选项 [绑定挂载](https://docs.docker.com/engine/reference/commandline/service_create/#add-bind-mounts-volumes-or-memory-filesystems) 主机目录。相同的技术可以用于持久化服务器的配置。以下命令创建一个 MySQL 服务器容器，并同时绑定挂载数据目录和服务器配置文件：

```sql
docker run --name=mysql1 \
--mount type=bind,src=*/path-on-host-machine/my.cnf*,dst=/etc/my.cnf \
--mount type=bind,src=*/path-on-host-machine/datadir*,dst=/var/lib/mysql \
-d container-registry.oracle.com/mysql/community-server:*tag*
```

该命令将 `*`path-on-host-machine/my.cnf`*` 挂载到 ``/etc/my.cnf``（容器内部的服务器配置文件），并将 `*`path-on-host-machine/datadir`*` 挂载到 `/var/lib/mysql`（容器内部的数据目录）。绑定挂载工作的前提条件如下：

+   配置文件 `*`path-on-host-machine/my.cnf`*` 必须已经存在，并且必须包含由用户 `mysql` 启动服务器的规范：

    ```sql
    [mysqld]
    user=mysql
    ```

    您还可以在文件中包含其他服务器配置选项。

+   数据目录 `*`path-on-host-machine/datadir`*` 必须已经存在。为了进行服务器初始化，目录必须为空。您还可以挂载一个预先填充了数据的目录并启动服务器；但是，您必须确保使用与创建数据的服务器相同的配置启动 Docker 容器，并在启动容器时挂载任何所需的主机文件或目录。

##### 运行额外的初始化脚本

如果有任何`.sh`或`.sql`脚本需要在数据库创建后立即运行，您可以将它们放入主机目录，然后将该目录挂载到容器内的`/docker-entrypoint-initdb.d/`。例如：

```sql
docker run --name=mysql1 \
--mount type=bind,src=*/path-on-host-machine/scripts/*,dst=/docker-entrypoint-initdb.d/ \
-d container-registry.oracle.com/mysql/community-server:*tag*
```

##### 从另一个 Docker 容器中的应用程序连接到 MySQL

通过设置一个 Docker 网络，您可以允许多个 Docker 容器相互通信，以便另一个 Docker 容器中的客户端应用程序可以访问服务器容器中的 MySQL 服务器。首先，创建一个 Docker 网络：

```sql
docker network create *my-custom-net*
```

然后，在创建和启动服务器和客户端容器时，使用`--network`选项将它们放在您创建的网络上。例如：

```sql
docker run --name=mysql1 --network=*my-custom-net* -d container-registry.oracle.com/mysql/community-server
```

```sql
docker run --name=myapp1 --network=*my-custom-net* -d myapp
```

`myapp1`容器可以通过`mysql1`主机名连接到`mysql1`容器，反之亦然，因为 Docker 会自动为给定的容器名称设置 DNS。在以下示例中，我们从`myapp1`容器内部运行**mysql**客户端，以连接到其自己容器中的主机`mysql1`：

```sql
docker exec -it myapp1 mysql --host=mysql1 --user=myuser --password
```

有关容器的其他网络技术，请参阅 Docker 文档中的[Docker 容器网络](https://docs.docker.com/engine/userguide/networking/)部分。

##### 服务器错误日志

当首次使用服务器容器启动 MySQL 服务器时，如果以下条件之一为真，则不会生成服务器错误日志：

+   从主机挂载了一个服务器配置文件，但该文件不包含系统变量`log_error`（参见持久化数据和配置更改关于绑定挂载服务器配置文件）。

+   未从主机挂载服务器配置文件，但 Docker 环境变量`MYSQL_LOG_CONSOLE`为`true`（这是 MySQL 8.0 服务器容器的默认状态）。然后，MySQL 服务器的错误日志被重定向到`stderr`，因此错误日志进入 Docker 容器的日志，并可以使用**docker logs *`mysqld-container`***命令查看。

要使 MySQL Server 在以下两种情况之一为真时生成错误日志，请使用`--log-error`选项来配置服务器以在容器内的特定位置生成错误日志。要持久化错误日志，请按照持久化数据和配置更改中的说明，在容器内的错误日志位置挂载主机文件。但是，您必须确保 MySQL Server 在其容器内具有对挂载主机文件的写访问权限。

##### 使用 Docker 的 MySQL 企业备份

MySQL 企业备份是 MySQL Server 的商业许可备份实用程序，可在[MySQL 企业版](https://www.mysql.com/products/enterprise/)中使用。MySQL 企业备份包含在 MySQL 企业版的 Docker 安装中。

在以下示例中，我们假设您已经在 Docker 容器中运行了一个 MySQL Server（请参阅 Section 2.5.6.1，“使用 Docker 部署 MySQL Server 的基本步骤”以了解如何使用 Docker 启动 MySQL Server 实例）。为了让 MySQL 企业备份备份 MySQL Server，它必须能够访问服务器的数据目录。例如，可以通过在启动服务器时将主机目录绑定到 MySQL Server 的数据目录来实现这一点：

```sql
docker run --name=mysqlserver \
--mount type=bind,src=*/path-on-host-machine/datadir/*,dst=/var/lib/mysql \
-d mysql/enterprise-server:8.0
```

使用此命令，MySQL Server 将使用 MySQL 企业版的 Docker 镜像启动，并且主机目录*`/path-on-host-machine/datadir/`*已挂载到服务器容器内的数据目录(`/var/lib/mysql`)。我们还假设，在服务器启动后，已为 MySQL 企业备份设置了访问服务器所需权限（有关详细信息，请参见授予 MySQL 备份管理员权限）。按照以下步骤备份和恢复 MySQL Server 实例。

要使用 Docker 的 MySQL 企业备份备份在 Docker 容器中运行的 MySQL Server 实例，请按照这里列出的步骤操作：

1.  在运行 MySQL 服务器容器的同一主机上，使用 MySQL 企业版镜像启动另一个容器，使用 MySQL 企业版备份命令 `backup-to-image` 执行备份。使用我们在上一步创建的绑定挂载访问服务器的数据目录。此外，将主机目录（*在此示例中为 `/path-on-host-machine/backups/`*）挂载到容器中用于存储备份的存储文件夹（在示例中为 `/data/backups`）以持久保存我们正在创建的备份。以下是执行此步骤的示例命令，其中使用从 [My Oracle Support](https://support.oracle.com/) 下载的 Docker 镜像启动 MySQL 企业版备份：

    ```sql
    $> docker run \
    --mount type=bind,src=*/path-on-host-machine/datadir/*,dst=/var/lib/mysql \
    --mount type=bind,src=*/path-on-host-machine/backups/*,dst=/data/backups \
    --rm mysql/enterprise-server:8.0 \
    mysqlbackup -u*mysqlbackup* -p*password* --backup-dir=/tmp/backup-tmp --with-timestamp \
    --backup-image=/data/backups/db.mbi backup-to-image

    [Entrypoint] MySQL Docker Image 8.0.11-1.1.5
    MySQL Enterprise Backup version 8.0.11 Linux-4.1.12-61.1.16.el7uek.x86_64-x86_64 [2018-04-08  07:06:45]
    Copyright (c) 2003, 2018, Oracle and/or its affiliates. All Rights Reserved.

    180921 17:27:25 MAIN    INFO: A thread created with Id '140594390935680'
    180921 17:27:25 MAIN    INFO: Starting with following command line ...
    ...

    -------------------------------------------------------------
       Parameters Summary
    -------------------------------------------------------------
       Start LSN                  : 29615616
       End LSN                    : 29651854
    -------------------------------------------------------------

    mysqlbackup completed OK!
    ```

    重要的是通过 **mysqlbackup** 的输出末尾检查备份是否已成功完成。

1.  一旦备份作业完成，容器就会退出，并且使用 `--rm` 选项启动时，容器在退出后会被删除。已经创建了一个镜像备份，并且可以在最后一步挂载备份存储的主机目录中找到，如下所示：

    ```sql
    $> ls /tmp/backups
    db.mbi
    ```

要在 Docker 容器中使用 MySQL 企业版备份还原 MySQL 服务器实例，请按照这里列出的步骤进行操作：

1.  停止 MySQL 服务器容器，这也会停止运行在其中的 MySQL 服务器：

    ```sql
    docker stop mysqlserver
    ```

1.  在主机上，删除 MySQL 服务器数据目录的绑定挂载中的所有内容：

    ```sql
    rm -rf */path-on-host-machine/datadir*/*
    ```

1.  使用 MySQL 企业版镜像启动一个容器，使用 MySQL 企业版备份命令 `copy-back-and-apply-log` 执行还原操作。像我们备份服务器时所做的那样，将服务器的数据目录和备份存储文件夹进行绑定挂载：

    ```sql
    $> docker run \
    --mount type=bind,src=*/path-on-host-machine/datadir/*,dst=/var/lib/mysql \
    --mount type=bind,src=*/path-on-host-machine/backups/*,dst=/data/backups \
    --rm mysql/enterprise-server:8.0 \
    mysqlbackup --backup-dir=/tmp/backup-tmp --with-timestamp \
    --datadir=/var/lib/mysql --backup-image=/data/backups/db.mbi copy-back-and-apply-log

    [Entrypoint] MySQL Docker Image 8.0.11-1.1.5
    MySQL Enterprise Backup version 8.0.11 Linux-4.1.12-61.1.16.el7uek.x86_64-x86_64 [2018-04-08  07:06:45]
    Copyright (c) 2003, 2018, Oracle and/or its affiliates. All Rights Reserved.

    180921 22:06:52 MAIN    INFO: A thread created with Id '139768047519872'
    180921 22:06:52 MAIN    INFO: Starting with following command line ...
    ...
    180921 22:06:52 PCR1    INFO: We were able to parse ibbackup_logfile up to
              lsn 29680612.
    180921 22:06:52 PCR1    INFO: Last MySQL binlog file position 0 155, file name binlog.000003
    180921 22:06:52 PCR1    INFO: The first data file is '/var/lib/mysql/ibdata1'
                                  and the new created log files are at '/var/lib/mysql'
    180921 22:06:52 MAIN    INFO: No Keyring file to process.
    180921 22:06:52 MAIN    INFO: Apply-log operation completed successfully.
    180921 22:06:52 MAIN    INFO: Full Backup has been restored successfully.

    mysqlbackup completed OK! with 3 warnings
    ```

    一旦备份作业完成，容器就会退出，并且使用 `--rm` 选项启动时，容器在退出后会被删除。

1.  使用以下命令重新启动服务器容器，这也会重新启动已还原的服务器：

    ```sql
    docker restart mysqlserver
    ```

    或者，在还原的数据目录上启动一个新的 MySQL 服务器，如下所示：

    ```sql
    docker run --name=mysqlserver2 \
    --mount type=bind,src=*/path-on-host-machine/datadir/*,dst=/var/lib/mysql \
    -d mysql/enterprise-server:8.0
    ```

    登录服务器检查服务器是否正在运行并使用还原的数据。

##### 使用 **mysqldump** 与 Docker

除了在 Docker 容器中运行的 MySQL 服务器使用 MySQL 企业版备份备份外，您还可以通过在 Docker 容器中运行的 **mysqldump** 实用程序执行逻辑备份。

以下说明假定您已经在 Docker 容器中运行了一个 MySQL 服务器，并且在容器首次启动时，已经将主机目录*`/path-on-host-machine/datadir/`*挂载到服务器的数据目录`/var/lib/mysql`上（有关详细信息，请参见将主机目录绑定到 MySQL 服务器数据目录上），该目录包含 Unix 套接字文件，通过该文件**mysqldump**和**mysql**可以连接到服务器。我们还假设，在服务器启动后，已经创建了一个具有适当权限（在本例中为`admin`）的用户，**mysqldump**可以使用该用户访问服务器。请使用以下步骤备份和恢复 MySQL 服务器数据：

*使用 Docker 使用**mysqldump**备份 MySQL 服务器数据*：

1.  在运行 MySQL 服务器容器的同一主机上，启动另一个容器，其中包含 MySQL 服务器的镜像，以使用**mysqldump**实用程序执行备份（请参阅该实用程序的文档以了解其功能、选项和限制）。通过绑定挂载*`/path-on-host-machine/datadir/`*提供对服务器数据目录的访问。此外，将主机目录（在本例中为*`/path-on-host-machine/backups/`*）挂载到容器内用于存储备份的存储文件夹（在本例中使用`/data/backups`）以持久保存您创建的备份。以下是使用此设置备份服务器上所有数据库的示例命令：

    ```sql
    $> docker run --entrypoint "/bin/sh" \ 
    --mount type=bind,src=*/path-on-host-machine/datadir/*,dst=/var/lib/mysql \
    --mount type=bind,src=*/path-on-host-machine/backups/*,dst=/data/backups \
    --rm container-registry.oracle.com/mysql/community-server:8.0 \
    -c "mysqldump -u*admin* --password='*password*' --all-databases > /data/backups/all-databases.sql"
    ```

    在命令中，使用`--entrypoint`选项启动容器后调用系统 shell，并使用`-c`选项指定要在 shell 中运行的**mysqldump**命令，其输出被重定向到备份目录中的文件`all-databases.sql`。

1.  一旦备份作业完成，容器就会退出，并且使用`--rm`选项启动后，容器在退出后将被删除。已创建了一个逻辑备份，并且可以在用于存储备份的主机目录中找到，如下所示：

    ```sql
    $> ls */path-on-host-machine/backups/*
    all-databases.sql
    ```

*使用 Docker 使用**mysqldump**恢复 MySQL 服务器数据*：

1.  确保您在容器中运行了一个 MySQL 服务器，您希望将备份数据还原到该服务器上。

1.  启动一个包含 MySQL 服务器镜像的容器，使用**mysql**客户端执行恢复操作。绑定挂载服务器的数据目录，以及包含您备份的存储文件夹：

    ```sql
    $> docker run  \
    --mount type=bind,src=*/path-on-host-machine/datadir/*,dst=/var/lib/mysql \
    --mount type=bind,src=*/path-on-host-machine/backups/*,dst=/data/backups \
    --rm container-registry.oracle.com/mysql/community-server:8.0 \
    mysql -u*admin* --password='*password*' -e "source /data/backups/all-databases.sql"
    ```

    一旦备份作业完成，容器就会退出，并且在启动时使用`--rm`选项后，容器在退出后会被删除。

1.  登录服务器检查已恢复的数据是否已在服务器上。

##### 已知问题

+   当使用服务器系统变量`audit_log_file`来配置审计日志文件名时，请使用`loose` 选项修饰符；否则，Docker 无法启动服务器。

##### Docker 环境变量

当创建 MySQL 服务器容器时，可以使用`--env`选项（简写为`-e`）并指定一个或多个环境变量来配置 MySQL 实例。如果挂载的数据目录不为空，则不会执行服务器初始化，在这种情况下设置这些变量之一也没有效果（参见持久化数据和配置更改），并且在容器启动期间不会修改目录的任何现有内容，包括服务器设置。

可用于配置 MySQL 实例的环境变量列在这里：

+   包括`MYSQL_RANDOM_ROOT_PASSWORD`、`MYSQL_ONETIME_PASSWORD`、`MYSQL_ALLOW_EMPTY_PASSWORD`和`MYSQL_LOG_CONSOLE`在内的布尔变量通过将它们设置为任何非零长度的字符串来设为真。因此，将它们设置为“0”、“false”或“no”并不会使它们为假，而实际上会使它们为真。这是一个已知问题。

+   `MYSQL_RANDOM_ROOT_PASSWORD`：当此变量为真时（这是其默认状态，除非设置了`MYSQL_ROOT_PASSWORD`或`MYSQL_ALLOW_EMPTY_PASSWORD`为真），在启动 Docker 容器时会为服务器的 root 用户生成一个随机密码。密码会打印到容器的`stdout`中，并且可以通过查看容器的日志（参见启动 MySQL 服务器实例）找到。

+   `MYSQL_ONETIME_PASSWORD`: 当该变量为 true（这是默认状态，除非设置了`MYSQL_ROOT_PASSWORD`或`MYSQL_ALLOW_EMPTY_PASSWORD`为 true），root 用户的密码将被设置为过期，必须在 MySQL 可以正常使用之前更改。

+   `MYSQL_DATABASE`: 此变量允许您在镜像启动时指定要创建的数据库的名称。如果使用`MYSQL_USER`和`MYSQL_PASSWORD`提供了用户名和密码，则将创建用户并授予该数据库的超级用户访问权限（对应于`GRANT ALL`）。指定的数据库是通过 CREATE DATABASE IF NOT EXIST 语句创建的，因此如果数据库已经存在，则该变量不起作用。

+   `MYSQL_USER`, `MYSQL_PASSWORD`: 这些变量一起用于创建用户并设置该用户的密码，用户被授予指定由`MYSQL_DATABASE`变量的数据库的超级用户权限。要创建用户，`MYSQL_USER`和`MYSQL_PASSWORD`都是必需的 - 如果两个变量中的任何一个未设置，则另一个将被忽略。如果两个变量都设置了但`MYSQL_DATABASE`没有设置，则创建用户时不授予任何权限。

    注意

    不需要使用此机制来创建默认情况下使用`MYSQL_ROOT_PASSWORD`和`MYSQL_RANDOM_ROOT_PASSWORD`中讨论的任一机制设置密码的根超级用户，除非`MYSQL_ALLOW_EMPTY_PASSWORD`为 true。

+   `MYSQL_ROOT_HOST`：默认情况下，MySQL 创建`'root'@'localhost'`帐户。此帐户只能从容器内部连接，如从容器内部连接到 MySQL 服务器中所述。要允许来自其他主机的 root 连接，请设置此环境变量。例如，值`172.17.0.1`，这是默认的 Docker 网关 IP，允许来自运行容器的主机机器的连接。该选项只接受一个条目，但允许使用通配符（例如，`MYSQL_ROOT_HOST=172.*.*.*`或`MYSQL_ROOT_HOST=%`）。

+   `MYSQL_LOG_CONSOLE`：当变量为 true（对于 MySQL 8.0 服务器容器而言，这是其默认状态）时，MySQL 服务器的错误日志被重定向到`stderr`，因此错误日志进入 Docker 容器的日志，并且可以使用**docker logs *`mysqld-container`***命令查看。

    注意

    如果主机已挂载服务器配置文件（参见 Persisting Data and Configuration Changes 中的绑定挂载配置文件），则该变量不起作用。

+   `MYSQL_ROOT_PASSWORD`：此变量指定为 MySQL root 帐户设置的密码。

    警告

    在命令行上设置 MySQL root 用户密码是不安全的。作为明确指定密码的替代方案，您可以设置一个容器文件路径的变量，用于密码文件，然后挂载来自主机的包含密码的文件到容器文件路径。这仍然不是非常安全的，因为密码文件的位置仍然暴露。最好使用`MYSQL_RANDOM_ROOT_PASSWORD`和`MYSQL_ONETIME_PASSWORD`的默认设置，两者都设置为 true。

+   `MYSQL_ALLOW_EMPTY_PASSWORD`。将其设置为 true 以允许使用空密码启动容器以用于 root 用户。

    警告

    将此变量设置为 true 是不安全的，因为它将使您的 MySQL 实例完全无保护，允许任何人获得完全的超级用户访问权限。最好使用`MYSQL_RANDOM_ROOT_PASSWORD`和`MYSQL_ONETIME_PASSWORD`的默认设置，两者都设置为 true。
