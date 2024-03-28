> 原文：[`dev.mysql.com/doc/refman/8.0/en/checking-rpm-signature.html`](https://dev.mysql.com/doc/refman/8.0/en/checking-rpm-signature.html)

#### 2.1.4.4 使用 RPM 进行签名检查

对于 RPM 软件包，没有单独的签名。RPM 软件包具有内置的 GPG 签名和 MD5 校验和。你可以通过运行以下命令来验证一个软件包：

```sql
$> rpm --checksig *package_name*.rpm
```

示例：

```sql
$> rpm --checksig mysql-community-server-8.0.36-1.el8.x86_64.rpm
mysql-community-server-8.0.36-1.el8.x86_64.rpm: digests signatures OK
```

注意

如果你正在使用 RPM 4.1，并且它抱怨`(GPG) NOT OK (MISSING KEYS: GPG#3a79bd29)`，即使你已经将 MySQL 公共构建密钥导入到你自己的 GPG 密钥环中，你需要先将该密钥导入到 RPM 密钥环中。RPM 4.1 不再使用你的个人 GPG 密钥环（或 GPG 本身）。相反，RPM 维护一个单独的密钥环，因为它是一个系统范围的应用程序，而用户的 GPG 公钥环是一个特定于用户的文件。要将 MySQL 公共密钥导入到 RPM 密钥环中，首先获取该密钥，然后使用**rpm --import**导入该密钥。例如：

```sql
$> gpg --export -a 3a79bd29 > 3a79bd29.asc
$> rpm --import 3a79bd29.asc
```

或者，**rpm**也支持直接从 URL 加载密钥：

```sql
$> rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
```

你也可以从本手册页面获取 MySQL 公共密钥：第 2.1.4.2 节，“使用 GnuPG 进行签名检查”。
