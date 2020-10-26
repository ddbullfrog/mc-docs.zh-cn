---
title: 快速入门：创建 Azure SignalR 服务 - ARM 模板
description: 本快速入门介绍如何使用 Azure 资源管理器模板创建 Azure SignalR 服务。
author: mgblythe
ms.service: signalr
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: v-tawe
origin.date: 10/02/2020
ms.date: 10/19/2020
ms.openlocfilehash: d801e6fdaf9687f1a60bc63680baac5bf8524146
ms.sourcegitcommit: e2e418a13c3139d09a6b18eca6ece3247e13a653
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/19/2020
ms.locfileid: "92170866"
---
# <a name="quickstart-use-an-arm-template-to-deploy-azure-signalr-service"></a>快速入门：使用 ARM 模板部署 Azure SignalR 服务

本快速入门介绍如何使用 Azure 资源管理器模板（ARM 模板）来创建 Azure SignalR 服务。 可以通过 Azure 门户、PowerShell 或 CLI 部署 Azure SignalR 服务。

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

如果你的环境满足先决条件，并且你熟悉如何使用 ARM 模板，请选择“部署到 Azure”按钮  。 登录后，该模板将在 Azure 门户中打开。

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="使用 Azure 门户中的 ARM 模板将 Azure SignalR 服务部署到 Azure 的按钮。":::](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-signalr%2fazuredeploy.json)

## <a name="prerequisites"></a>先决条件

# <a name="portal"></a>[Portal](#tab/azure-portal)

