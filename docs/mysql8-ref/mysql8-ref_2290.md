# A.17 MySQL 8.0 FAQ：InnoDB 数据静态加密

> 原文：[`dev.mysql.com/doc/refman/8.0/en/faqs-tablespace-encryption.html`](https://dev.mysql.com/doc/refman/8.0/en/faqs-tablespace-encryption.html)

A.17.1\. [数据是否解密给了有权限查看的用户？](https://dev.mysql.com/doc/refman/8.0/en/faqs-tablespace-encryption.html#faq-tablespace-encryption-access)

A.17.2\. [InnoDB 数据静态加密相关的开销是多少？](https://dev.mysql.com/doc/refman/8.0/en/faqs-tablespace-encryption.html#faq-tablespace-encryption-overhead)

A.17.3\. [InnoDB 数据静态加密使用哪些加密算法？](https://dev.mysql.com/doc/refman/8.0/en/faqs-tablespace-encryption.html#faq-tablespace-encryption-algorithm)

A.17.4\. [是否可以使用第三方加密算法代替 InnoDB 数据静态加密功能提供的算法？](https://dev.mysql.com/doc/refman/8.0/en/faqs-tablespace-encryption.html#faq-tablespace-encryption-other-algorithms)

A.17.5\. [索引列可以加密吗？](https://dev.mysql.com/doc/refman/8.0/en/faqs-tablespace-encryption.html#faq-tablespace-encryption-indexed-columns)

A.17.6\. [InnoDB 数据静态加密支持哪些数据类型和数据长度？](https://dev.mysql.com/doc/refman/8.0/en/faqs-tablespace-encryption.html#faq-tablespace-encryption-data-types)

A.17.7\. [数据在网络上保持加密吗？](https://dev.mysql.com/doc/refman/8.0/en/faqs-tablespace-encryption.html#faq-tablespace-encryption-network)

A.17.8\. [数据库内存包含明文还是加密数据？](https://dev.mysql.com/doc/refman/8.0/en/faqs-tablespace-encryption.html#faq-tablespace-encryption-database-memory)

A.17.9\. [我如何知道哪些数据需要加密？](https://dev.mysql.com/doc/refman/8.0/en/faqs-tablespace-encryption.html#faq-tablespace-encryption-data-to-encrypt)

A.17.10\. [InnoDB 数据静态加密与 MySQL 已提供的加密函数有何不同？](https://dev.mysql.com/doc/refman/8.0/en/faqs-tablespace-encryption.html#faq-tablespace-encryption-mysql-encryption)

A.17.11\. [可传输表空间功能与 InnoDB 数据静态加密兼容吗？](https://dev.mysql.com/doc/refman/8.0/en/faqs-tablespace-encryption.html#faq-tablespace-encryption-transportable-tablespaces)

A.17.12\. [压缩与 InnoDB 数据静态加密是否兼容？](https://dev.mysql.com/doc/refman/8.0/en/faqs-tablespace-encryption.html#faq-tablespace-encryption-compression)

A.17.13\. [我可以在加密表上使用 mysqldump 吗？](https://dev.mysql.com/doc/refman/8.0/en/faqs-tablespace-encryption.html#faq-tablespace-encryption-mysqldump)

A.17.14\. [如何更改（旋转，重新生成密钥）主加密密钥？](https://dev.mysql.com/doc/refman/8.0/en/faqs-tablespace-encryption.html#faq-tablespace-encryption-key-rotation)

A.17.15\. [我如何将数据从明文 InnoDB 表空间迁移到加密的 InnoDB 表空间？](https://dev.mysql.com/doc/refman/8.0/en/faqs-tablespace-encryption.html#faq-tablespace-encryption-data-migration)

| **A.17.1.** | 数据是否解密给了有权限查看的用户？ |
| --- | --- |
|  | 是的。`InnoDB`数据静态加密旨在在数据库内部透明地应用加密，而不影响现有应用程序。以加密格式返回数据将破坏大多数现有应用程序。`InnoDB`数据静态加密提供了加密的好处，而不会带来传统数据库加密解决方案所带来的开销，后者通常需要昂贵且重大的应用程序、数据库触发器和视图更改。 |
| **A.17.2.** | `InnoDB`数据静态加密相关的开销是多少？ |
|  | 没有额外的存储开销。根据内部基准测试，性能开销仅相当于单个数字百分比的差异。 |
| **A.17.3.** | `InnoDB`数据静态加密使用哪些加密算法？ |
|  | `InnoDB`数据静态加密支持高级加密标准（AES256）基于块的加密算法。它使用电子密码本（ECB）块加密模式用于表空间密钥加密，使用密码块链接（CBC）块加密模式用于数据加密。 |
| **A.17.4.** | 是否可以使用第三方加密算法来替代`InnoDB`数据静态加密功能提供的算法？ |
|  | 不，不可能使用其他加密算法。提供的加密算法被广泛接受。 |
| **A.17.5.** | 索引列可以加密吗？ |
|  | `InnoDB`数据静态加密透明地支持所有索引。 |
| **A.17.6.** | `InnoDB`数据静态加密支持哪些数据类型和数据长度？ |
|  | `InnoDB`数据静态加密支持所有支持的数据类型。没有数据长度限制。 |
| **A.17.7.** | 数据在网络上保持加密吗？ |
|  | 通过`InnoDB`数据静态加密功能加密的数据在从表空间文件读取时解密。因此，如果数据在网络上，它是明文形式。然而，网络上的数据可以使用 MySQL 网络加密进行加密，该加密使用 SSL/TLS 加密数据库之间传输的数据。 |
| **A.17.8.** | 数据库内存包含明文还是加密数据？ |
|  | 使用`InnoDB`数据静态加密，内存中的数据被解密，提供完全透明性。 |
| **A.17.9.** | 我如何知道要加密哪些数据？ |
|  | 遵守 PCI-DSS 标准要求信用卡号（主帐号号码，或'PAN'）以加密形式存储。违反通知法律（例如，CA SB 1386，CA AB 1950 以及 43 多个美国州的类似法律）要求对名字、姓氏、驾驶执照号码和其他 PII 数据进行加密。2008 年初，CA AB 1298 将医疗和健康保险信息添加到 PII 数据中。此外，行业特定的隐私和安全标准可能要求对某些资产进行加密。例如，像制药研究结果、油田勘探结果、金融合同或执法线人的个人数据等资产可能需要加密。在医疗保健行业，患者数据、健康记录和 X 射线图像的隐私至关重要。 |
| **A.17.10.** | `InnoDB`数据静止加密与 MySQL 已提供的加密函数有何不同？ |
|  | MySQL 中有对称和非对称加密 API，可用于在数据库内部手动加密数据。但是，应用程序必须管理加密密钥，并通过调用 API 函数执行所需的加密和解密操作。`InnoDB`数据静止加密不需要应用程序更改，对终端用户透明，并提供自动化的内置密钥管理。 |
| **A.17.11.** | 可以使用可传输表空间功能与`InnoDB`数据静止加密一起使用吗？ |
|  | 是的。对于加密的文件表表空间是支持的。有关更多信息，请参见导出加密表空间。 |
| **A.17.12.** | 压缩与`InnoDB`数据静止加密兼容吗？ |
|  | 使用`InnoDB`数据静止加密的客户可以充分利用压缩，因为压缩是在数据块加密之前应用的。 |
| **A.17.13.** | 我可以在加密表上使用`mysqldump`吗？ |
|  | 是的。因为这些实用程序创建逻辑备份，从加密表中转储的数据未加密。 |
| **A.17.14.** | 如何更改（旋转，重新生成）主加密密钥？ |
|  | `InnoDB`数据静止加密使用两层密钥机制。当使用数据静止加密时，单独的表空间密钥存储在底层表空间数据文件的头部。表空间密钥使用主加密密钥加密。主加密密钥在启用表空间加密时生成，并存储在数据库外部。主加密密钥使用`ALTER INSTANCE ROTATE INNODB MASTER KEY`语句进行旋转，生成新的主加密密钥，存储密钥，并将密钥旋转到使用中。 |
| **A.17.15.** | 如何将明文`InnoDB`表空间中的数据迁移到加密`InnoDB`表空间中？ |
|  | 不需要将数据从一个表空间转移到另一个表空间。要在`InnoDB`文件每表表空间中加密数据，请运行`ALTER TABLE *`tbl_name`* ENCRYPTION = 'Y'`。要加密通用表空间或`mysql`表空间，请运行`ALTER TABLESPACE *`tablespace_name`* ENCRYPTION = 'Y'`。MySQL 8.0.13 引入了对通用表空间的加密支持。MySQL 8.0.16 引入了对`mysql`系统表空间的加密支持。 |
