# -   8.4.1 认证插件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/authentication-plugins.html`](https://dev.mysql.com/doc/refman/8.0/en/authentication-plugins.html)

8.4.1.1 本地可插拔认证

8.4.1.2 缓存 SHA-2 可插拔认证

8.4.1.3 SHA-256 可插拔认证

8.4.1.4 客户端明文可插拔认证

8.4.1.5 PAM 可插拔认证

8.4.1.6 Windows 可插拔认证

8.4.1.7 LDAP 可插拔认证

8.4.1.8 Kerberos 可插拔认证

8.4.1.9 无登录可插拔认证

8.4.1.10 套接字对等凭证可插拔认证

8.4.1.11 FIDO 可插拔认证

8.4.1.12 测试可插拔认证

8.4.1.13 可插拔认证系统变量

注意

如果您正在寻找关于`authentication_oci`插件的信息，这仅适用于 MySQL HeatWave 服务。请参阅[authentication_oci 插件](https://docs.oracle.com/en-us/iaas/mysql-database/doc/connecting-db-system.html#MYAAS-GUID-232CA959-1FDD-4AA8-A77D-0A551C881C09)，在*MySQL HeatWave Service*手册中。

以下各节描述了 MySQL 中可用的可插拔认证方法以及实现这些方法的插件。有关认证过程的一般讨论，请参阅第 8.2.17 节，“可插拔认证”。

-   默认认证插件的确定方式如默认认证插件中所述。
