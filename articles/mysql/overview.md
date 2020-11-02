---
title: 概述 - Azure Database for MySQL
description: 了解 Azure Database for MySQL 服务，这是 Microsoft 云中基于 MySQL 社区版的关系数据库服务。
author: WenJason
ms.service: mysql
ms.author: v-jay
ms.custom: mvc
ms.topic: overview
origin.date: 3/18/2020
ms.date: 10/29/2020
ms.openlocfilehash: 08879d17ffeaa5dbd570e63edf202a1e26c2a558
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92470486"
---
# <a name="what-is-azure-database-for-mysql"></a>什么是用于 MySQL 的 Azure 数据库？

> [!NOTE] 
> 将要查看的是 Azure Database for MySQL 的新服务。 若要查看经典 MySQL Database for Azure 的文档，请访问[此页](https://docs.azure.cn/zh-cn/mysql-database-on-azure/)。

Azure Database for MySQL 是 Azure 云中基于 [MySQL 社区版](https://www.mysql.com/products/community/)（可在 GPLv2 许可证下使用）数据库引擎版本 5.6、5.7 和 8.0 的一种关系数据库服务。 用于 MySQL 的 Azure 数据库提供以下功能：

- 内置的高可用性。
- 使用自动备份和时间点还原对数据进行长达 35 天的保护。
- 自动维护基础硬件、操作系统和数据库引擎，使服务保持安全和最新状态。
- 使用非独占预付费定价，实现可预测性能。
- 在几秒钟内实现弹性缩放。
- 具有企业级安全性和行业领先的符合性，可保护静态和动态敏感数据。
- 具有监视和自动化功能，可简化大规模部署的管理和监视。
- 行业领先的支持体验。

这些功能几乎都不需要进行任何管理，并且都是在不另外收费的情况下提供的。 借助这些功能，用户可将注意力集中在如何快速进行应用开发、加快推向市场，而不需要投入宝贵的时间和资源来管理虚拟机与基础结构。 此外，可以继续使用选择的开源工具和平台来开发应用程序，以提供业务所需的速度和效率，这些都不需要学习新技能。

:::image type="content" source="media/overview/1-azure-db-for-mysql-conceptual-diagram.png" alt-text="用于 MySQL 的 Azure 数据库概念图":::

### <a name="azure-database-for-mysql"></a>Azure Database for MySQL

Azure Database for MySQL 单一服务器是一项完全托管的数据库服务，对数据库自定义的要求最低。 单一服务器平台旨在以最少的用户配置和控制来处理大多数数据库管理功能，例如修补、备份、高可用性、安全性。 此体系结构已进行优化，提供内置的高可用性，在单个区域的可用性为 99.99%。 它支持 MySQL 社区版 5.6、5.7 和 8.0。 目前，该服务已在各种 [Azure 区域](https://azure.microsoft.com/global-infrastructure/services/?regions=china-non-regional,china-east,china-east-2,china-north,china-north-2&products=all)中正式发布。

“单一服务器”部署选项提供三个定价层：“基本”、“常规用途”和“内存优化”。 每个层提供不同的资源功能以支持数据库工作负荷。 可以在一个月内花费很少的费用基于小型数据库构建第一个应用，然后根据解决方案的需求调整规模。 动态可伸缩性使得数据库能够以透明方式对不断变化的资源需求做出响应。 只需在需要资源时为所需的资源付费。 有关详细信息，请参阅[定价层](concepts-pricing-tiers.md)。

单一服务器最适合用于云原生应用程序，这些应用程序旨在处理自动修补，而无需对修补计划和自定义 MySQL 配置设置进行精细控制。 

有关单一服务器部署模式的详细概述，请参阅[单一服务器概述](single-server-overview.md)。

## <a name="contacts"></a>联系人
有关使用 Azure Database for MySQL 的任何问题或建议，请根据需要考虑以下联系点：

- 若要联系 Azure 支持，请[从 Azure 门户提交票证](https://portal.azure.cn/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。
- 若要修复帐户问题，请提交[支持请求](https://support.azure.cn/zh-cn/support/support-azure/)。

## <a name="next-steps"></a>后续步骤

详细了解 Azure Database for MySQL，并根据需要选择合适的选项。

- [单一服务器](single-server/index.yml)
- [为工作负载选择正确的 MySQL 部署选项](select-right-deployment-type.md)
