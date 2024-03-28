> 原文：[`dev.mysql.com/doc/refman/8.0/en/checking-gpg-signature.html`](https://dev.mysql.com/doc/refman/8.0/en/checking-gpg-signature.html)

#### 2.1.4.2 使用 GnuPG 进行签名检查

验证软件包的完整性和真实性的另一种方法是使用加密签名。这比使用 MD5 校验和更可靠，但需要更多工作。

我们使用**GnuPG**（GNU 隐私卫士）对 MySQL 可下载包进行签名。**GnuPG** 是 Phil Zimmermann 的著名 Pretty Good Privacy（**PGP**）的开源替代方案。大多数 Linux 发行版默认安装了**GnuPG**。否则，请参阅[`www.gnupg.org/`](http://www.gnupg.org/)获取有关**GnuPG**的更多信息以及如何获取和安装它。

要验证特定包的签名，您首先需要获取我们的公共 GPG 构建密钥的副本，您可以从[`pgp.mit.edu/`](http://pgp.mit.edu/)下载。您想要获取的密钥名为`mysql-build@oss.oracle.com`。MySQL 8.0.28 到 8.0.35 包的 keyID 为`3A79BD29`。在获取此密钥后，您应该在使用它验证 MySQL 包之前将其与下面显示的密钥进行比较。或者，您可以直接从下面的文本中复制并粘贴密钥。

重要

`3A79BD29` 密钥将于 2023-12-14 过期。一个新的替代密钥（`A8D3785C`）将签署即将发布的 MySQL 8.0.36 及更高版本的软件包。这两个密钥都将随 MySQL 8.0.35 发布的 MySQL 仓库设置软件包一起安装，并且这两个密钥也可以在[`repo.mysql.com/`](https://repo.mysql.com/)上找到。

注意

以下公共 GPG 构建密钥适用于 MySQL 8.0.28 到 8.0.35 包。对于早期 MySQL 发行包的公共 GPG 构建密钥（keyID `5072E1F5`），请参阅 Section 2.1.4.5, “存档包的 GPG 公共构建密钥”。

```sql
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: SKS 1.1.6
Comment: Hostname: pgp.mit.edu

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
sTSKHe+QnnnoFmu4gnmDU31i
=Xqbo
-----END PGP PUBLIC KEY BLOCK-----
```

要将构建密钥导入到您的个人公共 GPG 密钥环中，请使用**gpg --import**。例如，如果您已将密钥保存在名为`mysql_pubkey.asc`的文件中，则导入命令如下：

```sql
$> gpg --import mysql_pubkey.asc
gpg: key 3A79BD29: public key "MySQL Release Engineering
<mysql-build@oss.oracle.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

您还可以使用公钥 id `3A79BD29` 从公共密钥服务器下载密钥：

```sql
$> gpg --recv-keys 3A79BD29
gpg: requesting key 3A79BD29 from hkp server keys.gnupg.net
gpg: key 3A79BD29: "MySQL Release Engineering <mysql-build@oss.oracle.com>"
1 new user ID
gpg: key 3A79BD29: "MySQL Release Engineering <mysql-build@oss.oracle.com>"
53 new signatures
gpg: no ultimately trusted keys found
gpg: Total number processed: 1
gpg:           new user IDs: 1
gpg:         new signatures: 53
```

如果您想将密钥导入到您的 RPM 配置中以验证 RPM 安装包，您应该能够直接导入密钥：

```sql
$> rpm --import mysql_pubkey.asc
```

如果您遇到问题或需要 RPM 特定信息，请参阅 Section 2.1.4.4, “使用 RPM 进行签名检查”。

在下载并导入公共构建密钥后，下载您需要的 MySQL 包以及相应的签名，这些也可以从下载页面获取。签名文件与分发文件具有相同的名称，扩展名为`.asc`，如下表中的示例所示。

**表 2.1 MySQL 源文件的软件包和签名文件**

| 文件类型 | 文件名 |
| --- | --- |
| 发行文件 | `mysql-standard-8.0.36-linux-i686.tar.gz` |
| 签名文件 | `mysql-standard-8.0.36-linux-i686.tar.gz.asc` |

确保这两个文件存储在同一目录中，然后运行以下命令验证发行文件的签名：

```sql
$> gpg --verify *package_name*.asc
```

如果下载的软件包有效，您应该看到类似于此的`Good signature`消息：

```sql
$> gpg --verify mysql-standard-8.0.36-linux-i686.tar.gz.asc
gpg: Signature made Tue 01 Feb 2011 02:38:30 AM CST using DSA key ID 3A79BD29
gpg: Good signature from "MySQL Release Engineering <mysql-build@oss.oracle.com>"
```

`Good signature`消息表示文件签名有效，与我们网站上列出的签名相比。但您可能也会看到警告，如下所示：

```sql
$> gpg --verify mysql-standard-8.0.36-linux-i686.tar.gz.asc
gpg: Signature made Wed 23 Jan 2013 02:25:45 AM PST using DSA key ID 3A79BD29
gpg: checking the trustdb
gpg: no ultimately trusted keys found
gpg: Good signature from "MySQL Release Engineering <mysql-build@oss.oracle.com>"
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: A4A9 4068 76FC BD3C 4567  70C8 8C71 8D3B 5072 E1F5
```

这是正常的，因为它们取决于您的设置和配置。以下是这些警告的解释：

+   *gpg: 未找到最终受信任的密钥*：这意味着您或您的信任网络没有“最终受信任”特定密钥，这对于验证文件签名是可以接受的。

+   *警告：此密钥未经受信任签名！没有迹象表明签名属于所有者。*：这涉及您对自己拥有我们真实公钥的信任水平。这是个人决定。理想情况下，MySQL 开发人员会亲自交给您密钥，但更常见的是，您下载了它。下载是否被篡改？可能不会，但这个决定取决于您。建立信任网络是信任它们的一种方法。

请参阅 GPG 文档，了解如何使用公钥。
