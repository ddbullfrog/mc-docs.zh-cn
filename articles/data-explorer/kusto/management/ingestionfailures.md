---
title: 引入失败 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的引入失败。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 01/20/2019
ms.date: 09/24/2020
ms.openlocfilehash: 643da246c14c486b7df98a65b3246e54b3e593f8
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146347"
---
# <a name="ingestion-failures"></a>引入失败

## <a name="show-ingestion-failures"></a>.show ingestion failures


此命令返回一个结果集，该结果集包含[数据引入控制命令](../../ingest-data-overview.md#kusto-query-language-ingest-control-commands)运行时发生的任何引入失败。


> [!NOTE]
> 引入流的其他部分发生的引入失败不会出现在此命令的结果集中。 例如，在将数据引入控制命令发送到 Kusto 数据引擎服务之前，可能会发生这种故障。

> 若要详细了解如何监视在涉及[排队引入](../api/netfx/about-kusto-ingest.md#queued-ingestion)的流中发生的故障，请参阅[此指南](../api/netfx/kusto-ingest-client-status.md)。

**语法**

|语法选项|描述|
|---|---| 
|`.show` `ingestion` `failures`                                       |返回所有已记录的引入失败  
|`.show` `ingestion` `failures` <code>&#124;</code> `where` ...       |返回一组筛选的引入失败
|`.show` `ingestion` `failures` `with(OperationId="`*OperationId*`")` |返回特定操作 ID 的引入失败

**结果**
 
|输出参数           |类型     |说明                                                                              |
|---------------------------|---------|-----------------------------------------------------------------------------------------|
|OperationId                |String   |操作标识符，可用于通过 <br> [.show operations](operations.md) 命令查看其他操作详细信息 </br> 
|数据库                   |字符串   |发生失败的数据库
|表                      |String   |发生失败的表
|FailedOn                   |DateTime |对失败进行注册的日期/时间 (UTC) 
|IngestionSourcePath        |String   |标识引入源（通常为 Azure Blob URI） 
|详细信息                    |String   |失败详细信息。 提供对实际引入失败根本原因的见解
|FailureKind                |String   |失败类型（永久/暂时）
|RootActivityId             |String   |根活动 ID。
|OperationKind              |String   |引入操作类型（阶段），在此期间已对失败进行注册
|OriginatesFromUpdatePolicy |布尔 | 指示在执行[更新策略](update-policy.md)时是否注册了失败
 
**示例**
 
|OperationId |数据库 |表 |FailedOn |IngestionSourcePath |详细信息 |FailureKind |RootActivityId |OperationKind |OriginatesFromUpdatePolicy
|--|--|--|--|--|--|--|--|--|--
|3827def6-0773-4f2a-859e-c02cf395deaf |DB1 |Table1 |2017-02-14 22:25:03.1147331 |...url... |ID 为“*****.csv”的流出现 CSV 格式错误* |永久性 |3c883942-e446-4999-9b00-d4c664f06ef6 |DataIngestPull | 0
|841fafa4-076a-4cba-9300-4836da0d9c75 |DB1 |Table1 |2017-02-14 22:34:11.2565943 |...url... |ID 为“*****.csv”的流出现 CSV 格式错误* |永久性 |48571bdb-b714-4f32-8ddc-4001838a956c |DataIngestPull | 0
|e198c519-5263-4629-a158-8d68f7a1022f |DB1 |Table1 |2017-02-14 22:34:44.5824741 |...url... |ID 为“*****.csv”的流出现 CSV 格式错误* |永久性 |5e31ab3c-e2c7-489a-827e-e89d2d691ec4 |DataIngestPull | 0
|a9f287a1-f3e6-4154-ad18-b86438da0929 |DB1 |Table1 |2017-02-14 22:36:26.5525250 |...url... |发生了未知错误：引发了“System.Exception”类型的异常 |暂时性 |9b7bb017-471e-48f6-9c96-d16fcf938d2a |DataIngestPull | 0
|9edb3ecc-f4b4-4738-87e1-648eed2bd998 |DB1 |Table1 |2017-02-14 23:52:31.5460071 |...url... |下载 blob 失败：客户端无法在指定的超时时间内完成操作 |永久性 |21fa0dd6-cd7d-4493-b6f7-78916ce0d617 |DataIngestPull | 0
