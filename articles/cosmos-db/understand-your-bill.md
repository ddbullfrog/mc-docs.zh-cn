---
title: 了解 Azure Cosmos DB 帐单
description: 本文通过一些示例介绍如何了解 Azure Cosmos DB 帐单。
ms.service: cosmos-db
ms.topic: conceptual
origin.date: 08/19/2020
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: yes
ms.testdate: 09/28/2020
ms.author: v-yeche
ms.reviewer: sngun
ms.openlocfilehash: fe7849812c5091e53371bfdcd6d108dda9a348dd
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246300"
---
<!--VERIFIED SUCCESSFULLY-->
<!--UPDATE CAREFULLY-->
# <a name="understand-your-azure-cosmos-db-bill"></a>了解 Azure Cosmos DB 帐单

Azure Cosmos DB 是完全托管的云原生数据库服务，仅针对数据库操作和消耗的存储收费，因而简化了计费。 与本地或 IaaS 托管的替代方案相比，无需额外的许可费、硬件、使用成本或设施成本。 若想使用 Azure Cosmos DB 的多区域功能，与现有本地或 IaaS 解决方案相比，数据库服务可显着降低成本。

- **数据库操作**：数据库操作的收费方式取决于你使用的 Azure Cosmos 帐户的类型。

  - **预配的吞吐量**：按给定小时内的最大预配吞吐量（增量为 100 RU/秒）以小时来计费。
  - **无服务器**：每小时按数据库操作消耗的请求单位总数计费。

- **存储**：给定小时内数据和索引所消耗的存储总量（以 GB 为单位）按统一费率计费。

