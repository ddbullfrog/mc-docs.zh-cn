---
title: 快速入门 - 通过 Azure 资源管理器模板创建 Azure 流分析作业
description: 本快速入门介绍如何使用 Azure 资源管理器模板创建 Azure 流分析作业。
services: stream-analytics
ms.service: stream-analytics
author: Johnnytechn
ms.author: v-johya
ms.reviewer: jasonh
ms.workload: big-data
ms.topic: quickstart
ms.custom: mvc, subject-armqs
ms.date: 10/09/2020
ms.openlocfilehash: 2e8ad21ca62c79331b3647ae6c92684b9e395f52
ms.sourcegitcommit: 465c166998f0c24405e573e6ec91e6da90e54f98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91936846"
---
# <a name="quickstart-create-an-azure-stream-analytics-job-by-using-an-arm-template"></a>快速入门：使用 ARM 模板创建 Azure 流分析作业

本快速入门将使用 Azure 资源管理器模板（ARM 模板）创建 Azure 流分析作业。 创建作业后，将验证部署。

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

如果你的环境满足先决条件，并且你熟悉如何使用 ARM 模板，请选择“部署到 Azure”按钮。 Azure 门户中会打开模板。

[![部署到 Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-streamanalytics-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>先决条件

若要完成本文，需要做好以下准备：

* 拥有 Azure 订阅 - [创建试用订阅](https://www.azure.cn/pricing/1rmb-trial/)。

## <a name="review-the-template"></a>查看模板

本快速入门中使用的模板来自 [Azure 快速启动模板](https://azure.microsoft.com/resources/templates/101-streamanalytics-create/)。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.1.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for the resources."
      }
    },
    "streamAnalyticsJobName": {
      "type": "string",
      "minLength": 3,
      "maxLength": 63,
      "metadata": {
        "description": "Stream Analytics Job Name, can contain alphanumeric characters and hypen and must be 3-63 characters long"
      }
    },
    "numberOfStreamingUnits": {
      "type": "int",
      "minValue": 1,
      "maxValue": 48,
      "allowedValues": [
        1,
        3,
        6,
        12,
        18,
        24,
        30,
        36,
        42,
        48
      ],
      "metadata": {
        "description": "Number of Streaming Units"
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.StreamAnalytics/StreamingJobs",
      "apiVersion": "2019-06-01",
      "name": "[parameters('streamAnalyticsJobName')]",
      "location": "[parameters('location')]",
      "properties": {
        "sku": {
          "name": "standard"
        },
        "outputErrorPolicy": "stop",
        "eventsOutOfOrderPolicy": "adjust",
        "eventsOutOfOrderMaxDelayInSeconds": 0,
        "eventsLateArrivalMaxDelayInSeconds": 5,
        "dataLocale": "en-US",
        "transformation": {
          "name": "Transformation",
          "properties": {
            "streamingUnits": "[parameters('numberOfStreamingUnits')]",
            "query": "SELECT\r\n    *\r\nINTO\r\n    [YourOutputAlias]\r\nFROM\r\n    [YourInputAlias]"
          }
        }
      }
    }
  ]
}
```

模板中定义的 Azure 资源是 [Microsoft.StreamAnalytics/StreamingJobs](https://docs.microsoft.com/azure/templates/microsoft.streamanalytics/streamingjobs)：创建 Azure 流分析作业。

## <a name="deploy-the-template"></a>部署模板

本部分介绍如何使用 ARM 模板创建 Azure 流分析作业。

1. 选择下图登录到 Azure 并打开一个模板。 该模板会创建 Azure 流分析作业。

   [![部署到 Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-streamanalytics-create%2Fazuredeploy.json)

2. 提供所需的值以创建 Azure 流分析作业。

   ![使用 Azure 资源管理器模板创建 Azure 流分析作业](./media/quick-create-azure-resource-manager/create-stream-analytics-job-resource-manager-template.png "使用 Azure 资源管理器模板创建 Azure 流分析作业")

   提供以下值：

   |属性  |描述  |
   |---------|---------|
   |**订阅**     | 从下拉列表中选择自己的 Azure 订阅。        |
   |**资源组**     | 指定是要创建新的资源组还是使用现有的资源组。 资源组是用于保存 Azure 解决方案相关资源的容器。 有关详细信息，请参阅 [Azure 资源组概述](../azure-resource-manager/management/overview.md)。 |
   |**区域**     | 选择“中国北部”。 有关其他可用区域，请参阅[各区域推出的 Azure 服务](https://azure.microsoft.com/regions/services/)。        |
   |流分析作业名称     | 提供流分析作业的名称。      |
   |**流单元数**     |  选择所需的流单元数。 有关详细信息，请参阅[了解和调整流单元](stream-analytics-streaming-unit-consumption.md)。       |

3. 选择“查看 + 创建”，然后选择“创建” 。

## <a name="review-deployed-resources"></a>查看已部署的资源

可以使用 Azure 门户检查 Azure 流分析作业，或者使用以下 Azure CLI 或 Azure PowerShell 脚本来列出资源。

### <a name="azure-cli"></a>Azure CLI

```azurecli
echo "Enter your Azure Stream Analytics job name:" &&
read streamAnalyticsJobName &&
echo "Enter the resource group where the Azure Stream Analytics job exists:" &&
read resourcegroupName &&
az stream-analytics job show -g $resourcegroupName -n $streamAnalyticsJobName
```

### <a name="azure-powershell"></a>Azure PowerShell

```azurepowershell
$resourceGroupName = Read-Host -Prompt "Enter the resource group name where your Azure Stream Analytics job exists"
(Get-AzResource -ResourceType "Microsoft.StreamAnalytics/StreamingJobs" -ResourceGroupName $resourceGroupName).Name
 Write-Host "Press [ENTER] to continue..."
```

## <a name="clean-up-resources"></a>清理资源

如果打算继续学习后续教程，可能需要保留这些资源。 如果不再需要资源组，可以将其删除，这会删除 Azure 流分析作业。 使用 Azure CLI 或 Azure PowerShell 删除资源组：

### <a name="azure-cli"></a>Azure CLI

```azurecli
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

### <a name="azure-powershell"></a>Azure PowerShell

```azurepowershell
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

<!--Not available in MC: resource-manager-export.md-->

