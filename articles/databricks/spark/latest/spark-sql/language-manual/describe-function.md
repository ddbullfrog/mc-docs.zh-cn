---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 11/22/2019
title: 描述函数 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 DESCRIBE FUNCTION 语法。
ms.openlocfilehash: 693f13c818b47b67908c13bef8baaec4130069d3
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473119"
---
# <a name="describe-function"></a>描述函数

```sql
DESCRIBE FUNCTION [EXTENDED] [db_name.]function_name
```

返回现有函数（实现类和用法）的元数据。 如果该函数不存在，则会引发异常。

**`EXTENDED`**

显示扩展的用法信息。