具有活动订阅的 Azure 帐户。 [创建一个试用帐户](https://wd.azure.cn/pricing/1rmb-trial/)。

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

* 具有活动订阅的 Azure 帐户。 [创建一个试用帐户](https://wd.azure.cn/pricing/1rmb-trial/)。
* 若要在本地运行代码，请安装 [Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps)。

# <a name="cli"></a>[CLI](#tab/CLI)

* 具有活动订阅的 Azure 帐户。 [创建一个试用帐户](https://wd.azure.cn/pricing/1rmb-trial/)。
* 若要在本地运行代码，请安装：
    * Bash shell（例如 [Git for Windows](https://gitforwindows.org) 中包含的 Git Bash）。
    * [Azure CLI](/cli/install-azure-cli)。

---

## <a name="review-the-template"></a>查看模板

本快速入门中使用的模板来自 [Azure 快速启动模板](https://azure.microsoft.com/resources/templates/101-signalr/)。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "name": {
      "type": "string",
      "defaultValue": "[uniqueString(resourceGroup().id)]",
      "metadata": {
        "description": "The globally unique name of the SignalR resource to create."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for the SignalR resource."
      }
    },
    "pricingTier": {
      "type": "string",
      "defaultValue": "Standard_S1",
      "allowedValues": [
        "Free_F1",
        "Standard_S1"
      ],
      "metadata": {
        "description": "The pricing tier of the SignalR resource."
      }
    },
    "capacity": {
      "type": "int",
      "defaultValue": 1,
      "allowedValues": [
        1,
        2,
        5,
        10,
        20,
        50,
        100
      ],
      "metadata": {
        "description": "The number of SignalR Unit."
      }
    },
    "serviceMode": {
      "type": "string",
      "defaultValue": "Default",
      "allowedValues": [
        "Default",
        "Serverless",
        "Classic"
      ],
      "metadata": {
        "description": "Visit https://github.com/Azure/azure-signalr/blob/dev/docs/faq.md#service-mode to understand SignalR Service Mode."
      }
    },
    "enableConnectivityLogs": {
      "type": "string",
      "defaultValue": "true",
      "allowedValues": [
        "true",
        "false"
      ]
    },
    "enableMessagingLogs": {
      "type": "string",
      "defaultValue": "true",
      "allowedValues": [
        "true",
        "false"
      ]
    },
    "allowedOrigins": {
      "type": "array",
      "defaultValue": [
        "https://foo.com",
        "https://bar.com"
      ],
      "metadata": {
        "description": "Set the list of origins that should be allowed to make cross-origin calls."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.SignalRService/SignalR",
      "apiVersion": "2020-05-01",
      "name": "[parameters('name')]",
      "location": "[parameters('location')]",
      "sku": {
        "capacity": "[parameters('capacity')]",
        "name": "[parameters('pricingTier')]"
      },
      "kind": "SignalR",
      "properties": {
        "hostNamePrefix": "[parameters('name')]",
        "features": [
          {
            "flag": "ServiceMode",
            "value": "[parameters('serviceMode')]"
          },
          {
            "flag": "EnableConnectivityLogs",
            "value": "[parameters('enableConnectivityLogs')]"
          },
          {
            "flag": "EnableMessagingLogs",
            "value": "[parameters('enableMessagingLogs')]"
          }
        ],
        "cors": {
          "allowedOrigins": "[parameters('allowedOrigins')]"
        },
        "networkACLs": {
          "defaultAction": "deny",
          "publicNetwork": {
            "allow": [
              "ClientConnection"
            ]
          },
          "privateEndpoints": [
            {
              "name": "mySignalRService.1fa229cd-bf3f-47f0-8c49-afb36723997e",
              "allow": [
                "ServerConnection"
              ]
            }
          ]
        },
        "upstream": {
          "templates": [
            {
              "categoryPattern": "*",
              "eventPattern": "connect,disconnect",
              "hubPattern": "*",
              "urlTemplate": "https://example.com/chat/api/connect"
            }
          ]
        }
      }
    }
  ]
}
```

该模板定义了一个 Azure 资源：

* [**Microsoft.SignalRService/SignalR**](https://docs.microsoft.com/azure/templates/microsoft.signalrservice/signalr)

## <a name="deploy-the-template"></a>部署模板

# <a name="portal"></a>[Portal](#tab/azure-portal)

选择以下链接以使用 Azure 门户中的 ARM 模板部署 Azure SignalR 服务：

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="使用 Azure 门户中的 ARM 模板将 Azure SignalR 服务部署到 Azure 的按钮。":::](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-signalr%2fazuredeploy.json)

在“部署 Azure SignalR 服务”页上：

1. 如果需要，可以更改“订阅”的默认值。

2. 对于“资源组”，请选择“新建”，输入新资源组的名称，然后选择“确定”  。

3. 如果创建了新的资源组，请为该资源组选择一个区域。

4. 如果需要，请输入新版名称和 Azure SignalR 服务的位置（例如 chinaeast2）  。 如果未指定名称，则会自动生成名称。 Azure SignalR 服务的位置可以与资源组所在的区域相同，也可以与资源组所在的区域不同。 如果未指定位置，则将其设置为与资源组相同的区域。

5. 选择“定价层”（“Free_F1”或“Standard_S1”），输入“容量”（SignalR 单位数），然后选择“服务模式”：“默认”（需要中心服务器）、“无服务器”（不允许任何服务器连接）或“经典”（仅当中心具有服务器连接时才路由到中心服务器）       。 然后，选择是“启用连接日志”，还是“启用消息日志” 。

    > [!NOTE]
    > 对于“Free_F1”定价层，容量限制为 1 个单位。

    :::image type="content" source="./media/signalr-quickstart-azure-signalr-service-arm-template/deploy-azure-signalr-service-arm-template-portal.png" alt-text="使用 Azure 门户中的 ARM 模板将 Azure SignalR 服务部署到 Azure 的按钮。":::

6. 选择“查看 + 创建”  。

7. 阅读条款和条件，然后选择“创建”。

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

> [!NOTE]
> 若要在本地运行 PowerShell 脚本，请首先输入 `Connect-AzAccount` 来设置 Azure 凭据。

使用以下代码通过 ARM 模板部署 Azure SignalR 服务。 该代码将提示你输入以下项：

* 新的 Azure SignalR 服务的名称和区域
* 新资源组的名称和区域
* Azure 定价层（“Free_F1”或“Standard_S1”） 
* SignalR 单位容量（1、2、5、10、20、50 或 100）
  > [!NOTE]
  > 对于“Free_F1”定价层，容量限制为 1 个单位。
* 服务模式：“默认”要求具有中心服务器，“无服务器”不允许任何服务器连接，或“经典”仅在中心具有服务器连接时才路由到中心服务器  
* 是否启用日志以进行连接或消息传递（“是”或“否”） 

```azurepowershell
$serviceName = Read-Host -Prompt "Enter a name for the new Azure SignalR service"
$serviceLocation = Read-Host -Prompt "Enter an Azure region (for example, chinanorth2) for the service"
$resourceGroupName = Read-Host -Prompt "Enter a name for the new resource group to contain the service"
$resourceGroupRegion = Read-Host -Prompt "Enter an Azure region (for example, centralus) for the resource group"

$priceTier = Read-Host -Prompt "Enter the pricing tier (Free_F1 or Standard_S1)"
$unitCapacity = Read-Host -Prompt "Enter the number of SignalR units (1, 2, 5, 10, 20, 50, or 100)"
$servicingMode = Read-Host -Prompt "Enter the service mode (Default, Serverless, or Classic)"
$enableConnectionLogs = Read-Host -Prompt "Specify whether to enable connectivity logs (true or false)"
$enableMessageLogs = Read-Host -Prompt "Specify whether to enable messaging logs (true or false)"

Write-Verbose "New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion" -Verbose
New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion

$paramObjHashTable = @{
    name = $serviceName
    location = $serviceLocation
    pricingTier = $priceTier
    capacity = [int]$unitCapacity
    serviceMode = $servicingMode
    enableConnectivityLogs = $enableConnectionLogs
    enableMessagingLogs = $enableMessageLogs
}

Write-Verbose "Run New-AzResourceGroupDeployment to create an Azure SignalR service using an ARM template" -Verbose
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
    -TemplateParameterObject $paramObjHashTable `
    -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-signalr/azuredeploy.json
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[CLI](#tab/CLI)

使用以下代码通过 ARM 模板部署 Azure SignalR 服务。 该代码将提示你输入以下项：

* 新的 Azure SignalR 服务的名称和区域
* 新资源组的名称和区域
* Azure 定价层（“Free_F1”或“Standard_S1”） 
* SignalR 单位容量（1、2、5、10、20、50 或 100）
    > [!NOTE]
    > 对于“Free_F1”定价层，容量限制为 1 个单位。
* 服务模式：“默认”要求具有中心服务器，“无服务器”不允许任何服务器连接，或“经典”仅在中心具有服务器连接时才路由到中心服务器  
* 是否启用日志以进行连接或消息传递（“是”或“否”） 

```azurecli
read -p "Enter a name for the new Azure SignalR service: " serviceName &&
read -p "Enter an Azure region (for example, chinanorth2) for the service: " serviceLocation &&
read -p "Enter a name for the new resource group to contain the service: " resourceGroupName &&
read -p "Enter an Azure region (for example, centralus) for the resource group: " resourceGroupRegion &&
read -p "Enter the pricing tier (Free_F1 or Standard_S1): " priceTier &&
read -p "Enter the number of SignalR units (1, 2, 5, 10, 20, 50, or 100): " unitCapacity &&
read -p "Enter the service mode (Default, Serverless, or Classic): " servicingMode &&
read -p "Specify whether to enable connectivity logs (true or false): " enableConnectionLogs &&
read -p "Specify whether to enable messaging logs (true or false): " enableMessageLogs &&
params='name='$serviceName' location='$serviceLocation' pricingTier='$priceTier' capacity='$unitCapacity' serviceMode='$servicingMode' enableConnectivityLogs='$enableConnectionLogs' enableMessagingLogs='$enableMessageLogs &&
echo "CREATE RESOURCE GROUP:  az group create --name $resourceGroupName --location $resourceGroupRegion" &&
az group create --name $resourceGroupName --location $resourceGroupRegion &&
echo "RUN az deployment group create, which creates an Azure SignalR service using an ARM template" &&
az deployment group create --resource-group $resourceGroupName --parameters $params --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-signalr/azuredeploy.json &&
read -p "Press [ENTER] to continue: "
```

---

> [!NOTE]
> 部署可能需要几分钟才能完成。 记下 Azure SignalR 服务和资源组的名称，稍后将使用它们来查看已部署的资源。

## <a name="review-deployed-resources"></a>查看已部署的资源

# <a name="portal"></a>[Portal](#tab/azure-portal)

按照以下步骤查看新 Azure SignalR 服务的概览：

1. 在 [Azure 门户](https://portal.azure.cn)中，搜索并选择“SignalR”。

2. 在 SignalR 列表中，选择你的新服务。 此时将显示新 Azure SignalR 服务的“概述”页。

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

运行以下交互式代码来查看有关 Azure SignalR 服务的详细信息。 必须输入新服务和资源组的名称。

```azurepowershell
$serviceName = Read-Host -Prompt "Enter the name of your Azure SignalR service"
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
Write-Verbose "Get-AzSignalR -ResourceGroupName $resourceGroupName -Name $serviceName" -Verbose
Get-AzSignalR -ResourceGroupName $resourceGroupName -Name $serviceName
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[CLI](#tab/CLI)

运行以下交互式代码来查看有关 Azure SignalR 服务的详细信息。 必须输入新服务和资源组的名称。

```azurecli
read -p "Enter the name of your Azure SignalR service: " serviceName &&
read -p "Enter the resource group name: " resourceGroupName &&
echo "SHOW SERVICE DETAILS:  az signalr show --resource-group $resourceGroupName --name $serviceName" &&
az signalr show --resource-group $resourceGroupName --name $serviceName &&
read -p "Press [ENTER] to continue: "
```

## <a name="clean-up-resources"></a>清理资源

如果不再需要该资源组，可以将其删除，这将删除资源组中的资源。

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. 在 [Azure 门户](https://portal.azure.cn)中，搜索并选择“资源组”。

2. 在资源组列表中，选择你的资源组的名称。

3. 在资源组的“概览”页中，选择“删除资源组” 。

4. 在确认对话框中，键入资源组的名称，然后选择“删除”。

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell
$resourceGroupName = Read-Host -Prompt "Enter the name of the resource group to delete"
Write-Verbose "Remove-AzResourceGroup -Name $resourceGroupName" -Verbose
Remove-AzResourceGroup -Name $resourceGroupName
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
read -p "Enter the name of the resource group to delete: " resourceGroupName &&
echo "DELETE A RESOURCE GROUP (AND ITS RESOURCES):  az group delete --name $resourceGroupName" &&
az group delete --name $resourceGroupName &&
read -p "Press [ENTER] to continue: "
```

## <a name="next-steps"></a>后续步骤

有关引导你完成 ARM 模板创建过程的分步教程，请参阅：

> [!div class="nextstepaction"]
> [教程：创建和部署你的第一个 ARM 模板](../azure-resource-manager/templates/template-tutorial-create-first-template.md)
