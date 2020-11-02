---
title: basket 插件 - Azure 数据资源管理库
description: 本文介绍了 Azure 数据资源管理器中的 basket 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 05/26/2019
ms.date: 10/29/2020
ms.openlocfilehash: bbdb01c63c728514878e298ba26332337d4a9797
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104040"
---
# <a name="basket-plugin"></a>basket 插件

```kusto
T | evaluate basket()
```

Basket 在数据中查找所有离散属性（维度）的频繁模式。 然后，它返回在原始查询中通过了频率阈值的频繁模式。 Basket 能确保查找出数据中的所有频繁模式，但不能保证存在多项式运行时。 在行数方面，查询的运行时是线性的，但在列数（维度）方面，却可能是指数的。 Basket 基于最初开发用于 basket 分析数据挖掘的 Apriori 算法。

## <a name="syntax"></a>语法

`T | evaluate basket(` *arguments* `)`

## <a name="returns"></a>返回

Basket 返回出现在行的比率阈值以上的所有频繁模式。 默认阈值为 0.05。 每种模式均由结果中的一行表示。

第一列是段 ID。 后两列是此模式从原始查询中所捕获行的数量和百分比 。 其余列均来自原始查询。
它们的值是来自列中的特定值，或者是表示变量值的通配符值（默认为 null）。

**参数（全部可选）**

`T | evaluate basket(`[ *Threshold* , *WeightColumn* , *MaxDimensions* , *CustomWildcard* , *CustomWildcard* , ...]`)`

所有参数都为可选参数，但必须按上述方式进行排序。 若要指示应使用默认值，请使用字符串波形值 -“~”。 请参阅以下示例。

可用参数：

* Threshold - 0.015 < double < 1 [默认值:0.05]

    为被视为频繁的行设置最小比率。 不会返回比率更小的模式。
    
    示例： `T | evaluate basket(0.02)`

* WeightColumn - column_name

    根据指定的权重考虑输入中的每一行。 默认情况下，每一行的权重都是“1”。 该参数必须是数值列的名称，例如，int、long、real。 权重列的常见用法是对已嵌入每一行的数据进行采样或存储/聚合。

    示例： `T | evaluate basket('~', sample_Count)`

* MaxDimensions - 1 < *int* [默认值：5]

    设置默认情况下受限制的每个 basket 不相关维度的最大数量，以最大程度地减少查询运行时。

    示例： `T | evaluate basket('~', '~', 3)`

* CustomWildcard - *"any_value_per_type"*

    为结果表中的特定类型设置通配符，表明当前模式对此列没有任何限制。
    默认值为 null。 字符串的默认值为空字符串。 如果默认值是可行数据值，应使用其他通配符值，例如 `*`。

    例如：

     `T | evaluate basket('~', '~', '~', '*', int(-1), double(-1), long(0), datetime(1900-1-1))`

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents 
| where monthofyear(StartTime) == 5
| extend Damage = iff(DamageCrops + DamageProperty > 0 , "YES" , "NO")
| project State, EventType, Damage, DamageCrops
| evaluate basket(0.2)
```

|段 ID|计数|百分比|状态|EventType|损害|损害农作物|
|---|---|---|---|---|---|---|---|---|
|0|4574|77.7|||是|0
|1|2278|38.7||冰雹|是|0
|2|5675|96.4||||0
|3|2371|40.3||冰雹||0
|4|1279|21.7||雷雨大风||0
|5|2468|41.9||冰雹||
|6|1310|22.3|||是|
|7|1291|21.9||雷雨大风||

使用自定义通配符的示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents 
| where monthofyear(StartTime) == 5
| extend Damage = iff(DamageCrops + DamageProperty > 0 , "YES" , "NO")
| project State, EventType, Damage, DamageCrops
| evaluate basket(0.2, '~', '~', '*', int(-1))
```

|段 ID|计数|百分比|状态|EventType|损害|损害农作物|
|---|---|---|---|---|---|---|---|---|
|0|4574|77.7|\*|\*|是|0
|1|2278|38.7|\*|冰雹|是|0
|2|5675|96.4|\*|\*|\*|0
|3|2371|40.3|\*|冰雹|\*|0
|4|1279|21.7|\*|雷雨大风|\*|0
|5|2468|41.9|\*|冰雹|\*|-1
|6|1310|22.3|\*|\*|是|-1
|7|1291|21.9|\*|雷雨大风|\*|-1
