---
title: External_table() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 external_table()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 08/21/2019
ms.date: 08/18/2020
ms.openlocfilehash: d2dbccfb406ef604b4506b1ff7f973d04537316b
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146254"
---
# <a name="external_table"></a>external_table()

按名称引用外部表。

```kusto
external_table('StormEvent')
```

> [!NOTE]
> * `external_table` 函数与 [table](tablefunction.md) 函数具有类似的限制。
> * [外部表](schema-entities/externaltables.md)
> * [用于管理外部表的命令](../management/external-sql-tables.md)

## <a name="syntax"></a>语法

`external_table` `(` *TableName* [`,` *MappingName* ] `)`

## <a name="arguments"></a>参数

* TableName：正在查询的外部表的名称。
  必须是引用 `blob` 或 `adl` 类型的外部表的字符串文本。 <!-- TODO: Document data formats supported -->

* *MappingName*：映射对象的可选名称，该对象将实际（外部）数据分片中的字段映射到此函数输出的列。
