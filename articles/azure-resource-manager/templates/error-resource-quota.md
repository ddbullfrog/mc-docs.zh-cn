---
title: 配额错误
description: 说明在使用 Azure 资源管理器部署资源时如何解决资源配额错误。
ms.topic: troubleshooting
origin.date: 03/09/2018
author: rockboyfor
ms.date: 10/12/2020
ms.author: v-yeche
ms.openlocfilehash: 8a183ccf557e211ad95adf7b92bb0af2f0f58508
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937530"
---
# <a name="resolve-errors-for-resource-quotas"></a>解决资源配额错误

本文介绍了部署资源时可能遇到的配额错误。

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="symptom"></a>症状

如果部署的模板创建的资源超出 Azure 配额，则会收到如下所示的部署错误消息：

```
Code=OperationNotAllowed
Message=Operation results in exceeding quota limits of Core.
Maximum allowed: 4, Current in use: 4, Additional requested: 2.
```

或者，你可能看到：

```
Code=ResourceQuotaExceeded
Message=Creating the resource of type <resource-type> would exceed the quota of <number>
resources of type <resource-type> per resource group. The current resource count is <number>,
please delete some resources of this type before creating a new one.
```

## <a name="cause"></a>原因

已按资源组、订阅、帐户和其他范围应用配额。 例如，订阅可能配置为限制某个区域的核心数目。 如果尝试部署超过允许核心数目的虚拟机，则会收到指出超过配额的错误消息。
有关完整的配额信息，请参阅 [Azure 订阅和服务限制、配额与约束](../../azure-resource-manager/management/azure-subscription-service-limits.md)。

## <a name="troubleshooting"></a>故障排除

### <a name="azure-cli"></a>Azure CLI

对于 Azure CLI，可使用 `az vm list-usage` 命令查找虚拟机配额。

```azurecli
az vm list-usage --location "China East"
```

将返回：

```output
[
  {
    "currentValue": 0,
    "limit": 2000,
    "name": {
      "localizedValue": "Availability Sets",
      "value": "availabilitySets"
    }
  },
  ...
]
```

### <a name="powershell"></a>PowerShell

对于 PowerShell，可使用 Get-AzVMUsage 命令查找虚拟机配额  。

```powershell
Get-AzVMUsage -Location "China East"
```

将返回：

```output
Name                             Current Value Limit  Unit
----                             ------------- -----  ----
Availability Sets                            0  2000 Count
Total Regional Cores                         0   100 Count
Virtual Machines                             0 10000 Count
```

## <a name="solution"></a>解决方案

若要请求增加配额，请转到门户并提出支持问题。 在支持问题中，为你想要在其中进行部署的区域请求增加配额。

> [!NOTE]
> 请记住，对于资源组，配额针对每个单独的区域，而不是针对整个订阅。 如果需要在中国北部部署 30 个核心，则必须在中国北部寻求 30 个 Resource Manager 核心。 如果需要在有权访问的任何区域内部署 30 个核心，则应在所有区域内请求 30 个 Resource Manager 核心。
>
>

1. 选择 **订阅**。

   :::image type="content" source="./media/error-resource-quota/subscriptions.png" alt-text="显示选中了“订阅”的 Azure 门户菜单的屏幕截图。":::

2. 选择需要增加配额的订阅。

   :::image type="content" source="./media/error-resource-quota/select-subscription.png" alt-text="显示选中了“订阅”的 Azure 门户菜单的屏幕截图。":::

3. 选择“使用情况 + 配额” 

   :::image type="content" source="./media/error-resource-quota/select-usage-quotas.png" alt-text="显示选中了“订阅”的 Azure 门户菜单的屏幕截图。":::

4. 在右上角选择“请求增加”  。

   :::image type="content" source="./media/error-resource-quota/request-increase.png" alt-text="显示选中了“订阅”的 Azure 门户菜单的屏幕截图。":::

    <!--MOONCAKE CUSTOMIZE-->
   
5. 在完成“常规信息”和“问题”部分后，请填写你需要增加的配额类型的表单。  

    :::image type="content" source="./media/error-resource-quota/forms.png" alt-text="显示选中了“订阅”的 Azure 门户菜单的屏幕截图。":::
   
    <!--MOONCAKE CUSTOMIZE-->

<!-- Update_Description: update meta properties, wording update, update link -->
