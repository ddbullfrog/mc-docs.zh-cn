---
title: Azure 安全中心的容器安全性 | Azure
description: 了解 Azure 安全中心的容器安全功能。
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/09/2020
ms.author: v-tawe
origin.date: 02/11/2020
ms.openlocfilehash: c42c45abaa96b0afe094b961b880c06e187875a7
ms.sourcegitcommit: cdb7228e404809c930b7709bcff44b89d63304ec
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/28/2020
ms.locfileid: "91402624"
---
# <a name="container-security-in-security-center"></a>安全中心的容器安全性

Azure 安全中心是针对容器安全性提供的 Azure 原生解决方案。 安全中心还是最理想的管理工具，可用于集中管理云工作负荷、VM、服务器和容器的安全性。

本文介绍如何通过安全中心来提高、监视和维护容器及其应用的安全性。 你将了解 Azure 安全中心如何在容器安全性的以下核心方面提供帮助：

* 漏洞管理
* 强化容器的环境
* 运行时保护

[![Azure 安全中心的“容器安全性”选项卡](media/container-security/container-security-tab.png)](media/container-security/container-security-tab.png#lightbox)

## <a name="vulnerability-management---scanning-container-images"></a>漏洞管理 - 扫描容器映像
若要监视基于 ARM 的 Azure 容器注册表，请确保你在安全中心的标准层（请参阅[定价](https://docs.azure.cn/security-center/security-center-pricing)）上。 然后，启用可选的容器注册表捆绑包。 推送新映像时，安全中心会使用行业领先的漏洞扫描供应商 Qualys 提供的扫描程序来扫描映像。

当 Qualys 或安全中心发现问题时，你会在安全中心仪表板中收到通知。 安全中心会针对每个漏洞提供可行的建议、严重性分类，以及有关如何修正问题的指南。 若要详细了解安全中心针对容器提供的建议，请参阅[建议的参考列表](recommendations-reference.md#recs-containers)。

安全中心会对扫描程序的扫描结果进行筛选和分类。 当映像正常运行时，安全中心会将其标为正常。 安全中心仅为存在待解决问题的映像生成安全建议。 安全中心仅在出现问题时发出通知，这样会降低发送不必要的信息警报的可能性。

## <a name="environment-hardening"></a>环境强化

### <a name="continuous-monitoring-of-your-docker-configuration"></a>持续监视 Docker 配置
Azure 安全中心会识别在 IaaS Linux VM 上或其他运行 Docker 容器的 Linux 计算机上承载的非托管容器。 安全中心会持续评估这些容器的配置。 然后，它会将其与 [Internet 安全中心 (CIS) 的 Docker 基准](https://www.cisecurity.org/benchmark/docker/)进行比较。

安全中心包含 CIS 的 Docker 基准的完整规则集，并会在容器不符合控件标准的情况下发出警报。 在发现错误配置时，安全中心会生成安全建议。 请使用**建议页面**来查看建议和修正问题。 也可在“容器”选项卡上查看建议，该选项卡会显示使用 Docker 部署的所有虚拟机。 

若要详细了解可能会针对此功能显示的相关的安全中心建议，请参阅建议参考表的[容器部分](recommendations-reference.md#recs-containers)。

浏览 VM 的安全问题时，安全中心会提供计算机上有关容器的其他信息。 此类信息包括 Docker 版本以及主机上运行的映像数。 

>[!NOTE]
> 这些 CIS 基准检查不会在 AKS 托管实例或 Databricks 托管 VM 上运行。

### <a name="continuous-monitoring-of-your-kubernetes-clusters"></a>持续监视 Kubernetes 群集
安全中心可以与 Azure Kubernetes 服务 (AKS) 协同工作，后者是 Microsoft 的托管容器业务流程服务，用于开发、部署和管理容器化应用程序。

AKS 提供安全控制，并且可用于了解群集的安全状况。 安全中心使用这些功能来完成以下任务：
* 持续监视 AKS 群集的配置
* 生成符合行业标准的安全建议

若要详细了解可能会针对此功能显示的相关的安全中心建议，请参阅建议参考表的[容器部分](recommendations-reference.md#recs-containers)。

## <a name="run-time-protection---real-time-threat-detection"></a>运行时保护 - 实时检测威胁

安全中心为容器化环境提供实时检测威胁的功能，并在出现可疑活动时生成警报。 可以使用此信息快速补救安全问题，并提高容器的安全性。

我们在主机和 AKS 群集级别检测威胁。 如需完整的详细信息，请参阅[针对 Azure 容器进行威胁检测](threat-protection.md)。


## <a name="container-security-faq"></a>容器安全性常见问题解答

### <a name="what-types-of-images-can-azure-security-center-scan"></a>Azure 安全中心可以扫描哪些类型的映像？
安全中心扫描基于 Linux OS 的映像，这些映像提供 shell 访问权限。 

Qualys 扫描程序不支持极简映像，例如 [Docker 暂存](https://hub.docker.com/_/scratch/)映像或仅包含应用程序及其运行时依赖项而没有包管理器、shell 或 OS 的“Distroless”映像。

### <a name="how-does-azure-security-center-scan-an-image"></a>Azure 安全中心如何扫描映像？
先从注册表中拉取映像。 然后在独立沙盒中运行该映像，沙盒中包含的 Qualys 扫描程序会提取已知漏洞的列表。

安全中心会对扫描程序的扫描结果进行筛选和分类。 当映像正常运行时，安全中心会将其标为正常。 安全中心仅为存在待解决问题的映像生成安全建议。 安全中心仅在出现问题时发出通知，这样会降低发送不必要的信息警报的可能性。

### <a name="how-often-does-azure-security-center-scan-my-images"></a>Azure 安全中心扫描映像的频率如何？
每次推送都会触发映像扫描。

### <a name="can-i-get-the-scan-results-via-rest-api"></a>是否可以通过 REST API 获取扫描结果？
是的。 结果位于[子评估 Rest API](https://docs.microsoft.com/rest/api/securitycenter/subassessments/list/) 下。 此外，还可以对所有资源使用 Azure Resource Graph (ARG)，一个类似于 Kusto 的 API：查询可以提取特定扫描。
 

## <a name="next-steps"></a>后续步骤

若要详细了解 Azure 安全中心的容器安全性，请参阅以下相关文章：

* 若要查看容器相关资源的安全状况，请参阅[保护计算机和应用程序](security-center-virtual-machine-protection.md#containers)的容器部分。

* [与 Azure Kubernetes 服务集成](azure-kubernetes-service-integration.md)的详细信息

