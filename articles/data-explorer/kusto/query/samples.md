---
title: 示例 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的示例。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: d1387e39191d0dfa54257989ed721754adbd6d8b
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104578"
---
# <a name="samples"></a>示例

下面是一些常见的查询需求，以及如何使用 Kusto 查询语言来满足这些需求。

## <a name="display-a-column-chart"></a>显示柱形图

投影两列或更多列，将其用作图表的 x 轴和 y 轴。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples  -->
```kusto 
StormEvents
| where isnotempty(EndLocation) 
| summarize event_count=count() by EndLocation
| top 10 by event_count
| render columnchart
```

* 第一列形成 x 轴。 它可以是数字、日期/时间或字符串。 
* 使用 `where`、`summarize` 和 `top` 来限制显示的数据量。
* 对结果进行排序以定义 x 轴的顺序。

:::image type="content" source="images/samples/060.png" alt-text="柱形图的屏幕截图。y 轴的范围是从 0 到 50 左右。10 个彩色柱分别表示 10 个位置的值。":::

## <a name="get-sessions-from-start-and-stop-events"></a>从启动和停止事件获取会话

假设你有一个事件日志。 某些事件标志着某个扩展活动或会话的开始或结束。 

|名称|城市|SessionId|Timestamp|
|---|---|---|---|
|开始|London|2817330|2015-12-09T10:12:02.32|
|游戏|London|2817330|2015-12-09T10:12:52.45|
|开始|曼彻斯特|4267667|2015-12-09T10:14:02.23|
|停止|London|2817330|2015-12-09T10:23:43.18|
|取消|曼彻斯特|4267667|2015-12-09T10:27:26.29|
|停止|曼彻斯特|4267667|2015-12-09T10:28:31.72|

每个事件都有一个 SessionId。 问题是如何将具有相同 ID 的启动事件和停止事件进行匹配。

```kusto
let Events = MyLogTable | where ... ;

Events
| where Name == "Start"
| project Name, City, SessionId, StartTime=timestamp
| join (Events 
        | where Name="Stop"
        | project StopTime=timestamp, SessionId) 
    on SessionId
| project City, SessionId, StartTime, StopTime, Duration = StopTime - StartTime
```

1. 使用 [`let`](./letstatement.md) 为表的投影命名，在进行联接操作之前尽量削减该表的内容。
1. 使用 [`Project`](./projectoperator.md) 更改时间戳的名称，使开始和停止时间都可以出现在结果中。 
   它还会选择要在结果中显示的其他列。 
1. 使用 [`join`](./joinoperator.md) 将同一活动的开始和结束条目进行匹配，为每个活动创建一行。 
1. 最后，`project` 又添加一个列，用于显示活动的持续时间。


|城市|SessionId|StartTime|StopTime|持续时间|
|---|---|---|---|---|
|London|2817330|2015-12-09T10:12:02.32|2015-12-09T10:23:43.18|00:11:40.46|
|曼彻斯特|4267667|2015-12-09T10:14:02.23|2015-12-09T10:28:31.72|00:14:29.49|

### <a name="get-sessions-without-session-id"></a>在没有会话 ID 的情况下获取会话

假设不是那么方便，启动和停止事件没有可以用来进行匹配的会话 ID。 但是，我们有进行会话时客户端的 IP 地址。 假设每个客户端地址一次只进行一个会话，我们可以将每个启动事件与同一 IP 地址发出的下一个停止事件进行匹配。

```kusto
Events 
| where Name == "Start" 
| project City, ClientIp, StartTime = timestamp
| join  kind=inner
    (Events
    | where Name == "Stop" 
    | project StopTime = timestamp, ClientIp)
    on ClientIp
| extend duration = StopTime - StartTime 
    // Remove matches with earlier stops:
| where  duration > 0  
    // Pick out the earliest stop for each start and client:
| summarize arg_min(duration, *) by bin(StartTime,1s), ClientIp
```

联接会将每个启动时间与同一客户端 IP 地址的所有停止时间进行匹配。 
1. 删除停止时间较早的匹配项。
1. 按启动时间和 IP 分组，以获取每个会话的组。 
1. 提供适用于 StartTime 参数的 `bin` 函数。 如果你不这样做，则 Kusto 会自动使用 1 小时的箱，这样会将一些启动时间与错误的停止时间相匹配。

`arg_min` 会选取每个组中持续时间最短的行，而 `*` 参数则会传递给所有其他列。 此参数会为每个列名添加前缀“min_”。 

:::image type="content" source="images/samples/040.png" alt-text="一个列出结果的表，其中的列表示每个客户端启动时间组合的启动时间、客户端 IP、持续时间、城市和最早停止时间。"::: 

添加代码，用于对持续时间进行计数（使用以便捷方式设置了大小的箱作为单位）。在此示例中，由于我们首选条形图，因此将通过除以 `1s` 将时间跨度转换为数字。 

