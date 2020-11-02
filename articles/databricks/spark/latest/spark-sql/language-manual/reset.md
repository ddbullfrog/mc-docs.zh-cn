---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 11/22/2019
title: Reset - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 RESET 语法。
ms.openlocfilehash: d21673ad4cd844b31f1eeff6b855ba77792bc20b
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473013"
---
# <a name="reset"></a>重置

```sql
RESET
```

将所有属性重置为其默认值。 此后，[Set](set.md) 命令的输出将为空。