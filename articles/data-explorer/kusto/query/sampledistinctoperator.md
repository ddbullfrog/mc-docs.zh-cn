---
title: sample-distinct 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 sample-distinct 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: ebd472c4a6d3b5d2ea46dc4558b5bf1fd0b95192
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105510"
---
# <a name="sample-distinct-operator"></a>sample-distinct 运算符

返回单个列，该列包含所请求列的非重复值，且数目不超过指定数目。 

运算符的默认的、仅限目前的风格是尝试尽快返回答案（而不是尝试生成一个公平的示例）

```kusto
T | sample-distinct 5 of DeviceId
```

## <a name="syntax"></a>语法

*T* `| sample-distinct` *NumberOfValues* `of` *ColumnName*

## <a name="arguments"></a>参数
* NumberOfValues：要返回的 T 的非重复值的数目。 可以指定任何数值表达式。

**提示**

 通过在 let 语句中使用 `sample-distinct`，然后使用 `in` 运算符进行筛选，可以轻松地进行总体抽样（参见示例） 

 如果要获得最靠前的值，而非仅仅一个示例，可以使用 [top-hitters](tophittersoperator.md) 运算符 

 如果要对数据行（而不是特定列的值）进行抽样，请参阅 [sample 运算符](sampleoperator.md)

## <a name="examples"></a>示例  

从总体获取 10 个非重复值

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents | sample-distinct 10 of EpisodeId

```

进行总体抽样，并执行进一步的计算，了解汇总不会超过查询限制。 

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let sampleEpisodes = StormEvents | sample-distinct 10 of EpisodeId;
StormEvents 
| where EpisodeId in (sampleEpisodes) 
| summarize totalInjuries=sum(InjuriesDirect) by EpisodeId
```
