---
title: 为实现应用程序的高可用性路由流量 - Azure CLI - 流量管理器
description: Azure CLI 脚本示例 - 为实现应用程序的高可用性路由流量
services: traffic-manager
documentationcenter: traffic-manager
manager: twooley
tags: azure-infrastructure
ms.assetid: ''
ms.service: traffic-manager
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: traffic-manager
origin.date: 04/26/2018
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: no
ms.testdate: 09/28/2020
ms.author: v-yeche
ms.openlocfilehash: 1a788c8d4a8dd544d9ac851720493eaa221d982f
ms.sourcegitcommit: 71953ae66ddfc07c5d3b4eb55ff8639281f39b40
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2020
ms.locfileid: "91395161"
---
# <a name="route-traffic-for-high-availability-of-applications-using-azure-cli"></a>使用 Azure CLI 为实现应用程序的高可用性路由流量

此脚本将创建一个资源组、两个应用服务计划、两个 Web 应用、一个流量管理器配置文件和两个流量管理器终结点。 流量管理器将流量引导到一个区域（作为主要区域）中的应用程序；主要区域中的应用程序不可用时，引导到次要区域。 执行脚本前，必须将 MyWebApp、MyWebAppL1 和 MyWebAppL2 值更改为 Azure 内的唯一值。 运行脚本后，可以使用 URL mywebapp.trafficmanager.cn 访问主要区域中的应用。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>示例脚本

```azurecli
#!/bin/bash

RgName1="MyResourceGroup1"
RgName2="MyResourceGroup2"
Location1="chinaeast"
Location2="chinanorth"

# The values of the variables below must be unique (replace with your own names).
WebApp="MyWebApp"
WebAppL1="MyWebAppL1"
WebAppL2="MyWebAppL2"

# Create a resource group in location one.
az group create \
  --name $RgName1 \
  --location $Location1

# Create a resource group in location two.
az group create \
  --name $RgName2 \
  --location $Location2

# Create a website deployed from GitHub in both regions (replace with your own GitHub URL).
gitrepo="https://github.com/Azure-Samples/app-service-web-dotnet-get-started.git"

# Create a hosting plan and website and deploy it in location one (requires Standard 1 minimum SKU).
az appservice plan create \
  --name $WebAppL1 \
  --resource-group $RgName1 \
  --sku S1
az webapp create \
  --name $WebAppL1 \
  --resource-group $RgName1 \
  --plan $WebAppL1
az webapp deployment source config \
  --name $WebAppL1 \
  --resource-group $RgName1 \
  --repo-url $gitrepo \
  --branch master \
  --manual-integration

# Create a hosting plan and website and deploy it in chinanorth (requires Standard 1 minimum SKU).
az appservice plan create \
  --name $WebAppL2 \
  --resource-group $RgName2 \
  --sku S1
az webapp create \
  --name $WebAppL2 \
  --resource-group $RgName2 \
  --plan $WebAppL2
az webapp deployment source config \
  --name $WebAppL2 \
  --resource-group $RgName2 \
  --repo-url $gitrepo \
  --branch master --manual-integration

# Create a Traffic Manager profile.
az network traffic-manager profile create \
  --name MyTrafficManagerProfile \
  --resource-group $RgName1 \
  --routing-method Priority \
  --unique-dns-name $WebApp

# Create an endpoint for the location one website deployment and set it as the priority target.
L1Id=$(az webapp show \
  --resource-group $RgName1 \
  --name $WebAppL1 \
  --query id \
  --out tsv)
az network traffic-manager endpoint create \
  --name MyEndPoint1 \
  --profile-name MyTrafficManagerProfile \
  --resource-group $RgName1 \
  --type azureEndpoints \
  --priority 1 \
  --target-resource-id $L1Id

# Create an endpoint for the location two website deployment and set it as the secondary target.
L2Id=$(az webapp show \
  --resource-group $RgName2 \
  --name $WebAppL2 \
  --query id --out tsv)
az network traffic-manager endpoint create \
  --name MyEndPoint2 \
  --profile-name MyTrafficManagerProfile \
  --resource-group $RgName1 \
  --type azureEndpoints \
  --priority 2 \
  --target-resource-id $L2Id

```

## <a name="clean-up-deployment"></a>清理部署 

运行脚本示例后，可以使用以下命令删除资源组、应用服务应用以及所有相关资源。

```azurecli
az group delete --name myResourceGroup1 --yes
az group delete --name myResourceGroup2 --yes
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、Web 应用、流量管理器配置文件和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 注释 |
|---|---|
| [az group create](https://docs.azure.cn/cli/group#az-group-create) | 创建用于存储所有资源的资源组。 |
| [az appservice plan create](https://docs.azure.cn/cli/appservice/plan#az-appservice-plan-create) | 创建应用服务计划。 这与 Azure Web 应用的服务器场类似。 |
| [az webapp web create](https://docs.azure.cn/cli/webapp#az-webapp-create) | 创建应用服务计划中的 Azure Web 应用。 |
| [az network traffic-manager profile create](https://docs.azure.cn/cli/network/traffic-manager/profile#az-network-traffic-manager-profile-create) | 创建 Azure 流量管理器配置文件。 |
| [az network traffic-manager endpoint create](https://docs.azure.cn/cli/network/traffic-manager/endpoint#az-network-traffic-manager-endpoint-create) | 将终结点添加到 Azure 流量管理器配置文件。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.azure.cn/cli/index)。

可在 [Azure 网络文档](../cli-samples.md)中找到其他应用服务 CLI 脚本示例。

<!-- Update_Description: update meta properties, wording update, update link -->