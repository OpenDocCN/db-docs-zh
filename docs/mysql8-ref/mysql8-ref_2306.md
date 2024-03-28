> 原文：[`dev.mysql.com/doc/refman/8.0/en/communication-errors.html`](https://dev.mysql.com/doc/refman/8.0/en/communication-errors.html)

#### B.3.2.9 通信错误和中止连接

如果发生连接问题，如通信错误或中止连接，请使用以下信息源诊断问题：

+   错误日志。参见 Section 7.4.2, “The Error Log”。

+   一般查询日志。参见 Section 7.4.3, “The General Query Log”。

+   `Aborted_*`xxx`*`和`Connection_errors_*`xxx`*`状态变量。参见 Section 7.1.10, “Server Status Variables”。

+   主机缓存，可通过性能模式`host_cache`表访问。参见 Section 7.1.12.3, “DNS Lookups and the Host Cache”，以及 Section 29.12.21.3, “The host_cache Table”。

如果`log_error_verbosity`系统变量设置为 3，您可能会在错误日志中找到类似以下消息：

```sql
[Note] Aborted connection 854 to db: 'employees' user: 'josh'
```

如果客户端甚至无法连接，服务器会增加`Aborted_connects`状态变量。连接尝试失败可能出现以下原因：

+   客户端尝试访问数据库但没有权限。

+   客户端使用了错误的密码。

+   连接数据包不包含正确的信息。

+   超过`connect_timeout`秒才能获取连接数据包。参见 Section 7.1.8, “Server System Variables”。

如果发生这些情况，可能表明有人试图入侵您的服务器！如果启用了一般查询日志，这些类型的问题的消息将记录在其中。

如果客户端成功连接但后来断开连接不当或被终止，服务器会增加`Aborted_clients`状态变量，并在错误日志中记录一个中止连接的消息。可能的原因有以下几种：

+   客户端程序在退出之前没有调用`mysql_close()`。

+   客户端在超过`wait_timeout`或`interactive_timeout`秒的时间内没有向服务器发出任何请求而处于休眠状态。参见 Section 7.1.8, “Server System Variables”。

+   客户端程序在数据传输过程中突然结束。

其他导致连接中断或客户端中断的问题原因：

+   `max_allowed_packet` 变量值过小，或查询需要的内存超过为 **mysqld** 分配的内存。参见 Section B.3.2.8, “Packet Too Large”。

+   在 Linux 上使用以太网协议，包括半双工和全双工。一些 Linux 以太网驱动程序存在此 bug。您可以通过在客户端和服务器机器之间使用 FTP 传输大文件来测试此 bug。如果传输以突发-暂停-突发-暂停的模式进行，那么您遇到了 Linux 双工综合症。将网络卡和集线器/交换机的双工模式切换为全双工或半双工，并测试结果以确定最佳设置。

+   导致读取中断的线程库问题。

+   TCP/IP 配置不当。

+   故障的以太网、集线器、交换机、电缆等。只有通过更换硬件才能正确诊断此问题。

参见 Section B.3.2.7, “MySQL server has gone away”。
