# 2.1.4 使用 MD5 校验和或 GnuPG 验证软件包完整性

> 原文：[`dev.mysql.com/doc/refman/8.0/en/verifying-package-integrity.html`](https://dev.mysql.com/doc/refman/8.0/en/verifying-package-integrity.html)

2.1.4.1 验证 MD5 校验和

2.1.4.2 使用 GnuPG 进行签名检查

2.1.4.3 使用 Gpg4win 在 Windows 上进行签名检查

2.1.4.4 使用 RPM 进行签名检查

2.1.4.5 存档软件包的 GPG 公共构建密钥

在下载适合您需求的 MySQL 软件包并尝试安装之前，请确保其完整且未被篡改。有三种完整性检查方法：

+   MD5 校验和

+   使用 `GnuPG`，GNU 隐私卫士的加密签名

+   对于 RPM 包，内置的 RPM 完整性验证机制

以下部分描述了如何使用这些方法。

如果您注意到 MD5 校验和或 GPG 签名不匹配，请首先尝试再次下载相应的软件包，可能从另一个镜像站点。
