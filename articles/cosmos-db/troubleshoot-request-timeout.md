---
title: 排除 Azure Cosmos DB 服务请求超时异常的故障
description: 了解如何诊断和修复 Azure Cosmos DB 服务请求超时异常。
ms.service: cosmos-db
origin.date: 07/13/2020
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: 7d1e47e634b0738e7f570021c06d0243f1d438e0
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246497"
---
<!--Verified successfully-->
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-request-timeout-exceptions"></a>诊断和排查 Azure Cosmos DB 请求超时异常
Azure Cosmos DB 返回了 HTTP 408 请求超时。

## <a name="troubleshooting-steps"></a>疑难解答步骤
下面的列表包含请求超时异常的已知原因和解决方案。

### <a name="check-the-sla"></a>检查 SLA
检查 [Azure Cosmos DB 监视](monitor-cosmos-db.md)，了解 408 异常的数目是否违反了 Azure Cosmos DB SLA。

#### <a name="solution-1-it-didnt-violate-the-azure-cosmos-db-sla"></a>解决方案 1：它不违反 Azure Cosmos DB SLA
应用程序应处理此方案，并在发生这些暂时性故障时重试。

#### <a name="solution-2-it-did-violate-the-azure-cosmos-db-sla"></a>解决方案 2：它确实违反了 Azure Cosmos DB SLA
联系 [Azure 支持](https://portal.azure.cn/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)。

### <a name="hot-partition-key"></a>热分区键
Azure Cosmos DB 在物理分区之间均匀分配预配的总吞吐量。 存在热分区时，物理分区上的一个或多个逻辑分区键会消耗物理分区的所有请求单位/秒 (RU/s)。 同时，将无法使用其他物理分区上的 RU/s。 症状是，消耗的总 RU/s 将小于数据库或容器中预配的总 RU/s。 你仍然会看到针对热逻辑分区键的请求的限制 (429s)。

<!--Not Available on [Normalized RU Consumption metric](monitor-normalized-request-units.md)-->

#### <a name="solution"></a>解决方案：
选择均匀分配请求量和存储的适当分区键。 了解如何[更改分区键](https://devblogs.microsoft.com/cosmosdb/how-to-change-your-partition-key/)。

## <a name="next-steps"></a>后续步骤
* [诊断和排查](troubleshoot-dot-net-sdk.md)在使用 Azure Cosmos DB .NET SDK 时遇到的问题。
* 了解 [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) 和 [.NET v2](performance-tips.md) 的性能准则。

<!-- Update_Description: update meta properties, wording update, update link -->