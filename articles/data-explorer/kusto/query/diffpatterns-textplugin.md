---
title: diffpatterns_text 插件 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 diffpatterns_text 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 9af0d9e271fb6d94f722320badd73dfb13203b0c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104306"
---
# <a name="diffpatterns_text-plugin"></a>diffpatterns_text 插件

比较两个字符串值数据集，找出可以将这两个数据集之间的差异特征化的文本模式。

```kusto
T | evaluate diffpatterns_text(TextColumn, BooleanCondition)
```

`diffpatterns_text` 返回一组文本模式，这些模式捕获两个数据集中数据的差异部分（例如，当条件为 `true` 时，模式捕获占较大百分比的行；当条件为 `false` 时，模式捕获占较小百分比的行）。 模式是使用连续的标记（以空格分隔，来自文本列）或是使用表示通配符的 `*` 构建的。 每种模式均由结果中的一行表示。

## <a name="syntax"></a>语法

`T | evaluate diffpatterns_text(`TextColumn, BooleanCondition [, MinTokens, Threshold , MaxTokens]`)` 

## <a name="arguments"></a>参数

### <a name="required-arguments"></a>必需参数

* TextColumn - column_name

    要分析的文本列，必须是字符串类型的。
    
* BooleanCondition - 布尔表达式

    定义如何生成要与输入表进行比较的两个记录子集。 算法根据条件将查询拆分为两个数据集：“True”和“False”，然后分析它们之间的（文本）差异。 

### <a name="optional-arguments"></a>可选自变量

其他所有参数都为可选参数，但必须按以下方式进行排序。 

* MinTokens  - 0 < *int* < 200 [默认值：1]

    为每个结果模式设置非通配符标记的最小数目。

* Threshold - 0.015 < double < 1 [默认值:0.05]

    设置两个集之间的最小模式（比例）差异（请参阅 [diffpatterns](diffpatternsplugin.md)）。

* MaxTokens  - 0 < *int* [默认值：20]

    设置每个结果模式的最大标记数量（从开始算起），指定一个较低的限制会减少查询运行时间。

## <a name="returns"></a>返回

diffpatterns_text 的结果返回以下列：

* Count_of_True：当条件为 `true` 时与模式匹配的行数。
* Count_of_False：当条件为 `false` 时与模式匹配的行数。
* Percent_of_True：当条件为 `true` 时，行中与模式匹配的行所占百分比。
* Percent_of_False：当条件为 `false` 时，行中与模式匹配的行所占百分比。
* 模式：文本模式，其中包含来自文本字符串的标记和表示通配符的“`*`”。 

> [!NOTE]
> 这些模式不一定截然不同，可能未涵盖完整的数据集范围。 这些模式可能重叠，某些行可能与任何模式都不匹配。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents     
| where EventNarrative != "" and monthofyear(StartTime) > 1 and monthofyear(StartTime) < 9
| where EventType == "Drought" or EventType == "Extreme Cold/Wind Chill"
| evaluate diffpatterns_text(EpisodeNarrative, EventType == "Extreme Cold/Wind Chill", 2)
```

|Count_of_True|Count_of_False|Percent_of_True|Percent_of_False|模式|
|---|---|---|---|---|
|11|0|6.29|0|Winds shifting northwest in * wake * a surface trough brought heavy lake effect snowfall downwind * Lake Superior from|
|9|0|5.14|0|Canadian high pressure settled * * region * produced the coldest temperatures since February * 2006. Durations * freezing temperatures|
|0|34|0|6.24|* * * * * * * * * * * * * * * * * * West Tennessee,|
|0|42|0|7.71|* * * * * * caused * * * * * * * * across western Colorado. *|
|0|45|0|8.26|* * below normal *|
|0|110|0|20.18|Below normal *|