```
    // Count the frequency of each duration:
    | summarize count() by duration=bin(min_duration/1s, 10) 
      // Cut off the long tail:
    | where duration < 300
      // Display in a bar chart:
    | sort by duration asc | render barchart 
```

:::image type="content" source="images/samples/050.png" alt-text="此柱形图描绘了指定范围内具有持续时间的会话数。超过 400 个会话持续 10 秒。不到 100 个会话持续 290 秒。":::

### <a name="real-example"></a>实际示例

```kusto
Logs  
| filter ActivityId == "ActivityId with Blablabla" 
| summarize max(Timestamp), min(Timestamp)  
| extend Duration = max_Timestamp - min_Timestamp 
 
wabitrace  
| filter Timestamp >= datetime(2015-01-12 11:00:00Z)  
| filter Timestamp < datetime(2015-01-12 13:00:00Z)  
| filter EventText like "NotifyHadoopApplicationJobPerformanceCounters"      
| extend Tenant = extract("tenantName=([^,]+),", 1, EventText) 
| extend Environment = extract("environmentName=([^,]+),", 1, EventText)  
| extend UnitOfWorkId = extract("unitOfWorkId=([^,]+),", 1, EventText)  
| extend TotalLaunchedMaps = extract("totalLaunchedMaps=([^,]+),", 1, EventText, typeof(real))  
| extend MapsSeconds = extract("mapsMilliseconds=([^,]+),", 1, EventText, typeof(real)) / 1000 
| extend TotalMapsSeconds = MapsSeconds  / TotalLaunchedMaps 
| filter Tenant == 'DevDiv' and Environment == 'RollupDev2'  
| filter TotalLaunchedMaps > 0 
| summarize sum(TotalMapsSeconds) by UnitOfWorkId  
| extend JobMapsSeconds = sum_TotalMapsSeconds * 1 
| project UnitOfWorkId, JobMapsSeconds 
| join ( 
wabitrace  
| filter Timestamp >= datetime(2015-01-12 11:00:00Z)  
| filter Timestamp < datetime(2015-01-12 13:00:00Z)  
| filter EventText like "NotifyHadoopApplicationJobPerformanceCounters"  
| extend Tenant = extract("tenantName=([^,]+),", 1, EventText) 
| extend Environment = extract("environmentName=([^,]+),", 1, EventText)  
| extend UnitOfWorkId = extract("unitOfWorkId=([^,]+),", 1, EventText)   
| extend TotalLaunchedReducers = extract("totalLaunchedReducers=([^,]+),", 1, EventText, typeof(real)) 
| extend ReducesSeconds = extract("reducesMilliseconds=([^,]+)", 1, EventText, typeof(real)) / 1000 
| extend TotalReducesSeconds = ReducesSeconds / TotalLaunchedReducers 
| filter Tenant == 'DevDiv' and Environment == 'RollupDev2'  
| filter TotalLaunchedReducers > 0 
| summarize sum(TotalReducesSeconds) by UnitOfWorkId  
| extend JobReducesSeconds = sum_TotalReducesSeconds * 1 
| project UnitOfWorkId, JobReducesSeconds ) 
on UnitOfWorkId 
| join ( 
wabitrace  
| filter Timestamp >= datetime(2015-01-12 11:00:00Z)  
| filter Timestamp < datetime(2015-01-12 13:00:00Z)  
| filter EventText like "NotifyHadoopApplicationJobPerformanceCounters"  
| extend Tenant = extract("tenantName=([^,]+),", 1, EventText) 
| extend Environment = extract("environmentName=([^,]+),", 1, EventText)  
| extend JobName = extract("jobName=([^,]+),", 1, EventText)  
| extend StepName = extract("stepName=([^,]+),", 1, EventText)  
| extend UnitOfWorkId = extract("unitOfWorkId=([^,]+),", 1, EventText)  
| extend LaunchTime = extract("launchTime=([^,]+),", 1, EventText, typeof(datetime))  
| extend FinishTime = extract("finishTime=([^,]+),", 1, EventText, typeof(datetime)) 
| extend TotalLaunchedMaps = extract("totalLaunchedMaps=([^,]+),", 1, EventText, typeof(real))  
| extend TotalLaunchedReducers = extract("totalLaunchedReducers=([^,]+),", 1, EventText, typeof(real)) 
| extend MapsSeconds = extract("mapsMilliseconds=([^,]+),", 1, EventText, typeof(real)) / 1000 
| extend ReducesSeconds = extract("reducesMilliseconds=([^,]+)", 1, EventText, typeof(real)) / 1000 
| extend TotalMapsSeconds = MapsSeconds  / TotalLaunchedMaps  
| extend TotalReducesSeconds = (ReducesSeconds / TotalLaunchedReducers / ReducesSeconds) * ReducesSeconds  
| extend CalculatedDuration = (TotalMapsSeconds + TotalReducesSeconds) * time(1s) 
| filter Tenant == 'DevDiv' and Environment == 'RollupDev2') 
on UnitOfWorkId 
| extend MapsFactor = TotalMapsSeconds / JobMapsSeconds 
| extend ReducesFactor = TotalReducesSeconds / JobReducesSeconds 
| extend CurrentLoad = 1536 + (768 * TotalLaunchedMaps) + (1536 * TotalLaunchedMaps) 
| extend NormalizedLoad = 1536 + (768 * TotalLaunchedMaps * MapsFactor) + (1536 * TotalLaunchedMaps * ReducesFactor) 
| summarize sum(CurrentLoad), sum(NormalizedLoad) by  JobName  
| extend SaveFactor = sum_NormalizedLoad / sum_CurrentLoad 
```

