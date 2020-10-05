---
title: Azure Resource Graph 中的顾问数据
description: 对 Azure Resource Graph 中的顾问数据进行查询
ms.topic: article
ms.date: 09/22/2020
ms.author: v-johya
origin.date: 03/12/2020
ms.openlocfilehash: 471986713ab841449a2830baa09c1915ecf5e84d
ms.sourcegitcommit: cdb7228e404809c930b7709bcff44b89d63304ec
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/28/2020
ms.locfileid: "91402652"
---
# <a name="query-for-advisor-data-in-resource-graph-explorer-azure-resource-graph"></a>在 Resource Graph 资源管理器 (Azure Resource Graph) 中查询顾问数据

顾问资源现已加入到 [Azure Resource Graph](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyMenuBlade/ResourceGraph)。 这为顾问建议的许多大规模客户方案奠定了基础。 以前无法大规模实现但现在可以使用 Resource Graph 来实现的少数方案如下：
* 提供在 Azure 门户中对你的所有订阅执行复杂查询的功能
* 按类别类型（如可靠性、性能）和影响类型（高、中、低）汇总的建议
* 特定建议类型的所有建议
* 按建议类别列出的受影响资源计数

![Azure Resource Graph 资源管理器中的顾问](./media/azure-resource-graph-1.png)  


## <a name="advisor-resource-types-in-azure-graph"></a>Azure Graph 中的顾问资源类型

[Resource Graph](../governance/resource-graph/index.yml) 中的可用顾问资源类型：在顾问资源下有 3 种资源类型可供查询。 下面是现在可在 Resource Graph 中查询的资源列表。
* Microsoft.Advisor/configurations
* Microsoft.Advisor/recommendations
* Microsoft.Advisor/suppressions

这些资源类型列在名为“AdvisorResources”的新表下，你也可以在 Azure 门户的 Resource Graph 资源管理器中查询该表。


## <a name="next-steps"></a>后续步骤

有关顾问建议的详细信息，请参阅以下资源：
* [Azure 顾问简介](advisor-overview.md)
* [顾问入门](advisor-get-started.md)
* [顾问成本建议](advisor-cost-recommendations.md)
* [顾问可靠性建议](advisor-high-availability-recommendations.md)
* [顾问性能建议](advisor-performance-recommendations.md)
* [顾问安全性建议](advisor-security-recommendations.md)
* [顾问卓越运营建议](advisor-operational-excellence-recommendations.md)
* [顾问 REST API](https://docs.microsoft.com/rest/api/advisor/)

