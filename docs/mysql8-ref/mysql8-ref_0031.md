> 原文：[`dev.mysql.com/doc/refman/8.0/en/checking-gpg-signature-windows.html`](https://dev.mysql.com/doc/refman/8.0/en/checking-gpg-signature-windows.html)

#### 2.1.4.3 使用 Gpg4win 在 Windows 上进行签名检查

第 2.1.4.2 节，“使用 GnuPG 进行签名检查”部分描述了如何使用 GPG 验证 MySQL 下载。该指南也适用于 Microsoft Windows，但另一种选择是使用类似[Gpg4win](http://www.gpg4win.org/)的 GUI 工具。您可以使用不同的工具，但我们的示例是基于 Gpg4win，并利用其捆绑的`Kleopatra` GUI。

下载并安装 Gpg4win，然后加载 Kleopatra。对话框应该类似于：

**图 2.1 Kleopatra：初始屏幕**

![显示默认的 Kleopatra 屏幕。顶部菜单包括“文件”，“查看”，“证书”，“工具”，“设置”，“窗口”和“帮助”。顶部菜单下方是一个水平操作栏，包括“导入证书”，“重新显示”和“在服务器上查找证书”的可用按钮。灰色按钮为“导出证书”和“停止操作”。下方是一个名为“查找”的搜索框。在下面是三个选项卡：“我的证书”，“受信任的证书”和“其他证书”，其中选择了“我的证书”选项卡。���我的证书”包含六列：“名称”，“电子邮件”，“有效期从”，“有效期至”，“详细信息”和“密钥 ID”。没有示例值。](img/ba52471d309a541735a23a085c2fd06e.png)

接下来，添加 MySQL Release Engineering 证书。通过单击文件，查找服务器上的证书来执行此操作。在搜索框中键入“Mysql Release Engineering”并按搜索。

**图 2.2 Kleopatra：在服务器上查找证书向导：查找证书**

![显示一个名为“查找”的搜索输入字段，标题为“mysql release engineering”，输入的内容为“mysql release engineering”。一个结果包含以下值：名称=MySQL Release Engineering，电子邮件=mysql-build@oss.oracle.com，有效期从 2003-02-03 开始，有效期至“”，详细信息=OpenPGP，指纹=5072E1F5，密钥 ID=5072E1F5。可用的操作按钮有：搜索，全选，取消全选，详细信息，导入和关闭。](img/cffe6a9378752d7086a8febf20afa0b9.png)

选择“MySQL Release Engineering”证书。对于 MySQL 8.0.28 及更高版本，指纹和密钥 ID 必须为“3A79BD29”，对于 MySQL 8.0.27 及更早版本，必须为“5072E1F5”，或选择“详细信息...”以确认证书是否有效。现在，通过单击“导入”来导入它。当显示导入对话框时，选择“确定”，此证书现在应该列在“导入的证书”选项卡下。

接下来，配置我们证书的信任级别。选择我们的证书，然后从主菜单中选择证书，更改所有者信任....我们建议选择“我相信检查非常准确”以验证我们的证书，否则您可能无法验证我们的签名。选择“我相信检查非常准确”以启用“完全信任”，然后按“确定”。

**图 2.3 Kleopatra：更改 MySQL 发布工程的信任级别**

![显示了一系列信任选项，包括"I don't know (unknown trust)"、"I do NOT trust them (never trust)"、"I believe checks are casual (marginal trust)"、"I believe checks are very accurate (full trust)"和"This is my certificate (ultimate trust)"。选择了"I believe checks are very accurate (full trust)"选项。](img/d6d6dbc58f8240eb95bbc274e030c962.png)

接下来，验证已下载的 MySQL 软件包文件。这需要打包文件和签名文件。签名文件必须与打包文件同名，但要附加一个 .asc 扩展名，如下表中的示例所示。签名在每个 MySQL 产品的下载页面上都有链接。您必须使用此签名创建 .asc 文件。

**表 2.2 MySQL Installer for Microsoft Windows 的 MySQL 软件包和签名文件**

| 文件类型 | 文件名 |
| --- | --- |
| 分发文件 | `mysql-installer-community-8.0.36.msi` |
| 签名文件 | `mysql-installer-community-8.0.36.msi.asc` |

确保两个文件存储在同一目录中，然后运行以下命令来验证分发文件的签名。要么将签名（.asc）文件拖放到 Kleopatra 中，要么从文件中加载对话框，选择 Decrypt/Verify Files...，然后选择 .msi 或 .asc 文件。

**图 2.4 Kleopatra：解密和验证文件对话框**

![显示了可用的解密和验证选项。示例中使用了一个 MySQL Installer MSI 文件，其中 .asc 文件列为"输入文件"，.msi 文件列为"签名数据"。选中了"Input file is detached signature"选项的复选框。显示了一个"Input file is an archive; unpack with:"选项，但是被灰掉了。下方是"Create all output files in a single folder"选项的复选框，以及一个"Output folder"输入框，示例中输入了"C:/docs"。可用的按钮有"Back"（灰掉）、"Decrypt/Verify"和"Cancel"。](img/b89ddc50f5a754a080dd0038ef9cdc4c.png)

点击 Decrypt/Verify 来检查文件。最常见的两种结果看起来像下图；尽管黄色警告可能看起来有问题，但以下内容表示文件检查已成功通过。您现在可以运行此安装程序。

**图 2.5 Kleopatra：解密和验证结果对话框：所有操作已完成**

![结果窗口的黄色部分显示了"Not enough information to check signature validity"和"The validity of the signature cannot be verified."。还显示了关键信息，如 KeyID 和电子邮件地址，密钥的签名日期，并显示了 ASC 文件的名称。](img/66000da868c4452a8605060f167fd3d3.png)

如果看到红色的"The signature is bad"错误，意味着文件无效。如果看到此错误，请不要执行 MSI 文件。

**图 2.6 Kleopatra：解密和验证结果对话框：错误**

![结果窗口中的红色部分显示“无效签名”，“使用未知证书签名”，“签名有问题”，并显示 ASC 文件的名称。](img/a6c38233eb472ee44199774bd4c2647a.png)

第 2.1.4.2 节，“使用 GnuPG 进行签名检查”解释了为什么你看不到绿色的`Good signature`结果。
