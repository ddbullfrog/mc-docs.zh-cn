---
title: 教程 - 使用快速启动模板
description: 了解如何使用 Azure 快速入门模板来完成模板开发。
origin.date: 03/27/2020
author: rockboyfor
ms.date: 10/12/2020
ms.testscope: yes
ms.testdate: 08/24/2020
ms.topic: tutorial
ms.author: v-yeche
ms.custom: devx-track-azurecli
ms.openlocfilehash: 1aff1da81348d375c790eb938557f987c4604134
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937541"
---
# <a name="tutorial-use-azure-quickstart-templates"></a>教程：使用 Azure 快速入门模板

[Azure 快速入门模板](https://github.com/Azure/azure-quickstart-templates/)是一个存储库，其中包含社区贡献的模板。 可以在模板开发中使用示例模板。 在本教程中，我们需找到一个网站资源定义，然后将其添加到自己的模板中。 完成该过程需要大约 **12 分钟**。

## <a name="prerequisites"></a>先决条件

建议完成[有关已导出模板的教程](template-tutorial-export-template.md)，但这不是必需的。

必须已安装带有资源管理器工具扩展的 Visual Studio Code，以及 Azure PowerShell 或 Azure CLI。 有关详细信息，请参阅[模板工具](template-tutorial-create-first-template.md#get-tools)。

## <a name="review-template"></a>审阅模板

在上一篇教程的结束时，模板包含以下 JSON：

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
      "type": "string",
      "minLength": 3,
      "maxLength": 11
    },
    "storageSKU": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_RAGRS",
        "Premium_LRS"
      ]
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    },
    "appServicePlanName": {
      "type": "string",
      "defaultValue": "exampleplan"
    }
  },
  "variables": {
    "uniqueStorageName": "[concat(parameters('storagePrefix'), uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[variables('uniqueStorageName')]",
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
      "name": "[parameters('appServicePlanName')]",
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
    }
  ],
  "outputs": {
    "storageEndpoint": {
      "type": "object",
      "value": "[reference(variables('uniqueStorageName')).primaryEndpoints]"
    }
  }
}
```

此模板适用于部署存储帐户和应用服务计划，但你可能需要向其添加网站。 可以使用预生成的模板来快速发现部署资源所需的 JSON。

## <a name="find-template"></a>查找模板

1. 打开 [Azure 快速入门模板](https://github.com/Azure/azure-quickstart-templates/)
1. 在“搜索”中，输入“部署 linux web 应用”。  
1. 选择标题为“部署基本的 linux web 应用”的项。  如果找不到它，请单击此处的[直接链接](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-basic-linux/)。
1. 选择“在 GitHub 上浏览”。 
1. 选择“azuredeploy.json”。 
1. 查看模板。 具体说来，请查找 `Microsoft.Web/sites` 资源。

    :::image type="content" source="./media/template-tutorial-quickstart-template/resource-manager-template-quickstart-template-web-site.png" alt-text="资源管理器模板快速入门网站":::

## <a name="revise-existing-template"></a>修订现有模板

将快速入门模板与现有模板合并：

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
      "type": "string",
      "minLength": 3,
      "maxLength": 11
    },
    "storageSKU": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_RAGRS",
        "Premium_LRS"
      ]
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    },
    "appServicePlanName": {
      "type": "string",
      "defaultValue": "exampleplan"
    },
    "webAppName": {
      "type": "string",
      "metadata": {
        "description": "Base name of the resource such as web app name and app service plan "
      },
      "minLength": 2
    },
    "linuxFxVersion": {
      "type": "string",
      "defaultValue": "php|7.0",
      "metadata": {
        "description": "The Runtime stack of current web app"
      }
    }
  },
  "variables": {
    "uniqueStorageName": "[concat(parameters('storagePrefix'), uniqueString(resourceGroup().id))]",
    "webAppPortalName": "[concat(parameters('webAppName'), uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[variables('uniqueStorageName')]",
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
      "name": "[parameters('appServicePlanName')]",
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
      "name": "[variables('webAppPortalName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', parameters('appServicePlanName'))]"
      ],
      "kind": "app",
      "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', parameters('appServicePlanName'))]",
        "siteConfig": {
            "linuxFxVersion": "[parameters('linuxFxVersion')]"
          }
      }
    }
  ],
  "outputs": {
    "storageEndpoint": {
      "type": "object",
      "value": "[reference(variables('uniqueStorageName')).primaryEndpoints]"
    }
  }
}
```

Web 应用名称必须在 Azure 中独一无二。 为了防止出现重复名称，我们已将 **webAppPortalName** 变量从 **"webAppPortalName": "[concat(parameters('webAppName'), '-webapp')]"** 更新为 **"webAppPortalName": "[concat(parameters('webAppName'), uniqueString(resourceGroup().id))]"** 。

在 `Microsoft.Web/serverfarms` 定义末尾添加一个逗号，以便将资源定义与 `Microsoft.Web/sites` 定义分开。

在这个新资源中，有一些需要注意的重要功能。

你会注意到，它有一个名为 **dependsOn** 的元素，该元素设置为应用服务计划。 此设置是必需的，因为在创建 Web 应用之前，必须存在应用服务计划。 **dependsOn** 元素告知资源管理器如何将用于部署的资源排序。

**serverFarmId** 属性使用 [resourceId](template-functions-resource.md#resourceid) 函数。 此函数获取资源的唯一标识符。 在此示例中，它获取应用服务计划的唯一标识符。 Web 应用与一个特定的应用服务计划相关联。

## <a name="deploy-template"></a>部署模板

使用 Azure CLI 或 Azure PowerShell 来部署模板。

如果尚未创建资源组，请参阅[创建资源组](template-tutorial-create-first-template.md#create-resource-group)。 此示例假设已根据[第一篇教程](template-tutorial-create-first-template.md#deploy-template)中所述，将 **templateFile** 变量设置为模板文件的路径。

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addwebapp `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile `
  -storagePrefix "store" `
  -storageSKU Standard_LRS `
  -webAppName demoapp
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

若要运行此部署命令，必须具有 Azure CLI 的 [最新版本](https://docs.azure.cn/cli/install-azure-cli)。

```azurecli
az deployment group create \
  --name addwebapp \
  --resource-group myResourceGroup \
  --template-file $templateFile \
  --parameters storagePrefix=store storageSKU=Standard_LRS webAppName=demoapp
```

---

> [!NOTE]
> 如果部署失败，请使用“详细”开关获取有关正在创建的资源的信息。 使用“调试”开关获取调试的详细信息。

## <a name="clean-up-resources"></a>清理资源

若要继续学习下一篇教程，则不需删除该资源组。

如果你不打算继续学习，请删除该资源组以清理部署的资源。

1. 在 Azure 门户上的左侧菜单中选择“资源组”  。
2. 在“按名称筛选”字段中输入资源组名称。 
3. 选择资源组名称。
4. 在顶部菜单中选择“删除资源组”。 

## <a name="next-steps"></a>后续步骤

我们学习了如何使用快速入门模板进行模板开发。 在下一教程中，我们向资源添加标记。

> [!div class="nextstepaction"]
> [添加标记](template-tutorial-add-tags.md)

<!-- Update_Description: update meta properties, wording update, update link -->