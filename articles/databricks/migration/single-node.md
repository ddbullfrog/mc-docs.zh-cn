---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/10/2020
title: 将单节点工作负载迁移到 Azure Databricks -Azure Databricks
description: 了解如何将单节点工作负载迁移到 Azure Databricks。
ms.openlocfilehash: 220a439879da2b4edb57379e1f8211801ef21486
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937756"
---
# <a name="migrate-single-node-workloads-to-azure-databricks"></a>将单节点工作负荷迁移到 Azure Databricks

本文解答将单节点工作负载迁移到 Azure Databricks 时出现的典型问题。

我刚刚创建了 20 个节点的 Spark 群集，但 pandas 代码并没有更快运行。出什么问题了吗？

如果你使用的是任意单节点库，则在你切换为使用 Azure Databricks 时，它们本质上不会变成分布式库。 你需要使用 Apache Spark Python API（即 [PySpark](https://spark.apache.org/docs/latest/api/python/index.html)）重写代码。

或者，你可以考虑安装 [Koalas](../languages/koalas.md)，通过它你可以使用 pandas DataFrame API 来访问 Apache Spark DataFrames 中的数据。

我发现提供有 MLLib 和 SparkML。两者有何不同？

MLLib 是基于 RDD 的 API，而 SparkML 是基于 DataFrame 的 API。 我们建议你使用 SparkML，因为所有活动开发都集中在 SparkML。 然而，有时人们使用术语 MLLib 来泛指 Spark 的分布式 ML 库。

我喜欢 sklearn 中的一种算法，但 SparkML 不支持它（例如 DBSCAN）。我有哪些替代选择？

Spark-sklearn。 请参阅 [spark_sklearn 文档](https://databricks.github.io/spark-sklearn-docs/)。

SparkML 的部署选项有哪些？

* 批处理预计算
* 结构化流。 请参阅[结构化流](../spark/latest/structured-streaming/index.md)。
* 利用 MLeap 进行实时推理。 请参阅 [MLeap ML 模型导出](../applications/machine-learning/model-export/mleap-model-export.md#mleap-model-export)。

为什么我的 matplotlib 图像无法显示？

在大多数情况下，必须将所有图包装在 `display()` 函数中。 如果你运行 Databricks Runtime 6.3 及更高版本，则可以将群集配置为以内联方式显示图像。 请参阅 [Matplotlib](../notebooks/visualizations/matplotlib.md)。

如何安装或升级 pandas 或其他库？

下面是几个选项：

* 使用 Azure Databricks [库 UI 或 API](../libraries/index.md)。 它们将在群集中的每个节点上安装库。
* 使用[库实用工具](../dev-tools/databricks-utils.md#dbutils-library)。
* 通过 `%sh /databricks/python/bin/pip install` 安装库。

  此命令仅在 Apache Spark 驱动程序（而不是工作程序）上安装库，如果重启该群集，此库将被删除。 若要在群集上的所有节点上安装库，请使用 shell 命令重启，然后使用[初始化脚本](../clusters/init-scripts.md)。

为什么 `%sh pip install <library-name>` 安装 Python 2 版本，即使我正在 Python 3 群集上运行也是如此？

默认 pip 适用于 Python 2，因此你必须使用 `%sh /databricks/python/bin/pip` 以使用 Python 3。

如何只通过驱动程序查看 DBFS 上的数据？

将 `/dbfs/` 添加到文件路径的开头。 请参阅[本地文件 API](../data/databricks-file-system.md#fuse)。

如何将数据导入 Azure Databricks？

* 装载。 请参阅[将对象存储装载到 DBFS](../data/databricks-file-system.md#mount-storage)。
* “数据”选项卡。请参阅[数据概述](../data/data.md#access-data)。
* `%sh wget`

  如果 URL 中包含数据文件，则可以使用 `%sh wget <url>/<filename>` 将数据导入到 Spark 驱动程序节点。

  > [!NOTE]
  >
  > 单元格输出打印 `Saving to: '<filename>'`，但文件实际上保存到 `file:/databricks/driver/<filename>`。

  例如，如果你通过以下命令下载文件 `https://data.cityofnewyork.us/api/views/25th-nujf/rows.csv?accessType=DOWNLOAD`：

  ```bash
  %sh wget https://data.cityofnewyork.us/api/views/25th-nujf/rows.csv?accessType=DOWNLOAD
  ```

  若要加载此数据，请运行：

  ```python
  pandas_df = pd.read_csv("file:/databricks/driver/rows.csv?accessType=DOWNLOAD", header='infer')
  ```