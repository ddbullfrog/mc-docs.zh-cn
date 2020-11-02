---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/16/2020
title: sparklyr - Azure Databricks
description: 了解如何使用 Azure Databricks 中的 sparklyr 来处理 R 中的 Apache Spark。
ms.openlocfilehash: a6b1b3de820674c01e2c3584e16b2fc94fff713e
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473063"
---
# <a name="sparklyr"></a>sparklyr

Azure Databricks 支持笔记本、作业和 RStudio 桌面中的 [sparklyr](https://spark.rstudio.com/)。

## <a name="requirements"></a>要求

Azure Databricks 通过每个运行时版本分发 sparklyr 的最新稳定版本。 可以通过导入已安装的 sparklyr 版本，在 Azure Databricks R 笔记本或托管在 Azure Databricks 上的 RStudio Server 内使用 sparklyr。

在 RStudio Desktop 中，Databricks Connect 允许将 Sparklyr 从本地计算机连接到 Azure Databricks 群集并运行 Apache Spark 代码。 请参阅[将 sparklyr 和 RStudio Desktop 与 Databricks Connect 配合使用](../../../dev-tools/databricks-connect.md#sparklyr-rstudio)。

## <a name="connect-sparklyr-to-azure-databricks-clusters"></a>将 sparklyr 连接到 Azure Databricks 群集

若要建立 sparklyr 连接，可以使用 `"databricks"` 作为 `spark_connect()` 中的连接方法。
不需要 `spark_connect()` 的其他参数，也不需要调用 `spark_install()`，因为 Spark 已安装在 Azure Databricks 群集上。

```r
# create a sparklyr connection
sc <- spark_connect(method = "databricks")
```

## <a name="progress-bars-and-spark-ui-with-sparklyr"></a>具有 sparklyr 的进度条和 Spark UI

如上例所示，如果将 sparklyr 连接对象分配给名为 `sc` 的变量，则在每个触发 Spark 作业的命令后，你将在笔记本中看到 Spark 进度条。
此外，还可以单击进度条旁边的链接，查看与给定 Spark 作业关联的 Spark UI。

> [!div class="mx-imgBorder"]
> ![Sparklyr 进度](../../../_static/images/third-party-integrations/sparklyr/sparklyr-progress.png)

## <a name="use-sparklyr"></a>使用 sparklyr

安装 sparklyr 并建立连接后，所有其他 [sparklyr API](https://spark.rstudio.com/reference/index.html) 将正常工作。
有关一些示例，请参阅[示例笔记本](#notebook)。

sparklyr 通常与其他 [tidyverse 包](https://www.tidyverse.org/)（例如 [dplyr](https://cran.rstudio.com/web/packages/dplyr/vignettes/dplyr.html)）一起使用。
为方便起见，其中大多数包已预安装在 Databricks 上。
只需导入它们即可开始使用 API。

## <a name="use-sparklyr-and-sparkr-together"></a>结合使用 sparklyr 和 SparkR

SparkR 和 sparklyr 可在单个笔记本或作业中一起使用。
可以将 SparkR 与 sparklyr 一起导入，并使用其功能。
在 Azure Databricks 笔记本中，SparkR 连接是预先配置的。

SparkR 中的某些函数屏蔽了 dplyr 中的部分函数：

```r
> library(SparkR)
The following objects are masked from ‘package:dplyr’:

arrange, between, coalesce, collect, contains, count, cume_dist,
dense_rank, desc, distinct, explain, filter, first, group_by,
intersect, lag, last, lead, mutate, n, n_distinct, ntile,
percent_rank, rename, row_number, sample_frac, select, sql,
summarize, union
```

如果在导入 dplyr 后导入 SparkR，则可以通过使用完全限定的名称（例如 `dplyr::arrange()`）引用 dplyr 中的函数。
同样，如果在 SparkR 后导入 dplyr，则 SparkR 中的函数将被 dplyr 屏蔽。

或者，可以在不需要时选择性地拆离这两个包中的一个。

```r
detach("package:dplyr")
```

## <a name="use-sparklyr-in-spark-submit-jobs"></a>在 spark-submit 作业中使用 sparklyr

可以在 Azure Databricks 上运行使用 sparklyr 的脚本作为 spark-submit 作业，并进行少量代码修改。 上述部分说明不适用于在 Azure Databricks 上的 spark-submit 作业中使用 sparklyr。 特别是，必须向 `spark_connect` 提供 Spark 主 URL。 有关示例，请参阅[创建并运行适用于 R 脚本的 spark-submit 作业](../../../dev-tools/api/latest/examples.md#spark-submit-api-example-r)。

## <a name="unsupported-features"></a>不支持的功能

Azure Databricks 不支持需要本地浏览器的 sparklyr 方法，如 `spark_web()` 和 `spark_log()`。 不过，由于 Spark UI 是内置在 Azure Databricks 上的，因此你可以轻松地查看 Spark 作业和日志。
请参阅[群集驱动程序和工作器日志](../../../clusters/clusters-manage.md#driver-logs)。

### <a name="sparklyr-notebook"></a><a id="notebook"> </a><a id="sparklyr-notebook"> </a>Sparklyr 笔记本

[获取笔记本](../../../_static/notebooks/sparklyr.html)