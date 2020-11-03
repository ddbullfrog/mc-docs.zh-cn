---
title: 快速入门 - 使用 PowerShell 创建 Azure Databricks 工作区
description: 此快速入门介绍如何使用 PowerShell 创建 Azure Databricks 工作区。
services: azure-databricks
ms.service: azure-databricks
author: mamccrea
ms.author: mamccrea
ms.reviewer: jasonh
ms.workload: big-data
ms.topic: quickstart
ms.date: 06/02/2020
ms.custom: mvc
ms.openlocfilehash: 1bf790ca5b6db6c5077a29f2bf77afabdce428b2
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106562"
---
# <a name="quickstart-create-an-azure-databricks-workspace-using-powershell"></a>快速入门：使用 PowerShell 创建 Azure Databricks 工作区

此快速入门介绍如何使用 PowerShell 创建 Azure Databricks 工作区。 可以使用 PowerShell 以交互方式或者通过脚本创建和管理 Azure 资源。

## <a name="prerequisites"></a>先决条件

如果没有 Azure 订阅，请在开始之前创建一个[免费](https://azure.microsoft.com/free/)帐户。

如果选择在本地使用 PowerShell，则本文要求安装 Az PowerShell 模块，并使用 [Connect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/Connect-AzAccount) cmdlet 连接到 Azure 帐户。 有关安装 Az PowerShell 模块的详细信息，请参阅[安装 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps)。

> [!IMPORTANT]
> 尽管 Az.Databricks PowerShell 模块为预览版，但必须使用以下命令从 Az PowerShell 模块单独安装它：`Install-Module -Name Az.Databricks -AllowPrerelease`。
> Az.Databricks PowerShell 模块正式版推出后，它会包含在将来的 Az PowerShell 模块发行版中，并在 Azure Cloud Shell 中原生提供。

> [!Note]
> 如果要在持有美国政府合规性认证（如 FedRAMP High）的 Azure 商业云中创建 Azure Databricks 工作区，请联系你的 Microsoft 代表或 Databricks 代表以获得这种体验的访问权限。

如果这是你第一次使用 Azure Databricks，则必须注册 **Microsoft.Databricks** 资源提供程序。

```azurepowershell
Register-AzResourceProvider -ProviderNamespace Microsoft.Databricks
```

## <a name="use-azure-cloud-shell"></a>使用 Azure Cloud Shell

Azure 托管 Azure Cloud Shell（一个可通过浏览器使用的交互式 shell 环境）。 可以将 Bash 或 PowerShell 与 Cloud Shell 配合使用来使用 Azure 服务。 可以使用 Azure Cloud Shell 预安装的命令来运行本文中的代码，而不必在本地环境中安装任何内容。

若要启动 Azure Cloud Shell，请执行以下操作：

| 选项 | 示例/链接 |
|-----------------------------------------------|---|
| 选择代码块右上角的“试用”。 选择“试用”不会自动将代码复制到 Cloud Shell。 | ![Azure Cloud Shell 的“试用”示例](./media/cloud-shell-try-it/hdi-azure-cli-try-it.png) |
| 转到 [https://shell.azure.com](https://shell.azure.com) 或选择“启动 Cloud Shell”按钮可在浏览器中打开 Cloud Shell。 | [![在新窗口中启动 Cloud Shell](./media/cloud-shell-try-it/hdi-launch-cloud-shell.png)](https://shell.azure.com) |
| 选择 [Azure 门户](https://portal.azure.com)右上角菜单栏上的 **Cloud Shell** 按钮。 | ![Azure 门户中的“Cloud Shell”按钮](./media/cloud-shell-try-it/hdi-cloud-shell-menu.png) |

若要在 Azure Cloud Shell 中运行本文中的代码，请执行以下操作：

1. 启动 Cloud Shell。

1. 选择代码块上的“复制”按钮以复制代码。

1. 在 Windows 和 Linux 上选择 **Ctrl**+**Shift**+**V** 将代码粘贴到 Cloud Shell 会话中，或在 macOS 上选择 **Cmd**+**Shift**+**V** 将代码粘贴到 Cloud Shell 会话中。

1. 选择 **Enter** 运行此代码。


如果有多个 Azure 订阅，请选择应当计费的资源所在的相应订阅。 使用 [Set-AzContext](https://docs.microsoft.com/powershell/module/az.accounts/set-azcontext) cmdlet 选择特定的订阅 ID。

```azurepowershell
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>创建资源组

使用 [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) cmdlet 创建 [Azure 资源组](/azure-resource-manager/resource-group-overview)。 资源组是在其中以组的形式部署和管理 Azure 资源的逻辑容器。

以下示例在“美国西部 2”区域创建名为“myresourcegroup”的资源组。

```azurepowershell
New-AzResourceGroup -Name myresourcegroup -Location westus2
```

## <a name="create-an-azure-databricks-workspace"></a>创建 Azure Databricks 工作区

在本部分，使用 PowerShell 创建 Azure Databricks 工作区。

```azurepowershell
New-AzDatabricksWorkspace -Name mydatabricksws -ResourceGroupName myresourcegroup -Location westus2 -ManagedResourceGroupName databricks-group -Sku standard
```

提供以下值：

|       **属性**       |                                                                                **说明**                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 名称                     | 提供 Databricks 工作区的名称                                                                                                                                   |
| ResourceGroupName        | 指定现有资源组名称                                                                                                                                        |
| 位置                 | 选择“美国西部 2”。 有关其他可用区域，请参阅[各区域推出的 Azure 服务](https://azure.microsoft.com/regions/services/)                                     |
| ManagedResourceGroupName | 指定是要创建新的受管理资源组还是使用现有受管理资源组。                                                                                        |
| SKU                      | 在“标准”、“高级”和“试用”之间进行选择。 有关这些层的详细信息，请参阅 [Databricks 定价](https://azure.microsoft.com/pricing/details/databricks/) |

创建工作区需要几分钟时间。 完成此过程后，你的用户帐户将自动添加为工作区的管理员用户。

当工作区部署失败时，仍然会在失败状态下创建工作区。 删除失败的工作区，并创建一个解决部署错误的新工作区。 删除失败的工作区时，托管资源组和任何成功部署的资源也将被删除。

## <a name="determine-the-provisioning-state-of-a-databricks-workspace"></a>确定 Databricks 工作区的预配状态

若要确定 Databricks 工作区是否已成功预配，可以使用 `Get-AzDatabricksWorkspace` cmdlet。

```azurepowershell
Get-AzDatabricksWorkspace -Name mydatabricksws -ResourceGroupName myresourcegroup |
  Select-Object -Property Name, SkuName, Location, ProvisioningState
```

```Output
Name            SkuName   Location  ProvisioningState
----            -------   --------  -----------------
mydatabricksws  standard  westus2   Succeeded
```

## <a name="clean-up-resources"></a>清理资源

如果其他快速入门或教程不需要使用本快速入门中创建的资源，可以运行以下示例将其删除。

> [!CAUTION]
> 以下示例删除指定的资源组及其包含的所有资源。
> 如果指定的资源组中存在本快速入门范围外的资源，这些资源也会被删除。

```azurepowershell
Remove-AzResourceGroup -Name myresourcegroup
```

若要仅删除本快速入门中创建的服务器而不删除资源组，请使用 `Remove-AzDatabricksWorkspace` cmdlet。

```azurepowershell
Remove-AzDatabricksWorkspace -Name mydatabricksws -ResourceGroupName myresourcegroup
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [在 Databricks 中创建 Spark 群集](quickstart-create-databricks-workspace-portal.md#create-a-spark-cluster-in-databricks)
