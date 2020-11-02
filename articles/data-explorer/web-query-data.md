---
title: 快速入门：在 Azure 数据资源管理器 Web UI 中查询数据
description: 本快速入门介绍如何在 Azure 数据资源管理器 Web UI 中查询和共享数据。
author: orspod
ms.author: v-tawe
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: quickstart
origin.date: 06/15/2020
ms.date: 09/30/2020
ms.openlocfilehash: b0555c426d8d9c37a5bf13e1e97171eb8db60e79
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104804"
---
# <a name="quickstart-query-data-in-azure-data-explorer-web-ui"></a>快速入门：在 Azure 数据资源管理器 Web UI 中查询数据

Azure 数据资源管理器是一项快速且高度可缩放的数据探索服务，适用于日志和遥测数据。 Azure 数据资源管理器提供了可用于运行和共享查询的 Web 应用程序。 该应用程序可在 Azure 门户中使用，也可作为独立的 Web 应用程序使用。 在本文中，将使用独立版本，该版本允许连接到多个群集并共享指向查询的深层链接。

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

## <a name="prerequisites"></a>先决条件

除 Azure 订阅外，还需[测试群集和数据库](create-cluster-database-portal.md)才能完成此快速入门。

## <a name="sign-in-to-the-application"></a>登录应用程序

