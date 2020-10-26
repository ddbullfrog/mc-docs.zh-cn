---
title: 使用 Azure Cosmos DB 在多个区域分配数据
description: 了解如何通过多区域分布式多模型数据库服务 Azure Cosmos DB，使用多区域数据库进行多区域范围的异地复制、多主数据库故障转移和数据恢复。
ms.service: cosmos-db
ms.topic: conceptual
origin.date: 07/23/2019
author: rockboyfor
ms.date: 10/19/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: ad341018a9d84f3ebe4687f7c6a413a0175a4837
ms.sourcegitcommit: 7320277f4d3c63c0b1ae31ba047e31bf2fe26bc6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92118062"
---
# <a name="distribute-your-data-multiple-regionally-with-azure-cosmos-db"></a>使用 Azure Cosmos DB 将数据分布在多个区域

如今的应用程序需要具备高响应能力并始终联机。 若要实现低延迟和高可用性，需要在靠近用户的数据中心部署这些应用程序的实例。 这些应用程序通常部署在多个数据中心，称为多区域分布式应用程序。 多区域分布式应用程序需要一个多区域分布式数据库，该数据库可以透明地将数据复制到中国任何位置，以使应用程序能够对靠近其用户的数据副本进行操作。 

<!-- Notice: 全球 to 多个区域分布 -->

Azure Cosmos DB 是一个多区域分布式数据库服务，旨在提供低延迟、吞吐量弹性缩放和妥善定义的语义，以实现数据一致性和高可用性。 简单而言，如果应用程序需要保证在中国任何位置都能提供快速响应、始终联机并需要无限且可弹性缩放的吞吐量和存储，则应在 Azure Cosmos DB 上生成应用程序。

可将数据库配置为多区域分布，并使其可在任何 Azure 中国区域中使用。 为了降低延迟，请将数据放置在更靠近用户的位置。 根据应用程序的多区域覆盖范围和用户所在的位置选择所需的区域数目。 Cosmos DB 以透明方式将数据复制到与 Cosmos 帐户关联的所有区域。 它提供多区域分布式 Azure Cosmos 数据库和容器的单个系统映像，使应用程序能够在本地读取和写入。 

<!-- Notice: Azure China Regions -->

使用 Azure Cosmos DB 可以随时添加或删除与帐户关联的区域。 无需暂停或重新部署应用程序即可添加或删除区域。 得益于该服务原生提供的多宿主功能，它始终都能保持高可用性。

:::image type="content" source="./media/distribute-data-globally/deployment-topology.png" alt-text="高度可用的部署拓扑" border="false":::

<!--MOONCAKE: CORRECT ON (./media/distribute-data-globally/deployment-topology.png)-->

<a name="key-benefits-of-global-distribution"></a>
## <a name="key-benefits-of-multiple-region-distribution"></a>多区域分布的主要优点

**构建多区域主动-主动应用。** 使用新式多主数据库复制协议，每个区域都支持写入和读取。 多主数据库功能还可以实现：

- 无限弹性写入和读取可伸缩性。 
- 在全中国实现 99.999% 的读写可用性。
- 在 99% 的时间内，在 10 毫秒内为读写提供服务。

使用 Azure Cosmos DB 的多宿主 API，应用程序始终知道最靠近的区域并将请求发送到该区域。 无需进行任何配置更改就能识别最靠近的区域。 在 Azure Cosmos 帐户中添加和删除区域时，无需重新部署或暂停应用程序，它始终会保持高可用性。

**生成高响应能力的应用。** 应用程序可以在为数据库选择的所有区域中执行近乎实时的读写。 Azure Cosmos DB 在内部处理区域之间的数据复制，并保证提供所选的一致性级别。

**生成高度可用的应用。** 在中国的多个区域中运行数据库可以提高数据库的可用性。 如果一个区域不可用，其他区域可自动处理应用程序请求。 Azure Cosmos DB 为多区域数据库提供 99.999% 的读取和写入可用性。

**在区域性中断期间保持业务连续性。** Azure Cosmos DB 支持在区域性中断期间进行[自动故障转移](how-to-manage-database-account.md#automatic-failover)。 在区域性中断期间，Azure Cosmos DB 会继续维持其延迟、可用性、一致性和吞吐量方面的 SLA。 为帮助确保整个应用程序高度可用，Cosmos DB 提供手动故障转移 API 来模拟区域性中断。 使用此 API 可以执行常规业务连续性演练。

**多区域缩放读写吞吐量** 。 可让每个区域变得可写，并弹性缩放中国各地的读取和写入。 保证应用程序针对 Azure Cosmos 数据库或容器配置的吞吐量可在与 Azure Cosmos 帐户关联的所有区域中实现。 预配的吞吐量有 [SLA 的资金保障](https://www.azure.cn/support/sla/cosmos-db/)。

**从多个明确定义的一致性模型中进行选择。** Azure Cosmos 数据库复制协议提供了五种明确定义、实用且直观的一致性模型。 每个模型在一致性与性能之间进行了权衡。 使用这些一致性模型可以轻松生成多区域分布式应用程序。

## <a name="next-steps"></a><a name="Next Steps"></a>后续步骤

阅读以下文章详细了解多区域分布：

* [多区域分布 - 揭秘](global-dist-under-the-hood.md)
* [如何在应用程序中配置多主数据库](how-to-multi-master.md)
* [配置多宿主客户端](how-to-manage-database-account.md#configure-multiple-write-regions)
* [在 Azure Cosmos DB 帐户中添加或删除区域](how-to-manage-database-account.md#addremove-regions-from-your-database-account)
* [为 SQL API 帐户创建自定义冲突解决策略](how-to-manage-conflicts.md#create-a-custom-conflict-resolution-policy)
* [Cosmos DB 中的可编程一致性模型](consistency-levels.md)
* [为你的应用程序选择适当的一致性级别](consistency-levels-choosing.md)
* [跨 Azure Cosmos DB API 的一致性级别](consistency-levels-across-apis.md)
* [各种一致性级别的可用性和性能权衡](consistency-levels-tradeoffs.md)

<!-- Update_Description: update meta properties, wording update, update link -->