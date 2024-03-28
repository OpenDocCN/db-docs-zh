> 原文：[`dev.mysql.com/doc/refman/8.0/en/deploy-mysql-nonlinux-docker.html`](https://dev.mysql.com/doc/refman/8.0/en/deploy-mysql-nonlinux-docker.html)

#### 2.5.6.3 在 Windows 和其他非 Linux 平台上使用 Docker 部署 MySQL

警告

Oracle 提供的 MySQL Docker 镜像专为 Linux 平台构建。其他平台不受支持，使用 Oracle 提供的 MySQL Docker 镜像在这些平台上运行属于自担风险。本节讨论了在非 Linux 平台上使用这些镜像时的一些已知问题。

使用 Oracle 在 Windows 上的 MySQL 服务器 Docker 镜像存在以下已知问题：

+   如果您在容器的 MySQL 数据目录上进行绑定挂载（详见持久化数据和配置更改），您必须使用 `--socket` 选项将服务器套接字文件的位置设置在 MySQL 数据目录之外的某个地方；否则，服务器将无法启动。这是因为 Docker for Windows 处理文件挂载的方式不允许将主机文件绑定挂载到套接字文件上。
