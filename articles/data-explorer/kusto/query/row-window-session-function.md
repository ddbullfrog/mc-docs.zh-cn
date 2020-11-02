---
title: row_window_session() - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 row_window_session()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 75a26b101104daeaac9ef0c5fd47ef60bd27afdb
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105210"
---
# <a name="row_window_session"></a>row_window_session()

`row_window_session()` 计算[序列化行集](./windowsfunctions.md#serialized-row-set)中列的会话开始值。

## <a name="syntax"></a>语法

`row_window_session` `(` *`Expr`* `,` *`MaxDistanceFromFirst`* `,` *`MaxDistanceBetweenNeighbors`* [`,` *`Restart`* ] `)`

* *`Expr`* 是一个表达式，其值在会话中分组在一起。
  Null 值生成 Null 值，下一个值将启动新的会话。
  *`Expr`* 必须是 `datetime` 类型的标量表达式。

* *`MaxDistanceFromFirst`* 建立用于启动新会话的一个条件：在会话开始时 *`Expr`* 的当前值与 *`Expr`* 的值之间的最大距离。
  它是 `timespan` 类型的标量常数。

* *`MaxDistanceBetweenNeighbors`* 建立用于启动新会话的另一个条件：从 *`Expr`* 的一个值到下一个值的最大距离。
  它是 `timespan` 类型的标量常数。

* Restart 是 `boolean` 类型的可选标量表达式。 如果已指定，则计算结果为 `true` 的每个值将立即重启会话。

## <a name="returns"></a>返回

该函数在每个会话的开头返回值。

**备注**

该函数具有以下概念计算模型：

1. 按顺序检查 *`Expr`* 的输入值序列。

1. 对于每个值，确定它是否建立新的会话。

1. 如果它建立新会话，则发出 *`Expr`* 的值。 否则将发出 *`Expr`* 的先前值。

确定值是否表示新会话的条件是以下条件的逻辑或：

* 如果没有以前的会话值，或者以前的会话值为 Null。

* 如果 *`Expr`* 的值等于或超过上一个会话值加上 *`MaxDistanceFromFirst`* 。

* 如果 *`Expr`* 的值等于或超过 *`Expr`* 的上一个值加上 *`MaxDistanceBetweenNeighbors`* 。

## <a name="examples"></a>示例

下面的示例演示如何为包含两列的表计算会话开始值：一个用于标识序列的 `ID` 列，以及一个提供每个记录发生时间的 `Timestamp` 列。 在此示例中，一个会话不能超过 1 小时，并且只要记录之间的间隔短于 5 分钟，该会话就会继续。

```kusto
datatable (ID:string, Timestamp:datetime) [
    // ...
]
| sort by ID asc, Timestamp asc
| extend SessionStarted = row_window_session(Timestamp, 1h, 5m, ID != prev(ID))
```