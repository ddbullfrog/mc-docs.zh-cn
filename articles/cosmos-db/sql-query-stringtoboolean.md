---
title: Azure Cosmos DB 查询语言中的 StringToBoolean
description: 了解 Azure Cosmos DB 中的 SQL 系统函数 StringToBoolean。
author: rockboyfor
ms.service: cosmos-db
ms.topic: conceptual
origin.date: 03/03/2020
ms.date: 04/27/2020
ms.author: v-yeche
ms.custom: query-reference
ms.openlocfilehash: 20cf9fdd479542131b384e25ad2f96b7d7c00485
ms.sourcegitcommit: f9c242ce5df12e1cd85471adae52530c4de4c7d7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/24/2020
ms.locfileid: "82134830"
---
# <a name="stringtoboolean-azure-cosmos-db"></a>StringToBoolean (Azure Cosmos DB)
 返回已转换为布尔值的表达式。 如果表达式无法转换，则返回未定义的表达式。  

## <a name="syntax"></a>语法

```sql
StringToBoolean(<str_expr>)  
```  

## <a name="arguments"></a>参数

*str_expr*  
  是要解析为布尔表达式的字符串表达式。  

## <a name="return-types"></a>返回类型

  返回一个布尔表达式或未定义的表达式。  

## <a name="examples"></a>示例

  以下示例演示 `StringToBoolean` 在不同类型中的行为方式。 

 下面是输入有效的示例。

只能在 "true"/"false" 之前或之后使用空格。

```sql
SELECT 
    StringToBoolean("true") AS b1, 
    StringToBoolean("    false") AS b2,
    StringToBoolean("false    ") AS b3
```  

 下面是结果集。  

```json
[{"b1": true, "b2": false, "b3": false}]
```  

下面是输入无效的示例。

 布尔值区分大小写，必须全用小写字符（即 "true" 和 "false"）来表示。

```sql
SELECT 
    StringToBoolean("TRUE"),
    StringToBoolean("False")
```  

下面是结果集。  

```json
[{}]
``` 

传递的表达式将会解析为布尔表达式；以下输入不会计算为布尔类型，因此会返回未定义。

```sql
SELECT 
    StringToBoolean("null"),
    StringToBoolean(undefined),
    StringToBoolean(NaN), 
    StringToBoolean(false), 
    StringToBoolean(true)
```  

下面是结果集。  

```json
[{}]
```  

## <a name="remarks"></a>备注

此系统函数不会使用索引。

## <a name="next-steps"></a>后续步骤

- [字符串函数 Azure Cosmos DB](sql-query-string-functions.md)
- [系统函数 Azure Cosmos DB](sql-query-system-functions.md)
- [Azure Cosmos DB 简介](introduction.md)

<!-- Update_Description: update meta properties, wording update, update link -->