有关最新定价信息，请参阅[定价页](https://www.azure.cn/pricing/details/cosmos-db/)。

本文通过一些示例来帮助你了解每月帐单上的详细信息。 若存在以下情况，则示例中显示的数字可能会有所不同：Azure Cosmos 容器预配的吞吐量不同；容器跨多个区域；或在一个月内在不同时期运行。 本文中的所有示例都基于[定价页](https://www.azure.cn/pricing/details/cosmos-db/)中显示的定价信息计算账单。

> [!NOTE]
> 计费按每个小时计算，而不是按 60 分钟的持续时间计算。 

<!--MOONCAKE CUSTOMIZATION -->

## <a name="billing-examples"></a>计费示例

### <a name="billing-example---provisioned-throughput-on-a-container-full-month"></a>计费示例 - 容器上预配的吞吐量（整月）

* 假设在容器上配置了 1,000 RU/秒的吞吐量，且在该月总共存在 24 小时* 31 天 = 744 小时。  

* 容器每小时存在的 1,000 RU/秒是 10 个每小时 100 RU/秒（即 1,000/100 = 10）。 

* 以每小时 10 个单位乘以成本 0.051 元（每小时每 100 RU/秒）= 每小时 0.51 元。 

* 以每小时 0.51 元乘以当月的小时数，即 0.51 * 24 小时 * 31 天 = 当月 610.1 元。  

* 月度总帐单将显示 7,440 个单位（100 个 RU），费用为 610.1 元。

### <a name="billing-example---provisioned-throughput-on-a-container-partial-month"></a>计费示例 - 容器上预配的吞吐量（不足一个月）

* 假设创建一个预配吞吐量为 2,500 RU/秒的容器。容器在一个月内存在 24 小时（例如，在创建容器后 24 小时将其删除）。  

* 则帐单上会显示 600 个单位（2,500 RU/秒/100 RU/秒/单位* 24小时）。 费用为 30.6 元（600 个单位 * 0.051 元/单位）。

* 当月总帐单将为 30.6 元。

<!--Not Available on ### Billing example - serverless container -->
<!--THIS MODULE IS NOT AVAILABLE ON THE PRICING PAGE-->

### <a name="billing-rate-if-storage-size-changes"></a>存储大小变化时的计费率

存储容量按一个月内每小时的最大数据存储量（以 GB 为单位）计费。 例如，如果前半个月使用了 100 GB 的存储空间，而后半个月使用了 50 GB 的存储空间，则该月将按 75 GB 的等效存储空间进行计费。

### <a name="billing-rate-when-container-or-a-set-of-containers-are-active-for-less-than-an-hour"></a>容器或一组容器活动时间在一小时内的计费率

将对容器或数据库存在的每小时按统一费率进行收费，而无论使用量是多少，也无论容器或数据库使用时间是否不足一个小时。 例如，如果创建一个容器或数据库，然后在 5 分钟后删除它，那么帐单显示 1 个小时的费用。

### <a name="billing-rate-when-provisioned-throughput-on-a-container-or-database-scales-updown"></a>容器或数据库上预配的吞吐量纵向扩展/缩减时的计费率

如果在上午 9:30 将预配的吞吐量从 400 RU/秒增加到 1,000 RU/秒，然后在上午 10:45 将预配的吞吐量重新减少到 400 RU/秒，则将收取两小时 1,000 RU/秒的费用。 

如果在上午 9:30 将某个容器或一组容器的预配吞吐量从 100K RU/秒增加到 200K RU/秒，然后在上午 10:45 将预配的吞吐量重新减少到 100K RU/秒，则将收取两小时 200K RU/秒的费用。

### <a name="billing-example-multiple-containers-each-with-dedicated-provisioned-throughput-mode"></a>计费示例：多个容器，每个容器都采用专用的预配吞吐量模式

* 如果在中国东部 2 区域创建包含两个容器的 Azure Cosmos 帐户，并且这两个容器分别预配了 500 RU/秒和 700 RU/秒的吞吐量，则预配的总吞吐量将为 1,200 RU/秒。  

* 需支付 1,200/100 * 0.051 元 = 0.612 元/小时。 

* 如果需要更改吞吐量，且每个容器的容量增加了 500 RU/秒，同时还以 20,000 RU/秒创建了新的无限容器，则预配的总体容量为 22,200 RU/秒（1,000 RU/秒 + 1,200 RU/秒 + 20,000 RU/秒）。  

* 此时需支付： 0.051 元 x 222 = 11.322 元/小时。 

* 假设一个月为 744 小时（24 小时 * 31 天），如果 500 小时的预配吞吐量为 1,200 RU/秒，其余 244 小时的预配吞吐量为 22,200 RU/秒，则月度帐单将显示：500 x 0.984 元/小时 + 244 x 18.21元/小时 = 13402.56 元/月。

    <!--Not Available on :::image type="content" source="./media/understand-your-bill/bill-example1.png" alt-text="Dedicated throughput bill example":::-->

### <a name="billing-example-containers-with-shared-provisioned-throughput-mode"></a>计费示例：具有共享（预配）吞吐量模式的容器

* 如果在中国东部 2 区域创建包含两个 Azure Cosmos 数据库（其中含有一组在数据库级别共享吞吐量的容器）的 Azure Cosmos 帐户，并且这两个容器分别预配了 50K RU/秒和 70K RU/秒的吞吐量，则预配的总吞吐量将为 120K RU/秒。  

* 需支付 1200 x 0.051 元 = 61.2 元/小时。 

* 如果需要更改吞吐量，且每个数据库的预配吞吐量增加了 10K RU/秒，同时还向第一个数据库添加了新容器，共享吞吐量数据库的专用吞吐量模式为 15K RU /秒，则预配的总体容量为 155K RU/秒（60K RU/秒 + 80K RU/秒 + 15K RU/秒）。  

* 此时需支付：1,550 * 0.051 元 = 79.05元/小时。  

<!--CURRECTLY $2,880 MAP TO CNY18,360-->
    
* 假设一个月为 744 小时，如果 300 小时的预配吞吐量为 120K RU/秒，其余 444 小时的预配吞吐量为 155K RU/秒，则月度帐单将显示：300 x 61.2元/小时 + 444 x 79.05元/小时 = 18,360 元+ 35,098.2 元 = 53,458.2 元/月。 

    <!--Not Available on :::image type="content" source="./media/understand-your-bill/bill-example2.png" alt-text="Dedicated throughput bill example":::-->

## <a name="billing-examples-with-geo-replication-and-multi-master"></a>采用异地复制和多主数据库的计费示例  

可随时向 Azure Cosmos 数据库帐户添加/删除中国 Azure 区域。 对于为不同 Azure Cosmos 数据库和容器配置的吞吐量，可在与 Azure Cosmos 数据库帐户关联的每个 Azure 区域中保留这些吞吐量。 如果在 Azure Cosmos 数据库帐户（按小时配置）中的所有数据库和容器中配置的总预配吞吐量（RU /秒）为 T，而与数据库帐户关联的 Azure 区域数为 N，则对于 Azure Cosmos 数据库帐户而言，在给定时间内（小时）总预配吞吐量分别为：(a) 配置了单个写入区域时，总吞吐量 = T x N RU/秒；(b) 配置了所有能够处理写入的区域时，总吞吐量 = T x (N + 1) RU/每秒。 预配吞吐量（单个写入区域）每 100 RU/秒的费用为 0.051 元/小时，对于多个写入区域（配置多主数据库），每 100 RU/秒的费用为 0.102 元/小时（参阅[定价页面](https://www.azure.cn/pricing/details/cosmos-db/)）。 无论是单个写入区域还是多个写入区域，Azure Cosmos DB 都支持从任何区域读取数据。

### <a name="billing-example-multi-region-azure-cosmos-account-single-region-writes"></a>计费示例：多区域 Azure Cosmos 帐户，单个区域写入

假定存在位于中国北部的 Azure Cosmos 容器。 创建容器时，指定的吞吐量为 10K RU/秒，本月可存储 1 TB 的数据。 假定向 Azure Cosmos 帐户添加 3 个区域（中国东部、中国北部 2 和中国东部 2），每个区域都具有相同的存储和吞吐量。 则月度帐单为（假定一个月 31 天）。 帐单情况如下： 

|**项目** |**使用情况（月）** |**费率** |**每月成本** |
|---------|---------|---------|-------|
|中国北部容器的吞吐量帐单      | 10K RU/秒 * 24 * 31    |每 100 RU/秒每小时 0.051 元   |3,794.4 元|
|3 个其他区域 - 中国东部、中国北部 2 和中国东部 2 的吞吐量帐单       | 3 * 10K RU/秒 * 24 * 31    |每 100 RU/秒每小时 0.051 元  |11,383.2 元|
|中国北部容器的存储帐单      | 250 GB    |2\.576 元/GB  |644 元|
|3 个其他区域 - 中国东部、中国北部 2 和中国东部 2 的存储帐单      | 3 * 250 GB    |2\.576 元/GB  |1,932 元|
|**总计** |     |  |17,753.6 元|

此外，假定每月从中国北部的容器中传出 100 GB 数据，将数据复制到中国东部、中国北部 2 和中国东部 2。则需要按数据传输速率为导出部分付费。

### <a name="billing-example-multi-region-azure-cosmos-account-multi-region-writes"></a>计费示例：多区域 Azure Cosmos 帐户，多个区域写入

假定创建位于中国北部的 Azure Cosmos 容器。 创建容器时，指定的吞吐量为 10K RU/秒，本月可存储 1 TB 的数据。 假定添加 3 个区域（中国东部、中国北部 2 和中国东部 2），每个区域的存储和吞吐量都相同，并且你希望能够写入到与 Azure Cosmos 帐户关联的所有区域中的容器。 则月度帐单（假定一个月 31 天）情况如下：

<!--Should be China East, China North 2, China East 2-->

|**项目** |**使用情况（月）**|**费率** |**每月成本** |
|---------|---------|---------|-------|
|中国北部容器的吞吐量帐单（所有区域都可写入）       | 10K RU/秒 * 24 * 31    |每 100 RU/秒每小时 0.102 元    |7589 元 |
|3 个其他区域 - 中国东部、中国北部 2 和中国东部 2（所有区域均可写入）的吞吐量帐单        | (3 + 1) * 10K RU/秒 * 24 * 31    |每 100 RU/秒每小时 0.102 元   |30,355.2 元 |
|中国北部容器的存储帐单      | 250 GB    |2\.576 元/GB  |644 元|
|3 个其他区域 - 中国东部、中国北部 2 和中国东部 2 的存储帐单      | 3 * 250 GB    |2\.576 元/GB  |1,932 元|
|**总计**     |     |  |**40,520.2 元**|

此外，假定每月从中国北部的容器中传出 100 GB 数据，将数据复制到中国东部、中国北部 2 和中国东部 2。则需要按数据传输速率为导出部分付费。

<!--Customize-->

### <a name="billing-example-azure-cosmos-account-with-multi-master-database-level-throughput-including-dedicated-throughput-mode-for-some-containers"></a>计费示例：包含多主数据库的 Azure Cosmos 帐户，数据库级别的吞吐量包括某些容器的专用吞吐量模式

我们来看看以下示例，示例中包含一个多区域 Azure Cosmos 帐户，其中所有区域都可写入（多主数据库配置）。 为简单起见，假定存储大小保持不变，且不会更改，此处不考虑存储大小，从而简化示例。 本月的预配吞吐量变化如下（假定一月为 31 天或 744 小时）： 

<!--Region Mapping: West US       China North-->
<!--Region Mapping: East US       China East-->
<!--Region Mapping: North Europe  China North 2-->

[0-100 小时]：  

* 创建了包含 3 个区域的 Azure Cosmos 帐户（中国北部、中国东部和中国北部 2），其中所有区域都可写入 

* 创建共享吞吐量为 10K RU/秒的数据库 (D1) 

* 创建共享吞吐量为 30k RU/秒的数据库 (D2)  

* 创建专用吞吐量为 20K RU/秒的容器 (C1) 

[101-200 小时]：  

* 将数据库 (D1) 纵向扩展到 50K RU/秒 

* 将数据库 (D2) 纵向扩展到 70K RU/秒  

* 删除容器 (C1)  

[201-300 小时]：  

* 再次创建专用吞吐量为 20K RU/秒的容器 (C1) 

[301-400 小时]：  

* 从 Azure Cosmos 帐户中删除 1 个区域（可写入的区域的数量现为 2 个） 

* 将数据库 (D1) 缩小到 10K RU/秒 

* 将数据库 (D2) 纵向扩展到 80K RU/秒  

* 再次删除容器 (C1) 

[401-500 小时]：  

* 将数据库 (D2) 缩小到 10K RU/秒  

* 再次创建专用吞吐量为 20K RU/秒的容器 (C1) 

[501-700 小时]：  

* 将数据库 (D1) 纵向扩展到 20K RU/秒  

* 将数据库 (D2) 纵向扩展到 100K RU/秒  

* 再次删除容器 (C1)  

[701-744 小时]：  

* 将数据库 (D2) 缩小到 50K RU/秒  

下图直观地呈现了本月 744 小时内总预配吞吐量的变化情况： 

:::image type="content" source="./media/understand-your-bill/bill-example3.png" alt-text="Dedicated throughput bill example":::

将按以下方式计算月度总帐单（假定一个月为 31 天/744 小时）：

|**小时数** |**RU/秒** |**项目** |**使用时间（小时）** |**成本** |
|---------|---------|---------|-------|-------|
|[0-100] |D1:10K <br/>D2:30K <br/>C1:20K |中国北部容器的吞吐量帐单（所有区域都可写入）  | `D1: 10K RU/sec/100 * CNY0.102 * 100 hours = CNY1,020` <br/>`D2: 30 K RU/sec/100 * CNY0.102 * 100 hours = CNY3,060` <br/>`C1: 20 K RU/sec/100 *CNY0.102 * 100 hours = CNY2,040` |6,120 元  |
| | |2 个其他区域的吞吐量帐单：中国东部、中国北部2（所有区域均可写入）  |`(2 + 1) * (60 K RU/sec /100 * CNY0.102) * 100 hours = CNY18,360`  |18,360 元  |
|[101-200] |D1:50K <br/>D2:70K <br/>C1: -- |中国北部容器的吞吐量帐单（所有区域都可写入）  |`D1: 50 K RU/sec/100 * CNY0.102 * 100 hours = CNY5,100` <br/>`D2: 70 K RU/sec/100 * CNY0.102 * 100 hours = CNY7,140` |12,240 元  |
| | |2 个其他区域的吞吐量帐单：中国东部、中国北部2（所有区域均可写入）  |`(2 + 1) * (120 K RU/sec /100 * CNY0.102) * 100 hours = CNY36,720`  |36,720 元  |
|[201-300]  |D1:50K <br/>D2:70K <br/>C1:20K |中国北部容器的吞吐量帐单（所有区域都可写入）  |`D1: 50 K RU/sec/100 * CNY0.102 * 100 hours = CNY5,100` <br/>`D2: 70 K RU/sec/100 * CNY0.102 * 100 hours = CNY7,140` <br/>`C1: 20 K RU/sec/100 *CNY0.102 * 100 hours = CNY2,040` |14,280 元  |
| | |2 个其他区域的吞吐量帐单：中国东部、中国北部2（所有区域均可写入）  |`(2 + 1) * (140 K RU/sec /100 * CNY0.102) * 100 hours = CNY42,840` |42,840 元 |
|[301-400] |D1:10K <br/>D2:80K <br/>C1: -- |中国北部容器的吞吐量帐单（所有区域都可写入）  |`D1: 10K RU/sec/100 * CNY0.102 * 100 hours = CNY1,020` <br/>`D2: 80 K RU/sec/100 * CNY0.102 * 100 hours = CNY8,160`  |9,180 元   |
| | |2 个其他区域的吞吐量帐单：中国东部、中国北部2（所有区域均可写入）  |`(1 + 1) * (90 K RU/sec /100 * CNY0.102) * 100 hours = CNY18,360`  |18,360 元  |
|[401-500] |D1:10K <br />D2:10K <br />C1:20K |中国北部容器的吞吐量帐单（所有区域都可写入）  |`D1: 10K RU/sec/100 * CNY0.102 * 100 hours = CNY1,020` <br />`D2: 10K RU/sec/100 * CNY0.102 * 100 hours = CNY1,020` <br />`C1: 20 K RU/sec/100 *CNY0.102 * 100 hours = CNY2,040` |4,080 元  |
| | |2 个其他区域的吞吐量帐单：中国东部、中国北部2（所有区域均可写入）  |`(1 + 1) * (40 K RU/sec /100 * CNY0.102) * 100 hours = CNY8,160`  |8,160 元  |
|[501-700] |D1:20K <br />D2:100K <br />C1: -- |中国北部容器的吞吐量帐单（所有区域都可写入）  |`D1: 20 K RU/sec/100 * CNY0.102 * 200 hours = CNY4,080` <br />`D2: 100 K RU/sec/100 * CNY0.102 * 200 hours = CNY20,400` |24,480 元  |
| | |2 个其他区域的吞吐量帐单：中国东部、中国北部2（所有区域均可写入）  |`(1 + 1) * (120 K RU/sec /100 * CNY0.102) * 200 hours = CNY8,160`  |8,160 元  |
|[701-744] |D1:20K <br/>D2:50K <br/>C1: -- |中国北部容器的吞吐量帐单（所有区域都可写入）  |`D1: 20 K RU/sec/100 *CNY0.102 * 44 hours = CNY896.7` <br/>`D2: 50 K RU/sec/100 *CNY0.102 * 44 hours = CNY2,244` |3,140.7 元  |
| | |2 个其他区域的吞吐量帐单：中国东部、中国北部2（所有区域均可写入）  |`(1 + 1) * (70 K RU/sec /100 * CNY0.102) * 44 hours = CNY6,283.2‬`  |6,283.2‬ 元  |
|| |**每月总成本** | |**212,403.9 元** |

## <a name="billing-examples-with-free-tier-accounts"></a><a name="billing-examples-with-free-tier-accounts"></a>免费层帐户的计费示例
使用 Azure Cosmos DB 免费层，你将在帐户中获得前 400 RU/s 免费吞吐量和 5 GB 免费存储，并在帐户级别应用。 超过 400 RU/s 和 5 GB 的任何吞吐量和存储都将按定价页面的常规定价费率计费。 在账单上，你不会看到免费的 400 RU/s 和 5 GB 的费用或行项，而只会看到超过免费层涵盖范围的吞吐量和存储。 400 RU/s 适用于任何类型的吞吐量，包括预配吞吐量、自动缩放吞吐量和多主数据库吞吐量。  

### <a name="billing-example---container-or-database-with-provisioned-throughput"></a>计费示例 - 具有预配吞吐量的容器或数据库
- 假设我们在免费层帐户中创建一个具有 400 RU/s 吞吐量和 5 GB 存储的数据库或容器。
- 你的帐单不会显示此资源的任何费用。 你的每小时和每月成本将为 0 元。
- 现在，假设在同一个帐户中，我们添加了另一个具有 1000 RU/s 吞吐量和 10 GB 存储的数据库或容器。
- 现在帐单将显示 1000 RU/s 吞吐量和 10 GB 存储的费用。 

### <a name="billing-example---container-with-autoscale-throughput"></a>计费示例 - 具有自动缩放吞吐量的容器
- 假设在免费层帐户中，我们创建一个启用自动缩放功能的容器，最大吞吐量为 4000 RU/s。 此资源将自动在 400-4000 RU/s 吞吐量之间缩放。 
- 假设在 1-10 小时内，资源的吞吐量保持在最小值 400 RU/s。 在第 11 小时内，该资源的吞吐量扩展到 1000 RU/s，然后在一小时内回落到 400 RU/s。
- 在第 1-10 小时内，你需要支付的吞吐量费用为 0 元，因为免费层涵盖 400 RU/s。 
- 在第 11 小时，你需要支付 1000 RU/s - 400 RU/s = 600 RU/s 的吞吐量费用，因为这是一小时内的最大吞吐量。 该小时内的吞吐量为 6 个单位的 100 RU/s，因此总吞吐量成本为 6 个单位 * 0.123 元 = 0.738 元。 
- 超出前 5 GB 的任何存储都将按正常存储费率计费。 

### <a name="billing-example---multi-region-single-write-region-account"></a>计费示例 - 多区域单个写入区域帐户
- 假设我们在免费层帐户中创建一个具有 1200 RU/s 吞吐量和 10 GB 存储的数据库或容器。 我们将帐户复制到 3 个区域，并且有单主数据库（单个写入区域）帐户。
- 如果不使用免费层，我们共需要支付 3 * 1200 RU/s = 3600 RU/s 吞吐量和 3 * 10 GB = 30 GB 存储的费用。
- 使用免费层折扣后，在扣除 400 RU/s 吞吐量和 5 GB 存储后，我们需要支付 3200 RU/s（32 个单位）的有效预配吞吐量（按单个写入区域费率计算）和 25 GB 的存储费用。
- 每月 RU/s 成本为：32 个单位 * 0.051 元 * 24 小时 * 31 天 = 1,214.21 元。 每月存储成本为：25 GB * 2.576 / GB = 64.4 元。 总成本将为 1,214.21 元 + 64.4 元 = 1,278.61 元。
- 注意：如果各区域的吞吐量或存储单价不同，则免费层的 400 RU/s 吞吐量和 5 GB 存储将反映创建帐户所在区域的费率。

### <a name="billing-example---multi-region-multi-master-multiple-write-region-account"></a>计费示例 - 多区域多主数据库（多写入区域）帐户

此示例反映了在 2019 年 12 月 1 日之后创建的帐户的[多主数据库定价](https://www.azure.cn/pricing/details/cosmos-db/)。 
- 假设我们在免费层帐户中创建一个具有 1200 RU/s 吞吐量和 10 GB 存储的数据库或容器。 我们将帐户复制到 3 个区域，并且有多主数据库（多写入区域）帐户。 
- 如果不使用免费层，我们共需要支付 3 * 1200 RU/s = 3600 RU/s 吞吐量和 3 * 10 GB = 30 GB 存储的费用。
- 使用免费层折扣后，在扣除 400 RU/s 吞吐量和 5 GB 存储后，我们需要支付 3200 RU/s（32 个单位）的有效预配吞吐量（按多个写入区域费率计算）和 25 GB 的存储费用。
- 每月 RU/s 成本为：32 个单位 * 0.102 元 * 24 小时 * 31 天 = 2,428.5 元。 每月存储成本为：25 GB * 2.576 / GB = 64.4 元。 总成本将为 2,428.5 元 + 64.4 元 = 2,492.9 元。

<!--Customize-->

## <a name="proactively-estimating-your-monthly-bill"></a>主动估算每月帐单  

我们来看看另一个示例，介绍希望在月末之前主动估算帐单的情况。 可按如下方式估算帐单：

**存储成本**

* 平均记录大小 (KB) = 1 
* 记录数 = 100,000,000 
* 总存储 (GB) = 100 
* 每 GB 的月度成本 = 2.576 元 
* 预期的月度存储成本 = 257.6 元 

**吞吐量成本**

|操作类型| 请求数/秒| 平均值RU/请求| 所需 RU|
|----|----|----|----|
|写入| 100 | 5 | 500|
|读取| 400| 1| 400|

总 RU/秒：500 + 400 = 900；每小时成本：900/100 * 0.051 元 = 0.459 元；预期的月度吞吐量成本（假定每月 31 天）：0.459 元 * 24 * 31 = 341.50 元

**每月总成本**

月度总成本 = 月度存储成本 + 月度吞吐量成本；月度总成本 = 257.6 元 + 341.50 元 = 599.1 元

区域不同定价可能有所不同。有关最新定价，请参阅[定价页面](https://www.azure.cn/pricing/details/cosmos-db/)。

<!--Not Available on ## Billing with Azure Cosmos DB reserved capacity-->

## <a name="next-steps"></a>后续步骤

接下来，可通过以下文章了解 Azure Cosmos DB 中的成本优化：

* 详细了解 [Cosmos DB 定价模型如何对客户而言更具经济效益](total-cost-ownership.md)
* 详细了解[开发和测试优化](optimize-dev-test.md)
* 详细了解如何[优化吞吐量成本](optimize-cost-throughput.md)
* 详细了解如何[优化存储成本](optimize-cost-storage.md)
* 详细了解如何[优化读取和写入成本](optimize-cost-reads-writes.md)
* 详细了解如何[优化查询成本](optimize-cost-queries.md)
* 详细了解[优化多区域 Azure Cosmos 帐户的成本](optimize-cost-regions.md)

<!-- Update_Description: update meta properties, wording update, update link -->