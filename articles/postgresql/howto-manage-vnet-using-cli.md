---
title: 使用虚拟网络规则 - Azure CLI - Azure Database for PostgreSQL - 单一服务器
description: 本文介绍如何使用 Azure CLI 命令行为 Azure Database for PostgreSQL 创建和管理 VNet 服务终结点和规则。
author: WenJason
ms.author: v-jay
ms.service: postgresql
ms.devlang: azurecli
ms.topic: how-to
origin.date: 5/6/2019
ms.date: 11/09/2020
ms.openlocfilehash: ab026f8ee4ef77ba1eeed48cde084c15730185b7
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328675"
---
# <a name="create-and-manage-vnet-service-endpoints-for-azure-database-for-postgresql---single-server-using-azure-cli"></a>使用 Azure CLI 为 Azure Database for PostgreSQL 创建和管理 VNet 服务终结点 - 单一服务器
虚拟网络 (VNet) 服务终结点和规则将虚拟网络的专用地址空间扩展到你的 Azure Database for PostgreSQL 服务器。 使用便捷的 Azure 命令行接口 (CLI) 命令，可创建、更新、删除、列出和显示 VNet 服务终结点和规则，用于管理服务器。 有关 Azure Database for PostgreSQL VNet 服务终结点（包括限制）的概述，请参阅 [Azure Database for PostgreSQL Server VNet 服务终结点](concepts-data-access-and-security-vnet.md)。 在 Azure Database for PostgreSQL 的所有支持区域中，VNet 服务终结点均可用。

## <a name="prerequisites"></a>先决条件
若要逐步执行本操作方法指南，需要：
- 安装 [Azure CLI](/cli/install-azure-cli) 命令行实用程序。
- [Azure Database for PostgreSQL 服务器和数据库](quickstart-create-server-database-azure-cli.md)。

> [!NOTE]
> 只有常规用途和内存优化服务器才支持 VNet 服务终结点。
> 在 VNet 对等互连的情况下，如果流量通过具有服务终结点的公共 VPN 网关流动，并且应该流向对等机，请创建 ACL/VNet 规则，以便网关 VNet 中的 Azure 虚拟机能够访问 Azure Database for PostgreSQL 服务器。


