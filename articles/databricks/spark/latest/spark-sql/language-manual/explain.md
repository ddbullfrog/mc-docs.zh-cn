---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 11/22/2019
title: Explain - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 EXPLAIN 语法。
ms.openlocfilehash: 68dc6625b7d882642cbb63e14dd9ca2ba5ba5748
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472994"
---
# <a name="explain"></a>说明

```sql
EXPLAIN [EXTENDED | CODEGEN] statement
```

提供有关 `statement` 的详细计划信息，而无需实际运行。 默认情况下，这只会输出有关物理计划的信息。 不支持说明 `DESCRIBE TABLE`。

**`EXTENDED`**

在分析和优化前后输出有关逻辑计划的信息。

**`CODEGEN`**

输出语句的生成代码（如果有）。