---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/24/2020
title: 图像 - Azure Databricks
description: 了解如何使用 Azure Databricks 读取图像文件 (.jpg)。
ms.openlocfilehash: 35912c5a8e380674d52fc791e0756ca0bd8c569b
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121825"
---
# <a name="image"></a>映像

[图像数据源](https://spark.apache.org/docs/latest/ml-datasource#image-data-source)从图像表示形式的详细信息中提取内容，并提供一个标准 API 来加载图像数据。 若要读取图像文件，请将数据源 `format` 指定为 `image`。

```python
df = spark.read.format("image").load("<path-to-image-data>")
```

Scala、Java 和 R 存在类似的 API。

你可以导入嵌套的目录结构（例如，使用 `/path/to/dir/` 之类的路径），也可以通过指定具有分区目录的路径（即 `/path/to/dir/date=2018-01-02/category=automobile` 之类的路径）来使用分区发现。

> [!NOTE]
>
> 如果不想对图像进行解码，Azure Databricks 建议你使用[二进制文件](binary-file.md)数据源。

## <a name="image-structure"></a>图像结构

图像文件将以数据帧的形式加载，其中包含一个名为 `image` 的单结构类型列，该列中包含以下字段：

```
image: struct containing all the image data
  |-- origin: string representing the source URI
  |-- height: integer, image height in pixels
  |-- width: integer, image width in pixels
  |-- nChannels
  |-- mode
  |-- data
```

其中的字段包括：

* `nChannels`：颜色通道数。 对于灰度图像，典型值为 1；对于彩色图像（例如 RGB），典型值为 3；对于具有 alpha 通道的彩色图像，典型值为 4。
* `mode`：一个整数标志，指示如何解释数据字段。 它指定数据类型和存储数据的通道顺序。 该字段的值应该（此处不是指强制）映射到下表中显示的 OpenCV 类型之一。 为 1、2、3 或 4 个通道定义了 OpenCV 类型，而为像素值定义了几种数据类型。 通道顺序指定了颜色存储的顺序。 例如，如果你有一个具有红色、蓝色和绿色成分的典型三通道图像，则有六种可能的排序。 大多数库使用 RGB 或 BGR。 三（四）通道 OpenCV 类型应采用 BGR(A) 顺序。

  **OpenCV 中从类型到数量的映射（数据类型 x 通道数）**

  | 类型           | C1               | C2               | C3               | C4               |
  |----------------|------------------|------------------|------------------|------------------|
  | CV_8U          | 0                | 8                | 16               | 24               |
  | CV_8S          | 1                | 9                | 17               | 25               |
  | CV_16U         | 2                | 10               | 18               | 26               |
  | CV_16S         | 3                | 11               | 19               | 27               |
  | CV_32S         | 4                | 12               | 20               | 28               |
  | CV_32S         | 5                | 13               | 21               | 29               |
  | CV_64F         | 6                | 14               | 22               | 30               |

* `data`：以二进制格式存储的图像数据。 图像数据以三维数组的形式表示，其中包含维度形状（height、width、nChannels）和由模式字段指定的 t 类型数组值。 数组以行优先顺序存储。

## <a name="display-image-data"></a>显示图像数据

Databricks `display` 函数支持显示图像数据。 请参阅[图像](../../notebooks/visualizations/index.md#display-image-type)。

## <a name="notebook"></a>笔记本

以下笔记本演示如何在图像文件中读取和写入数据。

### <a name="image-data-source-notebook"></a>图像数据源笔记本

[获取笔记本](../../_static/notebooks/image-data-source.html)