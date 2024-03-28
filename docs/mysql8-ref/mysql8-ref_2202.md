> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-reload-saved.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-setup-reload-saved.html)

#### 30.4.4.12 `ps_setup_reload_saved()` 过程

使用`ps_setup_save()` Procedure")在同一会话中重新加载先前保存的性能模式配置。有关更多信息，请参阅`ps_setup_save()` Procedure")的描述。

在执行此过程期间，通过操纵`sql_log_bin`系统变量的会话值来禁用二进制日志记录。这是一项受限制的操作，因此该过程需要具有足够权限以设置受限制会话变量的权限。请参阅 Section 7.1.9.1, “System Variable Privileges”。

##### 参数

无。
