---
title: 诊断信息 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的诊断信息。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/24/2020
ms.openlocfilehash: 6170d182a8dee508b9a8dd2dc370bcd7e1ed3bd5
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146432"
---
# <a name="diagnostic-information"></a>诊断信息

这些命令可用于显示系统诊断信息。

* [.show cluster](#show-cluster)
* [.show diagnostics](#show-diagnostics)
* [.show capacity](#show-capacity)
* [.show operations](#show-operations)

## <a name="show-cluster"></a>.show cluster

```kusto
.show cluster
```

返回一个集，该集包含群集中当前处于活动状态的每个节点的一条记录。  

**结果** 

|输出列 |类型 |说明
|---|---|---|
|NodeId|String|标识节点。 如果在 Azure 中部署了该群集，则节点 ID 就是节点的 Azure RoleId
|地址|String |由群集用于进行节点间通信的内部终结点
|名称 |String |节点的内部名称。 名称包括计算机名称、进程名称和进程 ID
|StartTime |DateTime |节点中当前的 Kusto 实例化开始的日期/时间（以 UTC 为单位）。 此值可用于检测节点（或在该节点上运行的 Kusto）最近是否重启
|IsAdmin |布尔 |如果此节点当前是群集的“领导者” 
|MachineTotalMemory  |Int64 |节点的 RAM 容量。
|MachineAvailableMemory  |Int64 |当前可在节点上使用的 RAM 容量。
|ProcessorCount  |Int32 |节点上处理器的数量。
|EnvironmentDescription  |string |有关将节点环境序列化为 JSON 的其他信息。 例如升级/容错域

**示例**

```kusto
.show cluster
```

NodeID|地址|名称|StartTime|IsAdmin|MachineTotalMemory|MachineAvailableMemory|ProcessorCount|EnvironmentDescription
---|---|---|---|---|---|---|---|---
Kusto.Azure.Svc_IN_1|net.tcp://100.112.150.30:23107/|Kusto.Azure.Svc_IN_4/RD000D3AB1E9BD/WaWorkerHost/3820|2016-01-15 02:00:22.6522152|True|274877435904|247797796864|16|{"UpdateDomain":0, "FaultDomain":0}
Kusto.Azure.Svc_IN_3|net.tcp://100.112.154.34:23107/|Kusto.Azure.Svc_IN_3/RD000D3AB1E062/WaWorkerHost/2760|2016-01-15 05:52:52.1434683|False|274877435904|258740346880|16|{"UpdateDomain":1, "FaultDomain":1}
Kusto.Azure.Svc_IN_2|net.tcp://100.112.128.40:23107/|Kusto.Azure.Svc_IN_2/RD000D3AB1E054/WaWorkerHost/3776|2016-01-15 07:17:18.0699790|False|274877435904|244232339456|16|{"UpdateDomain":2, "FaultDomain":2}
Kusto.Azure.Svc_IN_0|net.tcp://100.112.138.15:23107/|Kusto.Azure.Svc_IN_0/RD000D3AB0D6C6/WaWorkerHost/3208|2016-01-15 09:46:36.9865016|False|274877435904|238414581760|16|{"UpdateDomain":3, "FaultDomain":3}


## <a name="show-diagnostics"></a>.show diagnostics

```kusto
.show diagnostics
```

返回有关 Kusto 群集运行状况的信息。
 
**返回**

|输出参数 |类型 |说明|
|-----------------|-----|-----------| 
|IsHealthy|布尔|群集是否正常运行
|IsScaleOutRequired|布尔|是否应通过添加更多计算节点来增加群集的大小
|MachinesTotal|Int64|群集中的计算机数
|MachinesOffline|Int64|当前处于脱机状态的计算机数
|NodeLastRestartedOn|DateTime|群集中的任何节点上次重启的日期/时间
|AdminLastElectedOn|DateTime|群集管理员角色的所有权上次更改的日期/时间
|MemoryLoadFactor|Double|群集所容纳的数据量（相对于其最大容量 100.0）
|ExtentsTotal|Int64|群集当前在所有数据库和所有表中拥有的数据盘区总数
|保留|Int64|
|保留|Int64|
|InstancesTargetBasedOnDataCapacity|Int64|使 ClusterDataCapacityFactor 低于 80 所需的实例数。 该值仅在所有计算机大小相同时才有效
|TotalOriginalDataSize|Int64|最初引入数据的总大小
|TotalExtentSize|Int64|压缩和编制索引后的存储数据的总大小
|IngestionsLoadFactor|Double|使用的群集引入容量的百分比。 使用 `.show capacity` 命令可查看最大容量
|IngestionsInProgress|Int64|当前正在执行的引入操作数
|IngestionsSuccessRate|Double|在前 10 分钟内成功完成的引入操作的百分比
|MergesInProgress|Int64|当前正在执行的盘区合并操作数
|BuildVersion|String|部署到群集的 Kusto 软件版本
|BuildTime|DateTime|Kusto 软件的内部版本的日期/时间。
|ClusterDataCapacityFactor|Double|使用的群集数据容量的百分比。 百分比的计算方式是：SUM（盘区大小数据）/SUM（SSD 缓存大小）。
|IsDataWarmingRequired|布尔|内部：是否应运行群集的预热查询以将数据引入本地 SSD 缓存 
|DataWarmingLastRunOn|DateTime|最后在群集上运行 .warm 数据的日期/时间
|MergesSuccessRate|Double|在前 10 分钟内成功完成的合并操作的百分比。
|NotHealthyReason|String|指定群集运行不正常的原因 
|IsAttentionRequired|布尔|群集是否需要操作团队的关注
|AttentionRequiredReason|String|指定群集需要关注的原因
|ProductVersion|String|指定产品信息（分支、版本等）
|FailedIngestOperations|Int64|前 10 分钟内引入操作失败的次数
|FailedMergeOperations|Int64|前 1 小时内合并操作失败的次数
|MaxExtentsInSingleTable|Int64|表中最大盘区数 (TableWithMaxExtents)
|TableWithMaxExtents|String|具有最大盘区数的表 (MaxExtentsInSingleTable)
|WarmExtentSize|Double|热缓存中盘区的总大小
|NumberOfDatabases|Int32|群集中的数据库数

## <a name="show-capacity"></a>.show capacity

```kusto
.show capacity
```

返回每个资源的估计群集容量的计算结果。
 
**结果**

|输出参数 |类型 |说明 
|---|---|---
|资源 |String |资源的名称 
|总计 |Int64 |“资源”类型的可用资源总量。 例如，并发引入数
|已耗用 |Int64 |当前使用的“资源”类型的资源量
|剩余 |Int64 |“资源”类型的剩余资源量
 
**示例**

|资源 |总计 |已耗用 |剩余
|---|---|---|---
|引入 |576 |1 |575

## <a name="show-operations"></a>.show operations

此命令返回一个表，其中包含自选择了新管理节点以来的所有管理操作。

|语法选项 |说明|
|---|---| 
|`.show` `operations`              |返回群集正在处理或已处理的所有操作
|`.show` `operations` OperationId|返回特定 ID 的操作状态
|`.show` `operations` `(`*OperationId1*`,` *OperationId2*`,` ...)|返回特定 ID 的操作状态

**结果**
 
|输出参数 |类型 |说明
|---|---|---
|ID |String |操作标识符
|操作 |String |管理员命令别名
|NodeId |String |如果命令正在远程运行某些内容，例如 DataIngestPull。 节点 ID 将包含正在运行的远程节点的 ID
|StartedOn |DateTime |操作开始的日期/时间（以 UTC 为单位） 
|LastUpdatedOn |DateTime |上次更新操作的日期/时间（以 UTC 为单位）。 该操作可以是操作中的步骤，也可以是一个完成步骤
|持续时间 |DateTime |LastUpdateOn 和 StartedOn 之间的时间跨度
|状态 |String |命令状态，值可为“InProgress”、“Completed”或“Failed”
|状态 |String |包含操作失败错误的另一个帮助字符串
 
**示例**
 
|ID |操作 |节点 ID |开始日期 |上次更新日期 |持续时间 |状态 |状态 
|--|--|--|--|--|--|--|--
|3827def6-0773-4f2a-859e-c02cf395deaf |SchemaShow | |2015-01-06 08:47:01.0000000 |2015-01-06 08:47:01.0000000 |0001-01-01 00:00:00.0000000 |已完成 | 
|841fafa4-076a-4cba-9300-4836da0d9c75 |DataIngestPull |Kusto.Azure.Svc_IN_1 |2015-01-06 08:47:02.0000000 |2015-01-06 08:48:19.0000000 |0001-01-01 00:01:17.0000000 |已完成 | 
|e198c519-5263-4629-a158-8d68f7a1022f |OperationsShow | |2015-01-06 08:47:18.0000000 |2015-01-06 08:47:18.0000000 |0001-01-01 00:00:00.0000000 |已完成 |
|a9f287a1-f3e6-4154-ad18-b86438da0929 |ExtentsDrop | |2015-01-11 08:41:01.0000000 |0001-01-01 00:00:00.0000000 |0001-01-01 00:00:00.0000000 |正在进行 |
|9edb3ecc-f4b4-4738-87e1-648eed2bd998 |DataIngestPull | |2015-01-10 14:57:41.0000000 |2015-01-10 14:57:41.0000000 |0001-01-01 00:00:00.0000000 |已失败 |集合已修改。 枚举操作可能无法运行 |
