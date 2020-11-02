---
title: 使用 Azure 数据资源管理器创建业务连续性和灾难恢复解决方案
description: 本文介绍了如何使用 Azure 数据资源管理器创建业务连续性和灾难恢复解决方案
author: orspod
ms.author: v-tawe
ms.reviewer: herauch
ms.service: data-explorer
ms.topic: conceptual
origin.date: 08/05/2020
ms.date: 08/18/2020
ms.openlocfilehash: 518f5535d9eebf40b92ff39f5f7176221eb8e55f
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105198"
---
# <a name="create-business-continuity-and-disaster-recovery-solutions-with-azure-data-explorer"></a>使用 Azure 数据资源管理器创建业务连续性和灾难恢复解决方案

本文详细介绍了如何通过在不同的 Azure 区域中复制 Azure 数据资源管理器资源、管理活动和数据引入，为应对 Azure 区域性中断做准备。 提供了通过事件中心进行数据引入的一个示例。 此外还针对不同的体系结构配置讨论了成本优化。 有关体系结构注意事项和恢复解决方案的详细信息，请参阅[业务连续性概述](business-continuity-overview.md)。

## <a name="prepare-for-azure-regional-outage-to-protect-your-data"></a>为应对 Azure 区域性中断做准备以保护你的数据

Azure 数据资源管理器不支持针对整个 Azure 区域的中断进行自动保护。 在自然灾害（如地震）期间，会发生这种中断。 如果你需要一个用于灾难恢复的解决方案，请执行以下步骤以确保业务连续性。 在这些步骤中，需在两个 Azure 配对区域中复制群集、管理活动和数据引入。

