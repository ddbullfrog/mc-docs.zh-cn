---
title: new_activity_metrics 插件 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 new_activity_metrics 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/30/2020
ms.date: 10/29/2020
ms.openlocfilehash: 58960536d6a3cc8eec6c9c5a66230a190adc439d
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103858"
---
# <a name="new_activity_metrics-plugin"></a>new_activity_metrics 插件

为 `New Users` 的队列计算有用的活动指标（非重复计数值、非重复的新值计数、保留率和变动率）。 将 `New Users` 的每个队列（时间窗口内首先看到的所有用户）与以前的所有队列进行比较。 比较会考虑所有以前的时间窗口。 例如，在 from=T2 和 to=T3 的记录中，非重复用户计数是出现在 T3 中同时未出现在 T1 和 T2 中的所有用户。 
```kusto
T | evaluate new_activity_metrics(id, datetime_column, startofday(ago(30d)), startofday(now()), 1d, dim1, dim2, dim3)
```

## <a name="syntax"></a>语法

*T* `| evaluate` `new_activity_metrics(`*IdColumn*`,` *TimelineColumn*`,` *Start*`,` *End*`,` *Window* [`,` *Cohort* ] [`,` *dim1*`,` *dim2*`,` ...] [`,` *Lookback* ] `)`

## <a name="arguments"></a>参数

* *T* ：输入表格表达式。
* *IdColumn* ：列的名称，其 ID 值表示用户活动。 
* TimelineColumn：表示时间线的列的名称。
* *开始* ：带有分析开始时段值的标量。
* End：带有分析结束时段值的标量。
* Window：带有分析窗口时段值的标量。 既可以是数字/日期/时间/时间戳值，也可以是 `week`/`month`/`year` 中的一个字符串（在这种情况下，这些时段分别为 [startofweek](startofweekfunction.md)/[startofmonth](startofmonthfunction.md)/[startofyear](startofyearfunction.md)）。 
* *Cohort* ：（可选）指示特定队列的标量常数。 如未提供，则计算并返回与分析时间窗口相对应的所有队列。
* dim1, dim2, ... ：（可选）维度列的列表，用于切分活动指标计算。
* Lookback：（可选）一个表格表达式，其中包含一组属于回溯时段的 ID

## <a name="returns"></a>返回

返回一个表，该表包含每个“from”和“to”时间线时段组合和每个现有维度组合的非重复计数值、非重复的新值计数、保留率和变动率。

输出表架构如下：

|from_TimelineColumn|to_TimelineColumn|dcount_new_values|dcount_retained_values|dcount_churn_values|retention_rate|churn_rate|dim1|..|dim_n|
|---|---|---|---|---|---|---|---|---|---|
|类型：自 TimelineColumn 起|相同|long|long|Double|Double|Double|..|..|..|

* `from_TimelineColumn` - 新用户的队列。 此记录中的指标是指此时间段内首次见到的所有用户。 决定是否为首次见到要考虑分析时段中所有之前的时段。 
* `to_TimelineColumn` - 进行比较的时段。 
* `dcount_new_values` - 出现在 `to_TimelineColumn` 中同时未出现在 `from_TimelineColumn`（含）之前的所有时段中的非重复用户数。 
* `dcount_retained_values` - 在所有的新用户（即 `from_TimelineColumn` 中首次出现的用户）中，出现在 `to_TimelineCoumn` 中的非重复用户数。
* `dcount_churn_values` - 在所有的新用户（即 `from_TimelineColumn` 中首次出现的用户）中，未出现在 `to_TimelineCoumn` 中的非重复用户数。
* `retention_rate` - 队列（首次出现在 `from_TimelineColumn` 中的用户）中的 `dcount_retained_values` 百分比。
* `churn_rate` - 队列（首次出现在 `from_TimelineColumn` 中的用户）中的 `dcount_churn_values` 百分比。

**备注**

有关 `Retention Rate` 和 `Churn Rate` 的定义，请参阅 [activity_metrics 插件](./activity-metrics-plugin.md)文档中的“注意”部分。


**示例**

以下示例数据集显示了哪些用户将出现在哪些天。 该表基于源 `Users` 表而生成，如下所示： 

```kusto
Users | summarize tostring(make_set(user)) by bin(Timestamp, 1d) | order by Timestamp asc;
```

|Timestamp|set_user|
|---|---|
|2019-11-01 00:00:00.0000000|[0,2,3,4]|
|2019-11-02 00:00:00.0000000|[0,1,3,4,5]|
|2019-11-03 00:00:00.0000000|[0,2,4,5]|
|2019-11-04 00:00:00.0000000|[0,1,2,3]|
|2019-11-05 00:00:00.0000000|[0,1,2,3,4]|

