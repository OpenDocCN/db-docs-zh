> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-tde-implementation.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-tde-implementation.html)

#### 25.6.14.2 NDB 文件系统加密实现

对于`NDB`透明数据加密（TDE），数据节点在静止状态下加密用户数据，安全性由密码（文件系统密码）提供，该密码用于加密和解密每个数据节点上的秘密文件。秘密文件包含一个节点主密钥（NMK），稍后用于加密用于持久性的不同文件类型。`NDB` TDE 加密用户数据文件，包括 LCP 文件、重做日志文件、表空间文件和撤销日志文件。

您可以使用**ndbxfrm**实用程序查看文件是否已加密，如下所示：

```sql
> ndbxfrm -i ndb_5_fs/LCP/0/T2F0.Data
File=ndb_5_fs/LCP/0/T2F0.Data, compression=no, encryption=yes
> ndbxfrm -i ndb_6_fs/LCP/0/T2F0.Data
File=ndb_6_fs/LCP/0/T2F0.Data, compression=no, encryption=no
```

从 NDB 8.0.31 开始，可以使用在该版本中添加的**ndb_secretsfile_reader**程序从秘密文件中获取密钥，如下所示：

```sql
> ndb_secretsfile_reader --filesystem-password=54kl14 ndb_5_fs/D1/NDBCNTR/S0.sysfile
ndb_secretsfile_reader: [Warning] Using a password on the command line interface can be insecure.
cac256e18b2ddf6b5ef82d99a72f18e864b78453cc7fa40bfaf0c40b91122d18
```

每个节点的密钥层次结构可以表示如下：

+   用户提供的口令（P）通过使用随机盐的密钥派生函数处理，生成一个唯一的口令密钥（PK）。

+   PK（对每个节点唯一）加密每个节点上的数据在其自己的秘密文件中。

+   秘密文件中的数据包括一个唯一的、随机生成的节点主密钥（NMK）。

+   NMK（使用包装）加密每个加密文件的头部中的一个或多个随机生成的数据加密密钥（DEK）值（包括 LCP 和 TS 文件以及重做和撤销日志）。

+   数据加密密钥值（DEK[0]，...，DEK[n]）用于加密每个文件中的[子集的]数据。

口令间接加密包含随机 NMK 的秘密文件，该文件加密节点上每个加密文件的一部分头。加密文件头包含用于该文件中数据的随机数据密钥。

加密由数据节点内的`NDBFS`层透明实现。`NDBFS`内部客户端块对其文件进行正常操作；`NDBFS`用额外的头部和尾部信息包装物理文件，支持加密，并在读取和写入文件时加密和解密数据。包装文件格式称为`ndbxfrm1`。

节点密码通过 PBKDF2 和随机盐处理，用于加密包含用于加密每个加密文件中的随机生成数据加密密钥的秘密文件。

加密和解密工作是在 NDBFS I/O 线程中执行的（而不是在信号执行线程中，如主线程、tc 线程、ldm 线程或 rep 线程）。这类似于压缩的 LCP 和压缩的备份的处理方式，通常会导致增加 I/O 线程的 CPU 使用率；您可能希望根据 I/O 线程的情况调整`ThreadConfig`。
