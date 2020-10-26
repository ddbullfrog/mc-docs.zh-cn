---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: 任务抢占 - Azure Databricks
description: 了解如何使用任务抢占在 Azure Databricks 中强制公平共享。
ms.openlocfilehash: b838310d296ef0c4e3ea81499c6f9e4f558cf464
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121856"
---
# <a name="task-preemption"></a>任务抢占

Azure Databricks 中的 Apache Spark 计划程序会自动抢占任务以强制进行公平共享。 对于具有许多并发运行作业的群集，这保证了它们的交互式响应时间。

> [!TIP]
>
> 当任务被计划程序抢占时，其终止原因将设置为 `preempted by scheduler`。 此原因在 Spark UI 中可见，可用于调试抢占行为。

## <a name="preemption-options"></a>抢占选项

默认情况下，抢占是保守的：在计划程序干预之前，作业可能会缺少资源长达 30 秒。 可以通过在群集启动时设置以下 Spark 配置属性来调整抢占：

* 是否应启用抢占。

  ```ini
  spark.databricks.preemption.enabled true
  ```

* 用于保证每个作业的公平份额分数。 设置为 1.0 时意味着计划程序将积极尝试确保完美的公平份额。 设置为 0.0 时可有效禁用抢占。 默认设置为 0.5，这意味着在最坏的情况下，工作将得到其公平份额的一半。

  ```ini
  spark.databricks.preemption.threshold 0.5
  ```

* 在进行抢占之前，作业必须保持耗尽的时间。 设置为较低值时可提供更长的交互式响应时间，但会降低群集效率。 建议值为 1-100 秒。

  ```ini
  spark.databricks.preemption.timeout 30s
  ```

* 计划程序检查任务抢占的频率。 应设置为小于抢占超时的值。

  ```ini
  spark.databricks.preemption.interval 5s
  ```

有关作业计划的详细信息，请参阅[应用程序中的计划](https://spark.apache.org/docs/latest/job-scheduling.html#scheduling-within-an-application)。