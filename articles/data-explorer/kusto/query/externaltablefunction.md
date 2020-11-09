---
title: External_table() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 external_table()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 08/21/2019
ms.date: 10/29/2020
ms.openlocfilehash: 188a6e14187e8106a50357fef853d6684449b1e5
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105152"
---
# <a name="external_table"></a>external_table()

按名称引用[外部表](schema-entities/externaltables.md)。

```kusto
external_table('StormEvent')
```

> [!NOTE]
> * `external_table` 函数与 [table](tablefunction.md) 函数具有类似的限制。
> * 标准[查询限制](../concepts/querylimits.md)还适用于外部表查询。

## <a name="syntax"></a>语法

`external_table` `(` *TableName* [`,` *MappingName* ] `)`

## <a name="arguments"></a>参数

* TableName：正在查询的外部表的名称。
  必须是引用 `blob`、`adl` 或 `sql` 类型的外部表的字符串文本。

* *MappingName* ：映射对象的可选名称，该对象将实际（外部）数据分片中的字段映射到此函数输出的列。

## <a name="next-steps"></a>后续步骤

* [外部表常规控制命令](../management/external-table-commands.md)
* [在 Azure 存储或 Azure Data Lake 中创建和更改外部表](../management/external-tables-azurestorage-azuredatalake.md)
* [创建和更改外部 SQL 表](../management/external-sql-tables.md)