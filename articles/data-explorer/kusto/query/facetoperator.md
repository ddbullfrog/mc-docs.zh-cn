---
title: facet 运算符 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 facet 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 5c1396c69287a4e6e7bb1900ae71b75cea071006
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105155"
---
# <a name="facet-operator"></a>facet 运算符

返回一组表（每个指定列一个表）。
每个表指定其列所采用的值的列表。
可以使用 `with` 子句创建附加表。

## <a name="syntax"></a>语法

T `| facet by` ColumnName [`, ` ...] [`with (` filterPipe `)`

## <a name="arguments"></a>参数

* ColumnName：输入中的列的名称（要汇总为输出表）。
* filterPipe：一个查询表达式，应用于输入表以生成一个输出。

## <a name="returns"></a>返回

多个表：`with` 子句对应一个表，每个列对应一个表。

## <a name="example"></a>示例

```kusto
MyTable 
| facet by city, eventType 
    with (where timestamp > ago(7d) | take 1000)
```