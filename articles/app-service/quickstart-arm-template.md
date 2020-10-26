---
title: 使用 Azure 资源管理器模板创建应用服务应用
description: 使用 Azure 资源管理器模板（ARM 模板）在几秒钟时间内在 Azure 应用服务中创建你的第一个应用，这是将应用部署到应用服务的许多方式之一。
author: msangapu-msft
ms.author: v-tawe
ms.assetid: 582bb3c2-164b-42f5-b081-95bfcb7a502a
ms.topic: quickstart
origin.date: 10/16/2020
ms.date: 10/19/2020
ms.custom: subject-armqs
zone_pivot_groups: app-service-platform-windows-linux
ms.openlocfilehash: 1747a9c8156b66a92fbe79123163e780f2773445
ms.sourcegitcommit: e2e418a13c3139d09a6b18eca6ece3247e13a653
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/19/2020
ms.locfileid: "92170843"
---
# <a name="quickstart-create-app-service-app-using-an-arm-template"></a>快速入门：使用 ARM 模板创建应用服务应用

使用 Azure 资源管理器模板（ARM 模板）和 Cloud Shell 中的 [Azure CLI](/cli/get-started-with-azure-cli) 将应用部署到云，以便开始使用 [Azure 应用服务](overview.md)。 由于使用的是免费应用服务层，因此完成本快速入门不会产生费用。

 [!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

如果你的环境满足先决条件，并且你熟悉如何使用 ARM 模板，请选择“部署到 Azure”按钮  。 Azure 门户中会打开模板。

使用下面的按钮在 Linux 上部署：

[![部署到 Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-app-service-docs-linux%2Fazuredeploy.json)

使用下面的按钮在 Windows 上部署：

[![部署到 Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-app-service-docs-windows%2Fazuredeploy.json)

## <a name="prerequisites"></a>先决条件

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="review-the-template"></a>查看模板

::: zone pivot="platform-windows"
本快速入门中使用的模板来自 [Azure 快速启动模板](https://azure.microsoft.com/resources/templates/101-app-service-docs-windows)。 它在 Windows 上部署应用服务计划和应用服务应用。 它与 .NET Core、.NET Framework、PHP、Node.js 和静态 HTML 应用兼容。 对于 Java，请参阅[创建 Java 应用](./quickstart-java.md)。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "webAppName": {
      "type": "string",
      "defaultValue": "[concat('webApp-', uniqueString(resourceGroup().id))]",
      "minLength": 2,
      "metadata": {
        "description": "Web app name."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    },
    "sku": {
      "type": "string",
      "defaultValue": "F1",
      "metadata": {
        "description": "The SKU of App Service Plan."
      }
    },
    "language": {
      "type": "string",
      "defaultValue": ".net",
      "allowedValues": [
        ".net",
        "php",
        "node",
        "html"
      ],
      "metadata": {
        "description": "The language stack of the app."
      }
    },
    "helloWorld": {
      "type": "bool",
      "defaultValue": false,
      "metadata": {
        "description": "true = deploy a sample Hello World app."
      }
    },
    "repoUrl": {
      "type": "string",
      "defaultValue": "",
      "metadata": {
        "description": "Optional Git Repo URL"
      }
    }
  },
  "variables": {
    "appServicePlanPortalName": "[concat('AppServicePlan-', parameters('webAppName'))]",
    "gitRepoReference": {
      ".net": "https://github.com/Azure-Samples/app-service-web-dotnet-get-started",
      "node": "https://github.com/Azure-Samples/nodejs-docs-hello-world",
      "php": "https://github.com/Azure-Samples/php-docs-hello-world",
      "html": "https://github.com/Azure-Samples/html-docs-hello-world"
    },
    "gitRepoUrl": "[if(bool(parameters('helloWorld')), variables('gitRepoReference')[toLower(parameters('language'))], parameters('repoUrl'))]",
    "configReference": {
      ".net": {
        "comments": ".Net app. No additional configuration needed."
      },
      "html": {
        "comments": "HTML app. No additional configuration needed."
      },
      "php": {
        "phpVersion": "7.4"
      },
      "node": {
        "appSettings": [
          {
            "name": "WEBSITE_NODE_DEFAULT_VERSION",
            "value": "12.15.0"
          }
        ]
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Web/serverfarms",
      "apiVersion": "2020-06-01",
      "name": "[variables('appServicePlanPortalName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('sku')]"
      }
    },
    {
      "type": "Microsoft.Web/sites",
      "apiVersion": "2020-06-01",
      "name": "[parameters('webAppName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('appServicePlanPortalName'))]"
      ],
      "properties": {
        "siteConfig": "[variables('configReference')[parameters('language')]]",
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('appServicePlanPortalName'))]"
      },
      "resources": [
        {
          "condition": "[contains(variables('gitRepoUrl'),'http')]",
          "type": "sourcecontrols",
          "apiVersion": "2020-06-01",
          "name": "web",
          "location": "[parameters('location')]",
          "dependsOn": [
            "[resourceId('Microsoft.Web/sites', parameters('webAppName'))]"
          ],
          "properties": {
            "repoUrl": "[variables('gitRepoUrl')]",
            "branch": "master",
            "isManualIntegration": true
          }
        }
      ]
    }
  ]
}
```

该模板中定义了两个 Azure 资源：

* [**Microsoft.Web/serverfarms**](https://docs.microsoft.com/azure/templates/microsoft.web/serverfarms)：创建应用服务计划。
* [**Microsoft.Web/sites**](https://docs.microsoft.com/azure/templates/microsoft.web/sites)：创建应用服务应用。

为了便于你使用，此模板包含多个预定义的参数。 有关参数默认值及其说明，请参阅下表：

| 参数 | 类型    | 默认值                | DESCRIPTION |
|------------|---------|------------------------------|-------------|
| webAppName | 字符串  | "webApp- **[`<uniqueString>`](../azure-resource-manager/templates/template-functions-string.md#uniquestring)** " | 应用程序名称 |
| location   | 字符串  | "[[resourceGroup().location](../azure-resource-manager/templates/template-functions-resource.md#resourcegroup)]" | 应用区域 |
| sku        | 字符串  | "F1"                         | 实例大小（F1 = 免费层） |
| 语言   | 字符串  | ".net"                       | 编程语言堆栈（.net、php、node、html） |
| helloWorld | boolean | False                        | True = 部署“Hello World”应用 |
| repoUrl    | 字符串  | " "                          | 外部 Git 存储库（可选） |
::: zone-end
::: zone pivot="platform-linux"
本快速入门中使用的模板来自 [Azure 快速启动模板](https://azure.microsoft.com/resources/templates/101-app-service-docs-linux)。 它在 Linux 上部署应用服务计划和应用服务应用。 它与应用服务上所有受支持的编程语言兼容。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "webAppName": {
      "type": "string",
      "defaultValue": "[concat('webApp-', uniqueString(resourceGroup().id))]",
      "minLength": 2,
      "metadata": {
        "description": "Web app name."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    },
    "sku": {
      "type": "string",
      "defaultValue": "F1",
      "metadata": {
        "description": "The SKU of App Service Plan."
      }
    },
    "linuxFxVersion": {
      "type": "string",
      "defaultValue": "DOTNETCORE|3.0",
      "metadata": {
        "description": "The Runtime stack of current web app"
      }
    },
    "repoUrl": {
      "type": "string",
      "defaultValue": " ",
      "metadata": {
        "description": "Optional Git Repo URL"
      }
    }
  },
  "variables": {
    "appServicePlanPortalName": "[concat('AppServicePlan-', parameters('webAppName'))]"
  },
  "resources": [
    {
      "type": "Microsoft.Web/serverfarms",
      "apiVersion": "2020-06-01",
      "name": "[variables('appServicePlanPortalName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('sku')]"
      },
      "kind": "linux",
      "properties": {
        "reserved": true
      }
    },
    {
      "type": "Microsoft.Web/sites",
      "apiVersion": "2020-06-01",
      "name": "[parameters('webAppName')]",
      "location": "[parameters('location')]",

      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('appServicePlanPortalName'))]"
      ],
      "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('appServicePlanPortalName'))]",
        "siteConfig": {
          "linuxFxVersion": "[parameters('linuxFxVersion')]"
        },
        "resources": [
          {
            "condition": "[contains(parameters('repoUrl'),'http')]",
            "type": "sourcecontrols",
            "apiVersion": "2020-06-01",
            "name": "web",
            "location": "[parameters('location')]",
            "dependsOn": [
              "[resourceId('Microsoft.Web/sites', parameters('webAppName'))]"
            ],
            "properties": {
              "repoUrl": "[parameters('repoUrl')]",
              "branch": "master",
              "isManualIntegration": true
            }
          }
        ]
      }
    }
  ]
}
```

