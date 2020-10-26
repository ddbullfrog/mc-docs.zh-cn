---
title: 教程 - 部署本地 Azure 资源管理器模板
description: 了解如何从本地计算机部署 Azure 资源管理器模板
origin.date: 05/20/2020
author: rockboyfor
ms.date: 10/26/2020
ms.testscope: yes
ms.testdate: 08/24/2020
ms.topic: tutorial
ms.author: v-yeche
ms.custom: ''
ms.openlocfilehash: 04b2b7b0d3946a571c58f7a6feb5a90948f5a7e1
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92470509"
---
# <a name="tutorial-deploy-a-local-azure-resource-manager-template"></a>教程：部署本地 Azure 资源管理器模板

了解如何从本地计算机部署 Azure 资源管理器模板。 完成本教程大约需要 **8 分钟** 。

本文是相关教学系列的第一篇教程。 在学习该系列时，你将通过创建链接模板来将模板模块化，将链接模板存储在存储帐户中，使用 SAS 令牌保护链接模板，并了解如何创建 DevOp 管道来部署模板。 该系列重点介绍模板部署。  如果希望了解模板开发，请参阅[初学者教程](./template-tutorial-create-first-template.md)。

## <a name="get-tools"></a>获取工具

首先，请确保已获取部署模板所需的工具。

### <a name="command-line-deployment"></a>命令行部署

你需要使用 Azure PowerShell 或 Azure CLI 来部署模板。 有关安装说明，请参阅：

- [安装 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps)
- [在 Windows 上安装 Azure CLI](https://docs.azure.cn/cli/install-azure-cli-windows)
- [在 Linux 上安装 Azure CLI](https://docs.azure.cn/cli/install-azure-cli-linux)

安装 Azure PowerShell 或 Azure CLI 后，请务必完成首次登录。 有关帮助，请参阅[登录 - PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps#sign-in) 或[登录 - Azure CLI](https://docs.azure.cn/cli/get-started-with-azure-cli#sign_in)。

### <a name="editor-optional"></a>编辑器（可选）

模板是一些 JSON 文件。 若要查看/编辑模板，需要一个好用的 JSON 编辑器。 我们建议使用带有资源管理器工具扩展的 Visual Studio Code。 如果需要安装这些工具，请参阅[快速入门：使用 Visual Studio Code 创建 Azure 资源管理器模板](quickstart-create-templates-use-visual-studio-code.md)。

## <a name="review-template"></a>审阅模板

模板部署一个存储帐户、应用服务计划和 Web 应用。 如果对创建模板感兴趣，则可完成[关于快速入门模板的教程](template-tutorial-quickstart-template.md)。 但这不是完成本教程所必需的。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "projectName": {
      "type": "string",
      "minLength": 3,
      "maxLength": 11,
      "metadata": {
        "description": "Specify a project name that is used to generate resource names."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Specify a location for the resources."
      }
    },
    "storageSKU": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_RAGRS",
        "Premium_LRS",
      ],
      "metadata": {
        "description": "Specify the storage account type."
      }
    },
    "linuxFxVersion": {
      "type": "string",
      "defaultValue": "php|7.0",
      "metadata": {
        "description": "Specify the Runtime stack of current web app"
      }
    }
  },
  "variables": {
    "storageAccountEndPoint": "https://core.chinacloudapi.cn/",
    "storageAccountName": "[concat(parameters('projectName'), 'store')]",
    "webAppName": "[concat(parameters('projectName'), 'WebApp')]",
    "appServicePlanName": "[concat(parameters('projectName'), 'Plan')]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[variables('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('storageSKU')]"
      },
      "kind": "StorageV2",
      "properties": {
        "supportsHttpsTrafficOnly": true
      }
    },
    {
      "type": "Microsoft.Web/serverfarms",
      "apiVersion": "2016-09-01",
      "name": "[variables('appServicePlanName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "B1",
        "tier": "Basic",
        "size": "B1",
        "family": "B",
        "capacity": 1
      },
      "kind": "linux",
      "properties": {
        "perSiteScaling": false,
        "reserved": true,
        "targetWorkerCount": 0,
        "targetWorkerSizeId": 0
      }
    },
    {
      "type": "Microsoft.Web/sites",
      "apiVersion": "2018-11-01",
      "name": "[variables('webAppName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('appServicePlanName'))]"
      ],
      "kind": "app",
      "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('appServicePlanName'))]",
        "siteConfig": {
          "linuxFxVersion": "[parameters('linuxFxVersion')]"
        }
      }
    }
  ],
  "outputs": {
    "storageEndpoint": {
      "type": "object",
      "value": "[reference(variables('storageAccountName')).primaryEndpoints]"
    }
  }
}
```

> [!IMPORTANT]
> 存储帐户名称长度必须为 3 到 24 个字符，并且只能使用数字和小写字母。 该名称必须是唯一的。 在模板中，存储帐户名称是追加了“store”的项目名称，项目名称的长度必须介于 3 到 11 个字符之间。 因此，项目名称必须符合存储帐户名称要求，且短于 11 个字符。

以 .json 为扩展名将模板的副本保存到本地计算机，例如 azuredeploy.json。 稍后在本教程中将部署此模板。

## <a name="sign-in-to-azure"></a>登录 Azure

若要开始使用 Azure PowerShell/Azure CLI 来部署模板，请使用你的 Azure 凭据登录。

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Connect-AzAccount -Environment AzureChinaCloud
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[!INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

```azurecli
az cloud set -n AzureChinaCloud
az login
```

---

如果你有多个 Azure 订阅，请选择要使用的订阅：

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Set-AzContext [SubscriptionID/SubscriptionName]
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az account set --subscription [SubscriptionID/SubscriptionName]
```

