---
title: 使用 Azure CLI 创建 Azure 数据资源管理器群集和数据库
description: 了解如何使用 Azure CLI 创建 Azure 数据资源管理器群集和数据库
author: orspod
ms.author: v-tawe
ms.reviewer: radennis
ms.service: data-explorer
ms.topic: how-to
origin.date: 06/03/2019
ms.date: 09/30/2020
ms.openlocfilehash: 95f838db0179ed3c44f43e33ef5c9650737e6e88
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105186"
---
# <a name="create-an-azure-data-explorer-cluster-and-database-by-using-azure-cli"></a>使用 Azure CLI 创建 Azure 数据资源管理器群集和数据库

> [!div class="op_single_selector"]
> * [Portal](create-cluster-database-portal.md)
> * [CLI](create-cluster-database-cli.md)
> * [PowerShell](create-cluster-database-powershell.md)
> * [C#](create-cluster-database-csharp.md)
> * [Python](create-cluster-database-python.md)
> * [Go](create-cluster-database-go.md)
> * [ARM 模板](create-cluster-database-resource-manager.md)

Azure 数据资源管理器是一项快速、完全托管的数据分析服务，用于实时分析从应用程序、网站和 IoT 设备等资源流式传输的海量数据。 若要使用 Azure 数据资源管理器，请先创建群集，再在该群集中创建一个或多个数据库。 然后将数据引入（加载）到数据库，以便对其运行查询。 在本文中，将使用 Azure CLI 创建群集和数据库。

## <a name="prerequisites"></a>先决条件

若要完成本文，需要 Azure 订阅。 如果没有订阅，请在开始之前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

<!-- [!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)] -->

如果选择在本地安装并使用 Azure CLI，本文需要 Azure CLI 2.0.4 或更高版本。 请运行 `az --version` 检查版本。 如需进行安装或升级，请参阅[安装 Azure CLI](/cli/install-azure-cli)。

## <a name="configure-the-cli-parameters"></a>配置 CLI 参数

如果在 Azure CLI 中运行命令，则不需要执行以下步骤。 如果在本地运行 CLI，请按以下步骤登录到 Azure 并设置当前订阅：

1. 运行以下命令来登录到 Azure：

    ```azurecli
    az cloud set -n AzureChinaCloud
    az login
    ```

1. 设置将在其中创建群集的订阅。 将 `MyAzureSub` 替换为要使用的 Azure 订阅的名称：

    ```azurecli
    az account set --subscription MyAzureSub
    ```
   
1. 安装扩展以使用最新的 Kusto CLI 版本：

    ```azurecli
    az extension add -n kusto
    ```

## <a name="create-the-azure-data-explorer-cluster"></a>创建 Azure 数据资源管理器群集

1. 请使用以下命令创建群集：

    ```azurecli
    az kusto cluster create --cluster-name azureclitest --sku name="Standard_D13_v2" tier="Standard" --resource-group testrg --location chinaeast2
    ```

   |**设置** | **建议的值** | **字段说明**|
   |---|---|---|
   | name | *azureclitest* | 所需的群集名称。|
   | sku | *Standard_D13_v2* | 将用于群集的 SKU。 参数：name - SKU 名称。 tier - SKU 层。 |
   | resource-group | *testrg* | 将在其中创建群集的资源组名称。 |
   | location | *chinaeast2* | 将在其中创建群集的位置。 |

    可以使用其他可选参数，例如群集的容量。

1. 运行以下命令，检查群集是否已成功创建：

    ```azurecli
    az kusto cluster show --cluster-name azureclitest --resource-group testrg
    ```

如果结果包含带 `Succeeded` 值的 `provisioningState`，则表示已成功创建群集。

## <a name="create-the-database-in-the-azure-data-explorer-cluster"></a>在 Azure 数据资源管理器群集中创建数据库

1. 请使用以下命令创建数据库：

    ```azurecli
    az kusto database create --cluster-name azureclitest --database-name clidatabase --resource-group testrg --read-write-database soft-delete-period=P365D hot-cache-period=P31D location=chinaeast2
    ```

   |**设置** | **建议的值** | **字段说明**|
   |---|---|---|
   | cluster-name | *azureclitest* | 将在其中创建数据库的群集的名称。|
   | database-name | *clidatabase* | 数据库名称。|
   | resource-group | *testrg* | 将在其中创建群集的资源组名称。 |
   | read-write-database | *P365D* *P31D* *chinaeast2* | 数据库类型。 参数：soft-delete-period - 表示供查询使用的数据的保留时间。 有关详细信息，请参阅[保留策略](/data-explorer/kusto/management/retentionpolicy)。 hot-cache-period - 表示数据将在缓存中保留的时间。 有关详细信息，请参阅[缓存策略](/data-explorer/kusto/management/cachepolicy)。 location - 这是会在其中创建数据库的位置。 |

1. 若要查看已创建的数据库，请运行以下命令：

    ```azurecli
    az kusto database show --database-name clidatabase --resource-group testrg --cluster-name azureclitest
    ```

现在，你有了一个群集和一个数据库。

## <a name="clean-up-resources"></a>清理资源

* 如果计划学习我们的其他文章，请保留已创建的资源。
* 若要清理资源，请删除群集。 删除群集时，也会删除其中的所有数据库。 使用以下命令删除群集：

    ```azurecli
    az kusto cluster delete --cluster-name azureclitest --resource-group testrg
    ```

## <a name="next-steps"></a>后续步骤

* [使用 Azure 数据资源管理器 Python 库引入数据](python-ingest-data.md)
