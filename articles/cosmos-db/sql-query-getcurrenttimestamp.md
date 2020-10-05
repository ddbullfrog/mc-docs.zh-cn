---
title: Azure Cosmos DB 查询语言中的 GetCurrentTimestamp
description: 了解 Azure Cosmos DB 中的 SQL 系统函数 GetCurrentTimestamp。
ms.service: cosmos-db
ms.topic: conceptual
origin.date: 08/19/2020
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.custom: query-reference
ms.openlocfilehash: c51885aa78766f8a1a9c0a1912b87dad54320ffe
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246816"
---
# <a name="getcurrenttimestamp-azure-cosmos-db"></a>GetCurrentTimestamp (Azure Cosmos DB)

 返回自 1970 年 1 月 1 日星期四 00:00:00 开始消逝的毫秒数。

## <a name="syntax"></a>语法

```sql
GetCurrentTimestamp ()  
```  

## <a name="return-types"></a>返回类型

返回一个有符号的数值，表示自 Unix 纪元以来当前已经过的毫秒数，即自 1970 年 1 月 1 日星期四 00:00:00 以来已经过的毫秒数。

## <a name="remarks"></a>备注

GetCurrentTimestamp() 是非确定性的函数。 返回的结果采用 UTC（协调世界时）格式。

此系统函数不会使用索引。

## <a name="examples"></a>示例

  以下示例演示如何使用 GetCurrentTimestamp() 内置函数获取当前时间戳。

```sql
SELECT GetCurrentTimestamp() AS currentUtcTimestamp
```  

 下面是示例结果集。

```json
[{
  "currentUtcTimestamp": 1556916469065
}]  
```

## <a name="next-steps"></a>后续步骤

- [日期和时间函数 Azure Cosmos DB](sql-query-date-time-functions.md)
- [系统函数 Azure Cosmos DB](sql-query-system-functions.md)
- [Azure Cosmos DB 简介](introduction.md)

<!-- Update_Description: update meta properties, wording update, update link -->