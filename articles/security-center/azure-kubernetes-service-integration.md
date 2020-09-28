---
title: Azure 安全中心与 Azure Kubernetes 服务
description: 了解 Azure 安全中心与 Azure Kubernetes 服务的集成
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/13/2020
ms.author: v-tawe
origin.date: 11/04/2019
ms.openlocfilehash: 71a70342af57c7a2af02eb123183484d6c853339
ms.sourcegitcommit: cdb7228e404809c930b7709bcff44b89d63304ec
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/28/2020
ms.locfileid: "91402579"
---
# <a name="azure-kubernetes-services-integration-with-security-center"></a>Azure Kubernetes 服务与安全中心的集成

Azure Kubernetes 服务 (AKS) 是 Microsoft 的托管服务，用于开发、部署和管理容器化应用程序。 

结合使用 AKS 和 Azure 安全中心的标准层（请参阅[定价](security-center-pricing.md)），以便更深入地了解 AKS 节点、云流量和安全控制。

安全中心使用已由 AKS 主节点收集的数据为 AKS 群集带来了安全性优势。 

![Azure 安全中心与 Azure Kubernetes 服务 (AKS) 综合概述](./media/azure-kubernetes-service-integration/aks-asc-integration-overview.png)

这两个工具共同构成了最优秀的云原生 Kubernetes 安全产品/服务。 

## <a name="benefits-of-integration"></a>集成的好处

同时使用这两个服务可以提供：

* **安全建议** - 安全中心标识 AKS 资源并对其进行分类：从群集到单个虚拟机。 然后，可以查看每个资源的安全建议。 有关详细信息，请参阅[建议的参考列表](recommendations-reference.md#recs-computeapp)中的容器建议。 

* **环境强化** - 安全中心持续监视 Kubernetes 群集的配置和 Docker 配置。 然后，它将生成反映行业标准的安全建议。

* **运行时保护** - 通过连续分析以下 AKS 源，安全中心会提醒你主机上检测到的威胁和恶意活动，还会提醒你 AKS 群集级别：
    * 原始安全事件（如网络数据和进程创建）
    * Kubernetes 审核日志

    有关详细信息，请参阅[针对 Azure 容器的威胁防护](threat-protection.md)

    有关可能出现警报的列表，请参阅警报引用表中的以下部分：[AKS 群集级别警报](alerts-reference.md#alerts-akscluster)和[容器主机级别警报](alerts-reference.md#alerts-containerhost)。  

![详细了解 Azure 安全中心与 Azure Kubernetes 服务 (AKS)](./media/azure-kubernetes-service-integration/aks-asc-integration-detailed.png)

> [!NOTE]
> Azure 安全中心从 Kubernetes 环境扫描的某些数据可能会包含敏感信息。


## <a name="next-steps"></a>后续步骤

若要详细了解安全中心的容器安全功能，请参阅：

* [Azure 安全中心和容器安全性](container-security.md)

* [Microsoft 的数据管理](https://www.microsoft.com/trust-center/privacy/data-management) - 描述 Microsoft 服务（包括 Azure、Intune 和 Office 365）的数据策略、Microsoft 数据管理的详细信息以及影响数据的保留策略
