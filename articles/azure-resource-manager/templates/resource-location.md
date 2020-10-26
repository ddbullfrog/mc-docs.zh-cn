---
title: 模板资源位置
description: 介绍如何在 Azure 资源管理器模板中设置资源位置。
ms.topic: conceptual
origin.date: 09/04/2019
author: rockboyfor
ms.date: 10/26/2020
ms.testscope: yes
ms.testdate: 08/24/2020
ms.author: v-yeche
ms.custom: ''
ms.openlocfilehash: bf521ba5cfeea9a02d3a10d91e56ea303d6dec93
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92469997"
---
<!--Verify successfully-->
# <a name="set-resource-location-in-arm-template"></a>在 ARM 模板中设置资源位置

部署 Azure 资源管理器 (ARM) 模板时，必须提供每个资源的位置。 该位置不必与资源组位置相同。

## <a name="get-available-locations"></a>获取可用位置

不同位置支持的资源类型不一样。 若要获取资源类型支持的位置，请使用 Azure PowerShell 或 Azure CLI。

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
((Get-AzResourceProvider -ProviderNamespace Microsoft.Batch).ResourceTypes `
  | Where-Object ResourceTypeName -eq batchAccounts).Locations
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az provider show \
  --namespace Microsoft.Batch \
  --query "resourceTypes[?resourceType=='batchAccounts'].locations | [0]" \
  --out table
```

---

## <a name="use-location-parameter"></a>使用 location 参数

若要在部署模板时实现灵活性，请使用参数指定资源的位置。 将参数的默认值设置为 `resourceGroup().location`。

以下示例显示了部署到作为参数指定的位置的存储帐户：

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountType": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Premium_LRS"
      ],
      "metadata": {
        "description": "Storage Account type"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    }
  },
  "variables": {
    
    "storageAccountName": "[concat('storage', uniquestring(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2018-07-01",
      "name": "[variables('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('storageAccountType')]"
      },
      "kind": "StorageV2",
      "properties": {}
    }
  ],
  "outputs": {
    "storageAccountEndPoint": "https://core.chinacloudapi.cn/",
    "storageAccountName": {
      "type": "string",
      "value": "[variables('storageAccountName')]"
    }
  }
}
```

## <a name="next-steps"></a>后续步骤

* 有关模板函数的完整列表，请参阅 [Azure Resource Manager 模板函数](template-functions.md)。
* 若要详细了解模板文件，请参阅[了解 ARM 模板的结构和语法](template-syntax.md)。

<!-- Update_Description: update meta properties, wording update, update link -->