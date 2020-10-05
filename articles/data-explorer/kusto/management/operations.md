---
title: 操作管理 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的操作管理。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/24/2020
ms.openlocfilehash: 04ed71a78f9923fb8f9c6d12ba9836993a87deaf
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146601"
---
# <a name="operations-management"></a>操作管理

## <a name="show-operations"></a>.show operations 

`.show` `operations` 命令返回一个表，其中包含了最近两周内执行的、所有正在运行和已完成的管理操作（这当前是保留期配置）。

**语法**

|语法选项|说明|
|---|---| 
|`.show` `operations`              |返回集群正在处理的所有操作或集群已处理的操作
|`.show` `operations` *OperationId*|返回特定 ID 的操作状态 
|`.show` `operations` `(`*OperationId1*`,` *OperationId2*`,` ...)|返回特定 ID 的操作状态

**结果**
 
|输出参数 |类型 |说明
|---|---|---
|ID |String |操作标识符
|操作 |String |管理员命令别名
|NodeId |String |如果命令具有远程执行（例如，DataIngestPull）- NodeId 将包含正在执行的远程节点的 ID
|StartedOn |DateTime |操作开始的日期/时间（以 UTC 为单位）
|LastUpdatedOn |DateTime |上次更新操作的日期/时间（以 UTC 为单位）（可以是操作中的步骤，也可以是完成步骤）
|持续时间 |DateTime |LastUpdateOn 和 StartedOn 之间的 TimeSpan
|状态 |String |命令状态 - 值可能是“InProgress”、“Completed”或“Failed”
|状态 |String |包含失败操作错误的其他帮助字符串
 
**示例**
 
|ID |操作 |节点 ID |开始日期 |上次更新日期 |持续时间 |状态 |状态 
|--|--|--|--|--|--|--|--
|3827def6-0773-4f2a-859e-c02cf395deaf |SchemaShow | |2015-01-06 08:47:01.0000000 |2015-01-06 08:47:01.0000000 |0001-01-01 00:00:00.0000000 |已完成 |
|841fafa4-076a-4cba-9300-4836da0d9c75 |DataIngestPull |Kusto.Azure.Svc_IN_1 |2015-01-06 08:47:02.0000000 |2015-01-06 08:48:19.0000000 |0001-01-01 00:01:17.0000000 |已完成 |
|e198c519-5263-4629-a158-8d68f7a1022f |OperationsShow | |2015-01-06 08:47:18.0000000 |2015-01-06 08:47:18.0000000 |0001-01-01 00:00:00.0000000 |已完成 |
|a9f287a1-f3e6-4154-ad18-b86438da0929 |ExtentsDrop | |2015-01-11 08:41:01.0000000 |0001-01-01 00:00:00.0000000 |0001-01-01 00:00:00.0000000 |正在进行 |
|9edb3ecc-f4b4-4738-87e1-648eed2bd998 |DataIngestPull | |2015-01-10 14:57:41.0000000 |2015-01-10 14:57:41.0000000 |0001-01-01 00:00:00.0000000 |已失败 |集合已修改。 枚举操作可能不会执行。

## <a name="show-operation-details"></a>.show operation details

操作可以（可选）长期保留其结果，并且可以在操作完成时使用 `.show` `operation` `details` 来检索结果。

> [!NOTE]
> 并非所有控制命令都能长期保留其结果。 这些命令仅在默认情况下针对异步执行时才通过使用 `async` 关键字来这样做。 请参阅有关特定命令的文档，然后检查命令是否执行此操作。 例如，请参阅[数据导出](data-export/index.md)）。
> `.show` `operations` `details` 命令的输出架构与从同步执行命令返回的架构相同。
> 仅在成功完成操作后才能调用 `.show` `operation` `details` 命令。 运行此命令前，请使用 [show operations 命令](#show-operations)）检查操作的状态。

**语法**

`.show` `operation` *OperationId* `details`

**结果**

每种操作的结果都不同，并且在同步执行时，与操作结果的架构匹配。

**示例**

该示例中的 OperationId 从其中一个[数据导出](../management/data-export/index.md)命令的异步执行返回。

```kusto 
.export 
  async 
  to csv ( 
    h@"https://storage1.blob.core.chinacloudapi.cn/containerName;secretKey", 
    h@"https://storage1.blob.core.chinacloudapi.cn/containerName2;secretKey" 
  ) 
  <| myLogs 
```

async export 命令返回了以下操作 ID：

|OperationId|
|---|
|56e51622-eb49-4d1a-b896-06a03178efcd|

命令完成后，可以使用此操作 ID 查询导出的 blob。 

```kusto
.show operation 56e51622-eb49-4d1a-b896-06a03178efcd details 
```

|`Path`|NumRecords |
|---|---|
|http://storage1.blob.core.chinacloudapi.cn/containerName/1_d08afcae2f044c1092b279412dcb571b.csv|10|
|http://storage1.blob.core.chinacloudapi.cn/containerName/2_454c0f1359e24795b6529da8a0101330.csv|15|
