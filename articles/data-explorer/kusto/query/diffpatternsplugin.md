---
title: diffpatterns 插件 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 diffpatterns 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 6f588655a469345c953d3042772e41a43685023f
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104949"
---
# <a name="diff-patterns-plugin"></a>diffpatterns 插件

比较结构相同的两个数据集，找出描述这两个数据集之间差异的离散属性（维度）的模式。
 开发 `Diffpatterns` 的目的是帮助分析故障（例如，通过比较给定时间范围内的故障和非故障进行分析），但也可找出相同结构的任何两个数据集之间的差异。 

```kusto
T | evaluate diffpatterns(splitColumn)
```
> [!NOTE]
> `diffpatterns` 的目的是查找显著的模式（这将捕获集之间的数据差异部分），而不是查找逐行差异。

## <a name="syntax"></a>语法

`T | evaluate diffpatterns(SplitColumn, SplitValueA, SplitValueB [, WeightColumn, Threshold, MaxDimensions, CustomWildcard, ...])` 

## <a name="arguments"></a>参数 

### <a name="required-arguments"></a>必需参数

* SplitColumn - column_name 

    指示算法如何将查询分割成数据集。 算法根据指定的 SplitValueA 和 SplitValueB 参数的值（见下文），将查询分为“A”和“B”两个数据集，并分析二者间的差异。 因此，拆分列必须至少有两个不同的值。

* SplitValueA - string 

    指定的 SplitColumn 中一个值的字符串表示形式。 SplitColumn 中具有此值的所有行都会被视为数据集“A”。

* SplitValueB - string 

    指定的 SplitColumn 中一个值的字符串表示形式。 SplitColumn 中具有此值的所有行都会被视为数据集“B”。

    示例： `T | extend splitColumn=iff(request_responseCode == 200, "Success" , "Failure") | evaluate diffpatterns(splitColumn, "Success","Failure") `

### <a name="optional-arguments"></a>可选自变量

其他所有参数都为可选参数，但必须按以下方式进行排序。 若要指示应使用默认值，请输入字符串波形值 -“~”（请参阅下方示例）。

* WeightColumn - column_name

    根据指定的权重考虑输入中的每一行（默认情况下每行具有权重“1”）。 该参数必须是某个数值列（例如，`int`、`long`、`real`）的名称。
    权重列的常见用法是对已嵌入每一行的数据进行采样或存储/聚合。
    
    示例： `T | extend splitColumn=iff(request_responseCode == 200, "Success" , "Failure") | evaluate diffpatterns(splitColumn, "Success","Failure", sample_Count) `

* Threshold - 0.015 < double < 1 [默认值:0.05]

    设置两个集之间的最小模式（比例）差异。

    示例：`T | extend splitColumn = iff(request-responseCode == 200, "Success" , "Failure") | evaluate diffpatterns(splitColumn, "Success","Failure", "~", 0.04)`

* MaxDimensions  - 0 < *int* [默认值：无限制]

    为每个结果模式设置不相关维度的最大数目。 通过指定一个限制，可减少查询运行时间。

    示例：`T | extend splitColumn = iff(request-responseCode == 200, "Success" , "Failure") | evaluate diffpatterns(splitColumn, "Success","Failure", "~", "~", 3)`

* CustomWildcard - *"any-value-per-type"*

    为结果表中的特定类型设置通配符，表明当前模式对此列没有任何限制。
    默认值为 null，字符串的默认值为空字符串。 如果默认值是可行数据值，应使用其他通配符值（例如 `*`）。
    请参阅以下示例。

    示例： `T | extend splitColumn = iff(request-responseCode == 200, "Success" , "Failure") | evaluate diffpatterns(splitColumn, "Success","Failure", "~", "~", "~", int(-1), double(-1), long(0), datetime(1900-1-1))`

## <a name="returns"></a>返回

`Diffpatterns` 返回一小组模式，可捕获两个数据集中数据的不同部分（即，某一模式可捕获第一个数据集中占较大百分比的行和第二个数据集中占较小百分比的行）。 每种模式均由结果中的一行表示。

`diffpatterns` 的结果返回以下列：

* SegmentId：为当前查询中的模式分配的标识（注意：不保证 ID 在重复查询中是相同的）。

* CountA：集 A 中的模式捕获的行数（集 A 是 `where tostring(splitColumn) == SplitValueA` 的等效项）。

* CountB：集 B 中的模式捕获的行数（集 B 是 `where tostring(splitColumn) == SplitValueB` 的等效项）。

* PercentA：集 A 中由模式捕获的行所占百分比 (100.0 * CountA / count(SetA))。

* PercentB：集 B 中由模式捕获的行所占百分比 (100.0 * CountB / count(SetB))。

* PercentDiffAB：A 和 B 之间的百分比差绝对值 (|PercentA - PercentB|) 是描述两个集之间模式显著性差异的主要度量值。

* 其余列：是输入的原始架构，用于描述模式，每行（模式）表示列的非通配符值（等效于行中的每个非通配符值的 `where col1==val1 and col2==val2 and ... colN=valN`）的交集。

对于每个模式，未在该模式中设置（即，未对特定值施加限制）的列将包含一个通配符值，该值在默认情况下为 null。 请参阅下文中的“参数”部分，了解如何手动更改通配符。

* 注意：这些模式通常并非截然不同。 它们可能会重叠，且通常不涵盖所有原始行。 某些行可能不属于任何模式。

> [!TIP]
> * 在输入管道中使用 [where](./whereoperator.md) 和 [project](./projectoperator.md) 可将数据缩减到仅剩所需数据。
> * 找到所需行时，可通过将该行的特定值添加到 `where` 筛选器，来对其进行深入研究。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents 
| where monthofyear(StartTime) == 5
| extend Damage = iff(DamageCrops + DamageProperty > 0 , 1 , 0)
| project State , EventType , Source , Damage, DamageCrops
| evaluate diffpatterns(Damage, "0", "1" )
```

|段 ID|计数 A|计数 B|百分比 A|百分比 B|PercentDiffAB|状态|EventType|Source|损害农作物|
|---|---|---|---|---|---|---|---|---|---|
|0|2278|93|49.8|7.1|42.7||冰雹||0|
|1|779|512|17.03|39.08|22.05||雷雨大风|||
|2|1098|118|24.01|9.01|15|||专业观测员|0|
|3|136|158|2.97|12.06|9.09|||报纸||
|4|359|214|7.85|16.34|8.49||山洪|||
|5|50|122|1.09|9.31|8.22|衣阿华州||||
|6|655|279|14.32|21.3|6.98|||执法机构||
|7|150|117|3.28|8.93|5.65||洪水|||
|8|362|176|7.91|13.44|5.52|||灾害管理||
