> 原文：[`dev.mysql.com/doc/refman/8.0/en/clone-plugin-compressed-data.html`](https://dev.mysql.com/doc/refman/8.0/en/clone-plugin-compressed-data.html)

#### 7.6.7.6 克隆压缩数据

支持对页面压缩数据进行克隆。在克隆远程数据时，需要满足以下要求：

+   接收方文件系统必须支持稀疏文件和空洞打孔，以便在接收方上进行空洞打孔。

+   捐赠方和接收方文件系统必须具有相同的块大小。如果文件系统块大小不同，则会报告类似以下错误：ERROR 3868 (HY000): Clone Configuration FS Block Size: Donor value: 114688 is different from Recipient value: 4096.

关于页面压缩功能的信息，请参见第 17.9.2 节，“InnoDB 页面压缩”。
