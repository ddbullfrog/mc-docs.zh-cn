---
title: 配置服务器参数 - Azure CLI - Azure Database for MySQL
description: 本文介绍如何使用 Azure CLI 命令行实用工具在 Azure Database for MySQL 中配置服务参数。
author: WenJason
ms.author: v-jay
ms.service: mysql
ms.devlang: azurecli
ms.topic: how-to
origin.date: 10/1/2020
ms.date: 10/29/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 6aa88c6951ebfe64e7037b78ff9e69e03e552842
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92470481"
---
# <a name="configure-server-parameters-in-azure-database-for-mysql-using-the-azure-cli"></a>使用 Azure CLI 在 Azure Database for MySQL 中配置服务器参数

> [!NOTE]
> 将要查看的是 Azure Database for MySQL 的新服务。 若要查看经典 MySQL Database for Azure 的文档，请访问[此页](https://docs.azure.cn/zh-cn/mysql-database-on-azure/)。

可以使用 Azure CLI、Azure 命令行实用工具来列出、显示和更新 Azure Database for MySQL 服务器的配置参数。 在服务器级别会公开引擎配置的一个子集，并可以进行修改。 

>[!Note]
> 可在服务器级别全局更新服务器参数，只需使用 [Azure CLI](./howto-configure-server-parameters-using-cli.md)、[PowerShell](./howto-configure-server-parameters-using-powershell.md) 或 [Azure 门户](./howto-server-parameters.md)即可

## <a name="prerequisites"></a>先决条件
若要逐步执行本操作方法指南，需要：
- [Azure Database for MySQL 服务器](quickstart-create-mysql-server-database-using-azure-cli.md)
- [Azure CLI](/cli/install-azure-cli) 命令行实用工具。

## <a name="list-server-configuration-parameters-for-azure-database-for-mysql-server"></a>列出 Azure Database for MySQL 服务器的服务器配置参数
若要列出服务器中的所有可修改参数及其值，请运行 [az mysql server configuration list](/cli/mysql/server/configuration#az-mysql-server-configuration-list) 命令。

可以列出资源组“myresourcegroup”下服务器 **mydemoserver.mysql.database.chinacloudapi.cn** 的服务器配置参数。
```cli
az mysql server configuration list --resource-group myresourcegroup --server mydemoserver
```
有关每个列出参数的定义，请参阅[服务器系统变量](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html)上的 MySQL 引用部分。

## <a name="show-server-configuration-parameter-details"></a>显示服务器配置参数详细信息
若要显示服务器的某个特定配置参数的详细信息，请运行 [az mysql server configuration show](/cli/mysql/server/configuration#az-mysql-server-configuration-show) 命令。

本示例显示了资源组“myresourcegroup”下服务器 **mydemoserver.mysql.database.chinacloudapi.cn** 的服务器配置参数 **slow\_query\_log** 的详细信息。
```cli
az mysql server configuration show --name slow_query_log --resource-group myresourcegroup --server mydemoserver
```
## <a name="modify-a-server-configuration-parameter-value"></a>修改服务器配置参数值
此外，你还可以修改某个服务器配置参数的值，这会更新 MySQL 服务器引擎的基础配置值。 若要更新配置，请使用 [az mysql server configuration set](/cli/mysql/server/configuration#az-mysql-server-configuration-set) 命令。 

更新资源组“myresourcegroup”下服务器 **mydemoserver.mysql.database.chinacloudapi.cn** 的服务器配置参数 **slow\_query\_log** 。
```cli
az mysql server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver --value ON
```
若要重置配置参数的值，省去可选的 `--value` 参数，服务将应用默认值。 对于上述示例，它将如下所示：
```cli
az mysql server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver
```
此代码会将 slow\_query\_log 配置重置为默认值 OFF。 

## <a name="setting-parameters-not-listed"></a>设置参数未列出
如果 Azure 门户中未列出你要更新的服务器参数，则可以选择性地使用 `init_connect` 在连接级别设置参数。 此项可为每个连接到服务器的客户端设置服务器参数。 

更新资源组 myresourcegroup 下的服务器 mydemoserver.mysql.database.chinacloudapi.cn 的 init\_connect 服务器配置参数，以设置字符集之类的值  。
```azurecli
az mysql server configuration set --name init_connect --resource-group myresourcegroup --server mydemoserver --value "SET character_set_client=utf8;SET character_set_database=utf8mb4;SET character_set_connection=latin1;SET character_set_results=latin1;"
```

## <a name="working-with-the-time-zone-parameter"></a>使用时区参数

### <a name="populating-the-time-zone-tables"></a>填充时区表

可以通过从 MySQL 命令行或 MySQL Workbench 等工具调用 `mysql.az_load_timezone` 存储过程，填充服务器上的时区表。

> [!NOTE]
> 如果正在运行 MySQL Workbench 中的 `mysql.az_load_timezone` 命令，可能需要先使用 `SET SQL_SAFE_UPDATES=0;` 关闭安全更新模式。

```sql
CALL mysql.az_load_timezone();
```

> [!IMPORTANT]
> 应重启服务器，确保正确填充时区表。 要重启服务器，请使用 [Azure 门户](howto-restart-server-portal.md)或 [CLI](howto-restart-server-cli.md)。

要查看可用的时区值，请运行以下命令：

```sql
SELECT name FROM mysql.time_zone_name;
```

### <a name="setting-the-global-level-time-zone"></a>设置全局级时区

可以使用 [az mysql server configuration set](/cli/mysql/server/configuration#az-mysql-server-configuration-set) 命令来设置全局级时区。

以下命令将资源组 myresourcegroup 下的服务器 mydemoserver.mysql.database.chinacloudapi.cn 的服务器配置参数 time\_zone 更新为 Asia/Shanghai   。

```azurecli
az mysql server configuration set --name time_zone --resource-group myresourcegroup --server mydemoserver --value "Asia/Shanghai"
```

### <a name="setting-the-session-level-time-zone"></a>设置会话级时区

可以通过从 MySQL 命令行或 MySQL Workbench 等工具运行 `SET time_zone` 命令来设置会话级时区。 以下示例将时区设置为“亚洲/上海”时区。  

```sql
SET time_zone = 'Asia/Shanghai';
```

若要了解[日期和时间函数](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_convert-tz)，请参阅 MySQL 文档。


## <a name="next-steps"></a>后续步骤

- 如何配置 [Azure 门户中的服务器参数](howto-server-parameters.md)