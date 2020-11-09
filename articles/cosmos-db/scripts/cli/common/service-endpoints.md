---
title: 创建具有虚拟网络服务终结点的 Azure Cosmos 帐户
description: 创建具有虚拟网络服务终结点的 Azure Cosmos 帐户
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
origin.date: 07/29/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: yes
ms.testdate: 08/10/2020
ms.author: v-yeche
ms.openlocfilehash: 32d67fea62c8c2f3e6b402910467a6c0f29f7ea1
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328086"
---
<!--Verify successfully-->
# <a name="create-an-azure-cosmos-account-with-virtual-network-service-endpoints-using-azure-cli"></a>使用 Azure CLI 创建具有虚拟网络服务终结点的 Azure Cosmos 帐户
[!INCLUDE[appliesto-all-apis](../../../includes/appliesto-all-apis.md)]

[!INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../../../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

如果选择在本地安装并使用 CLI，此命令需要运行 Azure CLI 2.9.1 版或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI](https://docs.azure.cn/cli/install-azure-cli)。

## <a name="sample-script"></a>示例脚本

此示例创建一个具有前端子网和后端子网的新虚拟网络，并为 `Microsoft.AzureCosmosDB` 启用服务终结点。 然后，它将检索此子网的资源 ID 并将其应用到 Azure Cosmos 帐户，并为该帐户启用服务终结点。

> [!NOTE]
> 此示例演示如何使用 Core (SQL) API 帐户。 若要将此示例用于其他 API，请将以下脚本中的 `enable-virtual-network` 和 `virtual-network-rules` 参数应用于 API 特定的脚本。

```azurecli
#!/bin/bash
# Reference: az cosmosdb | https://docs.azure.cn/cli/cosmosdb
# --------------------------------------------------
#
# Service endpoint operations for an Azure Cosmos account
#
# Create an Azure Cosmos Account with a service endpoint connected to a backend subnet
#

# Generate a unique 10 character alphanumeric string to ensure unique resource names
uniqueId=$(env LC_CTYPE=C tr -dc 'a-z0-9' < /dev/urandom | fold -w 10 | head -n 1)

# Sign in the Azure China Cloud
az cloud set -n AzureChinaCloud
az login

# Resource group and Cosmos account variables
resourceGroupName="Group-$uniqueId"
location='chinanorth2'
accountName="cosmos-$uniqueId" #needs to be lower case

# Variables for a new Virtual Network with two subnets
vnetName='myVnet'
frontEnd='FrontEnd'
backEnd='BackEnd'

# Create a resource group
az group create -n $resourceGroupName -l $location

# Create a virtual network with a front-end subnet
az network vnet create \
    -n $vnetName \
    -g $resourceGroupName \
    --address-prefix 10.0.0.0/16 \
    --subnet-name $frontEnd \
    --subnet-prefix 10.0.1.0/24

# Create a back-end subnet
az network vnet subnet create \
    -n $backEnd \
    -g $resourceGroupName \
    --address-prefix 10.0.2.0/24 \
    --vnet-name $vnetName \
    --service-endpoints Microsoft.AzureCosmosDB

svcEndpoint=$(az network vnet subnet show -g $resourceGroupName -n $backEnd --vnet-name $vnetName --query 'id' -o tsv)

# Create a Cosmos DB account with default values and service endpoints
# Use appropriate values for --kind or --capabilities for other APIs
az cosmosdb create \
    -n $accountName \
    -g $resourceGroupName \
    --enable-virtual-network true \
    --virtual-network-rules $svcEndpoint

```

## <a name="clean-up-deployment"></a>清理部署

运行脚本示例后，可以使用以下命令删除资源组以及与其关联的所有资源。

```azurecli
az group delete --name $resourceGroupName
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| Command | 说明 |
|---|---|
| [az group create](https://docs.azure.cn/cli/group#az_group_create) | 创建用于存储所有资源的资源组。 |
| [az network vnet create](https://docs.azure.cn/cli/network/vnet#az_network_vnet_create) | 创建 Azure 虚拟网络。 |
| [az network vnet subnet create](https://docs.azure.cn/cli/network/vnet/subnet#az_network_vnet_subnet_create) | 为 Azure 虚拟网络创建子网。 |
| [az network vnet subnet show](https://docs.azure.cn/cli/network/vnet/subnet#az_network_vnet_subnet_show) | 返回 Azure 虚拟网络的子网。 |
| [az cosmosdb create](https://docs.azure.cn/cli/cosmosdb#az_cosmosdb_create) | 创建 Azure Cosmos DB 帐户。 |
| [az group delete](https://docs.azure.cn/cli/group#az_group_delete) | 删除资源组，包括所有嵌套的资源。 |

## <a name="next-steps"></a>后续步骤

有关 Azure Cosmos DB CLI 的详细信息，请参阅 [Azure Cosmos DB CLI 文档](https://docs.azure.cn/cli/cosmosdb)。

可以在 [Azure Cosmos DB CLI GitHub 存储库](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb)中找到所有 Azure Cosmos DB CLI 脚本示例。

<!-- Update_Description: update meta properties, wording update, update link -->