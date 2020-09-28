---
title: Azure Cosmos DB 查询语言中的 LOG10
description: 了解 Azure Cosmos DB 中的 LOG10 SQL 系统函数，以便返回指定数值表达式的以 10 为底的对数
ms.service: cosmos-db
ms.topic: conceptual
origin.date: 09/13/2019
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.custom: query-reference
ms.openlocfilehash: b2232045a8ef3057ea22aa295107f191b110ae95
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246700"
---
# <a name="log10-azure-cosmos-db"></a>LOG10 (Azure Cosmos DB)
 返回指定数值表达式以 10 为底的对数。  

## <a name="syntax"></a>语法

```sql
LOG10 (<numeric_expr>)  
```  

## <a name="arguments"></a>参数

*numeric_expression*  
  是一个数值表达式。  

## <a name="return-types"></a>返回类型

  返回一个数值表达式。  

## <a name="remarks"></a>备注

  LOG10 和 POWER 函数互为反函数。 例如，10 ^ LOG10(n) = n。 此系统函数不会使用索引。

## <a name="examples"></a>示例

  以下示例声明一个变量并返回指定变量 (100) 的 LOG10 值。  

```sql
SELECT LOG10(100) AS log10 
```  

 下面是结果集。  

```json
[{log10: 2}]  
```  

## <a name="next-steps"></a>后续步骤

- [数学函数 Azure Cosmos DB](sql-query-mathematical-functions.md)
- [系统函数 Azure Cosmos DB](sql-query-system-functions.md)
- [Azure Cosmos DB 简介](introduction.md)

<!-- Update_Description: update meta properties, wording update, update link -->