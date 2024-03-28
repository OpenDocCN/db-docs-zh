> 原文：[`dev.mysql.com/doc/refman/8.0/en/ipv6-brokers.html`](https://dev.mysql.com/doc/refman/8.0/en/ipv6-brokers.html)

#### 7.1.13.5 从代理获取 IPv6 地址

如果您没有一个能够使您的系统在本地网络之外通过 IPv6 进行通信的公共 IPv6 地址，您可以从 IPv6 代理处获取一个。[Wikipedia IPv6 隧道代理页面](http://en.wikipedia.org/wiki/List_of_IPv6_tunnel_brokers)列出了几个代理及其功能，例如它们是否提供静态地址和支持的路由协议。

在配置服务器主机使用代理提供的 IPv6 地址后，使用适当的`bind_address`设置启动 MySQL 服务器，以允许服务器接受 IPv6 连接。您可以将*（或`::`）指定为`bind_address`值，或将服务器绑定到代理提供的特定 IPv6 地址。有关更多信息，请参见第 7.1.8 节，“服务器系统变量”中的`bind_address`描述。

请注意，如果代理分配动态地址，则为您的系统提供的地址可能会在下次连接到代理时更改。如果是这样，任何使用原始地址命名的帐户都将变为无效。为了绑定到特定地址但避免这种地址更改问题，您可以尝试与代理安排获得静态 IPv6 地址。

以下示例展示了如何在 Gentoo Linux 上将 Freenet6 用作代理和**gogoc** IPv6 客户端包的使用方法。

1.  在 Freenet6 创建一个帐户，访问此 URL 并注册：

    ```sql
    http://gogonet.gogo6.com
    ```

1.  创建帐户后，转到此 URL，登录并为 IPv6 代理创建用户 ID 和密码：

    ```sql
    http://gogonet.gogo6.com/page/freenet6-registration
    ```

1.  作为`root`，安装**gogoc**：

    ```sql
    $> emerge gogoc
    ```

1.  编辑`/etc/gogoc/gogoc.conf`以设置`userid`和`password`的值。例如：

    ```sql
    userid=gogouser
    passwd=gogopass
    ```

1.  启动**gogoc**：

    ```sql
    $> /etc/init.d/gogoc start
    ```

    每次系统启动时启动**gogoc**，执行此命令：

    ```sql
    $> rc-update add gogoc default
    ```

1.  使用**ping6**尝试 ping 主机：

    ```sql
    $> ping6 ipv6.google.com
    ```

1.  查看你的 IPv6 地址：

    ```sql
    $> ifconfig tun
    ```
