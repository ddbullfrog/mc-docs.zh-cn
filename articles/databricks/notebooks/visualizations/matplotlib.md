---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/07/2020
title: Matplotlib - Azure Databricks
description: 了解如何在 Azure Databricks Python 笔记本中显示 Matplotlib 图形。
ms.openlocfilehash: 4de31583ba88f002fcc19094d1faab46c61fdcc4
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106766"
---
# <a name="matplotlib"></a>Matplotlib

用于显示 Matplotlib 图形的方法取决于群集正在运行的 Databricks Runtime 版本：

* Databricks Runtime 6.5 及更高版本：直接支持以内联方式显示图表。
* Databricks Runtime 6.4：调用 `%matplotlib inline` magic 命令。
* Databricks Runtime 6.3：将群集配置为 `spark.databricks.workspace.matplotlibInline.enabled = true`，并调用 `%matplotlib inline` magic 命令。
* Databricks Runtime 6.2 及更低版本：使用 `display` 函数。

以下笔记本演示如何在 Python 笔记本中显示 [Matplotlib](https://matplotlib.org/) 图形。

## <a name="matplotlib-python-notebook"></a>Matplotlib Python 笔记本

[获取笔记本](../../_static/notebooks/matplotlib.html)

## <a name="render-images-at-higher-resolution"></a>在更高的分辨率下呈现图像

可以在 Python 笔记本中以标准分辨率的两倍呈现 matplotlib 图像，为高分辨率屏幕用户提供更好的可视化效果体验。 在笔记本单元中设置以下项之一：

`retina` 选项：

```python
%config InlineBackend.figure_format = 'retina'

from IPython.display import set_matplotlib_formats
     set_matplotlib_formats('retina')
```

`png2x` 选项：

```python
%config InlineBackend.figure_format = 'png2x'

from IPython.display import set_matplotlib_formats
set_matplotlib_formats('png2x')
```

若要切换回标准分辨率，请将以下内容添加到笔记本单元：

```python
set_matplotlib_formats('png')

%config InlineBackend.figure_format = 'png'
```