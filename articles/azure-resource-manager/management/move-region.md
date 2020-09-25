---
title: 将 Azure 资源移到另一区域
description: 概述如何跨 Azure 区域移动 Azure 资源。
ms.service: azure-resource-manager
ms.topic: conceptual
origin.date: 09/10/2020
author: rockboyfor
ms.date: 09/21/2020
ms.testscope: yes|no
ms.testdate: 09/21/2020null
ms.author: v-yeche
ms.openlocfilehash: 4716a3d7cf1fc87c83c6eb3d9787c5d26a87fb7d
ms.sourcegitcommit: 41e986cd4a2879d8767dc6fc815c805e782dc7e6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/20/2020
ms.locfileid: "90822395"
---
# <a name="moving-azure-resources-across-regions"></a>跨区域移动 Azure 资源

本文提供有关跨 Azure 区域移动 Azure 资源的信息。

Azure 地理位置和区域构成了 Azure 全球基础结构的基础。 Azure [地理位置](https://azure.microsoft.com/global-infrastructure/geographies/)通常包含两个或更多个 [Azure 区域](https://azure.microsoft.com/global-infrastructure/regions/)。 区域是一个地理位置和多个数据中心内的地区。 

<!--Not Avaialble on  and Availability Zones-->
<!--Not Available on containing Availability Zones, -->
<!--MOONCAKE: CORRECT ON https://azure.microsoft.com/global-infrastructure/geographies/-->

在特定的 Azure 区域中部署资源后，你可能会出于以下多种原因，需要将资源移到其他区域。

- **与区域的推出相适应**：将资源移到新推出的 Azure 区域（以前不存在此区域）。
- **与服务/功能相适应**：移动资源以利用特定区域中提供的服务或功能。
- **应对业务发展**：将资源移到某个区域，以应对企业变化，例如合并或收购。
- **确保邻近性**：将资源移到企业的本地区域。
- **满足数据要求**：移动资源以符合数据驻留要求或数据分类需求。 [了解详细信息](https://azure.microsoft.com/mediahandler/files/resourcefiles/achieving-compliant-data-residency-and-security-with-azure/Achieving_Compliant_Data_Residency_and_Security_with_Azure.pdf)。
    
    <!--MOONCAKE: CORRECT ON https://azure.microsoft.com/global-infrastructure/geographies/-->
    
- **应对部署要求**：移动错误部署的资源，或者移动资源来应对容量需求。 
- **应对解除**：由于区域解除而移动资源。

<!--Not Available on ## Move resources with Resource Mover-->
## <a name="region-move-process"></a>区域移动过程

跨区域移动资源的实际过程取决于要移动的资源。 但是，有一些通用的关键步骤：

1. **验证先决条件**：先决条件包括，确保所需的资源在目标区域中可用，检查是否有足够的配额，以及验证订阅是否可以访问目标区域。
2. **分析依赖关系**：你的资源可能依赖于其他资源。 在移动之前，请先查明依赖关系，以便资源在移动后可以继续按预期方式工作。
3. **准备移动**：在移动之前，需在主要区域中执行一些步骤。 例如，可能需要导出 Azure 资源管理器模板，或开始将资源从源复制到目标。
4. **移动资源**：移动资源的方式取决于具体的资源是什么。 可能需要在目标区域中部署模板，或者将资源故障转移到目标区域。
5. **丢弃目标资源**：移动资源后，可能需要查看这些资源是否已进入目标区域，并确定是否存在不需要的任何资源。
6. **提交移动操作**：确认资源进入目标区域后，某些资源可能需要最终的提交操作。 例如，在现已成为主要区域的目标区域中，可能需要设置灾难恢复到新的次要区域。 
7. **清理源**：最后，在新区域中的所有活动正常运转后，可以清理并解除为移动过程创建的资源，以及主要区域中的资源。

## <a name="next-steps"></a>后续步骤

有关支持跨区域移动的资源列表，请参阅[资源的移动操作支持](region-move-support.md)。

<!-- Update_Description: update meta properties, wording update, update link -->
<!--NEW.date: 12/09/2019-->