## <a name="chart-concurrent-sessions-over-time"></a>为一段时间内的并发会话绘制图表

假设你有一个活动表，其中包含活动的开始和结束时间。 展示一个随时间变化的图表，其中会显示在任意时间并发运行的活动数。

下面是一个名为 `X` 的示例输入。

|SessionId | StartTime | StopTime |
|---|---|---|
| a | 10:01:03 | 10:10:08 |
| b | 10:01:29 | 10:03:10 |
| c | 10:03:02 | 10:05:20 |

对于箱为 1 分钟的图表，每隔 1 分钟会创建一些内容，因此针对每个正在运行的活动都有一个计数。

下面是一个中间结果。

```kusto
X | extend samples = range(bin(StartTime, 1m), StopTime, 1m)
```

`range` 按指定的间隔生成值的数组。

|SessionId | StartTime | StopTime  | 示例|
|---|---|---|---|
| a | 10:01:33 | 10:06:31 | [10:01:00,10:02:00,...10:06:00]|
| b | 10:02:29 | 10:03:45 | [10:02:00,10:03:00]|
| c | 10:03:12 | 10:04:30 | [10:03:00,10:04:00]|

请使用 [mv-expand](./mvexpandoperator.md) 展开这些数组，而不是保留它们。

```kusto
X | mv-expand samples = range(bin(StartTime, 1m), StopTime , 1m)
```

|SessionId | StartTime | StopTime  | 示例|
|---|---|---|---|
| a | 10:01:33 | 10:06:31 | 10:01:00|
| a | 10:01:33 | 10:06:31 | 10:02:00|
| a | 10:01:33 | 10:06:31 | 10:03:00|
| a | 10:01:33 | 10:06:31 | 10:04:00|
| a | 10:01:33 | 10:06:31 | 10:05:00|
| a | 10:01:33 | 10:06:31 | 10:06:00|
| b | 10:02:29 | 10:03:45 | 10:02:00|
| b | 10:02:29 | 10:03:45 | 10:03:00|
| c | 10:03:12 | 10:04:30 | 10:03:00|
| c | 10:03:12 | 10:04:30 | 10:04:00|

现在按采样时间对它们分组，并计算每个活动的发生次数。

```kusto
X
| mv-expand samples = range(bin(StartTime, 1m), StopTime , 1m)
| summarize count(SessionId) by bin(todatetime(samples),1m)
```

* 请使用 todatetime()，因为 [mv-expand](./mvexpandoperator.md) 会生成动态类型的列。
* 请使用 bin()，因为对于数值和日期，如果你未提供间隔，则 summarize 始终使用默认间隔来应用 bin 函数。 


| count_SessionId | 示例|
|---|---|
| 1 | 10:01:00|
| 2 | 10:02:00|
| 3 | 10:03:00|
| 2 | 10:04:00|
| 1 | 10:05:00|
| 1 | 10:06:00|

结果可以呈现为条形图或时间图。

## <a name="introduce-null-bins-into-summarize"></a>将空箱引入 summarize 中

将 `summarize` 运算符应用于包含 `datetime` 列的组键时，请将那些值“分箱”为固定宽度的箱。

```kusto
let StartTime=ago(12h);
let StopTime=now()
T
| where Timestamp > StartTime and Timestamp <= StopTime 
| where ...
| summarize Count=count() by bin(Timestamp, 5m)
```

上面的示例会生成一个表（即 `T`），其中的单个行/行组会划分到时长为五分钟的每个 bin 中。 它不会执行的操作是添加“空箱”-- 时间箱值在 `StartTime` 到 `StopTime` 之间且在 `T` 中没有对应行的行。 

需要将这些箱“填充”到表中。下面是执行此操作的一种方法。

```kusto
let StartTime=ago(12h);
let StopTime=now()
T
| where Timestamp > StartTime and Timestamp <= StopTime 
| summarize Count=count() by bin(Timestamp, 5m)
| where ...
| union ( // 1
  range x from 1 to 1 step 1 // 2
  | mv-expand Timestamp=range(StartTime, StopTime, 5m) to typeof(datetime) // 3
  | extend Count=0 // 4
  )
| summarize Count=sum(Count) by bin(Timestamp, 5m) // 5 
```

