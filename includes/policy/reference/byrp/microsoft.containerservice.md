---
ms.service: azure-policy
ms.topic: include
origin.date: 09/10/2020
author: rockboyfor
ms.date: 09/25/2020
ms.testscope: yes|no
ms.testdate: 09/21/2020null
ms.author: v-yeche
ms.custom: generated
ms.openlocfilehash: b50d4ff2823b399aba64799723d0b45463f99f0c
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246598"
---
<!--Verified successfully on 09/21/2020-->

<!--ALL THE PREVIEWS POLICY IS INVALID ON MOONCAKE-->

|名称<br /><sub>（Azure 门户）</sub> |说明 |效果 |版本<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[应在 Kubernetes 服务上定义经授权的 IP 范围](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0e246bcf-5f6f-4f87-bc6f-775d4712c7ea) |通过仅向特定范围内的 IP 地址授予 API 访问权限，来限制对 Kubernetes 服务管理 API 的访问。 建议将访问权限限制给已获授权的 IP 范围，以确保只有受允许网络中的应用程序可以访问群集。 |Audit、Disabled |[2.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableIpRanges_KubernetesService_Audit.json) |
|[Kubernetes 服务应升级到不易受攻击的 Kubernetes 版本](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffb893a29-21bb-418c-a157-e99480ec364c) |将 Kubernetes 服务群集升级到更高 Kubernetes 版本，以抵御当前 Kubernetes 版本中的已知漏洞。 Kubernetes 版本 1.11.9+、1.12.7+、1.13.5+ 和 1.14.0+ 中已修补漏洞 CVE-2019-9946 |Audit、Disabled |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_UpgradeVersion_KubernetesService_Audit.json) |
|[应在 Kubernetes 服务中使用基于角色的访问控制 (RBAC)](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fac4a19c2-fa67-49b4-8ae5-0b2e78c49457) |若要对用户可以执行的操作提供粒度筛选，请使用基于角色的访问控制 (RBAC) 来管理 Kubernetes 服务群集中的权限并配置相关授权策略。 |Audit、Disabled |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableRBAC_KubernetesService_Audit.json) |

<!-- Update_Description: new article about microsoft.containerservice -->
<!--NEW.date: 09/21/2020-->