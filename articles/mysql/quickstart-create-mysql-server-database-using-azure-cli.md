---
title: 快速入门：创建服务器 - Azure CLI - Azure Database for MySQL
description: 本快速入门教程介绍如何使用 Azure CLI 在 Azure 资源组中为 MySQL 服务器创建 Azure 数据库。
author: WenJason
ms.author: v-jay
ms.service: mysql
ms.devlang: azurecli
ms.topic: quickstart
origin.date: 07/15/2020
ms.date: 11/09/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: cf32e54c225efb80bfbf6557b3e870d4a144649e
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328881"
---
# <a name="quickstart-create-an-azure-database-for-mysql-server-using-azure-cli"></a>快速入门：使用 Azure CLI 为 MySQL 服务器创建 Azure 数据库

> [!NOTE]
> 将要查看的是 Azure Database for MySQL 的新服务。 若要查看经典 MySQL Database for Azure 的文档，请访问[此页](https://docs.azure.cn/zh-cn/mysql-database-on-azure/)。

> [!TIP]
> 考虑使用更简单的 [az mysql up](https://docs.microsoft.com/cli/azure/ext/db-up/mysql#ext-db-up-az-mysql-up) Azure CLI 命令（当前处于预览状态）。 试用[快速入门](./quickstart-create-server-up-azure-cli.md)。

本快速入门教程介绍如何使用 [Azure CLI](/cli/get-started-with-azure-cli) 在大约 5 分钟内创建 Azure Database for MySQL 服务器。 如果没有 Azure 订阅，请在开始前创建一个[试用帐户](https://wd.azure.cn/pricing/1rmb-trial-full/?form-type=identityauth)。

> [!NOTE]
> 本文要求运行 Azure CLI 2.0 或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI](/cli/install-azure-cli)。 

## <a name="prerequisites"></a>先决条件
本文要求在本地运行 Azure CLI 2.0 或更高版本。 若要查看安装的版本，请运行 `az --version` 命令。 如果需要进行安装或升级，请参阅[安装 Azure CLI](/cli/install-azure-cli)。

你将需要使用 [az login](/cli/reference-index#az-login) 命令登录到你的帐户。 请注意 id 属性，该属性指的是 Azure 帐户的订阅 ID。 

```azurecli
az login
```

使用 [az account set](/cli/account) 命令选择帐户下的特定订阅。 记下 az login 输出中的 id 值，以用作命令中订阅参数的值。 如果有多个订阅，请选择应计费的资源所在的相应订阅。 若要获取所有订阅，请使用 [az account list](/cli/account#az-account-list)。

```azurecli
az account set --subscription <subscription id>
```

## <a name="create-an-azure-database-for-mysql-server"></a>创建 Azure Database for MySQL 服务器
使用 [az group create](/cli/group) 命令创建 [Azure 资源组](../azure-resource-manager/management/overview.md)，然后在此资源组中创建 MySQL 服务器。 应提供唯一名称。 以下示例在 `chinaeast2` 位置创建名为 `myresourcegroup` 的资源组。

```azurecli
az group create --name myresourcegroup --location chinaeast2
```

使用 [az mysql server create](/cli/mysql/server#az-mysql-server-create) 命令为 MySQL 服务器创建 Azure 数据库。 一台服务器可以管理多个数据库。

```azurecli
az mysql server create --resource-group myresourcegroup --name mydemoserver --location chinaeast2 --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 
```

下面是上述参数的详细信息： 

**设置** | **示例值** | **说明**
---|---|---
name | mydemoserver | 输入 Azure Database for MySQL 服务器的唯一名称。 服务器名称只能包含小写字母、数字和连字符 (-) 字符。 必须包含 3 到 63 个字符。
resource-group | myresourcegroup | 提供 Azure 资源组的名称。
location | chinaeast2 | 服务器的 Azure 位置。
admin-user | myadmin | 管理员的登录用户名。 不能是 **azure_superuser** 、 **admin** 、 **administrator** 、 **root** 、 **guest** 或 **public** 。
admin-password | *安全密码* | 管理员用户的密码。 该密码必须包含 8 到 128 个字符。 密码必须包含以下三个类别的字符：英文大写字母、英文小写字母、数字和非字母数字字符。
sku-name|GP_Gen5_2|输入定价层和计算配置的名称。 请遵循简写约定 {pricing tier} _{compute generation}_ {vCores}。 有关详细信息，请参阅[定价层](./concepts-pricing-tiers.md)。

>[!IMPORTANT] 
>- 服务器上的默认 MySQL 版本为 5.7。 目前还提供了 5.6 和 8.0 版本。
>- 若要查看 **az mysql server create** 命令的所有参数，请参阅此 [参考文档](/cli/mysql/server#az-mysql-server-create)。
>- 默认情况下，SSL 在服务器上启用。 有关 SSL 的详细信息，请参阅[配置 SSL 连接](howto-configure-ssl.md)

## <a name="configure-a-server-level-firewall-rule"></a>配置服务器级防火墙规则 
默认情况下，创建的新服务器使用防火墙规则进行保护，并且无法公开访问。 可以使用 [az mysql server firewall-rule create](/cli/mysql/server/firewall-rule) 命令在服务器上配置防火墙规则。 这将允许你在本地连接到服务器。

以下示例创建名为 `AllowMyIP` 的防火墙规则，该规则允许从特定的 IP 地址 (192.168.0.1) 进行连接。 替换将从其连接的 IP 地址。 如果需要，可以使用一系列 IP 地址。 不知道如何查找你的 IP，则转到 [https://whatismyipaddress.com/](https://whatismyipaddress.com/) 获取 IP 地址。

```azurecli
az mysql server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address 192.168.0.1 --end-ip-address 192.168.0.1
```

> [!NOTE]
> 连接到 Azure Database for MySQL 时，经端口 3306 进行通信。 如果尝试从企业网络内部进行连接，则可能不允许经端口 3306 的出站流量。 如果是这样，则无法连接到服务器，除非 IT 部门打开了端口 3306。

## <a name="get-the-connection-information"></a>获取连接信息

若要连接到服务器，需要提供主机信息和访问凭据。

```azurecli
az mysql server show --resource-group myresourcegroup --name mydemoserver
```

结果采用 JSON 格式。 记下 fullyQualifiedDomainName  和 administratorLogin  。
```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.mysql.database.chinacloudapi.cn",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMySQL/servers/mydemoserver",
  "location": "chinaeast2",
  "name": "mydemoserver",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 2,
    "family": "Gen5",
    "name": "GP_Gen5_2",
    "size": null,
    "tier": "GeneralPurpose"
  },
  "sslEnforcement": "Enabled",
  "storageProfile": {
    "backupRetentionDays": 7,
    "geoRedundantBackup": "Disabled",
    "storageMb": 5120
  },
  "tags": null,
  "type": "Microsoft.DBforMySQL/servers",
  "userVisibleState": "Ready",
  "version": "5.7"
}
```

## <a name="connect-to-azure-database-for-mysql-server-using-mysql-command-line-client"></a>使用 mysql 命令行客户端连接到 Azure Database for MySQL 服务器
可以在本地环境中使用常用的客户端工具（[mysql.exe](https://dev.mysql.com/downloads/) 命令行工具）连接到服务器。
```bash
   mysql -h mydemoserver.mysql.database.chinacloudapi.cn -u myadmin@mydemoserver -p
```

## <a name="clean-up-resources"></a>清理资源
如果不需要将这些资源用于其他快速入门/教程，则可通过执行以下命令将其删除： 

```azurecli
az group delete --name myresourcegroup
```

若要删除新创建的服务器，可运行 [az mysql server delete](/cli/mysql/server#az-mysql-server-delete) 命令。

```azurecli
az mysql server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
>[使用 MySQL 在 Windows 上生成 PHP 应用](../app-service/tutorial-php-mysql-app.md)
>[使用 MySQL 在 Linux 上生成 PHP 应用](../app-service/tutorial-php-mysql-app.md?pivots=platform-linux%253fpivots%253dplatform-linux)
>[使用 MySQL 生成基于 Java 的 Spring 应用](https://docs.microsoft.com/azure/developer/java/spring-framework/spring-app-service-e2e?tabs=bash)