下面是上述查询的分步说明： 

1. `union` 运算符用于向表中添加更多行。 这些行由 `union` 表达式生成。
1. `range` 运算符生成一个包含单个行/列的表。
   该表仅供 `mv-expand` 使用。
1. 基于 `range` 函数的 `mv-expand` 运算符会根据 `StartTime` 到 `EndTime` 之间时长为 5 分钟的箱的数目创建相同数目的行。
1. 使用一个值为 `0` 的 `Count`。
1. `summarize` 运算符将 `union` 的原始（左侧或外部）参数中的箱分组到一起。 此运算符还会根据它的内部参数进行分箱（空箱行）。 此过程可确保每个箱在输出中都有一行，其值为零或原始计数。  

## <a name="get-more-out-of-your-data-in-kusto-with-machine-learning"></a>在 Kusto 中利用机器学习对数据进行更深入的分析 

可以通过很多有趣的用例来了解如何利用机器学习算法从遥测数据中获得有趣的见解。 这些算法通常要求使用结构化程度很高的数据集作为其输入。 原始日志数据通常与所要求的结构和大小不符。 

首先需查找特定必应推理服务的错误率中的异常。 日志表有 650 亿条记录。 下面的简单查询筛选出 25 万条错误，创建一个包含错误计数的时序数据，该数据使用异常检测函数 [series_decompose_anomalies](series-decompose-anomaliesfunction.md)。 这些异常是由 Kusto 服务检测到的，在时序图表上突出显示为红点。

```kusto
Logs
| where Timestamp >= datetime(2015-08-22) and Timestamp < datetime(2015-08-23) 
| where Level == "e" and Service == "Inferences.UnusualEvents_Main" 
| summarize count() by bin(Timestamp, 5min)
| render anomalychart 
```

该服务识别了几个具有可疑错误率的时间段。 请使用 Kusto 放大此时间范围，并运行一个基于“Message”列进行聚合的查询。 尝试找出最常见的错误。 

消息的整个堆栈跟踪的相关部分被截断，这样做是为了与页面大小更加吻合。 

你可以看到，最常见的八个错误已被成功识别。 但是，后面会出现一长串错误，因为错误消息是通过一个包含不断变化的数据的格式字符串创建的。 

```kusto
Logs
| where Timestamp >= datetime(2015-08-22 05:00) and Timestamp < datetime(2015-08-22 06:00)
| where Level == "e" and Service == "Inferences.UnusualEvents_Main"
| summarize count() by Message 
| top 10 by count_ 
| project count_, Message 
```

|count_|Message
|---|---
|7125|'RunCycleFromInterimData' 方法的 ExecuteAlgorithmMethod 已失败...
|7125|InferenceHostService 调用失败...System.NullReferenceException:对象引用未设置为某个对象的实例...
|7124|推理系统意外错误...System.NullReferenceException:对象引用未设置为某个对象的实例... 
|5112|推理系统意外错误...System.NullReferenceException:对象引用未设置为某个对象的实例...
|174|InferenceHostService 调用失败...System.ServiceModel.CommunicationException:写入到管道时出错:...
|10|'RunCycleFromInterimData' 方法的 ExecuteAlgorithmMethod 已失败...
|10|推理系统错误...Microsoft.Bing.Platform.Inferences.Service.Managers.UserInterimDataManagerException:...
|3|InferenceHostService 调用失败...System.ServiceModel.CommunicationObjectFaultedException:...
|1|推理系统错误...SocialGraph.BOSS.OperationResponse...AIS TraceId:8292FC561AC64BED8FA243808FE74EFD...
|1|推理系统错误...SocialGraph.BOSS.OperationResponse...AIS TraceId:5F79F7587FF943EC9B641E02E701AFBF...

这种情况下可以使用 `reduce` 运算符。 此运算符识别出 63 个不同错误是由代码中的同一跟踪检测点引起的，这有助于用户集中精力来处理该时间范围中更多有意义的错误跟踪。

```kusto
Logs
| where Timestamp >= datetime(2015-08-22 05:00) and Timestamp < datetime(2015-08-22 06:00)
| where Level == "e" and Service == "Inferences.UnusualEvents_Main"
| reduce by Message with threshold=0.35
| project Count, Pattern
```

|计数|模式
|---|---
|7125|'RunCycleFromInterimData' 方法的 ExecuteAlgorithmMethod 已失败...
|  7125|InferenceHostService 调用失败...System.NullReferenceException:对象引用未设置为某个对象的实例...
|  7124|推理系统意外错误...System.NullReferenceException:对象引用未设置为某个对象的实例...
|  5112|推理系统意外错误...System.NullReferenceException:对象引用未设置为某个对象的实例...
|  174|InferenceHostService 调用失败...System.ServiceModel.CommunicationException:写入到管道时出错:...
|  63|推理系统错误...Microsoft.Bing.Platform.Inferences.\*:写入 \* 是为了将数据写入对象 BOSS.\*:SocialGraph.BOSS.Reques...
|  10|'RunCycleFromInterimData' 方法的 ExecuteAlgorithmMethod 已失败...
|  10|推理系统错误...Microsoft.Bing.Platform.Inferences.Service.Managers.UserInterimDataManagerException:...
|  3|InferenceHostService 调用失败...System.ServiceModel.\*:对象 System.ServiceModel.Channels.\*+\*（针对 \*\*）为 \*...，位于 Syst...

