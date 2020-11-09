---
title: 使用 Power BI 的 Azure 数据资源管理器连接器直观显示数据
description: 本文介绍如何使用三个选项中的一个选项在 Power BI 中直观显示数据：Azure 数据资源管理器的 Power BI 连接器。
author: orspod
ms.author: v-tawe
ms.reviewer: gabil
ms.service: data-explorer
ms.topic: how-to
origin.date: 07/10/2019
ms.date: 09/30/2020
ms.openlocfilehash: 89c7d59e5b6a4a6d38f0507b46a46e7dcfd71484
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106089"
---
# <a name="visualize-data-using-the-azure-data-explorer-connector-for-power-bi"></a>使用 Power BI 的 Azure 数据资源管理器连接器直观显示数据

Azure 数据资源管理器是一项快速且高度可缩放的数据探索服务，适用于日志和遥测数据。 Power BI 是一种业务分析解决方案，可以用来可视化数据，并在组织内共享结果。 Azure 数据资源管理器提供三个可以在 Power BI 中连接到数据的选项：使用内置连接器、从 Azure 数据资源管理器导入查询，或者使用 SQL 查询。 本文介绍如何使用内置连接器获取数据并在 Power BI 报表中直观显示这些数据。 使用 Azure 数据资源管理器本机连接器创建 Power BI 仪表板非常简单。 Power BI 连接器支持[导入和直接查询连接模式](https://docs.microsoft.com/power-bi/desktop-directquery-about)。 根据方案、规模和性能需求，可以使用 **导入** 或 **DirectQuery** 模式构建仪表板。 

## <a name="prerequisites"></a>必备条件

若要完成本文，需要满足以下条件：

* 如果没有 Azure 订阅，可在开始前创建一个 [试用帐户](https://www.azure.cn/pricing/1rmb-trial) 。
* 一个属于 Azure Active Directory 成员的组织电子邮件帐户，以便连接到 [Azure 数据资源管理器帮助群集](https://dataexplorer.azure.cn/clusters/help/databases/samples)。
* [Power BI Desktop](https://powerbi.microsoft.com/get-started/)（选择“免费下载”  ）

## <a name="get-data-from-azure-data-explorer"></a>从 Azure 数据资源管理器获取数据

首先连接到 Azure 数据资源管理器帮助群集，然后从 *StormEvents* 表引入一部分数据。 [!INCLUDE [data-explorer-storm-events](includes/data-explorer-storm-events.md)]

1. 在 Power BI Desktop 的“主页”选项卡上选择“获取数据”，然后选择“更多”    。

    ![获取数据](media/power-bi-connector/get-data-more.png)

1. 搜索“Azure 数据资源管理器”  ，选择“Azure 数据资源管理器”，然后选择“连接”   。

    ![搜索和获取数据](media/power-bi-connector/search-get-data.png)

1. 在“Azure 数据资源管理器(Kusto)”  屏幕上，使用以下信息填写表单。

    ![群集、数据库、表选项](media/power-bi-connector/cluster-database-table.png)

    | 设置 | “值” | 字段说明
    |---|---|---
    | 群集 | *https://help.kusto.chinacloudapi.cn* | 帮助群集的 URL。 其他群集的 URL 采用 https://\<ClusterName\>.\<Region\>.kusto.chinacloudapi.cn 格式。 |
    | 数据库 | 留空 | 托管在要连接到的群集上的数据库。 我们会在后面的步骤中选择此项。 |
    | 表名称 | 留空 | 数据库中的一个表，或者类似 <code>StormEvents \| take 1000</code> 的查询。 我们会在后面的步骤中选择此项。 |
    | 高级选项 | 留空 | 查询选项，例如结果集大小。
    | 数据连接模式 | *DirectQuery* | 确定 Power BI 是导入数据还是直接连接到数据源。 可以对此连接器使用任一选项。 |

    > [!NOTE]
    > 在 **导入** 模式下，数据将移到 Power BI。 在 **DirectQuery** 模式下，直接从 Azure 数据资源管理器群集查询数据。
    >
    > 在以下情况下使用 **导入** 模式：
    >
    > * 数据集很小。
    > * 不需要接近实时的数据。
    > * 数据已聚合，或在 Kusto 中执行[聚合](kusto/query/summarizeoperator.md#list-of-aggregation-functions)
    >
    > 在以下情况下使用 **DirectQuery** 模式：
    > * 数据集非常大。
    > * 需要接近实时的数据。

    ### <a name="advanced-options"></a>高级选项

    | 设置 | 示例值 | 字段说明
    |---|---|---
    | 限制查询结果记录数| `300000` | 结果中返回的最大记录数 |
    | 限制查询结果数据大小 | `4194304` | 结果中返回的最大数据大小（以字节为单位） |
    | 禁用结果集截断 | `true` | 可以通过使用 notruncation 请求选项来启用/禁用结果截断 |
    | 附加的 set 语句 | `set query_datascope=hotcache` | 为查询的持续时间设置查询选项。 查询选项控制查询的执行方式并返回结果。 |

1. 如果还没有连接到帮助群集，请登录。 使用组织帐户登录，然后选择“连接”。 

    ![登录](media/power-bi-connector/sign-in.png)

1. 在“导航器”屏幕上，展开  **Samples** 数据库，选择“StormEvents”，然后选择“编辑”。  

    ![选择表](media/power-bi-connector/select-table.png)

    表在 Power Query 编辑器中打开，可以在其中编辑行和列，然后导入数据。

1. 在 Power Query 编辑器中，选择“DamageCrops”列旁边的箭头，然后选择“降序排序”。  

    ![将 DamageCrops 按降序排序](media/power-bi-connector/sort-descending.png)

1. 在“主页”选项卡中，选择“保留行”，然后选择“保留最前面几行”。    输入值 *1000* ，引入已存储表的前面 1000 行。

    ![保留最前面几行](media/power-bi-connector/keep-top-rows.png)

1. 在“主页”  选项卡上，选择“关闭并应用”  。

    ![关闭并应用](media/power-bi-connector/close-apply.png)

## <a name="visualize-data-in-a-report"></a>在报表中将数据可视化

[!INCLUDE [data-explorer-power-bi-visualize-basic](includes/data-explorer-power-bi-visualize-basic.md)]

## <a name="clean-up-resources"></a>清理资源

如果不再需要为本文创建的报表，请删除 Power BI Desktop (.pbix) 文件。

## <a name="next-steps"></a>后续步骤

[有关使用 Power BI 的 Azure 数据资源管理器连接器查询数据的提示](power-bi-best-practices.md#tips-for-using-the-azure-data-explorer-connector-for-power-bi-to-query-data)
