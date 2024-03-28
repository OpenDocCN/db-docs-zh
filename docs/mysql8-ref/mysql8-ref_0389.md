> 原文：[`dev.mysql.com/doc/refman/8.0/en/creating-ssl-files-using-openssl.html`](https://dev.mysql.com/doc/refman/8.0/en/creating-ssl-files-using-openssl.html)

#### 8.3.3.2 使用 openssl 创建 SSL 证书和密钥

本节描述了如何使用**openssl**命令设置供 MySQL 服务器和客户端使用的 SSL 证书和密钥文件。第一个示例显示了一个简化的过程，类似于您可能从命令行中使用的过程。第二个示例显示了一个包含更多细节的脚本。前两个示例适用于 Unix，并且都使用 OpenSSL 的一部分的**openssl**命令。第三个示例描述了如何在 Windows 上设置 SSL 文件。

注意

有比本文所述过程更简单的生成 SSL 所需文件的替代方法：让服务器自动生成它们或使用**mysql_ssl_rsa_setup**程序（自 8.0.34 起已弃用）。请参阅第 8.3.3.1 节，“使用 MySQL 创建 SSL 和 RSA 证书和密钥”。

重要

无论您使用何种方法生成证书和密钥文件，用于服务器和客户端证书/密钥的通用名称值必须与用于 CA 证书的通用名称值不同。否则，使用 OpenSSL 编译的服务器的证书和密钥文件将无法工作。在这种情况下的典型错误是：

```sql
ERROR 2026 (HY000): SSL connection error:
error:00000001:lib(0):func(0):reason(1)
```

重要

如果连接到 MySQL 服务器实例的客户端使用带有`extendedKeyUsage`扩展（X.509 v3 扩展）的 SSL 证书，则扩展密钥用途必须包括客户端认证（`clientAuth`）。如果 SSL 证书仅指定用于服务器认证（`serverAuth`）和其他非客户端证书目的，证书验证将失败，客户端连接到 MySQL 服务器实例将失败。在本主题中按照说明使用**openssl**命令创建的 SSL 证书中没有`extendedKeyUsage`扩展。如果您使用其他方式创建的自己的客户端证书，请确保任何`extendedKeyUsage`扩展包括客户端认证。

+   示例 1：在 Unix 命令行上创建 SSL 文件

+   示例 2：使用 Unix 脚本创建 SSL 文件

+   示例 3：在 Windows 上创建 SSL 文件

##### 示例 1：在 Unix 命令行上创建 SSL 文件

以下示例显示了一组命令，用于创建 MySQL 服务器和客户端证书和密钥文件。您必须通过**openssl**命令回答几个提示。要生成测试文件，可以对所有提示按 Enter 键。要生成用于生产的文件，应提供非空响应。

```sql
# Create clean environment
rm -rf newcerts
mkdir newcerts && cd newcerts

# Create CA certificate
openssl genrsa 2048 > ca-key.pem
openssl req -new -x509 -nodes -days 3600 \
        -key ca-key.pem -out ca.pem

# Create server certificate, remove passphrase, and sign it
# server-cert.pem = public key, server-key.pem = private key
openssl req -newkey rsa:2048 -days 3600 \
        -nodes -keyout server-key.pem -out server-req.pem
openssl rsa -in server-key.pem -out server-key.pem
openssl x509 -req -in server-req.pem -days 3600 \
        -CA ca.pem -CAkey ca-key.pem -set_serial 01 -out server-cert.pem

# Create client certificate, remove passphrase, and sign it
# client-cert.pem = public key, client-key.pem = private key
openssl req -newkey rsa:2048 -days 3600 \
        -nodes -keyout client-key.pem -out client-req.pem
openssl rsa -in client-key.pem -out client-key.pem
openssl x509 -req -in client-req.pem -days 3600 \
        -CA ca.pem -CAkey ca-key.pem -set_serial 01 -out client-cert.pem
```

生成证书后，请验证它们：

```sql
openssl verify -CAfile ca.pem server-cert.pem client-cert.pem
```

你应该看到如下响应：

```sql
server-cert.pem: OK
client-cert.pem: OK
```

要查看证书的内容（例如，检查证书有效日期范围），可以像这样调用**openssl**：

```sql
openssl x509 -text -in ca.pem
openssl x509 -text -in server-cert.pem
openssl x509 -text -in client-cert.pem
```

现在你有一组文件，可以按以下方式使用：

+   `ca.pem`：用于在服务器端设置`ssl_ca`系统变量和客户端端的`--ssl-ca`选项。（如果使用 CA 证书，则两端必须相同。）

+   `server-cert.pem`，`server-key.pem`：用于在服务器端设置`ssl_cert`和`ssl_key`系统变量。 

+   `client-cert.pem`，`client-key.pem`：将这些用作客户端端的`--ssl-cert`和`--ssl-key`选项的参数。

对于额外的使用说明，请参见第 8.3.1 节，“配置 MySQL 使用加密连接”。

##### 示例 2：在 Unix 上使用脚本创建 SSL 文件

这是一个示例脚本，展示如何为 MySQL 设置 SSL 证书和密钥文件。执行脚本后，按照第 8.3.1 节，“配置 MySQL 使用加密连接”中描述的方式用于 SSL 连接。

```sql
DIR=`pwd`/openssl
PRIV=$DIR/private

mkdir $DIR $PRIV $DIR/newcerts
cp /usr/share/ssl/openssl.cnf $DIR
replace ./demoCA $DIR -- $DIR/openssl.cnf

# Create necessary files: $database, $serial and $new_certs_dir
# directory (optional)

touch $DIR/index.txt
echo "01" > $DIR/serial

#
# Generation of Certificate Authority(CA)
#

openssl req -new -x509 -keyout $PRIV/cakey.pem -out $DIR/ca.pem \
    -days 3600 -config $DIR/openssl.cnf

# Sample output:
# Using configuration from /home/jones/openssl/openssl.cnf
# Generating a 1024 bit RSA private key
# ................++++++
# .........++++++
# writing new private key to '/home/jones/openssl/private/cakey.pem'
# Enter PEM pass phrase:
# Verifying password - Enter PEM pass phrase:
# -----
# You are about to be asked to enter information to be
# incorporated into your certificate request.
# What you are about to enter is what is called a Distinguished Name
# or a DN.
# There are quite a few fields but you can leave some blank
# For some fields there will be a default value,
# If you enter '.', the field will be left blank.
# -----
# Country Name (2 letter code) [AU]:FI
# State or Province Name (full name) [Some-State]:.
# Locality Name (eg, city) []:
# Organization Name (eg, company) [Internet Widgits Pty Ltd]:MySQL AB
# Organizational Unit Name (eg, section) []:
# Common Name (eg, YOUR name) []:MySQL admin
# Email Address []:

#
# Create server request and key
#
openssl req -new -keyout $DIR/server-key.pem -out \
    $DIR/server-req.pem -days 3600 -config $DIR/openssl.cnf

# Sample output:
# Using configuration from /home/jones/openssl/openssl.cnf
# Generating a 1024 bit RSA private key
# ..++++++
# ..........++++++
# writing new private key to '/home/jones/openssl/server-key.pem'
# Enter PEM pass phrase:
# Verifying password - Enter PEM pass phrase:
# -----
# You are about to be asked to enter information that will be
# incorporated into your certificate request.
# What you are about to enter is what is called a Distinguished Name
# or a DN.
# There are quite a few fields but you can leave some blank
# For some fields there will be a default value,
# If you enter '.', the field will be left blank.
# -----
# Country Name (2 letter code) [AU]:FI
# State or Province Name (full name) [Some-State]:.
# Locality Name (eg, city) []:
# Organization Name (eg, company) [Internet Widgits Pty Ltd]:MySQL AB
# Organizational Unit Name (eg, section) []:
# Common Name (eg, YOUR name) []:MySQL server
# Email Address []:
#
# Please enter the following 'extra' attributes
# to be sent with your certificate request
# A challenge password []:
# An optional company name []:

#
# Remove the passphrase from the key
#
openssl rsa -in $DIR/server-key.pem -out $DIR/server-key.pem

#
# Sign server cert
#
openssl ca -cert $DIR/ca.pem -policy policy_anything \
    -out $DIR/server-cert.pem -config $DIR/openssl.cnf \
    -infiles $DIR/server-req.pem

# Sample output:
# Using configuration from /home/jones/openssl/openssl.cnf
# Enter PEM pass phrase:
# Check that the request matches the signature
# Signature ok
# The Subjects Distinguished Name is as follows
# countryName           :PRINTABLE:'FI'
# organizationName      :PRINTABLE:'MySQL AB'
# commonName            :PRINTABLE:'MySQL admin'
# Certificate is to be certified until Sep 13 14:22:46 2003 GMT
# (365 days)
# Sign the certificate? [y/n]:y
#
#
# 1 out of 1 certificate requests certified, commit? [y/n]y
# Write out database with 1 new entries
# Data Base Updated

#
# Create client request and key
#
openssl req -new -keyout $DIR/client-key.pem -out \
    $DIR/client-req.pem -days 3600 -config $DIR/openssl.cnf

# Sample output:
# Using configuration from /home/jones/openssl/openssl.cnf
# Generating a 1024 bit RSA private key
# .....................................++++++
# .............................................++++++
# writing new private key to '/home/jones/openssl/client-key.pem'
# Enter PEM pass phrase:
# Verifying password - Enter PEM pass phrase:
# -----
# You are about to be asked to enter information that will be
# incorporated into your certificate request.
# What you are about to enter is what is called a Distinguished Name
# or a DN.
# There are quite a few fields but you can leave some blank
# For some fields there will be a default value,
# If you enter '.', the field will be left blank.
# -----
# Country Name (2 letter code) [AU]:FI
# State or Province Name (full name) [Some-State]:.
# Locality Name (eg, city) []:
# Organization Name (eg, company) [Internet Widgits Pty Ltd]:MySQL AB
# Organizational Unit Name (eg, section) []:
# Common Name (eg, YOUR name) []:MySQL user
# Email Address []:
#
# Please enter the following 'extra' attributes
# to be sent with your certificate request
# A challenge password []:
# An optional company name []:

#
# Remove the passphrase from the key
#
openssl rsa -in $DIR/client-key.pem -out $DIR/client-key.pem

#
# Sign client cert
#

openssl ca -cert $DIR/ca.pem -policy policy_anything \
    -out $DIR/client-cert.pem -config $DIR/openssl.cnf \
    -infiles $DIR/client-req.pem

# Sample output:
# Using configuration from /home/jones/openssl/openssl.cnf
# Enter PEM pass phrase:
# Check that the request matches the signature
# Signature ok
# The Subjects Distinguished Name is as follows
# countryName           :PRINTABLE:'FI'
# organizationName      :PRINTABLE:'MySQL AB'
# commonName            :PRINTABLE:'MySQL user'
# Certificate is to be certified until Sep 13 16:45:17 2003 GMT
# (365 days)
# Sign the certificate? [y/n]:y
#
#
# 1 out of 1 certificate requests certified, commit? [y/n]y
# Write out database with 1 new entries
# Data Base Updated

#
# Create a my.cnf file that you can use to test the certificates
#

cat <<EOF > $DIR/my.cnf
[client]
ssl-ca=$DIR/ca.pem
ssl-cert=$DIR/client-cert.pem
ssl-key=$DIR/client-key.pem
[mysqld]
ssl_ca=$DIR/ca.pem
ssl_cert=$DIR/server-cert.pem
ssl_key=$DIR/server-key.pem
EOF
```

##### 示例 3：在 Windows 上创建 SSL 文件

如果您的系统尚未安装 OpenSSL，请下载 Windows 版。可在此处查看可用软件包的概述：

```sql
http://www.slproweb.com/products/Win32OpenSSL.html
```

根据您的架构（32 位或 64 位）选择 Win32 OpenSSL Light 或 Win64 OpenSSL Light 软件包。默认安装位置为`C:\OpenSSL-Win32`或`C:\OpenSSL-Win64`，取决于您下载的软件包。以下说明假定默认位置为`C:\OpenSSL-Win32`。如果您使用 64 位软件包，则根据需要进行修改。

如果在安装过程中出现消息指示“...关键组件丢失：Microsoft Visual C++ 2019 Redistributables”，请取消安装并根据您的架构（32 位或 64 位）再次下载以下软件包之一：

+   Visual C++ 2008 Redistributables（x86），可在以下位置找到：

    ```sql
    http://www.microsoft.com/downloads/details.aspx?familyid=9B2DA534-3E03-4391-8A4D-074B9F2BC1BF
    ```

+   Visual C++ 2008 Redistributables（x64），可在以下位置找到：

    ```sql
    http://www.microsoft.com/downloads/details.aspx?familyid=bd2a6171-e2d6-4230-b809-9a8d7548c1b6
    ```

安装额外的软件包后，重新启动 OpenSSL 设置过程。

在安装过程中，将默认的`C:\OpenSSL-Win32`作为安装路径，并保持默认选项`'将 OpenSSL DLL 文件复制到 Windows 系统目录'`被选中。

安装完成后，将`C:\OpenSSL-Win32\bin`添加到服务器的 Windows 系统路径变量中（根据您的 Windows 版本，以下路径设置说明可能略有不同）：

1.  在 Windows 桌面上，右键单击“我的电脑”图标，然后选择“属性”。

1.  从出现的系统属性菜单中选择“高级”选项卡，然后点击“环境变量”按钮。

1.  在系统变量下，选择“Path”，然后点击“编辑”按钮。应该会出现“编辑系统变量”对话框。

1.  在末尾添加`';C:\OpenSSL-Win32\bin'`（注意分号）。

1.  依次按下 3 次“确定”按钮。

1.  通过打开一个新的命令控制台（**开始>运行>cmd.exe**）并验证 OpenSSL 是否可用来检查 OpenSSL 是否已正确集成到 Path 变量中：

    ```sql
    Microsoft Windows [Version ...]
    Copyright (c) 2006 Microsoft Corporation. All rights reserved.

    C:\Windows\system32>cd \

    C:\>openssl
    OpenSSL> exit <<< If you see the OpenSSL prompt, installation was successful.

    C:\>
    ```

安装 OpenSSL 后，使用类似于本节前面示例 1 的指令，但需进行以下更改：

+   更改以下 Unix 命令：

    ```sql
    # Create clean environment
    rm -rf newcerts
    mkdir newcerts && cd newcerts
    ```

    在 Windows 上，使用以下命令代替：

    ```sql
    # Create clean environment
    md c:\newcerts
    cd c:\newcerts
    ```

+   当命令行末尾显示一个`'\'`字符时，必须删除此`'\'`字符，并将命令行输入在同一行上。

生成证书和密钥文件后，要将它们用于 SSL 连接，请参阅第 8.3.1 节，“配置 MySQL 使用加密连接”。
