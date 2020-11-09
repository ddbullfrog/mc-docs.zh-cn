---
title: Kusto 引入客户端库 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 Kusto 引入客户端库。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.custom: has-adal-ref
origin.date: 03/18/2020
ms.date: 10/29/2020
ms.openlocfilehash: 6964d930cae1c06650af6db6b628ea61c4deaa6d
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105228"
---
# <a name="kusto-ingest-client-library"></a>Kusto 引入客户端库 

`Kusto.Ingest` 库是用于将数据发送到 Kusto 服务的 .NET 4.6.2 库。
它依赖于以下库和 SDK：

* 用于 Azure AD 身份验证的 ADAL
* Azure 存储客户端

引入方法通过 [IKustoIngestClient](kusto-ingest-client-reference.md#interface-ikustoingestclient) 接口定义。  这些方法可以同时采用同步和异步模式处理来自流、IDataReader、本地文件和 Azure blob 的数据引入。

## <a name="ingest-client-flavors"></a>引入客户端风格

引入客户端有两种基本风格：排队引入和直接引入。

### <a name="queued-ingestion"></a>排队引入

排队引入模式由 [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient) 定义，它限制了客户端代码对 Kusto 服务的依赖性。 通过将 Kusto 引入消息发布到 Azure 队列来完成引入，然后从 Kusto 数据管理（引入）服务获取该消息。 任何中间存储项都将由引入客户端使用来自 Kusto 数据管理服务的资源创建。

**排队模式的优点包括：**

* 将数据引入过程与 Kusto 引擎服务分离
* 能够在 Kusto 引擎（或引入）服务不可用时持久保存引入请求
* 借助引入服务对入站数据进行高效且可控的聚合，从而提高性能 
* 允许 Kusto 引入服务管理 Kusto 引擎服务上的引入负载
* 根据需要，在出现暂时性引入失败（例如因为 XStore 限制而导致的失败）时重试 Kusto 引入服务
* 提供了一种方便的机制来跟踪每个引入请求的进度和结果

下面的关系图概述了排队引入客户端与 Kusto 的交互：

:::image type="content" source="../images/about-kusto-ingest/queued-ingest.png" alt-text="图显示了 Kusto.Ingest 库如何在排队引入模式下将查询发送到 Kusto 服务。":::
 
### <a name="direct-ingestion"></a>直接引入

由 IKustoDirectIngestClient 定义的直接引入模式强制与 Kusto 引擎服务进行直接交互。 在此模式下，Kusto 引入服务不会限制或管理数据。 每个引入请求最终都转换为直接在 Kusto 引擎服务上执行的 `.ingest` 命令。

下面的关系图概述了直接引入客户端与 Kusto 的交互：

:::image type="content" source="../images/about-kusto-ingest/direct-ingest.png" alt-text="图显示了 Kusto.Ingest 库如何在直接引入模式下将查询发送到 Kusto 服务。":::

> [!NOTE]
> 建议不要将直接模式用于生产级引入解决方案。

**直接模式的优点包括：**

* 低延迟且无聚合。 不过，使用排队引入时也可以实现低延迟
* 使用同步方法时，方法完成即表示引入操作结束

**直接模式的缺点包括：**

* 客户端代码必须实现任何重试或错误处理逻辑
* 当 Kusto 引擎服务不可用时，无法进行引入
* 客户端代码可能会大量向 Kusto 引擎服务发送引入请求，因为它不知道引擎服务容量

## <a name="ingestion-best-practices"></a>引入最佳做法

[引入最佳做法](kusto-ingest-best-practices.md)提供了有关引入的 COG 和吞吐量 POV。

* **线程安全性 -** Kusto 引入客户端实现是线程安全的，可以重复使用。 无需为单个或多个引入操作创建 `KustoQueuedIngestClient` 类的实例。 每个目标 Kusto 群集的每个用户进程需要 `KustoQueuedIngestClient` 的单个实例。 运行多个实例会适得其反，可能会导致数据管理群集上出现 DoS。

* **支持的数据格式 -** 使用原生引入时，如果其尚未存在，请将数据上传到一个或多个 Azure 存储 blob。 目前支持的 blob 格式都记录在[受支持的数据格式](../../../ingestion-supported-formats.md)中。

* **架构映射 -** 
[架构映射](../../management/mappings.md)有助于确定性地将源数据字段绑定到目标表列。

* **引入权限 -** 
[Kusto 引入权限](kusto-ingest-client-permissions.md)解释了使用 `Kusto.Ingest` 包成功进行引入所需的权限设置。

* **用法 -** 如前所述，建议用于 Kusto 的可持续大规模引入解决方案的基础应该是 **KustoQueuedIngestClient** 。
为了最大限度地减少 Kusto 服务上的不必要负载，建议你为每个 Kusto 群集的每个进程使用 Kusto 引入客户端（排队引入或直接引入）的单个实例。 Kusto 引入客户端实现是线程安全的，并且是完全可重入的。

## <a name="next-steps"></a>后续步骤

* [Kusto.Ingest 客户端参考](kusto-ingest-client-reference.md)包含 Kusto 引入客户端接口和实现的完整参考。 你会在其中找到有关如何创建引入客户端、增加引入请求、管理引入进度等内容的信息

* [Kusto.Ingest 操作状态](kusto-ingest-client-status.md)解释了用于跟踪引入状态的 KustoQueuedIngestClient 功能

* [Kusto.Ingest 错误](kusto-ingest-client-errors.md)介绍了 Kusto 引入客户端错误和异常

* [Kusto.Ingest 示例](kusto-ingest-client-examples.md)显示的代码片段演示了将数据引入到 Kusto 中的各种技术

* [在不使用 Kusto.Ingest 库的情况下进行数据引入](kusto-ingest-client-rest.md)介绍了如何使用 Kusto REST API 来实现排队 Kusto 引入，而不依赖于 `Kusto.Ingest` 库

