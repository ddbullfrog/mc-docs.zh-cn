---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 11/22/2019
title: 删除函数 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 DROP FUNCTION 语法。
ms.openlocfilehash: 01a8c32edd4b9ff4cc922726cb98917c20eabf3a
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472998"
---
# <a name="drop-function"></a>删除函数

```sql
DROP [TEMPORARY] FUNCTION [IF EXISTS] [db_name.]function_name
```

删除现有函数。 如果要删除的函数不存在，则会引发异常。

> [!NOTE]
>
> 仅当启用了 Hive 支持时，才支持此命令。

**`TEMPORARY`**

要删除的函数是否是临时函数。

**`IF EXISTS`**

如果要删除的函数不存在，则什么都不会发生。