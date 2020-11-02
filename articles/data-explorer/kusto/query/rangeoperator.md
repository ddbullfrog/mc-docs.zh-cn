---
title: range 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 range 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: a32bed5ff6f02f914381816926ae2a925e2ce4b0
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104010"
---
# <a name="range-operator"></a>range 运算符

生成值的单列表。

请注意，该表不含管道输入。 

## <a name="syntax"></a>语法

`range` columnName `from` start `to` stop `step` step

## <a name="arguments"></a>参数

* *columnName* ：输出表中的单列名称。
* *start* ：输出中的最小值。
* stop：输出中正生成的最大值（如果 step 跳过此值，则为最大值边界）。
* step：两个连续值之间的差异。 

参数必须为数字、日期或时间跨度值。 它们不能引用任何表的列。 （如需基于输入表计算范围，请使用 range 函数，可能需配合使用 mv-expand 运算符。） 

## <a name="returns"></a>返回

一个包含单个列 columnName 的表，其值包括 start、start `+` step...，直至 stop    。

## <a name="example"></a>示例  

过去 7 天午夜时分的表。 bin (floor) 函数将每次时间缩减到当天的开始。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
range LastWeek from ago(7d) to now() step 1d
```

|LastWeek|
|---|
|2015-12-05 09:10:04.627|
|2015-12-06 09:10:04.627|
|...|
|2015-12-12 09:10:04.627|


一个包含单个列 `Steps` 的表，其类型是 `long`，其值是 `1`、`4` 和 `7`。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
range Steps from 1 to 8 step 3
```

下一个示例演示如何使用 `range` 运算符创建小型的临时维度表，并使用该表在源数据不具有值的位置引入零。

```kusto
range TIMESTAMP from ago(4h) to now() step 1m
| join kind=fullouter
  (Traces
      | where TIMESTAMP > ago(4h)
      | summarize Count=count() by bin(TIMESTAMP, 1m)
  ) on TIMESTAMP
| project Count=iff(isnull(Count), 0, Count), TIMESTAMP
| render timechart  
```
