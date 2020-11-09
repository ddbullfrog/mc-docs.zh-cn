---
title: 为 Azure Cosmos DB 创建表 API 表（具有自动缩放功能）
description: 为 Azure Cosmos DB 创建表 API 表（具有自动缩放功能）
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: sample
origin.date: 07/29/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: yes
ms.testdate: 08/17/2020
ms.author: v-yeche
ms.openlocfilehash: fb5ad71b9294a1237f75cb79be21400b3dd4a9ce
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328305"
---
# <a name="create-an-azure-cosmos-table-api-account-and-table-with-autoscale-using-azure-cli"></a>使用 Azure CLI 创建 Azure Cosmos 表 API 帐户和表（具有自动缩放功能）
[!INCLUDE[appliesto-table-api](../../../includes/appliesto-table-api.md)]

[!INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../../../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

选择在本地安装并使用 CLI 时，本主题要求运行 Azure CLI 2.9.1 或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI](https://docs.azure.cn/cli/install-azure-cli)。

## <a name="sample-script"></a>示例脚本

```azurecli
#!/bin/bash
# Reference: az cosmosdb | https://docs.azure.cn/cli/cosmosdb
# --------------------------------------------------
#
# Create a Table API table with autoscale
#
#

# Sign in the Azure China Cloud
az cloud set -n AzureChinaCloud
az login

# Generate a unique 10 character alphanumeric string to ensure unique resource names
uniqueId=$(env LC_CTYPE=C tr -dc 'a-z0-9' < /dev/urandom | fold -w 10 | head -n 1)

# Variables for Cassandra API resources
resourceGroupName="Group-$uniqueId"
location='chinanorth2'
accountName="cosmos-$uniqueId" #needs to be lower case
tableName='table1'
maxThroughput=4000 #minimum = 4000

# Create a resource group
az group create -n $resourceGroupName -l $location

# Create a Cosmos account for Table API
az cosmosdb create \
    -n $accountName \
    -g $resourceGroupName \
    --capabilities EnableTable \
    --default-consistency-level Eventual \
    --locations regionName='China North 2' failoverPriority=0 isZoneRedundant=False

# Create a Table API Table with autoscale
az cosmosdb table create \
    -a $accountName \
    -g $resourceGroupName \
    -n $tableName \
    --max-throughput $maxThroughput

```

## <a name="clean-up-deployment"></a>清理部署

运行脚本示例后，可以使用以下命令删除资源组以及与其关联的所有资源。

```azurecli
az group delete --name $resourceGroupName
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.azure.cn/cli/group#az_group_create) | 创建用于存储所有资源的资源组。 |
| [az cosmosdb create](https://docs.azure.cn/cli/cosmosdb#az_cosmosdb_create) | 创建 Azure Cosmos DB 帐户。 |
| [az cosmosdb table create](https://docs.azure.cn/cli/cosmosdb/table#az_cosmosdb_table_create) | 创建 Azure Cosmos 表 API 表。 |
| [az group delete](https://docs.azure.cn/cli/group#az_group_delete) | 删除资源组，包括所有嵌套的资源。 |

<!--CORRECT ON [az cosmosdb table create]-->

## <a name="next-steps"></a>后续步骤

有关 Azure Cosmos DB CLI 的详细信息，请参阅 [Azure Cosmos DB CLI 文档](https://docs.azure.cn/cli/cosmosdb)。

可以在 [Azure Cosmos DB CLI GitHub 存储库](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb)中找到所有 Azure Cosmos DB CLI 脚本示例。

<!-- Update_Description: update meta properties, wording update, update link -->