---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 06/11/2020
title: htmlwidgets - Azure Databricks
description: 了解如何通过在 Azure Databricks 笔记本中使用 R 的 htmlwidgets，利用 R 的灵活语法和环境生成交互式绘图。
ms.openlocfilehash: 47c8a4d6be8ddb0be91b2d1fd0c419faa123485a
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106638"
---
# <a name="htmlwidgets"></a>htmlwidgets

使用 [R 的 htmlwidgets](https://www.htmlwidgets.org/)，可以利用 R 的灵活语法和环境生成交互式绘图。 Azure Databricks 笔记本支持 htmlwidgets。

设置有两个步骤：

1. 安装 [pandoc](https://pandoc.org/)，这是 htmlwidgets 用来生成 HTML 的 Linux 包。
2. 更改 htmlwidgets 包中的一个函数，使其在 Azure Databricks 中正常工作。

你可以使用[初始化脚本](../../clusters/init-scripts.md)来自动执行第一步，以便群集在启动时安装 pandoc。 你应在使用 htmlwidgets 包的每个笔记本中执行第二步，即更改 htmlwidgets 函数。

笔记本演示如何将 htmlwidgets 与 [dygraphs](https://rstudio.github.io/dygraphs/)、[leaflet](https://rstudio.github.io/leaflet/) 和 [plotly](https://plot.ly/r/) 配合使用。

> [!IMPORTANT]
>
> 对于每个库调用，都将下载包含呈现的绘图的 HTML 文件。 该绘图不以内联方式显示。

## <a name="htmlwidgets-notebook"></a>htmlwidgets 笔记本

[获取笔记本](../../_static/notebooks/azure/htmlwidgets-azure.html)