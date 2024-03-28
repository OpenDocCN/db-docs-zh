> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-docker.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-docker.html)

#### 25.3.1.5 使用 Docker 容器部署 NDB 集群

##### 下载 MySQL NDB 集群 Docker 镜像

在单独的步骤中下载 Docker 镜像并不是绝对必要的；但是，在创建 Docker 容器之前执行此步骤可以确保您的本地镜像是最新的。要从[Oracle 容器注册表（OCR）](https://container-registry.oracle.com/)下载 MySQL NDB 集群社区版镜像，请运行此命令：

```sql
docker pull container-registry.oracle.com/mysql/community-cluster:*tag*
```

*`tag`*是您要拉取的镜像版本的标签（例如，`7.5`，`7.6`，`8.0`或`latest`）。如果省略了**`:*`tag`*`**，则使用`latest`标签，并下载 MySQL NDB 集群最新 GA 版本的镜像。

您可以使用此命令列出已下载的 Docker 镜像：

```sql
$> docker images
REPOSITORY                                              TAG       IMAGE ID       CREATED        SIZE
container-registry.oracle.com/mysql/community-cluster   8.0       d1b28e457ac5   5 weeks ago    636MB
```

要从 OCR 下载 MySQL 商业集群镜像，您需要首先接受许可协议。请按照以下步骤操作：

+   访问 OCR 网站[`container-registry.oracle.com/`](https://container-registry.oracle.com/)并选择 MySQL。

+   在 MySQL 存储库列表下，选择`commercial-cluster`。

+   如果您尚未登录到 OCR，请单击页面右侧的“登录”按钮，然后在提示时输入 Oracle 帐户凭据。

+   跟随页面右侧的说明接受许可协议。

使用此命令从 OCR 下载 MySQL 商业集群的 Docker 镜像：

```sql
docker pull  container-registry.oracle.com/mysql/commercial-cluster:*tag*
```

##### 使用默认配置启动 MySQL 集群

首先，为容器之间的通信创建一个名为`cluster`的内部 Docker 网络：

```sql
docker network create cluster --subnet=192.168.0.0/16
```

然后，启动管理节点：

```sql
docker run -d --net=cluster --name=management1 --ip=192.168.0.2 container-registry.oracle.com/mysql/community-cluster ndb_mgmd
```

接下来，启动两个数据节点

```sql
docker run -d --net=cluster --name=ndb1 --ip=192.168.0.3 container-registry.oracle.com/mysql/community-cluster ndbd
```

```sql
docker run -d --net=cluster --name=ndb2 --ip=192.168.0.4 container-registry.oracle.com/mysql/community-cluster ndbd
```

最后，启动 MySQL 服务器节点：

```sql
docker run -d --net=cluster --name=mysql1 --ip=192.168.0.10 -e MYSQL_RANDOM_ROOT_PASSWORD=true container-registry.oracle.com/mysql/community-cluster mysqld
```

然后，服务器将使用随机生成的密码进行初始化，需要更改密码。从日志中获取密码：

```sql
docker logs mysql1 2>&1 | grep PASSWORD
```

如果命令没有返回密码，则服务器尚未完成初始化。请等待一会儿，然后再试一次。一旦获得密码，通过`mysql`客户端登录到服务器并更改密码：

```sql
docker exec -it mysql1 mysql -uroot -p
```

一旦您在服务器上，使用以下语句更改根密码：

```sql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '*password*';
```

最后，启动一个带有交互式管理客户端**ndb_mgm**的容器来监视集群：

```sql
$> docker run -it --net=cluster container-registry.oracle.com/mysql/community-cluster ndb_mgm
[Entrypoint] MySQL Docker Image 8.0.34-1.2.10-cluster
[Entrypoint] Starting ndb_mgm
-- NDB Cluster -- Management Client --
```

运行`SHOW`命令以打印集群的状态。您应该看到以下内容：

```sql
ndb_mgm> SHOW
Connected to Management Server at: 192.168.0.2:1186
Cluster Configuration
---------------------
[ndbd(NDB)]	2 node(s)
id=2	@192.168.0.3  (mysql-8.0.34-ndb-8.0.34, Nodegroup: 0, *)
id=3	@192.168.0.4  (mysql-8.0.34-ndb-8.0.34, Nodegroup: 0)

[ndb_mgmd(MGM)]	1 node(s)
id=1	@192.168.0.2  (mysql-8.0.34-ndb-8.0.34)

[mysqld(API)]	1 node(s)
id=4	@192.168.0.10  (mysql-8.0.34-ndb-8.0.34)
```

##### 自定义 MySQL 集群

默认的 MySQL NDB 集群镜像包括两个配置文件，这些文件也可以在[MySQL NDB 集群的 GitHub 存储库](https://github.com/mysql/mysql-docker/tree/mysql-cluster)中找到。

+   `/etc/my.cnf`

+   `/etc/mysql-cluster.cnf`

要更改集群（例如，添加更多节点或更改网络设置），必须更新这些文件。有关更多信息，请参见第 25.4.3 节，“NDB 集群配置文件”。在启动容器时使用自定义配置文件，使用`-v`标志加载外部文件。例如（将整个命令输入在同一行）：

```sql
$> docker run -d --net=cluster --name=management1 \
      --ip=192.168.0.2 -v /etc/my.cnf:/etc/my.cnf -v \ 
      /etc/mysql-cluster.cnf:/etc/mysql-cluster.cnf \
      container-registry.oracle.com/mysql/community-cluster ndb_mgmd
```
