---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: Set - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 SET 属性语法。
ms.openlocfilehash: 14eac0f4fcf89d4e63fd3126c53f9fab09fd8887
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472842"
---
# <a name="set"></a>设置

```sql
SET [-v]
SET property_key[=property_value]
```

设置属性、返回现有属性的值或列出所有现有属性。 如果为现有属性键提供值，则将替代旧值。

**`-v`**

输出现有属性的含义。

**`property_key`**

设置或返回单个属性的值。