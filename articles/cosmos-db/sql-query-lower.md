---
title: Azure Cosmos DB 查询语言中的 LOWER
description: 了解 Azure Cosmos DB 中的 LOWER SQL 系统函数，以便返回在将大写字符数据转换为小写后的字符串表达式
author: rockboyfor
ms.service: cosmos-db
ms.topic: conceptual
origin.date: 09/13/2019
ms.date: 04/27/2020
ms.author: v-yeche
ms.custom: query-reference
ms.openlocfilehash: a1fc979ffb969b80e76cd4c64141baed7faae8d7
ms.sourcegitcommit: f9c242ce5df12e1cd85471adae52530c4de4c7d7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/24/2020
ms.locfileid: "82134555"
---
# <a name="lower-azure-cosmos-db"></a>LOWER (Azure Cosmos DB)
 返回在将大写字符数据转换为小写后的字符串表达式。  

LOWER 系统函数不使用索引。 如果计划频繁进行不区分大小写的比较，则 LOWER 系统函数可能会消耗大量 RU。 如果是这种情况，可以在插入时将大小写规范化，而不是每次都使用 LOWER 系统函数将数据规范化以进行比较。 于是，诸如 SELECT * FROM c WHERE LOWER(c.name) = 'bob' 的查询只需变成 SELECT * FROM c WHERE c.name = 'bob'。

## <a name="syntax"></a>语法

```sql
LOWER(<str_expr>)  
```  

## <a name="arguments"></a>参数

*str_expr*  
  是一个字符串表达式。  

## <a name="return-types"></a>返回类型

  返回一个字符串表达式。  

## <a name="examples"></a>示例

  以下示例演示如何在查询中使用 `LOWER`。  

```sql
SELECT LOWER("Abc") AS lower
```  

 下面是结果集。  

```json
[{"lower": "abc"}]  

```  

## <a name="remarks"></a>备注

此系统函数不会使用索引。

## <a name="next-steps"></a>后续步骤

- [字符串函数 Azure Cosmos DB](sql-query-string-functions.md)
- [系统函数 Azure Cosmos DB](sql-query-system-functions.md)
- [Azure Cosmos DB 简介](introduction.md)

<!-- Update_Description: update meta properties, wording update, update link -->