原始表插件的输出如下所示： 

```kusto
let StartDate = datetime(2019-11-01 00:00:00);
let EndDate = datetime(2019-11-07 00:00:00);
Users 
| evaluate new_activity_metrics(user, Timestamp, StartDate, EndDate-1tick, 1d) 
| where from_Timestamp < datetime(2019-11-03 00:00:00.0000000)
```

|R|from_Timestamp|to_Timestamp|dcount_new_values|dcount_retained_values|dcount_churn_values|retention_rate|churn_rate|
|---|---|---|---|---|---|---|---|
|1|2019-11-01 00:00:00.0000000|2019-11-01 00:00:00.0000000|4|4|0|1|0|
|2|2019-11-01 00:00:00.0000000|2019-11-02 00:00:00.0000000|2|3|1|0.75|0.25|
|3|2019-11-01 00:00:00.0000000|2019-11-03 00:00:00.0000000|1|3|1|0.75|0.25|
|4|2019-11-01 00:00:00.0000000|2019-11-04 00:00:00.0000000|1|3|1|0.75|0.25|
|5|2019-11-01 00:00:00.0000000|2019-11-05 00:00:00.0000000|1|4|0|1|0|
|6|2019-11-01 00:00:00.0000000|2019-11-06 00:00:00.0000000|0|0|4|0|1|
|7|2019-11-02 00:00:00.0000000|2019-11-02 00:00:00.0000000|2|2|0|1|0|
|8|2019-11-02 00:00:00.0000000|2019-11-03 00:00:00.0000000|0|1|1|0.5|0.5|
|9|2019-11-02 00:00:00.0000000|2019-11-04 00:00:00.0000000|0|1|1|0.5|0.5|
|10|2019-11-02 00:00:00.0000000|2019-11-05 00:00:00.0000000|0|1|1|0.5|0.5|
|11|2019-11-02 00:00:00.0000000|2019-11-06 00:00:00.0000000|0|0|2|0|1|

下面分析了输出中的数条记录： 
* 记录 `R=3`，`from_TimelineColumn` = `2019-11-01`，`to_TimelineColumn` = `2019-11-03`：
    * 此记录的用户即为 11/1 出现的所有新用户。 由于这是第一个时段，因此这些是该箱中的所有用户 - [0,2,3,4]
    * `dcount_new_values` - 出现在 11/3 同时未出现 11/1 的用户的数目。 这里有一个用户 - `5`。 
    * `dcount_retained_values` - 11/1 出现的所有新用户中，有多少保留到了 11/3？ 有三个 (`[0,2,4]`)，而 `count_churn_values` 为 1（用户 `3`）。 
    * `retention_rate` = 0.75 - 四个新用户（出现在 11/1 的用户）中保留了三个。 

* 记录 `R=9`，`from_TimelineColumn` = `2019-11-02`，`to_TimelineColumn` = `2019-11-04`：
    * 此记录关注首次出现在 11/2 的新用户 - 用户`1` 和 `5`。 
    * `dcount_new_values` - 出现在 11/4 同时未出现在 `T0 .. from_Timestamp` 间所有时段的用户的数目。 这意味着，出现在 11/4 但未出现在 11/1 或 11/2 的用户，没有这样的用户。 
    * `dcount_retained_values` - 11/2 出现的所有新用户 (`[1,5]`) 中，有多少保留到了 11/4？ 有一个这样的用户 (`[1]`)，count_churn_values 为一（用户 `5`）。 
    * `retention_rate` 为 0.5 - 11/2 出现的两个新用户中 11/4 保留了一个。 


### <a name="weekly-retention-rate-and-churn-rate-single-week"></a>每周保留率和变动率（一周）

