---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: 创建 Bloom 筛选器索引 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Delta Lake SQL 语言的 CREATE BLOOMFILTER INDEX 语法。
ms.openlocfilehash: 50cb8e27d0c7620b592e01d906340f735970becf
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472821"
---
# <a name="create-bloom-filter-index"></a>创建 Bloom 筛选器索引

```sql
CREATE BLOOMFILTER INDEX
ON [TABLE] table_name
[FOR COLUMNS(columnName1 [OPTIONS(..)], columnName2, ...)]
[OPTIONS(..)]
```

为新数据或重写的数据创建 Bloom 筛选器索引；它不会为现有数据创建 Bloom 筛选器。 如果表名或表中的某一列不存在，则该命令将失败。 如果对某一列启用了 Bloom 筛选，则新的选项将替换现有的 Bloom 筛选器选项。

尽管无法为已写入的数据生成 Bloom 筛选器索引，但 [OPTIMIZE](optimize.md) 命令会为已重新组织的数据更新 Bloom 筛选器。 因此，在以下情况下，可以通过对表运行 `OPTIMIZE` 来回填 Bloom 筛选器：

* 以前未对表进行优化。
* 使用不同的文件大小，要求重新写入数据文件。
* 使用 `ZORDER`（或不同的 `ZORDER`，如果已存在一个），要求重新写入数据文件。

可以通过在列级别或表级别定义选项来优化 Bloom 筛选器：

* `fpp`：假正概率。 每个写入的 Bloom 筛选器所需的假正率。 这会影响在 Bloom 筛选器中放置单个项时所需的位数，并影响 Bloom 筛选器的大小。 该值必须大于 0 且小于或等于 1。 默认值为 0.1，它要求每个项 5 位。
* `numItems`：文件可包含的非重复项的数目。 此设置对筛选质量非常重要，因为它会影响 Bloom 筛选器中使用的总位数（项数 * 每个项的位数）。 如果此设置不正确，则会对 Bloom 筛选器进行非常稀疏的填充，从而浪费磁盘空间并降低必须下载此文件的查询的速度，或者它会太满，并且准确率下降（更高的 FPP）。 该值必须大于 0。 默认值为 1000000 项。
* `maxExpectedFpp`：未写入到磁盘的 Bloom 筛选器的预期 FPP 阈值。 写入 Bloom 筛选器时的最大预期假正概率。 如果预期 FPP 大于此阈值，则 Bloom 筛选器的选择性太低；使用 Bloom 筛选器所花费的时间和资源超出了其有用性。 该值必须介于 0 和 1 之间。 默认值为 1.0（已禁用）。

这些选项仅在写入数据时才起作用。 可以在不同的层次结构级别（写入操作、表级别和列级别）配置这些属性。 列级别优先于表级别和操作级别，表级别优先于操作级别。

请参阅 [Bloom 筛选器索引](../../../../delta/optimizations/bloom-filters.md)。