## <a name="configure-vnet-service-endpoints-for-azure-database-for-postgresql"></a>配置 Azure Database for PostgreSQL Vnet 服务终结点
[az network vnet](https://docs.azure.cn/cli/network/vnet) 命令用于配置虚拟网络。

如果没有 Azure 订阅，请在开始前创建一个[试用帐户](https://www.azure.cn/zh-cn/pricing/1rmb-trial-full/?form-type=identityauth)。

如果选择在本地安装并使用 CLI，本文要求运行 Azure CLI 2.0 版或更高版本。 若要查看安装的版本，请运行 `az --version` 命令。 如果需要进行安装或升级，请参阅[安装 Azure CLI](/cli/install-azure-cli)。 

如果在本地运行 CLI，需要使用 [az login](/cli/authenticate-azure-cli) 命令登录帐户。 记下与订阅名称相对应的命令输出中的 **id** 属性。
```cli
az login
```

如果有多个订阅，请选择应计费的资源所在的相应订阅。 使用 [az account set](/cli/account#az-account-set) 命令选择帐户下的特定订阅 ID。 用订阅的 **az login** 输出中的 **id** 属性代替订阅 id 占位符。

- 该帐户必须拥有创建虚拟网络和服务终结点所需的必要权限。

对虚拟网络拥有写入访问权限的用户可在虚拟网络上单独配置服务终结点。

若要在 VNet 中保护 Azure 服务资源，用户必须对所添加的子网拥有“Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/”权限。 此权限默认包含在内置的服务管理员角色中，可以通过创建自定义角色进行修改。

详细了解[内置角色](../role-based-access-control/built-in-roles.md)以及将特定的权限分配到[自定义角色](../role-based-access-control/custom-roles.md)。

VNet 和 Azure 服务资源可以位于相同或不同的订阅中。 如果 VNet 和 Azure 服务资源位于不同的订阅中，资源应在相同的 Active Directory (AD) 租户下。 确保两个订阅都注册了 Microsoft.Sql 资源提供程序。 有关详细信息，请参阅[资源管理器注册][resource-manager-portal]

> [!IMPORTANT]
> 强烈建议在运行下面的示例脚本或配置服务终结点前先阅读本文有关服务终结点配置和注意事项的内容。 **虚拟网络服务终结点：** [虚拟网络服务终结点](../virtual-network/virtual-network-service-endpoints-overview.md)是一个子网，其属性值包括一个或多个正式的 Azure 服务类型名称。 VNet 服务终结点使用服务类型名称 Microsoft.Sql，可引用名为“SQL 数据库”的 Azure 服务。 此服务标记也适用于 Azure SQL 数据库、Azure Database for PostgreSQL 和 MySQL 服务。 请务必要注意，对 VNet 服务终结点应用 Microsoft.Sql 服务标记时，它会为所有 Azure 数据库服务配置服务终结点流量，其中包括 Azure SQL 数据库、Azure Database for PostgreSQL 和子网上的 Azure Database for MySQL 服务器。 
> 

### <a name="sample-script-to-create-an-azure-database-for-postgresql-database-create-a-vnet-vnet-service-endpoint-and-secure-the-server-to-the-subnet-with-a-vnet-rule"></a>示例脚本演示了如何创建 Azure Database for PostgreSQL 数据库，创建 VNet 和 VNet 服务终结点，以及如何使用 VNet 规则在子网中保护服务器
在此示例脚本中，更改突出显示的行，以自定义管理员用户名和密码。 将 `az account set --subscription` 命令中使用的 SubscriptionID 替换为你自己的订阅标识符。

```cli
#!/bin/bash

# To find the name of an Azure region in the CLI run this command: az account list-locations
# Substitute  <subscription id> with your identifier
az account set --subscription <subscription id>

# Create a resource group
az group create \
--name myresourcegroup \
--location chinaeast

# Create a PostgreSQL server in the resource group
# Name of a server maps to DNS name and is thus required to be globally unique in Azure.
# Substitute the <server_admin_password> with your own value.
az postgres server create \
--name mydemoserver \
--resource-group myresourcegroup \
--location chinaeast \
--admin-user mylogin \
--admin-password <server_admin_password> \
--sku-name GP_Gen4_2

# Get available service endpoints for Azure region output is JSON
# Use the command below to get the list of services supported for endpoints, for an Azure region, say "chinaeast".
az network vnet list-endpoint-services \
-l chinaeast

# Add Azure SQL service endpoint to a subnet *mySubnet* while creating the virtual network *myVNet* output is JSON
az network vnet create \
-g myresourcegroup \
-n myVNet \
--address-prefixes 10.0.0.0/16 \
-l chinaeast

# Creates the service endpoint
az network vnet subnet create \
-g myresourcegroup \
-n mySubnet \
--vnet-name myVNet \
--address-prefix 10.0.1.0/24 \
--service-endpoints Microsoft.SQL

# View service endpoints configured on a subnet
az network vnet subnet show \
-g myresourcegroup \
-n mySubnet \
--vnet-name myVNet

# Create a VNet rule on the sever to secure it to the subnet. Note: resource group (-g) parameter is where the database exists. VNet resource group if different should be specified using subnet id (URI) instead of subnet, VNet pair.
az postgres server vnet-rule create \
-n myRule \
-g myresourcegroup \
-s mydemoserver \
--vnet-name myVNet \
--subnet mySubnet
```

## <a name="clean-up-deployment"></a>清理部署
运行脚本示例后，可以使用以下命令删除资源组以及与其关联的所有资源。

```bash
#!/bin/bash
az group delete --name myresourcegroup
```

<!-- Link references, to text, Within this same GitHub repo. --> 
[resource-manager-portal]: ../azure-resource-manager/management/resource-providers-and-types.md