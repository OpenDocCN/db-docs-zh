# 2.5.6 在 Linux 上使用 Docker 容器部署 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/linux-installation-docker.html`](https://dev.mysql.com/doc/refman/8.0/en/linux-installation-docker.html)

2.5.6.1 使用 Docker 部署 MySQL 服务器的基本步骤

2.5.6.2 使用 Docker 部署 MySQL 服务器的更多主题

2.5.6.3 在 Windows 和其他非 Linux 平台上使用 Docker 部署 MySQL

本节介绍如何使用 Docker 容器部署 MySQL 服务器。

虽然在以下说明中为演示目的使用`docker`客户端，但总体上，由 Oracle 提供的 MySQL 容器镜像可与符合[OCI 1.0 规范](https://opencontainers.org/posts/announcements/2021-05-04-oci-dist-spec-v1/)的任何容器工具一起使用。

警告

在使用 Docker 容器部署 MySQL 之前，请确保您了解运行容器的安全风险并适当地加以缓解。
