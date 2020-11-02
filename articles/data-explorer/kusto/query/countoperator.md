---
title: count 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 count 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 04/16/2020
ms.date: 10/29/2020
ms.openlocfilehash: 4b142c851487373613337e1ce0429146cb456ee8
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104448"
---
# <a name="count-operator"></a>count 运算符

返回输入记录集中的记录数。

## <a name="syntax"></a>语法

`T | count`

## <a name="arguments"></a>参数

*T* ：待计算记录数的表格数据。

## <a name="returns"></a>返回

此函数返回包含单个记录且列类型为 `long` 的表。 唯一单元格的值是 *T* 中的记录数。 

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents | count
```