下个查询计算 `New Users` 队列（首周出现的用户）的每周保留率和变动率。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
// Generate random data of user activities
let _start = datetime(2017-05-01);
let _end = datetime(2017-05-31);
range Day from _start to _end  step 1d
| extend d = tolong((Day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+200*r-1), 1) 
| mv-expand id=_users to typeof(long) limit 1000000
// Take only the first week cohort (last parameter)
| evaluate new_activity_metrics(['id'], Day, _start, _end, 7d, _start)
| project from_Day, to_Day, retention_rate, churn_rate
```

|from_Day|to_Day|retention_rate|churn_rate|
|---|---|---|---|
|2017-05-01 00:00:00.0000000|2017-05-01 00:00:00.0000000|1|0|
|2017-05-01 00:00:00.0000000|2017-05-08 00:00:00.0000000|0.544632768361582|0.455367231638418|
|2017-05-01 00:00:00.0000000|2017-05-15 00:00:00.0000000|0.031638418079096|0.968361581920904|
|2017-05-01 00:00:00.0000000|2017-05-22 00:00:00.0000000|0|1|
|2017-05-01 00:00:00.0000000|2017-05-29 00:00:00.0000000|0|1|


### <a name="weekly-retention-rate-and-churn-rate-complete-matrix"></a>每周保留率和变动率（完整矩阵）

下个查询计算 `New Users` 队列的每周保留率和变动率。 如果前面的示例计算一周的统计信息，则下面将生成每个 from/to 组合的 NxN 表。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
// Generate random data of user activities
let _start = datetime(2017-05-01);
let _end = datetime(2017-05-31);
range Day from _start to _end  step 1d
| extend d = tolong((Day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+200*r-1), 1) 
| mv-expand id=_users to typeof(long) limit 1000000
// Last parameter is omitted - 
| evaluate new_activity_metrics(['id'], Day, _start, _end, 7d)
| project from_Day, to_Day, retention_rate, churn_rate
```

|from_Day|to_Day|retention_rate|churn_rate|
|---|---|---|---|
|2017-05-01 00:00:00.0000000|2017-05-01 00:00:00.0000000|1|0|
|2017-05-01 00:00:00.0000000|2017-05-08 00:00:00.0000000|0.190397350993377|0.809602649006622|
|2017-05-01 00:00:00.0000000|2017-05-15 00:00:00.0000000|0|1|
|2017-05-01 00:00:00.0000000|2017-05-22 00:00:00.0000000|0|1|
|2017-05-01 00:00:00.0000000|2017-05-29 00:00:00.0000000|0|1|
|2017-05-08 00:00:00.0000000|2017-05-08 00:00:00.0000000|1|0|
|2017-05-08 00:00:00.0000000|2017-05-15 00:00:00.0000000|0.405263157894737|0.594736842105263|
|2017-05-08 00:00:00.0000000|2017-05-22 00:00:00.0000000|0.227631578947368|0.772368421052632|
|2017-05-08 00:00:00.0000000|2017-05-29 00:00:00.0000000|0|1|
|2017-05-15 00:00:00.0000000|2017-05-15 00:00:00.0000000|1|0|
|2017-05-15 00:00:00.0000000|2017-05-22 00:00:00.0000000|0.785488958990536|0.214511041009464|
|2017-05-15 00:00:00.0000000|2017-05-29 00:00:00.0000000|0.237644584647739|0.762355415352261|
|2017-05-22 00:00:00.0000000|2017-05-22 00:00:00.0000000|1|0|
|2017-05-22 00:00:00.0000000|2017-05-29 00:00:00.0000000|0.621835443037975|0.378164556962025|
|2017-05-29 00:00:00.0000000|2017-05-29 00:00:00.0000000|1|0|


### <a name="weekly-retention-rate-with-lookback-period"></a>回溯时段的每周保留率

下面的查询在考虑 `lookback` 时段的情况下计算 `New Users` 队列的保留率：一个表格查询，其中包含用于定义 `New Users` 队列的一组 ID（此集中未出现的所有 ID 为 `New Users`）。 查询在分析期间检查 `New Users` 的保留行为。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
// Generate random data of user activities
let _lookback = datetime(2017-02-01);
let _start = datetime(2017-05-01);
let _end = datetime(2017-05-31);
let _data = range Day from _lookback to _end  step 1d
| extend d = tolong((Day - _lookback)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+200*r-1), 1) 
| mv-expand id=_users to typeof(long) limit 1000000;
//
let lookback_data = _data | where Day < _start | project Day, id;
_data
| evaluate new_activity_metrics(id, Day, _start, _end, 7d, _start, lookback_data)
| project from_Day, to_Day, retention_rate
```

|from_Day|to_Day|retention_rate|
|---|---|---|
|2017-05-01 00:00:00.0000000|2017-05-01 00:00:00.0000000|1|
|2017-05-01 00:00:00.0000000|2017-05-08 00:00:00.0000000|0.404081632653061|
|2017-05-01 00:00:00.0000000|2017-05-15 00:00:00.0000000|0.257142857142857|
|2017-05-01 00:00:00.0000000|2017-05-22 00:00:00.0000000|0.296326530612245|
|2017-05-01 00:00:00.0000000|2017-05-29 00:00:00.0000000|0.0587755102040816|
