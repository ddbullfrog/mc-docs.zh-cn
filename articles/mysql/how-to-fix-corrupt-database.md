---
title: 解决数据库损坏问题 - Azure Database for MySQL
description: 了解如何修复 Azure Database for MySQL 的数据库损坏问题
author: WenJason
ms.author: v-jay
ms.service: mysql
ms.topic: how-to
origin.date: 09/21/2020
ms.date: 10/19/2020
ms.openlocfilehash: 99e1fc9e889e156f7744d5d8bc4478ba4971543e
ms.sourcegitcommit: ba01e2d1882c85ebeffef344ef57afaa604b53a0
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/14/2020
ms.locfileid: "92041931"
---
# <a name="troubleshoot-database-corruption-on-azure-database-for-mysql"></a>排查 Azure Database for MySQL 的数据库损坏问题

数据库损坏可能会导致应用程序停机，及时解决该问题以避免数据丢失也是至关重要的。 发生数据库损坏时，你在服务器日志中会看到此错误： **InnoDB:磁盘上的数据库页面损坏或故障** 。

在此指南中，你将了解当数据库或表损坏时如何进行修复。 Azure Database for MySQL 使用 InnoDB 引擎，它能够自动检查损坏并执行修复操作。 InnoDB 通过对其读取的每个页面执行校验和来检查是否存在损坏的页面。如果发现校验和差异，它会自动停止 MySQL 服务器。

请尝试下列任一选项，以便快速缓解数据库损坏问题。

## <a name="restart-your-mysql-server"></a>重启 MySQL 服务器

你经常会注意到，当应用程序访问某个数据库或表时，该数据库或表已损坏。 由于 InnoDB 有一种崩溃恢复机制，因此可以在重启服务器时解决大多数问题。 因此，重启服务器应该有助于服务器从导致数据库处于错误状态的故障中恢复。

##  <a name="resolve-data-corruption-with-dump-and-restore-method"></a>用转储和还原方法解决数据损坏问题

建议使用 **转储和还原方法** 解决损坏问题。 这涉及到访问损坏的表，使用 **mysqldump** 实用工具来创建该表的逻辑备份，这会保留表结构和其中的数据，然后将该表重新加载到数据库中。

### <a name="back-up-your-database-or-tables"></a>备份数据库或表

> [!Important]
> - 确保你已配置了防火墙规则，以便从客户端计算机访问服务器。 请参阅[如何配置单一服务器上的防火墙规则](howto-manage-firewall-using-portal.md)。
> - 如果已启用 SSL，请将 SSL 选项 ```--ssl-cert``` 用于 **mysqldump**

从命令行使用此命令通过 mysqldump 创建备份文件

```
$ mysqldump [--ssl-cert=/path/to/pem] -h [host] -u [uname] -p[pass] [dbname] > [backupfile.sql]
```

需要提供的参数包括：
- [ssl-cert=/path/to/pem] 在客户端计算机上下载 SSL 证书，并通过命令在此项中设置路径。 如果 SSL 被禁用，则不要使用此项。
- [host] 是你的 Azure Database for MySQL 服务器
- [uname] 是你的服务器管理员用户名
- [pass] 是你的管理员用户的密码
- [dbname] 是你的数据库的名称
- [backupfile.sql] 是你的数据库备份的文件名

> [!Important]
> - 对于单一服务器，请使用 ```admin-user@servername``` 格式来替换下面命令中的 ```myserveradmin```。
> - 对于灵活服务器，请使用 ```admin-user``` 格式来替换下面命令中的 ```myserveradmin```。

如果某个特定表已损坏，请使用此示例选择数据库中要备份的特定表
```
$ mysqldump --ssl-cert=</path/to/pem> -h mydemoserver.mysql.database.chinacloudapi.cn -u myserveradmin -p testdb table1 table2 > testdb_tables_backup.sql
```

若要备份一个或多个数据库，请使用--database 开关并列出以空格分隔的数据库名称。

```
$ mysqldump --ssl-cert=</path/to/pem>  -h mydemoserver.mysql.database.chinacloudapi.cn -u myserveradmin -p --databases testdb1 testdb3 testdb5 > testdb135_backup.sql
```

###  <a name="restore-your-database-or-tables"></a>还原数据库或表

以下步骤展示了如何还原数据库或表。 创建备份文件后，可以使用 * **mysql** 实用工具还原表或数据库。 运行如下所示的命令：

```
mysql  --ssl-cert=</path/to/pem> -h [hostname] -u [uname] -p[pass] [db_to_restore] < [backupfile.sql]
```
下面的示例说明了如何从通过 **mysqldump** 创建的备份文件还原 ```testdb```。 

> [!Important]
> - 对于单一服务器，请使用 ```admin-user@servername``` 格式来替换下面命令中的 ```myserveradmin```。
> - 对于灵活服务器，请使用 ```admin-user``` 格式来替换下面命令中的 ```myserveradmin```。 

```
$ mysql --ssl-cert=</path/to/pem> -h mydemoserver.mysql.database.chinacloudapi.cn -u myserveradmin -p testdb < testdb_backup.sql
```

## <a name="next-steps"></a>后续步骤
如果以上步骤无助于解决问题，你始终可以还原整个服务器。
- [还原 Azure Database for MySQL 单一服务器](howto-restore-server-portal.md)



