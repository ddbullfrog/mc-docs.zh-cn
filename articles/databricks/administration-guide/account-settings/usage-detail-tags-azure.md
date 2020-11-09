---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 05/14/2020
title: 使用群集、池和工作区标记监视使用情况 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用标记监视使用情况。
ms.openlocfilehash: 5ada06cd9af61ab6c30511643060074611ad0fee
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106484"
---
# <a name="monitor-usage-using-cluster-pool-and-workspace-tags"></a>使用群集、池和工作区标记监视使用情况

要监视成本并将 Azure Databricks 使用情况准确地划分到组织的业务部门和团队（例如退款），可以标记工作区（资源组）、群集和池。 这些标记传播到详细的[成本分析报表](/cost-management-billing/costs/quick-acm-cost-analysis)，你可以在 Azure 门户中访问这些报表。

例如，以下是 Azure 门户中的成本分析发票详细信息报表，该报表通过 `clusterid` 标记详细记录了一个月的成本：

> [!div class="mx-imgBorder"]
> ![按群集 ID 进行成本分析](../../_static/images/account-settings/tag-cost-analysis-clusterid.png)

## <a name="tagged-objects-and-resources"></a>标记的对象和资源

你可以为 Azure Databricks 托管的下列对象添加自定义标记：

| Object          | 标记界面 (UI)                                                                                     | 标记界面 (API)                                                   |
|-----------------|------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------|
| 工作区       | [Azure 门户](/azure-resource-manager/management/tag-resources#portal)    | [Azure 资源 API](https://docs.microsoft.com/rest/api/resources/tags) |
| 池            | Azure Databricks 工作区中的[池 UI](../../clusters/instance-pools/configure.md#instance-pool-tags) | [实例池 API](../../dev-tools/api/latest/instance-pools.md)         |
| 群集         | Azure Databricks 工作区中的[群集 UI](../../clusters/configure.md#cluster-tags)                   | [群集 API](../../dev-tools/api/latest/clusters.md)                    |

Azure Databricks 将以下默认标记添加到所有池和群集中：

| 池标记密钥名称                 | “值”                                                                        |
|-----------------------------------|------------------------------------------------------------------------------|
| `Vendor`                          | 常数“Databricks”                                                        |
| `DatabricksInstancePoolCreatorId` | 创建池的用户的 Azure Databricks 内部标识符        |
| `DatabricksInstancePoolId`        | 池的 Azure Databricks 内部标识符                             |

| 群集标记密钥名称 | “值”                                                        |
|----------------------|--------------------------------------------------------------|
| `Vendor`             | 常数“Databricks”                                        |
| `ClusterId`          | 群集的 Azure Databricks 内部标识符          |
| `ClusterName`        | 群集的名称                                          |
| `Creator`            | 创建群集的用户的用户名（电子邮件地址） |

在作业群集上，Azure Databricks 还应用以下默认标记：

| 群集标记密钥名称 | 值    |
|----------------------|----------|
| `RunName`            | 作业名称 |
| `JobId`              | 作业 ID   |

## <a name="tag-propagation"></a>标记传播

工作区、池和群集标记由 Azure Databricks 聚合并传播到 Azure VM，用于[成本分析报表](/cost-management-billing/costs/quick-acm-cost-analysis)。 但池和群集标记的传播方式彼此不同。

> [!div class="mx-imgBorder"]
> ![按群集 ID 进行成本分析](../../_static/images/account-settings/tag-propagation.png)

工作区和池标记进行聚合并分配为托管池的 Azure VM 的资源标记。

工作区和群集标记进行聚合并分配为托管群集的 Azure VM 的资源标记。

从池中创建群集时，只会将工作区标记和池标记传播到 VM。 不传播群集标记，以保持池群集启动性能。

### <a name="tag-conflict-resolution"></a>标记冲突解决

如果自定义群集标记、池标记或工作区标记与 Azure Databricks 默认群集或池标记具有相同的名称，则该自定义标记在传播时将以 `x_` 作为前缀。

例如，如果工作区标记有 `vendor = Azure Databricks`，则该标记将与默认的群集标记 `vendor = Databricks` 冲突。 因此，标记将作为 `x_vendor = Azure Databricks` 和 `vendor = Databricks` 传播。

### <a name="limitations"></a>限制

* 在进行任何更改后，自定义工作区标记传播到 Azure Databricks 可能需要长达一个小时的时间。
* 不能为 Azure 资源分配超过 50 个标记。 如果聚合标记的总计数超过此限制，带 `x_` 前缀的标记将按字母顺序计算，超出限制的标记将被忽略。 如果忽略所有带 `x_` 前缀的标记，并且一直计数直到超过限制，则剩余的标记将按照字母顺序计算，而超出限制的标记将被忽略。
* 标记键和值只能包含来自 ISO 8859-1 (latin1) 集的字符。 包含其他字符的标记将被忽略。
* 如果更改标记键名称或值，则这些更改仅在群集重启或池扩展之后才适用。