---
title: 使用 Powershell 创建 Azure 数据资源管理器群集和数据库
description: 了解如何使用 PowerShell 创建 Azure 数据资源管理器群集和数据库
author: orspod
ms.author: v-tawe
ms.reviewer: lugoldbe
ms.service: data-explorer
ms.topic: how-to
origin.date: 06/03/2019
ms.date: 09/30/2020
ms.openlocfilehash: d884b7de2534155e8330ee1d5ce299347a6f1b81
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105169"
---
# <a name="create-an-azure-data-explorer-cluster-and-database-by-using-powershell"></a>使用 PowerShell 创建 Azure 数据资源管理器群集和数据库

> [!div class="op_single_selector"]
> * [门户](create-cluster-database-portal.md)
> * [CLI](create-cluster-database-cli.md)
> * [PowerShell](create-cluster-database-powershell.md)
> * [C#](create-cluster-database-csharp.md)
> * [Python](create-cluster-database-python.md)
> * [Go](create-cluster-database-go.md)
> * [ARM 模板](create-cluster-database-resource-manager.md)  

Azure 数据资源管理器是一项快速、完全托管的数据分析服务，用于实时分析从应用程序、网站和 IoT 设备等资源流式传输的海量数据。 若要使用 Azure 数据资源管理器，请先创建群集，再在该群集中创建一个或多个数据库。 然后将数据引入（加载）到数据库，以便对其运行查询。 在本文中，将使用 Powershell 创建群集和数据库。 可以在 Windows、Linux 或 [Azure CLI](https://docs.azure.cn/cli/?view=azure-cli-latest) 中使用 [Az.Kusto](https://docs.microsoft.com/powershell/module/az.kusto/?view=azps-1.4.0#kusto) 运行 PowerShell cmdlet 和脚本，以创建和配置 Azure 数据资源管理器群集和数据库。

## <a name="prerequisites"></a>先决条件

[!INCLUDE [updated-for-az](includes/updated-for-az.md)]

如果没有 Azure 订阅，请在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

<!-- [!INCLUDE [cloud-shell-try-it.md](includes/cloud-shell-try-it.md)] -->

如果选择在本地安装并使用 Azure CLI，本文需要 Azure CLI 2.0.4 或更高版本。 请运行 `az --version` 检查版本。 如需进行安装或升级，请参阅[安装 Azure CLI](/cli/install-azure-cli)。

## <a name="configure-parameters"></a>配置参数

如果在 Azure PowerShell 中运行命令，则不需要执行以下步骤。 如果在本地运行 CLI，请按步骤 1 和 2 登录到 Azure 并设置当前订阅：

1. 运行以下命令来登录到 Azure：

    ```azurepowershell
    Connect-AzAccount -EnvironmentName AzureChinaCloud
    ```

1. 设置要在其中创建群集的订阅：

    ```azurepowershell
     Set-AzContext -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    ```
1. 在本地或 Azure PowerShell 中运行 Azure CLI 时，需要在设备上安装 Az.Kusto 模块：

    ```azurepowershell
     Install-Module -Name Az.Kusto
    ```

## <a name="create-the-azure-data-explorer-cluster"></a>创建 Azure 数据资源管理器群集

1. 请使用以下命令创建群集：

    ```azurepowershell
     New-AzKustoCluster -ResourceGroupName testrg -Name mykustocluster -Location 'China East 2' -Sku D13_v2 -Capacity 10
    ```

   |**设置** | **建议的值** | **字段说明**|
   |---|---|---|
   | 名称 | mykustocluster  | 所需的群集名称。|
   | SKU | *D13_v2* | 将用于群集的 SKU。 |
   | ResourceGroupName | *testrg* | 将在其中创建群集的资源组名称。 |

    可以使用其他可选参数，例如群集的容量。

1. 运行以下命令，检查群集是否已成功创建：

    ```azurepowershell
    Get-AzKustoCluster -Name mykustocluster -ResourceGroupName testrg
    ```

如果结果包含带 `Succeeded` 值的 `provisioningState`，则表示已成功创建群集。

## <a name="create-the-database-in-the-azure-data-explorer-cluster"></a>在 Azure 数据资源管理器群集中创建数据库

1. 请使用以下命令创建数据库：

    ```azurepowershell
    New-AzKustoDatabase -ResourceGroupName testrg -ClusterName mykustocluster -Name mykustodatabase -SoftDeletePeriod 3650:00:00:00 -HotCachePeriod 3650:00:00:00
    ```

   |**设置** | **建议的值** | **字段说明**|
   |---|---|---|
   | ClusterName | mykustocluster | 将在其中创建数据库的群集的名称。|
   | 名称 | mykustodatabase | 数据库名称。|
   | ResourceGroupName | *testrg* | 将在其中创建群集的资源组名称。 |
   | SoftDeletePeriod | *3650:00:00:00* | 供查询使用的数据的保留时间。 |
   | HotCachePeriod | *3650:00:00:00* | 数据将在缓存中保留的时间。 |

1. 若要查看已创建的数据库，请运行以下命令：

    ```azurepowershell
    Get-AzKustoDatabase -ClusterName mykustocluster -ResourceGroupName testrg -Name mykustodatabase
    ```

现在，你有了一个群集和一个数据库。

## <a name="clean-up-resources"></a>清理资源

* 如果计划学习我们的其他文章，请保留已创建的资源。
* 若要清理资源，请删除群集。 删除群集时，也会删除其中的所有数据库。 使用以下命令删除群集：

    ```azurepowershell
    Remove-AzKustoCluster -ResourceGroupName testrg -Name mykustocluster
    ```

## <a name="next-steps"></a>后续步骤

* [其他 Az.Kusto 命令](https://docs.microsoft.com/powershell/module/az.kusto/?view=azps-1.7.0#kusto)
* [使用 Azure 数据资源管理器 .NET Standard SDK（预览版）引入数据](./net-sdk-ingest-data.md)