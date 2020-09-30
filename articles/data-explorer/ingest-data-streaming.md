---
title: 使用 Azure 门户在 Azure 数据资源管理器群集上配置流式引入
description: 了解如何配置 Azure 数据资源管理器群集，并开始使用 Azure 门户通过流式引入加载数据。
author: orspod
ms.author: v-tawe
ms.reviewer: alexefro
ms.service: data-explorer
ms.topic: how-to
origin.date: 07/13/2020
ms.date: 09/24/2020
ms.openlocfilehash: 30c9188c737bda2428878647746cda7676869f0c
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146374"
---
# <a name="configure-streaming-ingestion-on-your-azure-data-explorer-cluster-using-the-azure-portal"></a>使用 Azure 门户在 Azure 数据资源管理器群集上配置流式引入

> [!div class="op_single_selector"]
> * [Portal](ingest-data-streaming.md)
> * [C#](ingest-data-streaming-csharp.md)

[!INCLUDE [ingest-data-streaming-intro](includes/ingest-data-streaming-intro.md)]

## <a name="prerequisites"></a>先决条件

* 如果没有 Azure 订阅，请在开始前创建一个[试用 Azure 帐户](https://www.azure.cn/pricing/1rmb-trial/)。
* 创建 [Azure 数据资源管理器群集和数据库](create-cluster-database-portal.md)

## <a name="enable-streaming-ingestion-on-your-cluster"></a>在群集上启用流式引入

### <a name="enable-streaming-ingestion-while-creating-a-new-cluster-in-the-azure-portal"></a>在 Azure 门户中创建新群集时启用流式引入

可以在创建新的 Azure 数据资源管理器群集时启用流式引入。 

在“配置”选项卡中，选择“流式引入” > “启用”  。

:::image type="content" source="media/ingest-data-streaming/cluster-creation-enable-streaming.png" alt-text="在 Azure 数据资源管理器中创建群集时启用流式引入":::

### <a name="enable-streaming-ingestion-on-an-existing-cluster-in-the-azure-portal"></a>在 Azure 门户中现有群集上启用流式引入

1. 在 Azure 门户中，转到 Azure 数据资源管理器群集。 
1. 在“设置”中选择“配置”。  
1. 在“配置”窗格中，选择“打开”以启用“流式引入”。  
1. 选择“保存” 。

    :::image type="content" source="media/ingest-data-streaming/streaming-ingestion-on.png" alt-text="在 Azure 数据资源管理器中创建群集时启用流式引入":::

> [!WARNING]
> 启用流式引入之前，请查看[限制](#limitations)。

## <a name="create-a-target-table-and-define-the-policy-in-the-azure-portal"></a>在 Azure 门户中创建目标表并定义策略

1. 在 Azure 门户中导航到群集。
1. 选择“查询”。

    :::image type="content" source="media/ingest-data-streaming/cluster-select-query-tab.png" alt-text="在 Azure 数据资源管理器中创建群集时启用流式引入":::

1. 将以下命令复制到“查询窗格”中，然后选择“运行”以创建将通过流式引入接收数据的表 。

    ```Kusto
    .create table TestTable (TimeStamp: datetime, Name: string, Metric: int, Source:string)
    ```

    :::image type="content" source="media/ingest-data-streaming/create-table.png" alt-text="在 Azure 数据资源管理器中创建群集时启用流式引入":::

1. 在已创建的表或包含此表的数据库上定义[流式引入策略](kusto/management/streamingingestionpolicy.md)。 
 
    > [!TIP]
    > 在数据库级别定义的策略将对数据库中的所有现有表和未来表应用相同的设置。 
    
1. 将以下命令之一复制到“查询窗格”并选择“运行” 。

    ```kusto
    .alter table TestTable policy streamingingestion enable
    ```

    或

    ```kusto
    .alter database StreamingTestDb policy streamingingestion enable
    ```

    :::image type="content" source="media/ingest-data-streaming/define-streaming-ingestion-policy.png" alt-text="在 Azure 数据资源管理器中创建群集时启用流式引入":::

[!INCLUDE [ingest-data-streaming-use](includes/ingest-data-streaming-types.md)]

[!INCLUDE [ingest-data-streaming-disable](includes/ingest-data-streaming-disable.md)]

## <a name="drop-the-streaming-ingestion-policy-in-the-azure-portal"></a>在 Azure 门户中删除流式引入策略

1. 在 Azure 门户中，转到 Azure 数据资源管理器群集，然后选择“查询”。 
1. 要从表中删除流引入策略，请将以下命令复制到“查询窗格”并选择“运行” 。

    ```Kusto
    .delete table TestTable policy streamingingestion 
    ```

    :::image type="content" source="media/ingest-data-streaming/delete-streaming-ingestion-policy.png" alt-text="在 Azure 数据资源管理器中创建群集时启用流式引入":::

1. 在“设置”中选择“配置”。 
1. 在“配置”窗格中，选择“关闭”以禁用“流式引入”。  
1. 选择“保存” 。

    :::image type="content" source="media/ingest-data-streaming/streaming-ingestion-off.png" alt-text="在 Azure 数据资源管理器中创建群集时启用流式引入":::

[!INCLUDE [ingest-data-streaming-limitations](includes/ingest-data-streaming-limitations.md)]

## <a name="next-steps"></a>后续步骤

* [在 Azure 数据资源管理器中查询数据](web-query-data.md)