现在，你已经很好地了解了导致所检测到的异常的最常见错误。

若要了解这些错误在整个示例系统中的影响，请注意以下事项： 
* “Logs”表包含其他维度数据，例如“Component”、“Cluster”，等等。
* 新的“autocluster”插件可以使用简单的查询来帮助获取该见解。 
* 在下面的示例中，可以清楚地看到，最常见的四个错误中的每一个都特定于一个组件。 此外，尽管最常见的三个错误特定于 DB4 群集，但第四个错误在所有群集中都发生。

```kusto
Logs
| where Timestamp >= datetime(2015-08-22 05:00) and Timestamp < datetime(2015-08-22 06:00)
| where Level == "e" and Service == "Inferences.UnusualEvents_Main"
| evaluate autocluster()
```

|计数 |百分号 (%)|组件|群集|Message
|---|---|---|---|---
|7125|26.64|InferenceHostService|DB4|...方法的 ExecuteAlgorithmMethod...
|7125|26.64|未知组件|DB4|InferenceHostService 调用失败...
|7124|26.64|InferenceAlgorithmExecutor|DB4|推理系统意外错误...
|5112|19.11|InferenceAlgorithmExecutor|*|推理系统意外错误... 

## <a name="map-values-from-one-set-to-another"></a>将值从一个集映射到另一个集

常见的用例是对值进行静态映射，这样可以使结果更易于演示。
有关示例，请查看下表。 
`DeviceModel` 指定设备的型号，这不是一种很方便的引用设备名称的形式。  


|DeviceModel |计数 
|---|---
|iPhone5,1 |32 
|iPhone3,2 |432 
|iPhone7,2 |55 
|iPhone5,2 |66 

  
下面是一个更好的表示形式。  

|FriendlyName |计数 
|---|---
|iPhone 5 |32 
|iPhone 4 |432 
|iPhone 6 |55 
|iPhone5 |66 

下面的两种方法演示了如何实现此表示形式。  

### <a name="mapping-using-dynamic-dictionary"></a>使用动态字典进行映射

此方法展示了使用动态字典和动态访问器进行映射。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
// Data set definition
let Source = datatable(DeviceModel:string, Count:long)
[
  'iPhone5,1', 32,
  'iPhone3,2', 432,
  'iPhone7,2', 55,
  'iPhone5,2', 66,
];
// Query start here
let phone_mapping = dynamic(
  {
    "iPhone5,1" : "iPhone 5",
    "iPhone3,2" : "iPhone 4",
    "iPhone7,2" : "iPhone 6",
    "iPhone5,2" : "iPhone5"
  });
Source 
| project FriendlyName = phone_mapping[DeviceModel], Count
```

|FriendlyName|计数|
|---|---|
|iPhone 5|32|
|iPhone 4|432|
|iPhone 6|55|
|iPhone5|66|

### <a name="map-using-static-table"></a>使用静态表进行映射

此方法展示了使用持久的表和联接运算符进行映射。
 
创建映射表（只需创建一次）。

```kusto
.create table Devices (DeviceModel: string, FriendlyName: string) 

.ingest inline into table Devices 
    ["iPhone5,1","iPhone 5"]["iPhone3,2","iPhone 4"]["iPhone7,2","iPhone 6"]["iPhone5,2","iPhone5"]
```

现在，Devices 表的内容如下。

|DeviceModel |FriendlyName 
|---|---
|iPhone5,1 |iPhone 5 
|iPhone3,2 |iPhone 4 
|iPhone7,2 |iPhone 6 
|iPhone5,2 |iPhone5 

使用同样的方法创建一个测试表源。

```kusto
.create table Source (DeviceModel: string, Count: int)

.ingest inline into table Source ["iPhone5,1",32]["iPhone3,2",432]["iPhone7,2",55]["iPhone5,2",66]
```

Join 和 project。

```kusto
Devices  
| join (Source) on DeviceModel  
| project FriendlyName, Count
```

结果:

|FriendlyName |计数 
|---|---
|iPhone 5 |32 
|iPhone 4 |432 
|iPhone 6 |55 
|iPhone5 |66 


## <a name="create-and-use-query-time-dimension-tables"></a>创建和使用查询时间维度表

你经常需要将查询的结果与未存储在数据库中的某些临时维度表联接起来。 可以定义一个表达式，使其结果是限定于单个查询的一个表。 例如：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
// Create a query-time dimension table using datatable
let DimTable = datatable(EventType:string, Code:string)
  [
    "Heavy Rain", "HR",
    "Tornado",    "T"
  ]
;
DimTable
| join StormEvents on EventType
| summarize count() by Code
```

