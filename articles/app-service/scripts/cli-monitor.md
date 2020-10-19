---
title: CLI：使用 Web 服务器日志监视应用
description: 了解如何使用 Azure CLI 自动部署和管理应用服务应用。 此示例演示如何使用 Web 服务器日志监视应用。
author: msangapu-msft
tags: azure-service-management
ms.assetid: 0887656f-611c-4627-8247-b5cded7cef60
ms.devlang: azurecli
ms.topic: sample
origin.date: 12/11/2017
ms.date: 10/19/2020
ms.author: v-tawe
ms.custom: mvc, seodec18, devx-track-azurecli
ms.openlocfilehash: eedca45b6d4636d507c5aced2190801791ecdcea
ms.sourcegitcommit: e2e418a13c3139d09a6b18eca6ece3247e13a653
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/19/2020
ms.locfileid: "92170628"
---
# <a name="monitor-an-app-service-app-with-web-server-logs-using-azure-cli"></a>使用 Azure CLI 通过 Web 服务器日志监视应用服务应用

此示例脚本将创建资源组、应用服务计划和应用，并配置应用以启用 Web 服务器日志。 然后，它将下载日志文件以供查看。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

如果选择在本地安装并使用 CLI，需要 Azure CLI 2.0 版或更高版本。 要查找版本，请运行 `az --version`。 如需进行安装或升级，请参阅[安装 Azure CLI](/cli/install-azure-cli?view=azure-cli-lastest)。

## <a name="sample-script"></a>示例脚本

```azurecli
#/bin/bash

# Variables
appName="AppServiceMonitor$random"
location="ChinaNorth"

# Create a Resource Group
az group create --name myResourceGroup --location $location

# Create an App Service Plan
az appservice plan create --name AppServiceMonitorPlan --resource-group myResourceGroup --location $location

# Create a Web App and save the URL
url=$(az webapp create --name $appName --plan AppServiceMonitorPlan --resource-group myResourceGroup --query defaultHostName | sed -e 's/^"//' -e 's/"$//')

# Enable all logging options for the Web App
az webapp log config --name $appName --resource-group myResourceGroup --application-logging true --detailed-error-messages true --failed-request-tracing true --web-server-logging filesystem

# Create a Web Server Log
curl -s -L $url/404

# Download the log files for review
az webapp log download --name $webappname --resource-group myResourceGroup
```

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、应用服务应用和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| Command | 说明 |
|---|---|
| [`az group create`](/cli/group?view=azure-cli-latest#az-group-create) | 创建用于存储所有资源的资源组。 |
| [`az appservice plan create`](/cli/appservice/plan?view=azure-cli-latest#az-appservice-plan-create) | 创建应用服务计划。 |
| [`az webapp create`](/cli/webapp?view=azure-cli-latest#az-webapp-create) | 创建应用服务应用。 |
| [`az webapp log config`](/cli/webapp/log?view=azure-cli-latest#az-webapp-log-config) | 配置应用服务应用将持久保留的日志。 |
| [`az webapp log download`](/cli/webapp/log?view=azure-cli-latest#az-webapp-log-download) | 将应用服务应用的日志下载到本地计算机。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](/cli/overview?view=azure-cli-lastest)。

可以在 [Azure 应用服务文档](../samples-cli.md)中找到其他应用服务 CLI 脚本示例。
