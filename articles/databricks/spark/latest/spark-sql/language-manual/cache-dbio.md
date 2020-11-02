---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/02/2020
title: 缓存 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 CACHE SELECT 语法。
ms.openlocfilehash: ee81afbd96fb7300873de628c9801d78fceabb5d
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473038"
---
# <a name="cache"></a>缓存

```sql
CACHE SELECT column_name[, column_name, ...] FROM [db_name.]table_name [ WHERE boolean_expression ]
```

在[增量缓存](../../../../delta/optimizations/delta-cache.md)中缓存由指定的简单 `SELECT` 查询访问的数据。 可以通过提供列名称列表来选择要缓存的列的子集，并通过提供谓词来选择行的子集。 这使得后续查询可以尽可能避免扫描原始文件。 此构造仅适用于 Parquet 表。 如上所述，还支持视图，但扩展的查询仅限于简单查询。

请参阅[增量和 Apache Spark 缓存](../../../../delta/optimizations/delta-cache.md#delta-and-rdd-cache-comparison)以了解 RDD 缓存和 Databricks IO 缓存之间的差异。

## <a name="examples"></a>示例

```sql
CACHE SELECT * FROM boxes
CACHE SELECT width, length FROM boxes WHERE height=3
```