---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/14/2020
title: Plotly - Azure Databricks
description: 了解如何将 Plotly 与 Azure Databricks 配合使用。
ms.openlocfilehash: b9cb9002c26de97a325992e95cc2fbb66548c3a7
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106756"
---
# <a name="plotly"></a>Plotly

[Plotly](https://pypi.org/project/plotly/) 是交互式绘图库。 Azure Databricks 支持 Plotly 2.0.7。 若要使用 Plotly，请[安装 Plotly PyPI 包](../../libraries/index.md)，并将其附加到群集。

> [!NOTE]
>
> 我们建议在 Azure Databricks 笔记本内部使用 [Plotly Offline](https://plot.ly/python/offline/#)。  处理大型数据集时，Plotly Offline 的[性能可能不太理想](https://community.plot.ly/t/offline-plotting-in-python-is-very-slow-on-big-data-sets/3077)。 如果你发现性能问题，则应减小数据集的大小。

若要显示 Plotly 绘图：

1. 将 `output_type='div'` 指定为 Plotly `plot()` 函数的参数。
2. 将 `plot()` 函数的输出传递到 Databricks `displayHTML()` 函数。

有关示例，请参阅笔记本。

## <a name="plotly-python-notebook"></a>Plotly Python 笔记本

[获取笔记本](../../_static/notebooks/plotly.html)