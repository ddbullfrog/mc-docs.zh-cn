---
title: 教程 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的教程。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/23/2020
ms.date: 09/30/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 4050a6a3b6a7214c4f3201eb94de3e3c9f162c54
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106006"
---
# <a name="tutorial"></a>教程

::: zone pivot="azuredataexplorer"

了解 Kusto 查询语言的最好方法是查看一些简单的查询，以便使用[包含一些示例数据的数据库](https://help.kusto.chinacloudapi.cn/Samples)来“感受”该语言。 本文中演示的查询应在该数据库上运行。 这个示例数据库中的 `StormEvents` 表提供了在美国发生的风暴的一些相关信息。

<!--
  TODO: Provide link to reference data we used originally in StormEvents
-->

<!--
  TODO: A few samples below reference non-existent tables, such as Events and Logs.
        We need to add these tables.
-->

## <a name="count-rows"></a>统计行数

我们的示例数据库有一个名为 `StormEvents` 的表。
为了了解其大小，可通过管道符号将其内容注入只计算行数的运算符中：

* *语法：* 查询是一个数据源（通常是表名称），可以选择后跟一对或多对管道字符和一些表格运算符。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents | count
```

结果如下：

|计数|
|-----|
|59066|
    
[count 运算符](./countoperator.md)。

## <a name="project-select-a-subset-of-columns"></a>project：选择列的子集

使用 [project](./projectoperator.md) 挑选出所需列。 请参阅下面的示例，它同时使用 [project](./projectoperator.md) 和 [take](./takeoperator.md) 运算符。

## <a name="where-filtering-by-a-boolean-expression"></a>where：按布尔表达式筛选

让我们只查看 2007 年 2 月期间 `California` 中的 `flood`：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents
| where StartTime > datetime(2007-02-01) and StartTime < datetime(2007-03-01)
| where EventType == 'Flood' and State == 'CALIFORNIA'
| project StartTime, EndTime , State , EventType , EpisodeNarrative
```

|StartTime|EndTime|状态|EventType|EpisodeNarrative|
|---|---|---|---|---|
|2007-02-19 00:00:00.0000000|2007-02-19 08:00:00.0000000|CALIFORNIA|洪水|19 日凌晨，一股锋面系统穿过南部的 San Joaquin 山谷，给西部的 Kern 县带来了短时强降雨。 据报道，Taft 附近的 166 号州高速公路发生不严重的洪水。|

## <a name="take-show-me-n-rows"></a>take：显示 n 行

让我们看看一些数据 - 示例 5 行中有什么？

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents
| take 5
| project  StartTime, EndTime, EventType, State, EventNarrative  
```

|StartTime|EndTime|EventType|状态|EventNarrative|
|---|---|---|---|---|
|2007-09-18 20:00:00.0000000|2007-09-19 18:00:00.0000000|暴雨|佛罗里达州|在 24 小时的时间里，Volusia 县沿海部分地区降雨量高达 9 英寸。|
|2007-09-20 21:57:00.0000000|2007-09-20 22:05:00.0000000|龙卷风|佛罗里达州|龙卷风袭击了位于 West Crooked 湖北端的 Eustis 镇。 该龙卷风迅速增强为 EF1 级别，并向西北偏北移动，穿过 Eustis。 其移动轨迹将近 2 英里长，最大宽度为 300 码。  龙卷风摧毁了 7 座房屋。 27 所房屋受到严重破坏，81 所房屋报告有轻微损坏。 没有发生严重的人员伤亡，财产损失估计为 620 万美元。|
|2007-09-29 08:11:00.0000000|2007-09-29 08:11:00.0000000|海龙卷|大西洋南部|Melbourne 海滩东南的大西洋上形成了一个海龙卷，并短暂地向海岸移动。|
|2007-12-20 07:50:00.0000000|2007-12-20 07:53:00.0000000|雷雨大风|密西西比州|许多大树被吹倒，有些倒在电线上。 东部 Adams 县遭到破坏。|
|2007-12-30 16:00:00.0000000|2007-12-30 16:05:00.0000000|雷雨大风|佐治亚州|县政府报告说，在 206 国道附近的 Quincey Batten 环线上，有几棵树被吹倒。 预估了树木移除费用。|

但是 [take](./takeoperator.md) 不按特定顺序显示表中的行，所以现在对它们进行排序。
* [limit](./takeoperator.md) 是 [take](./takeoperator.md) 的别名，具有相同的效果。

## <a name="sort-and-top"></a>sort 和 top

* *语法：* 有些运算符具有由关键字（如 `by`）引入的参数。
* `desc` = 降序，`asc` = 升序。

显示前 n 行，按特定列排序：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents
| top 5 by StartTime desc
| project  StartTime, EndTime, EventType, State, EventNarrative  
```

|StartTime|EndTime|EventType|状态|EventNarrative|
|---|---|---|---|---|
|2007-12-31 22:30:00.0000000|2007-12-31 23:59:00.0000000|冬季风暴|密歇根州|这场大雪持续到元旦凌晨。|
|2007-12-31 22:30:00.0000000|2007-12-31 23:59:00.0000000|冬季风暴|密歇根州|这场大雪持续到元旦凌晨。|
|2007-12-31 22:30:00.0000000|2007-12-31 23:59:00.0000000|冬季风暴|密歇根州|这场大雪持续到元旦凌晨。|
|2007-12-31 23:53:00.0000000|2007-12-31 23:53:00.0000000|疾风|CALIFORNIA|据报道，Ventura 县的山区有北到东北阵风，风速约为每小时 58 英里。|
|2007-12-31 23:53:00.0000000|2007-12-31 23:53:00.0000000|疾风|CALIFORNIA|温泉 RAWS 传感器报告说，北风达到每小时 58 英里。|

同样可以通过依次使用 [sort](./sortoperator.md) 和 [take](./takeoperator.md) 运算符来实现

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents
| sort by StartTime desc
| take 5
| project  StartTime, EndTime, EventType, EventNarrative
```

## <a name="extend-compute-derived-columns"></a>extend：计算派生的列

通过计算每行中的值来创建新列：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents
| limit 5
| extend Duration = EndTime - StartTime 
| project StartTime, EndTime, Duration, EventType, State
```

|StartTime|EndTime|持续时间|EventType|状态|
|---|---|---|---|---|
|2007-09-18 20:00:00.0000000|2007-09-19 18:00:00.0000000|22:00:00|暴雨|佛罗里达州|
|2007-09-20 21:57:00.0000000|2007-09-20 22:05:00.0000000|00:08:00|龙卷风|佛罗里达州|
|2007-09-29 08:11:00.0000000|2007-09-29 08:11:00.0000000|00:00:00|海龙卷|大西洋南部|
|2007-12-20 07:50:00.0000000|2007-12-20 07:53:00.0000000|00:03:00|雷雨大风|密西西比州|
|2007-12-30 16:00:00.0000000|2007-12-30 16:05:00.0000000|00:05:00|雷雨大风|佐治亚州|

可以重用列名并将计算结果分配给同一列。
例如：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
print x=1
| extend x = x + 1, y = x
| extend x = x + 1
```

|x|y|
|---|---|
|3|1|

[Scalar 表达式](./scalar-data-types/index.md)可包含所有常用运算符（`+`、`-`、`*`、`/`、`%`），并含有一系列有用的函数。

## <a name="summarize-aggregate-groups-of-rows"></a>summarize：聚合行组

统计每个国家/地区的事件数：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents
| summarize event_count = count() by State
```

[summarize](./summarizeoperator.md) 将在 `by` 子句中拥有相同值的行组合在一起，然后使用聚合函数（如 `count`）将每个组合并到单个行。 因此，在此情况下，每个州都有相应的行，还有一个列，用来表示该州的行计数。

有一系列[聚合函数](./summarizeoperator.md#list-of-aggregation-functions)，并且可以在一个 summarize 运算符中使用其中多个函数，以生成多个计算列。 例如，我们可以获取每个州的风暴计数，以及每个州每个独特类型的风暴总计数，  
然后我们可以用 [top](./topoperator.md) 来获取受风暴影响最严重的州：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents 
| summarize StormCount = count(), TypeOfStorms = dcount(EventType) by State
| top 5 by StormCount desc
```

|状态|StormCount|TypeOfStorms|
|---|---|---|
|德克萨斯|4701|27|
|KANSAS|3166|21|
|衣阿华州|2337|19|
|ILLINOIS|2022|23|
|MISSOURI|2016|20|

summarize 的结果是：

* 以 `by` 命名的各列；
* 每个计算表达式相应的列；
* 每个 `by` 值组合相应的行。

## <a name="summarize-by-scalar-values"></a>按标量值汇总

可以在 `by` 子句中使用标量（数值、时间或间隔）值，但需要将这些值放入箱中。  
[bin()](./binfunction.md) 函数可用于此目的：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents
| where StartTime > datetime(2007-02-14) and StartTime < datetime(2007-02-21)
| summarize event_count = count() by bin(StartTime, 1d)
```

这会将所有时间戳缩短为 1 天的间隔：

|StartTime|event_count|
|---|---|
|2007-02-14 00:00:00.0000000|180|
|2007-02-15 00:00:00.0000000|66|
|2007-02-16 00:00:00.0000000|164|
|2007-02-17 00:00:00.0000000|103|
|2007-02-18 00:00:00.0000000|22|
|2007-02-19 00:00:00.0000000|52|
|2007-02-20 00:00:00.0000000|60|

[bin()](./binfunction.md) 在许多语言中与 [floor()](./floorfunction.md) 函数等效。 它简单地将每个值缩减为最接近提供的系数的倍数，这样 [summarize](./summarizeoperator.md) 便可以将行分配给组。

## <a name="render-display-a-chart-or-table"></a>Render：显示图表或表

投影两列，将其用作图表的 x 轴和 y 轴：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents 
| summarize event_count=count(), mid = avg(BeginLat) by State 
| sort by mid
| where event_count > 1800
| project State, event_count
| render columnchart
```

:::image type="content" source="images/tutorial/event-counts-state.png" alt-text="按州划分的风暴事件计数柱状图":::

虽然我们在项目操作中删除了 `mid`，但是如果我们希望图表中按顺序显示国家/地区，则仍然需要它。

严格地说，“render”（呈现）是客户端的一项功能，而不是查询语言的一部分。 尽管如此，它已经集成到语言中，在预想结果时会非常有用。


## <a name="timecharts"></a>时间图表

回到数字箱，显示一个时序：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents
| summarize event_count=count() by bin(StartTime, 1d)
| render timechart
```

:::image type="content" source="images/tutorial/time-series-start-bin.png" alt-text="按时间划分的折线图事件":::

## <a name="multiple-series"></a>多个序列

在 `summarize by` 子句中使用多个值可为值的每个组合创建单独的行：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents 
| where StartTime > datetime(2007-06-04) and StartTime < datetime(2007-06-10) 
| where Source in ("Source","Public","Emergency Manager","Trained Spotter","Law Enforcement")
| summarize count() by bin(StartTime, 10h), Source
```

:::image type="content" source="images/tutorial/table-count-source.png" alt-text="按源划分的表计数":::

只需将呈现术语添加到上面：`| render timechart`。

:::image type="content" source="images/tutorial/line-count-source.png" alt-text="按源划分的折线图计数":::

注意，`render timechart` 使用第一列作为 x 轴，然后将其他列显示为单独的行。

## <a name="daily-average-cycle"></a>每日平均周期

平均一天的活动如何变化？

按一天的时间取模对事件计数，将其量化到小时。 请注意，我们使用 `floor` 而不是 bin：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents
| extend hour = floor(StartTime % 1d , 1h)
| summarize event_count=count() by hour
| sort by hour asc
| render timechart
```

:::image type="content" source="images/tutorial/time-count-hour.png" alt-text="按小时划分的时间图计数":::

目前，`render` 无法正确标记持续时间，但我们可以使用 `| render columnchart` 代替：

:::image type="content" source="images/tutorial/column-count-hour.png" alt-text="按小时计算的柱形图计数":::

## <a name="compare-multiple-daily-series"></a>比较多个每日系列

不同州一天中不同时间的活动如何变化？

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents
| extend hour= floor( StartTime % 1d , 1h)
| where State in ("GULF OF MEXICO","MAINE","VIRGINIA","WISCONSIN","NORTH DAKOTA","NEW JERSEY","OREGON")
| summarize event_count=count() by hour, State
| render timechart
```

:::image type="content" source="images/tutorial/time-hour-state.png" alt-text="按小时和州划分的的时间图表":::

除以 `1h` 可将 x 轴转换为小时数而不是持续时间：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents
| extend hour= floor( StartTime % 1d , 1h)/ 1h
| where State in ("GULF OF MEXICO","MAINE","VIRGINIA","WISCONSIN","NORTH DAKOTA","NEW JERSEY","OREGON")
| summarize event_count=count() by hour, State
| render columnchart
```

:::image type="content" source="images/tutorial/column-hour-state.png" alt-text="按小时和州划分的柱形图":::

## <a name="join"></a>联接

如何基于两个给定的 EventType 查找哪个州同时发生了这两种类型的事件？

可以先拉取类型为第一个 EventType 的风暴事件，再拉取第二个 EventType 的风暴事件，然后在“州”上联接这两个集合。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents
| where EventType == "Lightning"
| join (
    StormEvents 
    | where EventType == "Avalanche"
) on State  
| distinct State
```

:::image type="content" source="images/tutorial/join-events-la.png" alt-text="联接闪电和雪崩事件":::

## <a name="user-session-example-of-join"></a>关于联接的用户会话示例

本部分不使用 `StormEvents` 表。

假设你的数据包含标记每个用户会话的开始和结束的事件，并且每个会话都具有唯一的 ID。 

每个用户会话的持续时间是多少？

通过使用 `extend` 为这两个时间戳提供别名，就可以计算会话持续时间。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
Events
| where eventName == "session_started"
| project start_time = timestamp, stop_time, country, session_id
| join ( Events
    | where eventName == "session_ended"
    | project stop_time = timestamp, session_id
    ) on session_id
| extend duration = stop_time - start_time
| project start_time, stop_time, country, duration
| take 10
```

:::image type="content" source="images/tutorial/user-session-extend.png" alt-text="用户会话扩展":::

执行联接前，最好使用 `project` 选中所需列。
在同一子句中，重命名时间戳列。

## <a name="plot-a-distribution"></a>绘制分布图

不同长度的风暴分别有多少个？

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents
| extend  duration = EndTime - StartTime
| where duration > 0s
| where duration < 3h
| summarize event_count = count()
    by bin(duration, 5m)
| sort by duration asc
| render timechart
```

:::image type="content" source="images/tutorial/event-count-duration.png" alt-text="按持续时间划分的事件计数时间表":::

或使用 `| render columnchart`：

:::image type="content" source="images/tutorial/column-event-count-duration.png" alt-text="按持续时间划分的柱形图事件计数时间表":::

## <a name="percentiles"></a>百分位数

哪些持续时间范围涵盖不同风暴百分比？

使用上述查询，但将 `render` 替换为：

```kusto
| summarize percentiles(duration, 5, 20, 50, 80, 95)
```

在本例中，我们没有提供 `by` 子句，因此结果为单行：

:::image type="content" source="images/tutorial/summarize-percentiles-duration.png" alt-text="按持续时间划分的表汇总百分位数":::

从中我们可以发现：

* 5% 的风暴持续时间小于 5 分钟；
* 50%的风暴持续时间小于 1 小时 25 分钟；
* 5% 的风暴持续时间至少为 2 小时 50 分钟。

若要获取各个州的单独细分情况，只需通过两个 summarize 运算符单独运算州列：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents
| extend  duration = EndTime - StartTime
| where duration > 0s
| where duration < 3h
| summarize event_count = count()
    by bin(duration, 5m), State
| sort by duration asc
| summarize percentiles(duration, 5, 20, 50, 80, 95) by State
```

:::image type="content" source="images/tutorial/summarize-percentiles-state.png" alt-text="按州划分的表汇总百分位数持续时间":::

## <a name="let-assign-a-result-to-a-variable"></a>Let：将结果分配给变量

在上面的“联接”示例中，使用 [let](./letstatement.md) 分隔查询表达式的各个部分。 结果不变：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
let LightningStorms = 
    StormEvents
    | where EventType == "Lightning";
let AvalancheStorms = 
    StormEvents
    | where EventType == "Avalanche";
LightningStorms 
| join (AvalancheStorms) on State
| distinct State
```

> [!TIP]
> 在 Kusto Explorer 客户端中，请勿在此查询的各部分间放入空白行。 请务必执行所有运算。

## <a name="combining-data-from-several-databases-in-a-query"></a>在查询中组合来自多个数据库的数据

有关详细讨论，请参阅[跨数据库查询](./cross-cluster-or-database-queries.md)

编写样式查询时：

```kusto
Logs | where ...
```

名为 Logs 的表必须位于默认数据库中。 如果要从其他数据库访问表，请使用以下语法：

```kusto
database("db").Table
```

因此，如果你有名为 Diagnostics 和 Telemetry 的数据库，并想要将它们的一些数据关联起来，则可以编写（假设 Diagnostics 是默认数据库）  

```kusto
Logs | join database("Telemetry").Metrics on Request MachineId | ...
```

或者如果你的默认数据库是 Telemetry

```kusto
union Requests, database("Diagnostics").Logs | ...
```
    
上述所有操作都假设两个数据库都位于当前连接的群集中。 假设 Telemetry 数据库属于另一个名为 TelemetryCluster.kusto.chinacloudapi.cn 的群集，为了访问它，你需要 

```kusto
Logs | join cluster("TelemetryCluster").database("Telemetry").Metrics on Request MachineId | ...
```

> [!NOTE]
> 指定群集后，数据库是必需的

::: zone-end

::: zone pivot="azuremonitor"

Azure Monitor 不支持此功能

::: zone-end
