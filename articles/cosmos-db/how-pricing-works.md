---
title: Azure Cosmos DB 的定价模型
description: 本文介绍 Azure Cosmos DB 的定价模型，以及该模型如何简化成本管理和成本计划。
ms.service: cosmos-db
ms.topic: conceptual
origin.date: 08/19/2020
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 91c284b9211f2c7d1902eec6c4e623346eb8da88
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246292"
---
# <a name="pricing-model-in-azure-cosmos-db"></a>Azure Cosmos DB 中的定价模型

Azure Cosmos DB 的定价模型可简化成本管理和计划。 使用 Azure Cosmos DB，你需要为你针对数据库执行的操作以及你的数据使用的存储付费。

- 数据库操作：数据库操作的收费方式取决于你使用的 Azure Cosmos 帐户的类型。

  - **预配的吞吐量**：[预配的吞吐量](set-throughput.md)（也称为预留吞吐量）保证在任何规模都具有高性能。 请以每秒的[请求单位](request-units.md)数 (RU/s) 形式指定所需吞吐量，Azure Cosmos DB 会提供所需资源来保证已配置的吞吐量。 可以[在数据库或容器上预配吞吐量](set-throughput.md)。 根据工作负载需求，可以随时纵向扩展/缩减吞吐量或使用[自动缩放](provision-throughput-autoscale.md)（尽管数据库或容器上有一个保证 SLA 所需的最低吞吐量）。 按指定时间内最大的预配吞吐量以小时来收费。

   > [!NOTE]
   > 由于预配的吞吐量模型会将资源提供给容器或数据库，因此即使不运行任何工作负载，你也要为你已预配的吞吐量付费。

  - 无服务器：在 [无服务器](serverless.md)模式下，无需在 Azure Cosmos 帐户中创建资源时预配任何吞吐量。 在计费周期结束时，会针对你的数据库操作已使用的请求单位量计费。

- **存储**：对于一个给定小时内你的数据和索引所使用的存储总量（以 GB 为单位），将按单一费率计费。 存储按使用量计费，因此你不必提前预留任何存储。 仅为所使用的存储付费。

Azure Cosmos DB 中的定价模型在所有 API 中都是一致的。 有关详细信息，请参阅 [Azure Cosmos DB 定价页](https://www.azure.cn/pricing/details/cosmos-db/)、[了解 Azure Cosmos DB 帐单](understand-your-bill.md)和 [Azure Cosmos DB 定价模型如何对客户而言更具经济效益](total-cost-ownership.md)。

如果将 Azure Cosmos DB 帐户部署到某个区域，则对于处于预配吞吐量模式下基于数据库和容器的吞吐量，有一个最低价格。 在无服务器模式下没有最低价格。 定价取决于所使用的区域，有关最新定价信息，请参阅 [Azure Cosmos DB 定价页](https://www.azure.cn/pricing/details/cosmos-db/)。

<!--MOONCAKE CORRECT ON non-government region in the US-->

## <a name="try-azure-cosmos-db-for-free"></a>免费试用 Azure Cosmos DB

Azure Cosmos DB 免费为开发人员提供众多选项。 这些选项包括：

* Azure Cosmos DB 免费层：使用 Azure Cosmos DB 免费层，可以轻松上手、开发和测试应用程序，甚至免费运行小型生产工作负载。 如果在帐户上启用了免费层，那么在该帐户的生存期内，你将在该帐户中免费获得前 400 RU/秒的吞吐量和 5 GB 的存储空间。 每个 Azure 订阅最多可以有一个免费层帐户，并且必须在创建帐户时选择加入使用。 首先，[在 Azure 门户中创建一个启用了免费层的新帐户](create-cosmosdb-resources-portal.md) 或使用 [ARM 模板](manage-sql-with-resource-manager.md#free-tier)。

* **Azure 试用帐户**：Azure 提供了一个[免费层](https://www.azure.cn/pricing/1rmb-trial/)，它在前 30 天为你提供了价值人民币 1500 元的 Azure 额度。

<!--MOONCAKE CUSTOMIZE: Azure trial account-->
<!--Not Available on For more information [Azure trial account](../billing/billing-avoid-charges-free-account.md)-->
<!--Not Available on * **Try Azure Cosmos DB for free**-->

* **Azure Cosmos DB 模拟器**：为方便进行开发，Azure Cosmos DB 模拟器提供了一个模拟 Azure Cosmos DB 服务的本地环境。 模拟器免费提供，并且具有对云服务的高保真度。 使用 Azure Cosmos DB 模拟器可在本地开发和测试应用程序，无需创建 Azure 订阅且不会产生任何费用。 投入生产之前，可以在本地使用模拟器开发应用程序。 如果对模拟器的应用程序功能感到满意，可切换到云中的“使用 Azure Cosmos DB 帐户”，从而大幅节省成本。 有关模拟器的详细信息，请参阅[使用 Azure Cosmos DB 进行开发和测试](local-emulator.md)一文。

<!-- Not Available on ## Pricing with reserved capacity-->
## <a name="next-steps"></a>后续步骤

可在以下文章中了解更多关于优化 Azure Cosmos DB 资源成本的信息：

* 了解[开发和测试优化](optimize-dev-test.md)
* 详细了解[了解 Azure Cosmos DB 帐单](understand-your-bill.md)
* 详细了解如何[优化吞吐量成本](optimize-cost-throughput.md)
* 详细了解如何[优化存储成本](optimize-cost-storage.md)
* 详细了解如何[优化读取和写入成本](optimize-cost-reads-writes.md)
* 详细了解如何[优化查询成本](optimize-cost-queries.md)
* 详细了解如何[优化多区域 Cosmos 帐户的成本](optimize-cost-regions.md)
    
    <!--Not Available on * Learn about [Azure Cosmos DB reserved capacity](cosmos-db-reserved-capacity.md)-->
    
* 了解 [Azure Cosmos DB 模拟器](local-emulator.md)

<!-- Update_Description: update meta properties, wording update, update link -->