1. 在两个 Azure 配对区域中[创建两个或更多个独立的群集](#create-multiple-independent-clusters)。
1. [复制所有管理活动](#replicate-management-activities)，例如，在每个群集上创建新表或管理用户角色。
1. 以并行方式将数据引入到每个群集中。

### <a name="create-multiple-independent-clusters"></a>创建多个独立的群集

在多个区域中创建多个 [Azure 数据资源管理器群集](create-cluster-database-portal.md)。
请确保在 [Azure 配对区域](/best-practices-availability-paired-regions)中创建至少两个这样的群集。

下图显示的是副本，三个群集位于三个不同的区域中。 

:::image type="content" source="media/business-continuity-create-solution/independent-clusters.png" alt-text="创建独立的群集":::

### <a name="replicate-management-activities"></a>复制管理活动

复制管理活动，使每个副本中具有相同的群集配置。

1. 在每个副本上创建相同的项，如下所示： 
    * 数据库：可以使用 [Azure 门户](create-cluster-database-portal.md#create-a-database)或我们的 [SDK](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/kusto/Microsoft.Azure.Management.Kusto) 之一来创建新数据库。
    * [表](kusto/management/create-table-command.md)
    * [映射](kusto/management/create-ingestion-mapping-command.md)
    * [策略](kusto/management/policies.md)

1. 在每个副本上管理[身份验证和授权](kusto/management/security-roles.md)。

    :::image type="content" source="media/business-continuity-create-solution/regional-duplicate-management.png" alt-text="创建独立的群集":::    

### <a name="configure-data-ingestion"></a>配置数据引入

在每个群集上一致地配置数据引入。 以下引入方法使用以下高级业务连续性功能。

|引入方法  |灾难恢复功能  |
|---------|---------|
|[IoT 中心](/iot-hub/iot-hub-ha-dr#cross-region-dr)  |[Microsoft 发起的故障转移和手动故障转移](/iot-hub/iot-hub-ha-dr#cross-region-dr) |

<!-- |[Event Hub](ingest-data-event-hub.md) | Metadata disaster recovery using [primary and secondary disaster recovery namespaces](/event-hubs/event-hubs-geo-dr)     | -->

<!-- |[Ingest from storage using Event Grid subscription](ingest-data-event-grid.md) |  Implement a [geo-disaster recovery](/event-hubs/event-hubs-geo-dr) for the blob-created messages that are sent to Event Hub and the [disaster recovery and account failover strategy](/storage/common/storage-disaster-recovery-guidance)       | -->

## <a name="disaster-recovery-solution-using-event-hub-ingestion"></a>使用事件中心引入功能的灾难恢复解决方案

完成[为应对 Azure 区域性中断做准备以保护你的数据](#prepare-for-azure-regional-outage-to-protect-your-data)后，你的数据和管理活动会分发到多个区域。 如果某个区域发生服务中断，则 Azure 数据资源管理器将能够使用其他副本。 

### <a name="set-up-ingestion-using-event-hub"></a>使用事件中心设置引入

以下示例通过事件中心使用引入。 已设置故障转移流，并且 Azure 数据资源管理器通过别名引入数据。 使用每个群集副本的唯一使用者组[从事件中心引入数据](ingest-data-event-hub.md)。 否则，最终将分发流量，而不是复制流量。

> [!NOTE] 
> 通过事件中心/IoT 中心/存储进行引入是可靠的。 如果某个群集在一段时间内不可用，则它稍后会追赶进度并插入任何挂起的消息或 blob。 此过程依赖于[检查点设置](/event-hubs/event-hubs-features#checkpointing)。

:::image type="content" source="media/business-continuity-create-solution/event-hub-management-scheme.png" alt-text="创建独立的群集":::

如下图所示，数据源会生成发往已进行故障转移配置的事件中心的事件，而每个 Azure 数据资源管理器副本都会使用这些事件。 数据可视化效果组件（例如 Power BI、Grafana 或 SDK 支持的 WebApps）可以查询其中一个副本。

:::image type="content" source="media/business-continuity-create-solution/data-sources-visualization.png" alt-text="创建独立的群集":::

## <a name="optimize-costs"></a>优化成本

现在，你可以使用下面的一些方法优化副本：

* [创建主动-热备用服务器配置](#create-an-active-hot-standby-configuration)
* [启动和停止副本](#start-and-stop-the-replicas)
* [实施高度可用的应用程序服务](#implement-a-highly-available-application-service)
* [在主动-主动配置中优化成本](#optimize-cost-in-an-active-active-configuration)

### <a name="create-an-active-hot-standby-configuration"></a>创建主动-热备用服务器配置

复制和更新 Azure 数据资源管理器设置会导致成本随副本数量线性增加。 为了优化成本，你可以实施体系结构变体来平衡时间、故障转移和成本。
在主动-热备用服务器配置中，通过引入被动 Azure 数据资源管理器副本实现了成本优化。 只有当主要区域（例如区域 A）中发生灾难时，才会启用这些副本。 区域 B 和 C 中的副本不需要全天候处于活动状态，因而大大降低了成本。 但在大多数情况下，这些副本的性能不如主要群集。 有关详细信息，请参阅[主动-热备用服务器配置](business-continuity-overview.md#active-hot-standby-configuration)。

在下图中，只有一个群集从事件中心引入数据。 区域 A 中的主要群集执行[连续数据导出](kusto/management/data-export/continuous-data-export.md)来将所有数据导出到某个存储帐户。 次要副本有权使用[外部表](kusto/query/schema-entities/externaltables.md)访问数据。

:::image type="content" source="media/business-continuity-create-solution/active-hot-standby-scheme.png" alt-text="创建独立的群集":::

### <a name="start-and-stop-the-replicas"></a>启动和停止副本 

可以使用以下方法之一启动和停止次要副本：

<!-- * [Azure Data Explorer connector to Power Automate (Preview)](flow.md) -->

* Azure 门户中“概览”选项卡上的“停止”按钮。 有关详细信息，请参阅[停止和重启群集](create-cluster-database-portal.md#stop-and-restart-the-cluster)。

* Azure CLI： 

```kusto
az kusto cluster stop --name=<clusterName> --resource-group=<rgName> --subscription=<subscriptionId>” 
```

### <a name="implement-a-highly-available-application-service"></a>实施高度可用的应用程序服务

#### <a name="create-the-azure-app-service-bcdr-client"></a>创建 Azure 应用服务 BCDR 客户端

本部分介绍了如何创建 [Azure 应用服务](https://azure.microsoft.com/services/app-service/)，该服务支持与单个主要的和多个辅助的 Azure 数据资源管理器群集进行连接。 下图展示了 Azure 应用服务设置。

:::image type="content" source="media/business-continuity-create-solution/app-service-setup.png" alt-text="创建独立的群集":::

> [!TIP]
> 在同一服务中的副本之间建立多个连接可以提高可用性。 此设置不仅仅在发生区域性中断的情况下有用。  

1. 使用此[应用服务样板代码](https://github.com/Azure/azure-kusto-bcdr-boilerplate)。 为了实施多群集客户端，已经创建了 [AdxBcdrClient](https://github.com/Azure/azure-kusto-bcdr-boilerplate/blob/master/webapp/ADX/AdxBcdrClient.cs) 类。 使用此客户端执行的每个查询都将[首先发送到主要群集](https://github.com/Azure/azure-kusto-bcdr-boilerplate/blob/26f8c092982cb8a3757761217627c0e94928ee07/webapp/ADX/AdxBcdrClient.cs#L69)。 如果出现故障，则查询将发送到次要副本。

1. 使用[自定义应用程序见解指标](/azure-monitor/app/api-custom-events-metrics)来度量性能，并请求将内容分发到主要群集和辅助群集。 

#### <a name="test-the-azure-app-service-bcdr-client"></a>测试 Azure 应用服务 BCDR 客户端

我们使用多个 Azure 数据资源管理器副本运行了测试。 在模拟主要和辅助群集的服务中断后，可以看到应用服务 BCDR 客户端的行为符合预期。

:::image type="content" source="media/business-continuity-create-solution/simulation-verify-service.png" alt-text="创建独立的群集":::

> [!NOTE]
> 响应时间较慢是由不同的 SKU 和全球查询导致的。

#### <a name="perform-dynamic-or-static-routing"></a>执行动态或静态路由

使用 [Azure 流量管理器路由方法](/traffic-manager/traffic-manager-routing-methods)对请求进行动态或静态路由。  Azure 流量管理器是一个基于 DNS 的流量负载均衡器，可以用来分发应用服务流量。 此流量针对全球 Azure 区域中的服务进行了优化，同时提供高可用性和快速响应能力。 

<!-- You can also use [Azure Front Door based routing](/frontdoor/front-door-routing-methods). For comparison of these two methods, see [Load-balancing with Azure’s application delivery suite](/frontdoor/front-door-lb-with-azure-app-delivery-suite). -->

### <a name="optimize-cost-in-an-active-active-configuration"></a>在主动-主动配置中优化成本

使用主动-主动配置进行灾难恢复会线性增加成本。 成本包括节点、存储、标记成本以及因[带宽](https://www.azure.cn/pricing/details/bandwidth/)增加的网络成本。

<!-- #### Use optimized autoscale to optimize costs

Use the [optimized autoscale](manage-cluster-horizontal-scaling.md#optimized-autoscale) feature to configure horizontal scaling for the secondary clusters. They should be dimensioned so they can handle the ingestion load. Once the primary cluster isn't reachable, the secondary clusters will get more traffic and scale according to the configuration. 

Using optimized autoscale in this example saved roughly 50% of the cost in comparison to having the same horizontal and vertical scale on all replicas. -->

## <a name="next-steps"></a>后续步骤

从[业务连续性和灾难恢复概述](business-continuity-overview.md)着手。
