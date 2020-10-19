---
title: 连接字符串 - Azure Database for MySQL
description: 本文档中列出了目前支持的用于将应用程序连接到 Azure Database for MySQL 的连接字符串，包括 ADO.NET (C#)、JDBC、Node.js、ODBC、PHP、Python 和 Ruby。
author: WenJason
ms.author: v-jay
ms.service: mysql
ms.topic: how-to
origin.date: 3/18/2020
ms.date: 10/19/2020
ms.custom: devx-track-python, devx-track-js
ms.openlocfilehash: e37eb497523f7d7e85751c6c4109887f28a5abe8
ms.sourcegitcommit: ba01e2d1882c85ebeffef344ef57afaa604b53a0
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/14/2020
ms.locfileid: "92041740"
---
# <a name="how-to-connect-applications-to-azure-database-for-mysql"></a>如何将应用程序连接到 Azure Database for MySQL

> [!NOTE]
> 将要查看的是 Azure Database for MySQL 的新服务。 若要查看经典 MySQL Database for Azure 的文档，请访问[此页](https://docs.azure.cn/zh-cn/mysql-database-on-azure/)。

本主题列出了 Azure Database for MySQL 支持的连接字符串以及模板和示例。 你可能会在连接字符串中使用不同的参数和设置。

- 若要获取证书，请参阅[如何配置 SSL](./howto-configure-ssl.md)。
- {your_host} = \<servername>.mysql.database.chinacloudapi.cn
- {your_user}@{servername} = userID 格式是用于身份验证的正确格式。  如果只使用 userID，身份验证将失败。

## <a name="adonet"></a>ADO.NET
```ado.net
Server={your_host};Port={your_port};Database={your_database};Uid={username@servername};Pwd={your_password};[SslMode=Required;]
```

在此示例中，服务器名称为 `mydemoserver`，数据库名称为 `wpdb`，用户名为 `WPAdmin`，密码为 `mypassword!2`。 因此，连接字符串应为：

```ado.net
Server= "mydemoserver.mysql.database.chinacloudapi.cn"; Port=3306; Database= "wpdb"; Uid= "WPAdmin@mydemoserver"; Pwd="mypassword!2"; SslMode=Required;
```

## <a name="jdbc"></a>JDBC
```jdbc
String url ="jdbc:mysql://%s:%s/%s[?verifyServerCertificate=true&useSSL=true&requireSSL=true]",{your_host},{your_port},{your_database}"; myDbConn = DriverManager.getConnection(url, {username@servername}, {your_password}";
```

## <a name="nodejs"></a>Node.js
```node.js
var conn = mysql.createConnection({host: {your_host}, user: {username@servername}, password: {your_password}, database: {your_database}, Port: {your_port}[, ssl:{ca:fs.readFileSync({ca-cert filename})}}]);
```

## <a name="odbc"></a>ODBC
```odbc
DRIVER={MySQL ODBC 5.3 UNICODE Driver};Server={your_host};Port={your_port};Database={your_database};Uid={username@servername};Pwd={your_password}; [sslca={ca-cert filename}; sslverify=1; Option=3;]
```

## <a name="php"></a>PHP
```php
$con=mysqli_init(); [mysqli_ssl_set($con, NULL, NULL, {ca-cert filename}, NULL, NULL);] mysqli_real_connect($con, {your_host}, {username@servername}, {your_password}, {your_database}, {your_port});
```

## <a name="python"></a>Python
```python
cnx = mysql.connector.connect(user={username@servername}, password={your_password}, host={your_host}, port={your_port}, database={your_database}[, ssl_ca={ca-cert filename}, ssl_verify_cert=true])
```

## <a name="ruby"></a>Ruby
```ruby
client = Mysql2::Client.new(username: {username@servername}, password: {your_password}, database: {your_database}, host: {your_host}, port: {your_port}[, sslca:{ca-cert filename}, sslverify:false, sslcipher:'AES256-SHA'])
```

## <a name="get-the-connection-string-details-from-the-azure-portal"></a>从 Azure 门户获取连接字符串详细信息
在 [Azure 门户](https://portal.azure.cn)中，转到 Azure Database for MySQL 服务器，再单击“连接字符串”，以获取实例的字符串列表：:::image type="content" source="./media/howto-connection-strings/connection-strings-on-portal.png" alt-text="Azure 门户中的“连接字符串”窗格":::

字符串提供驱动程序、服务器和其他数据库连接参数等详细信息。 修改这些示例以使用自己的参数，例如数据库名称、密码等。 然后，可以在代码和应用程序中使用此字符串连接到服务器。

## <a name="next-steps"></a>后续步骤
- 有关连接库的详细信息，请参阅[概念 - 连接库](./concepts-connection-libraries.md)。
