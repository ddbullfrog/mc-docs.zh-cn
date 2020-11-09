---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 05/14/2020
title: 旧版折线图 - Azure Databricks
description: 了解 Azure Databricks 中的旧版折线图以及如何迁移到 Line。
ms.openlocfilehash: b673f1e5916026ac1cfb1440bbfdd4581b89e692
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106705"
---
# <a name="legacy-line-charts"></a>旧版折线图

Azure Databricks 具有三种类型的折线图：Line 以及旧版图表 Line (v2) 和 Line (v1)  。

## <a name="line-chart-comparison"></a>折线图比较

Line 具有自定义绘图选项：设置 Y 轴范围、显示或隐藏标记以及将对数标尺应用到 Y 轴和支持一组丰富的客户端交互的内置工具栏。

此外，Line、Line (v2) 和 Line (v1) 图表以不同的方式处理时序数据  ：

| Line 和 Line (v2)                                                                                                                      | Line (v1)                                                                                                               |
|-----------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| 支持日期和时间戳。<br><br>Line (v2) 将日期格式设置为本地时间。                                                        | 日期和时间戳被视为文本。                                                                                 |
| 键只能是日期、时间戳或数字。                                                                                             | 键可以是任何类型。                                                                                                 |
| X 轴是排序的：<br><br>* 日期、时间戳和数字的自然线性排序。<br>* 时间间隔显示为图表上的间隔。 | X 轴是分类的：<br><br>* 除非数据已排序，否则不排序。<br>* 图表中不显示时间间隔。 |

## <a name="migrate-to-line-from-legacy-line-chart-types"></a>从旧版折线图类型迁移到 Line

若要从 Line (v1) 或 Line (v2) 迁移到 Line  ：

1. 单击条形图![图表按钮](../../_static/images/notebooks/chart-button.png)旁边的![向下按钮](../../_static/images/button-down.png)，然后选择“Line”。

   > [!div class="mx-imgBorder"]
   > ![图表类型](../../_static/images/notebooks/display-charts.png)

2. 对于 Line (v1) 图表，如果键列不是日期、时间戳或数字，则必须按以下笔记本中所示将列显式分析为日期、时间戳或数字。

### <a name="timestamp-conversion-notebook"></a>时间戳转换笔记本

[获取笔记本](../../_static/notebooks/timestamp-conversion.html)

## <a name="use-legacy-line-charts"></a>使用旧版折线图

若要使用旧版图表，请从“旧版图表”下拉菜单中进行选择。

> [!div class="mx-imgBorder"]
> ![旧版图表类型](../../_static/images/notebooks/display-legacy-charts.png)