登录到[应用程序](https://dataexplorer.azure.cn/)。

## <a name="add-clusters"></a>添加群集

首次打开应用程序时，未进行任何连接。

![添加群集](media/web-query-data/add-cluster.png)

开始运行查询前，必须先将连接添加到至少一个群集中。 在本部分中，将连接添加到我们为帮助学习而设置的 Azure 数据资源管理器 help 群集，以及在之前的快速入门中创建的测试群集中。

1. 在应用程序的左上角，选择“添加群集”。

1. 在“添加群集”对话框中，输入 URI，然后选择“添加” 。

   可使用 help 群集 URI `https://help.kusto.chinacloudapi.cn`。 如果你有自己的群集，请提供自己的群集 URI。 例如，`https://mydataexplorercluster.chinanorth2.kusto.chinacloudapi.cn`，如下图所示：

    ![门户中的服务器 URI](media/web-query-data/server-uri.png)

1. 在左侧窗格中，现在应看到 help 群集。 展开“示例”数据库，以便可查看有权访问的示例表。

    ![示例数据库](media/web-query-data/sample-databases.png)

    我们在本快速入门后面部分以及其他 Azure 数据资源管理器文章中使用 StormEvents 表。

现在添加创建的测试群集。

1. 选择“添加群集”。

1. 在“添加群集”对话框中，以 `https://<ClusterName>.<Region>.kusto.chinacloudapi.cn/` 格式输入测试群集 URL，然后选择“添加” 。

    在以下示例中，会看到 help 群集和一个新群集 docscluster.chinaeast2（完整 URL 为 `https://docscluster.chinanorth2.kusto.chinacloudapi.cn/`）   。

    ![测试群集](media/web-query-data/test-cluster.png)

## <a name="run-queries"></a>运行查询

现在可针对连接到的任一群集运行查询（假设测试群集中有数据）。 我们将重点介绍 help 群集。

1. 在左窗格中的 help 群集下，选择示例数据库 。

1. 将以下查询复制并粘贴到查询窗口中。 在窗口顶部，选择“运行”。

    ```Kusto
    StormEvents
    | sort by StartTime desc
    | take 10
    ```
    此查询返回 StormEvents 表中的十条最新记录。 结果的左侧应如下表所示。

    :::image type="content" source="media/web-query-data/result-set-01.png" alt-text="表的屏幕截图，其中列出了 10 个风暴事件的开始时间、结束时间、事件叙述、事件 ID、状态和事件类型。" border="false":::

    下图显示了应用程序现在应处的状态，添加了群集以及带有结果的查询。

    ![完整应用程序](media/web-query-data/full-application.png)

1. 将以下查询复制并粘贴到第一个查询下方的查询窗口中。 请注意，它与首个查询在单独行上格式化的方式有何不同。

    ```Kusto
    StormEvents | sort by StartTime desc | project StartTime, EndTime, State, EventType, DamageProperty, EpisodeNarrative | take 10
    ```

1. 单击窗口中的新建查询，该窗口将选择该查询。 按 Shift+Alt+F 设置查询格式，如下所示。

    ![已格式化查询](media/web-query-data/formatted-query.png)

1. 按 Shift+Enter，这是运行查询的快捷方式。

   此查询返回与第一条记录相同的记录，但仅包括 `project` 语句中指定的列。 结果应如下表所示。

    :::image type="content" source="media/web-query-data/result-set-02.png" alt-text="表的屏幕截图，其中列出了 10 个风暴事件的开始时间、结束时间、事件叙述、事件 ID、状态和事件类型。" border="false":::

1. 在查询窗口顶部，选择“撤回”。

    查询窗口现在显示第一个查询的结果集，而无需重新运行该查询。 通常在分析期间，会运行多个查询，通过“撤回”可以重新访问先前查询的结果。

1. 让我们再运行一个查询来查看不同类型的输出。

    ```Kusto
    StormEvents
    | summarize event_count=count(), mid = avg(BeginLat) by State
    | sort by mid
    | where event_count > 1800
    | project State, event_count
    | render columnchart
    ```
    结果应如下图所示。

    ![柱形图](media/web-query-data/column-chart.png)

> [!NOTE]
> 查询表达式中的空白行可能影响执行查询的那一部分。
>
> 如果未选定文本，则假定以空行分隔查询或命令。
> 如果选定了文本，则运行选定的文本。

## <a name="work-with-the-table-grid"></a>使用网格型

现在你已了解了基本查询的工作原理，让我们看看如何使用网格型来自定义结果并进行进一步分析。

1. 重新运行第一个查询。 将鼠标悬停在“州”列上，选择菜单，然后选择“按州分组” 。

    ![按州分组](media/web-query-data/group-by.png)

1. 在网格中，展开加利福尼亚州，以查看该州的记录。

    :::image type="content" source="media/web-query-data/result-set-03.png" alt-text="表的屏幕截图，其中列出了 10 个风暴事件的开始时间、结束时间、事件叙述、事件 ID、状态和事件类型。" border="false":::

    进行探索性分析时，此类型的分组可能会有所帮助。

1. 将鼠标悬停在“组”列上，然后选择“重置列” 。

    ![重置列](media/web-query-data/reset-columns.png)

    此操作将网格返回到其原始状态。

1. 运行以下查询。

    ```Kusto
    StormEvents
    | sort by StartTime desc
    | where DamageProperty > 5000
    | project StartTime, State, EventType, DamageProperty, Source
    | take 10
    ```

1. 在网格的右侧，选择“列”以查看工具面板。

    ![工具面板](media/web-query-data/tool-panel.png)

    此面板的功能类似于 Excel 中的数据透视表字段列表，可在网格中执行更多分析。

1. 选择“透视模式”，然后按以下方式拖动列：将“状态”拖动到“行组”；将“DamageProperty”拖动到“值”，并将“EventType”拖动到“列标签”     。  

    ![透视模式](media/web-query-data/pivot-mode.png)

    结果应如以下数据透视表所示。

    ![数据透视表](media/web-query-data/pivot-table.png)

    请注意佛蒙特州和阿拉巴马州各有两个属于同一类别的事件，而德克萨斯州有两个不同类别的事件。 通过数据透视表可以快速发现这样的情况，它们是快速分析的绝佳工具。

## <a name="share-queries"></a>共享查询

很多时候，你希望共享创建的查询。 

1. 在查询窗口中，选择复制的第一个查询。

1. 在查询窗口顶部，选择“共享”。 

:::image type="content" source="media/web-query-data/share-menu.png" alt-text="表的屏幕截图，其中列出了 10 个风暴事件的开始时间、结束时间、事件叙述、事件 ID、状态和事件类型。":::

下拉列表中提供了以下选项：
* 链接到剪贴板
* [将查询链接到剪贴板](#provide-a-deep-link)
* 将查询结果链接到剪贴板
* [固定到仪表板](#pin-to-dashboard)
* [Power BI 查询](power-bi-imported-query.md)

### <a name="provide-a-deep-link"></a>提供深层链接

可提供深层链接，以便有权访问群集的其他用户可运行查询。

1. 在“共享”中，选择“将查询链接到剪贴板” 。

1. 将链接和查询复制到文本文件。

1. 将链接粘贴到新的浏览器窗口中。 运行查询后，结果应如下所示。

    :::image type="content" source="media/web-query-data/shared-query.png" alt-text="表的屏幕截图，其中列出了 10 个风暴事件的开始时间、结束时间、事件叙述、事件 ID、状态和事件类型。":::

### <a name="pin-to-dashboard"></a>固定到仪表板

在 Web UI 中通过查询完成数据探索并找到所需的数据后，可将其固定到仪表板中以便持续监视。 

若要固定查询：

1. 在“共享”中，选择“固定到仪表板” 。

1. 在“固定到仪表板”窗格中：
    1. 提供查询名称。
    1. 选择“使用现有查询”或“创建新查询” 。
    1. 提供仪表板名称
    1. 选择“创建后查看仪表板”复选框（如果是新仪表板）。
    1. 选择“固定”

    :::image type="content" source="media/web-query-data/pin-to-dashboard.png" alt-text="表的屏幕截图，其中列出了 10 个风暴事件的开始时间、结束时间、事件叙述、事件 ID、状态和事件类型。":::
    
> [!NOTE]
> “固定到仪表单”选项仅固定所选的查询。 要创建仪表板数据源并将显示命令转换为仪表板中的视觉对象，必须在数据库列表中选择相关数据库。

## <a name="export-query-results"></a>导出查询结果

要将查询结果导出到 CSV 文件，请选择“文件” > “导出到 CSV” 。

:::image type="content" source="media/web-query-data/export-results.png" alt-text="表的屏幕截图，其中列出了 10 个风暴事件的开始时间、结束时间、事件叙述、事件 ID、状态和事件类型。":::

## <a name="provide-feedback"></a>提供反馈

1. 在应用程序的右上角，选择反馈图标 ![反馈图标](media/web-query-data/icon-feedback.png).

1. 输入反馈，然后选择“提交”。

## <a name="clean-up-resources"></a>清理资源

未在此快速入门中创建任何资源，但如果要从应用程序中删除一个或两个群集，请右键单击群集并选择“删除连接”。

## <a name="next-steps"></a>后续步骤

[Azure 数据资源管理器的编写查询](write-queries.md)