下面是一个稍微复杂一些的示例。

```kusto
// Create a query-time dimension table using datatable
let TeamFoundationJobResult = datatable(Result:int, ResultString:string)
  [
    -1, 'None', 0, 'Succeeded', 1, 'PartiallySucceeded', 2, 'Failed',
    3, 'Stopped', 4, 'Killed', 5, 'Blocked', 6, 'ExtensionNotFound',
    7, 'Inactive', 8, 'Disabled', 9, 'JobInitializationError'
  ]
;
JobHistory
  | where PreciseTimeStamp > ago(1h)
  | where Service  != "AX"
  | where Plugin has "Analytics"
  | sort by PreciseTimeStamp desc
  | join kind=leftouter TeamFoundationJobResult on Result
  | extend ExecutionTimeSpan = totimespan(ExecutionTime)
  | project JobName, StartTime, ExecutionTimeSpan, ResultString, ResultMessage
```

## <a name="retrieve-the-latest-records-by-timestamp-per-identity"></a>检索每个标识的最新记录（按时间戳）

假设你有一个表，其中包含：
* 一个 `ID` 列（例如用户 ID 或节点 ID），标识与每行关联的实体
* 一个 `timestamp` 列，为行提供时间参考
* 其他列

如果要求查询针对 `ID` 列的每个值返回最新的两个记录（在这里，“最新”是指“其 `timestamp` 值最高”），则可使用 [top-nested 运算符](topnestedoperator.md)创建该查询。

例如：

```kusto
datatable(id:string, timestamp:datetime, bla:string)           // #1
  [
  "Barak",  datetime(2015-01-01), "1",
  "Barak",  datetime(2016-01-01), "2",
  "Barak",  datetime(2017-01-20), "3",
  "Donald", datetime(2017-01-20), "4",
  "Donald", datetime(2017-01-18), "5",
  "Donald", datetime(2017-01-19), "6"
  ]
| top-nested   of id        by dummy0=max(1),                  // #2
  top-nested 2 of timestamp by dummy1=max(timestamp),          // #3
  top-nested   of bla       by dummy2=max(1)                   // #4
| project-away dummy0, dummy1, dummy2                          // #5
```

注意：下面的编号指的是代码示例中的编号（在最右侧）。

1. `datatable` 是生成用于演示的一些测试数据的方法。 通常，在这里将使用实际数据。
1. 此行实质上表示“返回 `id` 的所有非重复值”。
1. 然后，对于通过 max 函数选取的前两个记录，此行会返回以下项：
     * `timestamp` 列
     * 上一级别的列（此处只有一个列：`id`）
     * 在此级别指定的列（此处为 `timestamp`）
1. 此行将为上一级别返回的每个记录添加 `bla` 列的值。 如果表中存在你感兴趣的其他列，则可针对每个这样的列重复执行此行。
1. 最后的行使用 [project-away 运算符](projectawayoperator.md)删除通过 `top-nested` 引入的“额外”列。

## <a name="extend-a-table-with-some-percent-of-total-calculation"></a>通过某项“占总数的百分比”计算扩展一个表

包含数值列的表格表达式如果采用“占总数的百分比”作为值，则对用户更有用。 例如，假设有一个查询生成了下表：

|SomeSeries|SomeInt|
|----------|-------|
|Foo       |    100|
|条形图​​       |    200|

如果要将该表显示为：

|SomeSeries|SomeInt|Pct |
|----------|-------|----|
|Foo       |    100|33.3|
|条形图​​       |    200|66.6|

则需要计算 `SomeInt` 列的总计（总和），然后将此列的每个值除以总计。 对任意结果使用 [as 运算符](asoperator.md)。

```kusto
// The following table literal represents a long calculation
// that ends up with an anonymous tabular value:
datatable (SomeInt:int, SomeSeries:string) [
  100, "Foo",
  200, "Bar",
]
// We now give this calculation a name ("X"):
| as X
// Having this name we can refer to it in the sub-expression
// "X | summarize sum(SomeInt)":
| extend Pct = 100 * bin(todouble(SomeInt) / toscalar(X | summarize sum(SomeInt)), 0.001)
```

## <a name="perform-aggregations-over-a-sliding-window"></a>对滑动窗口执行聚合

以下示例展示了如何使用滑动窗口来汇总列。
请使用下表，其中包含按时间戳列出的水果价格。 使用为期七天的滑动窗口计算每天每种水果的最小成本、最大成本和总成本。 结果集中的每条记录都是对前七天的聚合，分析期间的每一天在结果中都有一条对应的记录。  

fruits 表：

