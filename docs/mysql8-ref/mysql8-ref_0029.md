> 原文：[`dev.mysql.com/doc/refman/8.0/en/verifying-md5-checksum.html`](https://dev.mysql.com/doc/refman/8.0/en/verifying-md5-checksum.html)

#### 2.1.4.1 验证 MD5 校验和

下载 MySQL 软件包后，您应该确保其 MD5 校验和与 MySQL 下载页面上提供的校验和匹配。每个软件包都有一个您可以验证的独立校验和，以确保与您下载的软件包匹配。正确的 MD5 校验和在每个 MySQL 产品的下载页面上列出；您应该将其与您下载的文件（产品）的 MD5 校验和进行比较。

每个操作系统和设置都提供了用于检查 MD5 校验和的工具的自己版本。通常命令被命名为**md5sum**，或者可能被命名为**md5**，有些操作系统根本不提供。在 Linux 上，它是**GNU 文本工具**包的一部分，可用于广泛的平台。您还可以从[`www.gnu.org/software/textutils/`](http://www.gnu.org/software/textutils/)下载源代码。如果安装了 OpenSSL，您可以使用命令**openssl md5 *`package_name`***。Windows 实现的**md5**命令行实用程序可从[`www.fourmilab.ch/md5/`](http://www.fourmilab.ch/md5/)获取。**winMd5Sum**是一个图形化的 MD5 检查工具，可从[`www.nullriver.com/index/products/winmd5sum`](http://www.nullriver.com/index/products/winmd5sum)获取。我们的 Microsoft Windows 示例假定名称为**md5.exe**。

Linux 和 Microsoft Windows 示例：

```sql
$> md5sum mysql-standard-8.0.36-linux-i686.tar.gz
aaab65abbec64d5e907dcd41b8699945  mysql-standard-8.0.36-linux-i686.tar.gz
```

```sql
$> md5.exe mysql-installer-community-8.0.36.msi
aaab65abbec64d5e907dcd41b8699945  mysql-installer-community-8.0.36.msi
```

你应该验证生成的校验和（十六进制数字串）是否与下载页面上相应软件包下方显示的校验和匹配。

注意

请确保验证*存档文件*（例如`.zip`、`.tar.gz`或`.msi`文件）的校验和，而不是存档内包含的文件的校验和。换句话说，在提取内容之前验证文件。
