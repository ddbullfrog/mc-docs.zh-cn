---
title: Azure Cosmos DB 查询语言中的 LOG
description: 了解 Azure Cosmos DB 中的 LOG SQL 系统函数，以便返回指定数值表达式的自然对数
ms.service: cosmos-db
ms.topic: conceptual
origin.date: 09/13/2019
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.custom: query-reference
ms.openlocfilehash: 7940cae19384001e12d3ded312c8aa832df5f440
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246702"
---
# <a name="log-azure-cosmos-db"></a>LOG (Azure Cosmos DB)
 返回指定数值表达式的自然对数。  

## <a name="syntax"></a>语法

```sql
LOG (<numeric_expr> [, <base>])  
```  

## <a name="arguments"></a>参数

*numeric_expr*  
  是一个数值表达式。  

*base*  
  可选的数值参数，用于设置对数的底。  

## <a name="return-types"></a>返回类型

  返回一个数值表达式。  

## <a name="remarks"></a>备注

  默认情况下，LOG() 返回自然对数。 可以通过使用可选的 base 参数将对数的底更改为其他值。  

  自然对数是以 **e** 为底的对数，其中，**e** 是一个无理常量，约等于 2.718281828。 

  数字的指数的自然对数是数字本身：LOG( EXP( n ) ) = n。 以及数字的自然对数的指数是数字本身：EXP( LOG( n ) ) = n。

  此系统函数不会使用索引。

## <a name="examples"></a>示例

  以下示例声明一个变量并返回指定变量 (10) 的对数值。  

```sql
SELECT LOG(10) AS log  
```  

 下面是结果集。  

```json
[{log: 2.3025850929940459}]  
```  

 以下示例计算某个数字的指数的 `LOG`。  

```sql
SELECT EXP(LOG(10)) AS expLog  
```  

 下面是结果集。  

```json
[{expLog: 10.000000000000002}]  
```  

## <a name="next-steps"></a>后续步骤

- [数学函数 Azure Cosmos DB](sql-query-mathematical-functions.md)
- [系统函数 Azure Cosmos DB](sql-query-system-functions.md)
- [Azure Cosmos DB 简介](introduction.md)

<!-- Update_Description: update meta properties, wording update, update link -->