|Timestamp|水果|价格|
|---|---|---|
|2018-09-24 21:00:00.0000000|Bananas|3|
|2018-09-25 20:00:00.0000000|苹果|9|
|2018-09-26 03:00:00.0000000|Bananas|4|
|2018-09-27 10:00:00.0000000|Plums|8|
|2018-09-28 07:00:00.0000000|Bananas|6|
|2018-09-29 21:00:00.0000000|Bananas|8|
|2018-09-30 01:00:00.0000000|Plums|2|
|2018-10-01 05:00:00.0000000|Bananas|0|
|2018-10-02 02:00:00.0000000|Bananas|0|
|2018-10-03 13:00:00.0000000|Plums|4|
|2018-10-04 14:00:00.0000000|苹果|8|
|2018-10-05 05:00:00.0000000|Bananas|2|
|2018-10-06 08:00:00.0000000|Plums|8|
|2018-10-07 12:00:00.0000000|Bananas|0|

滑动窗口聚合查询。
查询结果后面是说明：

```kusto
let _start = datetime(2018-09-24);
let _end = _start + 13d; 
Fruits 
| extend _bin = bin_at(Timestamp, 1d, _start) // #1 
| extend _endRange = iif(_bin + 7d > _end, _end, 
                            iff( _bin + 7d - 1d < _start, _start,
                                iff( _bin + 7d - 1d < _bin, _bin,  _bin + 7d - 1d)))  // #2
| extend _range = range(_bin, _endRange, 1d) // #3
| mv-expand _range to typeof(datetime) limit 1000000 // #4
| summarize min(Price), max(Price), sum(Price) by Timestamp=bin_at(_range, 1d, _start) ,  Fruit // #5
| where Timestamp >= _start + 7d; // #6

```

|Timestamp|水果|min_Price|max_Price|sum_Price|
|---|---|---|---|---|
|2018-10-01 00:00:00.0000000|苹果|9|9|9|
|2018-10-01 00:00:00.0000000|Bananas|0|8|18|
|2018-10-01 00:00:00.0000000|Plums|2|8|10|
|2018-10-02 00:00:00.0000000|Bananas|0|8|18|
|2018-10-02 00:00:00.0000000|Plums|2|8|10|
|2018-10-03 00:00:00.0000000|Plums|2|8|14|
|2018-10-03 00:00:00.0000000|Bananas|0|8|14|
|2018-10-04 00:00:00.0000000|Bananas|0|8|14|
|2018-10-04 00:00:00.0000000|Plums|2|4|6|
|2018-10-04 00:00:00.0000000|苹果|8|8|8|
|2018-10-05 00:00:00.0000000|Bananas|0|8|10|
|2018-10-05 00:00:00.0000000|Plums|2|4|6|
|2018-10-05 00:00:00.0000000|苹果|8|8|8|
|2018-10-06 00:00:00.0000000|Plums|2|8|14|
|2018-10-06 00:00:00.0000000|Bananas|0|2|2|
|2018-10-06 00:00:00.0000000|苹果|8|8|8|
|2018-10-07 00:00:00.0000000|Bananas|0|2|2|
|2018-10-07 00:00:00.0000000|Plums|4|8|12|
|2018-10-07 00:00:00.0000000|苹果|8|8|8|

查询详细信息：

查询会在输入表中的每条记录实际出现后的七天内对其进行“延伸”（复制）。 每条记录实际出现 7 次。
因此，每日聚合包含之前七天的所有记录。

下面的分步说明编号指的是代码示例中的编号（在最右侧）：
1. 将每条记录分箱到某一天（相对于 _start）。 
2. 确定每条记录的范围的结束时间 - _bin + 7d，除非这超出了 (start, end) 范围，如果超出，会对其进行调整。 
3. 对于每条记录，创建一个 7 天（时间戳）的数组，从当前记录的日期开始。 
4. 对数组执行 mv-expand 操作，以便将每条记录复制成 7 条记录，彼此间隔 1 天。 
5. 针对每天执行聚合函数。 由于 #4，这实际上汇总了过去七天的数据。 
6. 第一个七天的数据不完整。 第一个七天没有 7 天的回溯期。 第一个七天会从最终结果中排除。 在示例中，它们仅参与 2018-10-01 的聚合。

## <a name="find-preceding-event"></a>查找之前的事件
下一示例演示了如何在 2 个数据集中查找之前的事件。  

目的：有两个数据集：A 和 B。对于 B 中的每条记录，请在 A 中查找其之前的事件（即，A 中比 B 要“早”的 arg_max 记录）。 下面是使用以下示例数据集时的预期输出。 

```kusto
let A = datatable(Timestamp:datetime, ID:string, EventA:string)
[
    datetime(2019-01-01 00:00:00), "x", "Ax1",
    datetime(2019-01-01 00:00:01), "x", "Ax2",
    datetime(2019-01-01 00:00:02), "y", "Ay1",
    datetime(2019-01-01 00:00:05), "y", "Ay2",
    datetime(2019-01-01 00:00:00), "z", "Az1"
];
let B = datatable(Timestamp:datetime, ID:string, EventB:string)
[
    datetime(2019-01-01 00:00:03), "x", "B",
    datetime(2019-01-01 00:00:04), "x", "B",
    datetime(2019-01-01 00:00:04), "y", "B",
    datetime(2019-01-01 00:02:00), "z", "B"
];
A; B
```

