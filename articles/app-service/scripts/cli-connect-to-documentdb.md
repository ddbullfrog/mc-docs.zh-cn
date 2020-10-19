---
title: CLI：将应用连接到 Cosmos DB
description: 了解如何使用 Azure CLI 自动部署和管理应用服务应用。 此示例演示如何将应用连接到 MongoDB (Cosmos DB)。
author: msangapu-msft
tags: azure-service-management
ms.assetid: bbbdbc42-efb5-4b4f-8ba6-c03c9d16a7ea
ms.devlang: azurecli
ms.topic: sample
origin.date: 12/11/2017
ms.date: 10/19/2020
ms.author: v-tawe
ms.custom: mvc, seodec18
ms.openlocfilehash: b4d4cf8d782783be2294e4ae80a228adb4b97fb8
ms.sourcegitcommit: e2e418a13c3139d09a6b18eca6ece3247e13a653
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/19/2020
ms.locfileid: "92170470"
---
# <a name="connect-an-app-service-app-to-cosmos-db-using-cli"></a>使用 CLI 将应用服务应用连接到 Cosmos DB

此示例脚本使用 Azure Cosmos DB 的用于 MongoDB 的 API 和应用服务应用来创建一个 Azure Cosmos DB 帐户。 然后，它将使用应用设置将 MongoDB 连接字符串链接到 Web 应用。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]



如果选择在本地安装并使用 CLI，需要 Azure CLI 2.0 版或更高版本。 要查找版本，请运行 `az --version`。 如果需要进行安装或升级，请参阅[安装 Azure CLI](/cli/install-azure-cli?view=azure-cli-lastest)。

## <a name="sample-script"></a>示例脚本

```azurecli
#/bin/bash

# Variables
appName="webappwithcosmosdb$RANDOM"
location="ChinaNorth"

# Create a Resource Group 
az group create --name myResourceGroup --location $location

# Create an App Service Plan
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --location $location
# Create a Web App
az webapp create --name $appName --plan myAppServicePlan --resource-group myResourceGroup 

# Create a Cosmos DB with MongoDB API
az cosmosdb create --name $appName --resource-group myResourceGroup --kind MongoDB

# Get the MongoDB URL
connectionString=$(az cosmosdb list-connection-strings --name $appName --resource-group myResourceGroup \
--query connectionStrings[0].connectionString --output tsv)

# Assign the connection string to an App Setting in the Web App
az webapp config appsettings set --name $appName --resource-group myResourceGroup \
--settings "MONGODB_URL=$connectionString" 
```

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、应用服务应用、Cosmos DB 和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| Command | 说明 |
|---|---|
| [`az group create`](/cli/group?view=azure-cli-latest#az-group-create) | 创建用于存储所有资源的资源组。 |
| [`az appservice plan create`](/cli/appservice/plan?view=azure-cli-latest#az-appservice-plan-create) | 创建应用服务计划。 |
| [`az webapp create`](/cli/webapp?view=azure-cli-latest#az-webapp-create) | 创建应用服务应用。 |
| [`az cosmosdb create`](/cli/cosmosdb?view=azure-cli-latest#az-cosmosdb-create) | 创建 Cosmos DB 帐户。 |
| [`az cosmosdb list-connection-strings`](/cli/cosmosdb?view=azure-cli-latest#az-cosmosdb-list-connection-strings) | 列出指定 Cosmos DB 帐户的连接字符串。 |
| [`az webapp config appsettings set`](/cli/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) | 创建或更新应用服务应用的应用设置。 应用设置将作为应用的环境变量公开。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](/cli/overview?view=azure-cli-lastest)。

可以在 [Azure 应用服务文档](../samples-cli.md)中找到其他应用服务 CLI 脚本示例。