---

## <a name="create-resource-group"></a>创建资源组

部署模板时，请指定一个包含资源的资源组。 在运行部署命令之前，请使用 Azure CLI 或 Azure PowerShell 创建该资源组。 使用以下代码部分中的选项卡在 Azure PowerShell 与 Azure CLI 之间进行选择。 本文中的 CLI 示例针对 Bash shell 编写。

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$projectName = Read-Host -Prompt "Enter a project name that is used to generate resource and resource group names"
$resourceGroupName = "${projectName}rg"

New-AzResourceGroup `
  -Name $resourceGroupName `
  -Location "China North"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
echo "Enter a project name that is used to generate resource and resource group names:"
read projectName
resourceGroupName="${projectName}rg"

az group create \
  --name $resourceGroupName \
  --location "China North"
```

---

## <a name="deploy-template"></a>部署模板

使用一个或两个部署选项来部署模板。

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$projectName = Read-Host -Prompt "Enter the same project name"
$templateFile = Read-Host -Prompt "Enter the template file path and file name"
$resourceGroupName = "${projectName}rg"

New-AzResourceGroupDeployment `
  -Name DeployLocalTemplate `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile $templateFile `
  -projectName $projectName `
  -verbose
```

有关使用 Azure PowerShell 来部署模板的详细信息，请参阅[使用资源管理器模板和 Azure PowerShell 来部署资源](./deploy-powershell.md)。

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
echo "Enter the same project name:"
read projectName
echo "Enter the template file path and file name:"
read templateFile
resourceGroupName="${projectName}rg"

az deployment group create \
  --name DeployLocalTemplate \
  --resource-group $resourceGroupName \
  --template-file $templateFile \
  --parameters projectName=$projectName \
  --verbose
```

若要详细了解如何使用 Azure CLI 来部署模板，请参阅[使用资源管理器模板和 Azure CLI 来部署资源](./deploy-cli.md)。

---

## <a name="clean-up-resources"></a>清理资源

通过删除资源组来清理你部署的资源。

1. 在 Azure 门户上的左侧菜单中选择“资源组”。
2. 在“按名称筛选”字段中输入资源组名称。
3. 选择资源组名称。
4. 在顶部菜单中选择“删除资源组”。

## <a name="next-steps"></a>后续步骤

现在，你已了解了如何部署本地模板。 在下一教程中，你需要将该模板拆分为一个主模板和一个链接模板，并了解如何存储和保护链接模板。

> [!div class="nextstepaction"]
> [部署链接模板](./deployment-tutorial-linked-template.md)

<!-- Update_Description: update meta properties, wording update, update link -->