|Timestamp|ID|EventB|
|---|---|---|
|2019-01-01 00:00:00.0000000|x|Ax1|
|2019-01-01 00:00:00.0000000|z|Az1|
|2019-01-01 00:00:01.0000000|x|Ax2|
|2019-01-01 00:00:02.0000000|y|Ay1|
|2019-01-01 00:00:05.0000000|y|Ay2|

</br>

|Timestamp|ID|EventA|
|---|---|---|
|2019-01-01 00:00:03.0000000|x|B|
|2019-01-01 00:00:04.0000000|x|B|
|2019-01-01 00:00:04.0000000|y|B|
|2019-01-01 00:02:00.0000000|z|B|

预期输出： 

|ID|Timestamp|EventB|A_Timestamp|EventA|
|---|---|---|---|---|
|x|2019-01-01 00:00:03.0000000|B|2019-01-01 00:00:01.0000000|Ax2|
|x|2019-01-01 00:00:04.0000000|B|2019-01-01 00:00:01.0000000|Ax2|
|y|2019-01-01 00:00:04.0000000|B|2019-01-01 00:00:02.0000000|Ay1|
|z|2019-01-01 00:02:00.0000000|B|2019-01-01 00:00:00.0000000|Az1|

对于此问题，可以采用两种不同的建议方法。 你应该基于你的特定数据集来测试这两种方法，找到最适合你的方法。

> [!NOTE] 
> 每种方法针对不同数据集的运行方式可能不同。

### <a name="suggestion-1"></a>建议 1

此建议按 ID 和时间戳对两个数据集进行序列化，然后将所有 B 事件与其之前的所有 A 事件分组到一起，然后从组中的所有 A 中选取 `arg_max`。

```kusto
A
| extend A_Timestamp = Timestamp, Kind="A"
| union (B | extend B_Timestamp = Timestamp, Kind="B")
| order by ID, Timestamp asc 
| extend t = iff(Kind == "A" and (prev(Kind) != "A" or prev(Id) != ID), 1, 0)
| extend t = row_cumsum(t)
| summarize Timestamp=make_list(Timestamp), EventB=make_list(EventB), arg_max(A_Timestamp, EventA) by t, ID
| mv-expand Timestamp to typeof(datetime), EventB to typeof(string)
| where isnotempty(EventB)
| project-away t
```

### <a name="suggestion-2"></a>建议 2

此建议需要一个最长回溯期（A 中的记录可能比 B 中的记录“早”多长时间）。满足该要求后，此方法会根据 ID 和此回溯期联接这两个数据集。 联接会生成所有可能的候选项（在回溯期内早于 B 的所有 A 记录），然后通过 arg_min(TimestampB – TimestampA) 筛选在时间上最靠近 B 的项。 回溯期越短，查询结果就会越好。

在下面的示例中，回溯期设置为 1m，ID 为“z”的记录没有相应的“A”事件，因为它的“A”早 2m。

```kusto 
let _maxLookbackPeriod = 1m;  
let _internalWindowBin = _maxLookbackPeriod / 2;
let B_events = B 
    | extend ID = new_guid()
    | extend _time = bin(Timestamp, _internalWindowBin)
    | extend _range = range(_time - _internalWindowBin, _time + _maxLookbackPeriod, _internalWindowBin) 
    | mv-expand _range to typeof(datetime) 
    | extend B_Timestamp = Timestamp, _range;
let A_events = A 
    | extend _time = bin(Timestamp, _internalWindowBin)
    | extend _range = range(_time - _internalWindowBin, _time + _maxLookbackPeriod, _internalWindowBin) 
    | mv-expand _range to typeof(datetime) 
    | extend A_Timestamp = Timestamp, _range;
B_events
    | join kind=leftouter (
        A_events
) on ID, _range
| where isnull(A_Timestamp) or (A_Timestamp <= B_Timestamp and B_Timestamp <= A_Timestamp + _maxLookbackPeriod)
| extend diff = coalesce(B_Timestamp - A_Timestamp, _maxLookbackPeriod*2)
| summarize arg_min(diff, *) by ID
| project ID, B_Timestamp, A_Timestamp, EventB, EventA
```

|ID|B_Timestamp|A_Timestamp|EventB|EventA|
|---|---|---|---|---|
|x|2019-01-01 00:00:03.0000000|2019-01-01 00:00:01.0000000|B|Ax2|
|x|2019-01-01 00:00:04.0000000|2019-01-01 00:00:01.0000000|B|Ax2|
|y|2019-01-01 00:00:04.0000000|2019-01-01 00:00:02.0000000|B|Ay1|
|z|2019-01-01 00:02:00.0000000||B||
