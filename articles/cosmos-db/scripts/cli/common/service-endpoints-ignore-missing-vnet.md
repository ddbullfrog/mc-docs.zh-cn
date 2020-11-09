---
title: 使用虚拟网络服务终结点连接现有 Azure Cosmos 帐户
description: 使用虚拟网络服务终结点连接现有 Azure Cosmos 帐户
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
origin.date: 07/29/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: yes
ms.testdate: 08/10/2020
ms.author: v-yeche
ms.openlocfilehash: 7f64e592022c5d600561745c259aaf2e6ac606b0
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328026"
---
<!--Verify successfully-->
# <a name="connect-an-existing-azure-cosmos-account-with-virtual-network-service-endpoints-using-azure-cli"></a>通过 Azure CLI 使用虚拟网络服务终结点连接现有 Azure Cosmos 帐户
[!INCLUDE[appliesto-all-apis](../../../includes/appliesto-all-apis.md)]

[!INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../../../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

选择在本地安装并使用 CLI 时，本主题要求运行 Azure CLI 2.9.1 或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI](https://docs.azure.cn/cli/install-azure-cli)。

## <a name="sample-script"></a>示例脚本

此示例旨在说明如何使用 `ignore-missing-vnet-service-endpoint` 参数将现有 Azure Cosmos 帐户连接到尚未为服务终结点配置子网的现有新虚拟网络。 这样就可以在完成对虚拟网络的子网的配置之前，完成 Cosmos 帐户的配置而不会出现错误。 子网配置完成后，便可通过配置的子网访问 Cosmos 帐户。

> [!NOTE]
> 此示例演示如何使用 SQL (Core) API 帐户。 若要将此示例用于其他 API，请将以下脚本中的 `enable-virtual-network` 和 `virtual-network-rules` 参数应用于 API 特定的脚本。

```azurecli
#!/bin/bash
# Reference: az cosmosdb | https://docs.azure.cn/cli/cosmosdb
# --------------------------------------------------
#
# Service endpoint operations for an Azure Cosmos account
#
# Create an Azure Cosmos Account with a service endpoint connected to a backend subnet
# that is not yet enabled for service endpoints.

# This sample demonstrates how to configure service endpoints for existing Cosmos account where
# the connected subnet is not yet configured for service endpoints.
# This sample will then configure the subnet for service endpoints.

# Sign in the Azure China Cloud
az cloud set -n AzureChinaCloud
az login

# Generate a unique 10 character alphanumeric string to ensure unique resource names
uniqueId=$(env LC_CTYPE=C tr -dc 'a-z0-9' < /dev/urandom | fold -w 10 | head -n 1)

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

# Create a back-end subnet but without specifying --service-endpoints Microsoft.AzureCosmosDB
az network vnet subnet create \
    -n $backEnd \
    -g $resourceGroupName \
    --address-prefix 10.0.2.0/24 \
    --vnet-name $vnetName

svcEndpoint=$(az network vnet subnet show -g $resourceGroupName -n $backEnd --vnet-name $vnetName --query 'id' -o tsv)

# Create a Cosmos DB account with default values
# Use appropriate values for --kind or --capabilities for other APIs
az cosmosdb create -n $accountName -g $resourceGroupName

# Add the virtual network rule but ignore the missing service endpoint on the subnet
az cosmosdb network-rule add \
    -n $accountName \
    -g $resourceGroupName \
    --virtual-network $vnetName \
    --subnet svcEndpoint \
    --ignore-missing-vnet-service-endpoint true

read -p'Press any key to configure the subnet for service endpoints'

az network vnet subnet update \
    -n $backEnd \
    -g $resourceGroupName \
    --vnet-name $vnetName \
    --service-endpoints Microsoft.AzureCosmosDB

```

<!--MOONCAKE: CORRECT ON LINE 78 az cosmosdb NETWORK-rule add-->

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
| [az network vnet subnet update](https://docs.azure.cn/cli/network/vnet/subnet#az_network_vnet_subnet_update) | 更新 Azure 虚拟网络的子网。 |
| [az group delete](https://docs.azure.cn/cli/group#az_group_delete) | 删除资源组，包括所有嵌套的资源。 |

## <a name="next-steps"></a>后续步骤

有关 Azure Cosmos DB CLI 的详细信息，请参阅 [Azure Cosmos DB CLI 文档](https://docs.azure.cn/cli/cosmosdb)。

可以在 [Azure Cosmos DB CLI GitHub 存储库](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb)中找到所有 Azure Cosmos DB CLI 脚本示例。

<!-- Update_Description: update meta properties, wording update, update link -->