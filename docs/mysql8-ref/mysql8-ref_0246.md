> 原文：[`dev.mysql.com/doc/refman/8.0/en/ipv6-system-support.html`](https://dev.mysql.com/doc/refman/8.0/en/ipv6-system-support.html)

#### 7.1.13.1 验证系统对 IPv6 的支持

在 MySQL 服务器可以接受 IPv6 连接之前，您的服务器主机上的操作系统必须支持 IPv6。要确定这一点是否成立，可以尝试运行以下命令：

```sql
$> ping6 ::1
16 bytes from ::1, icmp_seq=0 hlim=64 time=0.171 ms
16 bytes from ::1, icmp_seq=1 hlim=64 time=0.077 ms
...
```

要生成系统网络接口的描述，请调用**ifconfig -a**并查看输出中的 IPv6 地址。

如果您的主机不支持 IPv6，请查阅系统文档以获取启用指南。也许您只需要重新配置现有网络接口以添加 IPv6 地址。或者可能需要进行更广泛的更改，比如使用启用 IPv6 选项重新构建内核。

这些链接可能有助于在各种平台上设置 IPv6：

+   [Windows](https://msdn.microsoft.com/en-us/library/dd163569.aspx)

+   [Gentoo Linux](http://www.gentoo.org/doc/en/ipv6.xml)

+   [Ubuntu Linux](https://wiki.ubuntu.com/IPv6)

+   [Linux（通用）](http://www.tldp.org/HOWTO/Linux+IPv6-HOWTO/)

+   [macOS](https://support.apple.com/en-us/HT202237)
