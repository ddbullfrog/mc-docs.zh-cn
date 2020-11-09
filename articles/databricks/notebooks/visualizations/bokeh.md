---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/14/2020
title: Bokeh - Azure Databricks
description: 了解如何在 Azure Databricks 笔记本中使用 Bokeh（一个 Python 交互式可视化效果库）。
ms.openlocfilehash: 7bcf2670a55393b4ccd9553db6f6fae557d48eb3
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106646"
---
# <a name="bokeh"></a>Bokeh

[Bokeh](https://docs.bokeh.org/en/latest/) 是一个 Python 交互式可视化效果库。

若要使用 Bokeh，请通过[库](../../libraries/index.md) UI 安装 Bokeh PyPI 包，并将其附加到群集。

若要在 Azure Databricks 中显示 Bokeh 绘图，请执行以下操作：

1. 按照 [Bokeh 文档](https://docs.bokeh.org/en/latest/docs/user_guide/quickstart.html)中的说明生成绘图。
2. 通过某种方式（例如，使用 Bokeh 的 `file_html()` 或 `output_file()` 函数）生成一个包含绘图数据的 HTML 文件。
3. 将此 HTML 传递给 Azure Databricks `displayHTML()` 函数。

   > [!IMPORTANT]
   >
   > 笔记本单元格（内容和输出）的最大大小为 16MB。 请确保传递给 `displayHTML()` 函数的 HTML 大小不超过此值。

有关示例，请参阅以下笔记本。

## <a name="bokeh-demo-notebook"></a>bokeh 演示笔记本

[获取笔记本](../../_static/notebooks/bokeh-demo.html)