该模板中定义了两个 Azure 资源：

* [**Microsoft.Web/serverfarms**](https://docs.microsoft.com/azure/templates/microsoft.web/serverfarms)：创建应用服务计划。
* [**Microsoft.Web/sites**](https://docs.microsoft.com/azure/templates/microsoft.web/sites)：创建应用服务应用。

为了便于你使用，此模板包含多个预定义的参数。 有关参数默认值及其说明，请参阅下表：

| 参数 | 类型    | 默认值                | DESCRIPTION |
|------------|---------|------------------------------|-------------|
| webAppName | 字符串  | "webApp- **[`<uniqueString>`](../azure-resource-manager/templates/template-functions-string.md#uniquestring)** " | 应用程序名称 |
| location   | 字符串  | "[[resourceGroup().location](../azure-resource-manager/templates/template-functions-resource.md#resourcegroup)]" | 应用区域 |
| sku        | 字符串  | "F1"                         | 实例大小（F1 = 免费层） |
| linuxFxVersion   | 字符串  | "DOTNETCORE&#124;3.0        | "编程语言堆栈 &#124; 版本" |
| repoUrl    | 字符串  | " "                          | 外部 Git 存储库（可选） |

---
::: zone-end

## <a name="deploy-the-template"></a>部署模板

此处使用 Azure CLI 来部署模板。 还可以使用 Azure 门户、Azure PowerShell 和 REST API。 若要了解其他部署方法，请参阅[部署模板](../azure-resource-manager/templates/deploy-powershell.md)。

以下代码将创建一个资源组、一个应用服务计划和一个 Web 应用。 已为你设置了默认资源组、应用服务计划和位置。 请将 `<app-name>` 替换为全局唯一的应用名称（有效字符是 `a-z`、`0-9` 和 `-`）。

::: zone pivot="platform-windows"
运行以下代码，在 Windows 上部署 .NET Framework 应用。

```azurecli-interactive
az group create --name myResourceGroup --location "southcentralus" &&
az deployment group create --resource-group myResourceGroup \
--parameters language=".net" helloWorld="true" webAppName="<app-name>" \
--template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-app-service-docs-windows/azuredeploy.json"
::: zone-end
::: zone pivot="platform-linux"
Run the code below to create a Python app on Linux.

```azurecli-interactive
az group create --name myResourceGroup --location "southcentralus" &&
az deployment group create --resource-group myResourceGroup --parameters webAppName="<app-name>" linuxFxVersion="PYTHON|3.7" \
--template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-app-service-docs-linux/azuredeploy.json"
```

若要部署其他语言堆栈，请使用相应的值更新 `linuxFxVersion`。 下面显示了示例。 若要显示当前版本，请在 Cloud Shell 中运行以下命令：`az webapp config show --resource-group myResourceGroup --name <app-name> --query linuxFxVersion`

| 语言    | 示例：                                              |
|-------------|------------------------------------------------------|
| **.NET**    | linuxFxVersion="DOTNETCORE&#124;3.0"                 |
| **PHP**     | linuxFxVersion="PHP&#124;7.4"                        |
| **Node.js** | linuxFxVersion="NODE&#124;10.15"                     |
| **Java**    | linuxFxVersion="JAVA&#124;1.8 &#124;TOMCAT&#124;9.0" |
| **Python**  | linuxFxVersion="PYTHON&#124;3.7"                     |
| **Ruby**    | linuxFxVersion="RUBY&#124;2.6"                       |

---
::: zone-end

> [!NOTE]
> 可以[在此处找到更多 Azure 应用服务模板示例](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Sites)。

## <a name="validate-the-deployment"></a>验证部署

浏览到 `http://<app_name>.chinacloudsites.cn/` 并验证它是否已创建。

## <a name="clean-up-resources"></a>清理资源

不再需要资源组时，[可将其删除](../azure-resource-manager/management/delete-resource-group.md?tabs=azure-portal#delete-resource-group)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
>从本地 Git 进行部署

> [!div class="nextstepaction"]
>将 ASP.NET Core 与 SQL 数据库配合使用

> [!div class="nextstepaction"]
>将 Python 与 Postgres 配合使用

> [!div class="nextstepaction"]
> [带 MySQL 的 PHP](tutorial-php-mysql-app.md)

> [!div class="nextstepaction"]
> [使用 Java 连接到 Azure SQL 数据库](../azure-sql/database/connect-query-java.md?toc=%252fazure%252fjava%252ftoc.json)

> [!div class="nextstepaction"]
>映射自定义域
