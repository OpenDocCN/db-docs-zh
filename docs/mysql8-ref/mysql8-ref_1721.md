> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndb-transporter-errors.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndb-transporter-errors.html)

#### 25.6.2.4 NDB Cluster: NDB Transporter Errors

此部分列出了在传输器错误发生时写入集群日志的错误代码、名称和消息。

0x00

TE_NO_ERROR

无错误

0x01

TE_ERROR_CLOSING_SOCKET

关闭套接字时发现错误

0x02

TE_ERROR_IN_SELECT_BEFORE_ACCEPT

在接受之前发现错误。传输器将重试

0x03

TE_INVALID_MESSAGE_LENGTH

在消息中发现错误（无效的消息长度）

0x04

TE_INVALID_CHECKSUM

在消息中发现错误（校验和）

0x05

TE_COULD_NOT_CREATE_SOCKET

创建套接字时发现错误（无法创建套接字）

0x06

TE_COULD_NOT_BIND_SOCKET

绑定服务器套接字时发现错误

0x07

TE_LISTEN_FAILED

监听服务器套接字时发现错误

0x08

TE_ACCEPT_RETURN_ERROR

在 accept 期间发现错误（接受返回错误）

0x0b

TE_SHM_DISCONNECT

远程节点已断开连接

0x0c

TE_SHM_IPC_STAT

无法检查 shm 段

0x0d

TE_SHM_UNABLE_TO_CREATE_SEGMENT

无法创建 shm 段

0x0e

TE_SHM_UNABLE_TO_ATTACH_SEGMENT

无法附加 shm 段

0x0f

TE_SHM_UNABLE_TO_REMOVE_SEGMENT

无法移除 shm 段

0x10

TE_TOO_SMALL_SIGID

Sig ID 太小

0x11

TE_TOO_LARGE_SIGID

Sig ID 太大

0x12

TE_WAIT_STACK_FULL

等待堆栈已满

0x13

TE_RECEIVE_BUFFER_FULL

接收缓冲区已满

0x14

TE_SIGNAL_LOST_SEND_BUFFER_FULL

发送缓冲区已满，并且尝试强制发送失败

0x15

TE_SIGNAL_LOST

发送失败原因未知（信号丢失）

0x16

TE_SEND_BUFFER_FULL

发送缓冲区已满，但睡一会解决了问题

0x21

TE_SHM_IPC_PERMANENT

Shm ipc Permanent 错误

注意

传输器错误代码 0x17 到 0x20 和 0x22 保留用于 SCI 连接，在此版本的 NDB Cluster 中不支持，因此未包含在此处。
