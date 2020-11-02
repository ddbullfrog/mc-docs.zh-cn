---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: 显示函数 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 SHOW FUNCTIONS 语法。
ms.openlocfilehash: 311bf35606df29686c567caede7d81458fab0699
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472952"
---
# <a name="show-functions"></a>显示函数

```sql
SHOW [USER | SYSTEM |ALL ] FUNCTIONS ([LIKE] regex | [db_name.]function_name)
```

显示与给定 regex 或函数名称匹配的函数。 如果未提供 regex 或名称，则显示所有函数。 如果声明 `USER` 或 `SYSTEM`，则这些函数将只分别显示用户定义的 Spark SQL 函数和系统定义的 Spark SQL 函数。

**`LIKE`**

此限定符仅用于兼容性，不起作用。