---
title: 在 Azure 数据资源管理器中创建表
description: 了解如何通过一键式操作在 Azure 数据资源管理器中轻松创建表。
author: orspod
ms.author: v-tawe
ms.reviewer: tzgitlin
ms.service: data-explorer
ms.topic: how-to
origin.date: 09/06/2020
ms.date: 10/30/2020
ms.openlocfilehash: 358e19ba6982c1d8cce59d93bc2935bfa5499cbd
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106537"
---
# <a name="create-a-table-in-azure-data-explorer-preview"></a>在 Azure 数据资源管理器中创建表（预览）

创建表是 Azure 数据资源管理器中[数据引入](ingest-data-overview.md)和[查询](write-queries.md)过程中的一个重要步骤。 [在 Azure 数据资源管理器中创建群集和数据库后](create-cluster-database-portal.md)，可以创建一个表。 下文介绍如何使用 Azure 数据资源管理器 Web UI 快速轻松地创建表和架构映射。 

## <a name="prerequisites"></a>先决条件

* 如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://wd.azure.cn/pricing/1rmb-trial/)。
* 创建 [Azure 数据资源管理器群集和数据库](create-cluster-database-portal.md)。
* 登录到 [Azure 数据资源管理器 Web UI](https://dataexplorer.azure.cn/) 并[添加与群集的连接](web-query-data.md#add-clusters)。

## <a name="create-a-table"></a>创建表

1. 在 Web UI 的左侧菜单中，右键单击数据库名称“ExampleDB”，并选择“创建表(预览)” 。

    :::image type="content" source="./media/one-click-table/create-table.png" alt-text="使用 Azure 数据资源管理器 Web UI 创建表":::

“创建表”窗口随即打开，并已选中“源”选项卡 。
1. “数据库”字段自动填充数据库。 可以从下拉菜单中选择其他数据库。
1. 在“表名称”中，输入表的名称。 
    > [!TIP]
    >  表名称最多可包含 1024 个字符，包括字母数字、连字符和下划线。 不支持特殊字符。

### <a name="select-source-type"></a>选择源类型

1. 在“源类型”中，选择将用于创建表映射的数据源。 从以下选项中选择：“从 blob”、“从文件”或“从容器”  。
   
    
    * 如果使用的是容器：
        * 输入 blob 的存储 URL，可以选择输入样本大小。 
        * 使用文件筛选器筛选文件。 
        * 选择将在下一步中用于定义架构的文件。

        :::image type="content" source="media/one-click-table/storage.png" alt-text="使用 blob 创建表以创建架构映射":::
    
    * 如果使用的是本地文件：
        * 选择“浏览”以找到文件，或者将文件拖放到字段中。

        :::image type="content" source="./media/one-click-table/data-from-file.png" alt-text="基于本地文件中的数据创建表":::

    * 如果使用的是 blob：
        * 在“链接到存储”字段中，添加容器的 [SAS URL](/vs-azure-tools-storage-explorer-blobs#get-the-sas-for-a-blob-container)，可以选择输入样本大小。 

2. 选择“编辑架构”以继续转到“架构”选项卡 。

### <a name="edit-schema"></a>编辑架构

在“架构”选项卡中，将在左侧窗格中自动标识数据格式和压缩。 如果标识错误，请使用“数据格式”下拉菜单选择正确的格式。

   * 如果数据格式为 JSON，则还必须选择 JSON 级别（1 到 10）。 级别确定表列数据分割。
   * 如果数据格式为 CSV，请选中“包括列名”复选框，以忽略文件的标题行。

        :::image type="content" source="./media/one-click-table/schema-tab.png" alt-text="在 Azure 数据资源管理器中的一键式创建表体验中编辑架构选项卡":::
 
1. 在“映射”中，为此表的架构映射输入名称。 
    > [!TIP]
    >  表名称可以包含字母数字字符和下划线。 不支持空格、特殊字符和连字符。
2. 选择“创建”  。

## <a name="create-table-completed-window"></a>“创建表已完成”窗口

如果创建表成功完成，则“创建表已完成”窗口中的两个步骤都会标有绿色的勾选标记。

* 选择“查看命令”，打开编辑器以执行每个步骤。 
    * 在编辑器中，可以查看和复制基于输入生成的自动命令。
    
    :::image type="content" source="./media/one-click-table/table-completed.png" alt-text="在一键式创建表体验中完成表创建 - Azure 数据资源管理器":::
 
在“创建表”进程下方的磁贴中，浏览“快速查询”或“工具”  ：

* “快速查询”包含指向 Web UI（其中包含示例查询）的链接。

* “工具”包含用于撤销正在运行的相关 .drop 命令的链接 。

> [!NOTE]
> 使用 .drop 命令时，可能会丢失数据。<br>
> 此工作流中的 drop 命令将仅还原由创建表进程（新表和架构映射）所做的更改。

## <a name="next-steps"></a>后续步骤

* [数据引入概述](ingest-data-overview.md)
* [Azure 数据资源管理器的编写查询](write-queries.md)  
