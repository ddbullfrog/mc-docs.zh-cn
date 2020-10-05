---
author: rockboyfor
ms.service: azure-policy
ms.topic: include
origin.date: 07/22/2020
ms.date: 08/10/2020
ms.testscope: yes
ms.testdate: 08/10/2020
ms.author: v-yeche
ms.custom: generated
ms.openlocfilehash: 99ebecae16dafa5bae12dcf0dd9a2aae433a34ba
ms.sourcegitcommit: fce0810af6200f13421ea89d7e2239f8d41890c0
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/06/2020
ms.locfileid: "91146289"
---
<!--Verifed successfully-->
## <a name="azure-security-benchmark"></a>Azure 安全基准

[Azure 安全基准](../../../../articles/security/benchmarks/overview.md)提供有关如何在 Azure 上保护云解决方案的建议。 若要查看此服务如何完全映射到 Azure 安全基准，请参阅 [Azure 安全基准映射文件](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines)。

<!--Not Available on [Azure Policy Regulatory Compliance - Azure Security Benchmark](../../../../articles/governance/policy/samples/azure-security-benchmark.md)-->

|域 |控制 ID |控制标题 |策略<br /><sub>（Azure 门户）</sub> |Policy 版本<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|网络安全 |1.1 |在虚拟网络上使用网络安全组或 Azure 防火墙来保护资源 |[应在 Kubernetes 服务上定义经授权的 IP 范围](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0e246bcf-5f6f-4f87-bc6f-775d4712c7ea) |[1.0.1-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableIpRanges_KubernetesService_Audit.json) |
|数据保护 |4.6 |使用 Azure RBAC 控制对资源的访问 |[应在 Kubernetes 服务中使用基于角色的访问控制 (RBAC)](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fac4a19c2-fa67-49b4-8ae5-0b2e78c49457) |[1.0.1-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableRBAC_KubernetesService_Audit.json) |
|漏洞管理 |5.3 |部署第三方软件修补程序自动化管理解决方案 |[Kubernetes 服务应升级到不易受攻击的 Kubernetes 版本](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffb893a29-21bb-418c-a157-e99480ec364c) |[1.0.1-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_UpgradeVersion_KubernetesService_Audit.json) |
|安全配置 |7.3 |维护安全的 Azure 资源配置 |[[预览]：应在 Kubernetes 服务上定义 Pod 安全策略](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3abeb944-26af-43ee-b83d-32aaf060fb94) |[1.0.0-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnablePSP_KubernetesService_Audit.json) |
|安全配置 |7.9 |针对 Azure 服务实现自动化配置监视 |[[预览]：应在 Kubernetes 服务上定义 Pod 安全策略](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3abeb944-26af-43ee-b83d-32aaf060fb94) |[1.0.0-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnablePSP_KubernetesService_Audit.json) |

## <a name="cis-azure-foundations-benchmark"></a>CIS Azure 基础基准检验

<!--Not Available on [Azure Policy Regulatory Compliance - CIS Azure Foundations Benchmark 1.1.0](../../../../articles/governance/policy/samples/cis-azure-1-1-0.md)-->

有关此符合性标准的详细信息，请参阅 [CIS Azure 基础基准检验](https://www.cisecurity.org/benchmark/azure/)。

|域 |控制 ID |控制标题 |策略<br /><sub>（Azure 门户）</sub> |Policy 版本<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|其他安全注意事项 |8.5 |在 Azure Kubernetes 服务中启用基于角色的访问控制 (RBAC) |[应在 Kubernetes 服务中使用基于角色的访问控制 (RBAC)](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fac4a19c2-fa67-49b4-8ae5-0b2e78c49457) |[1.0.1-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableRBAC_KubernetesService_Audit.json) |

## <a name="nist-sp-800-171-r2"></a>NIST SP 800-171 R2

<!--Not Available on [Azure Policy Regulatory Compliance - NIST SP 800-171 R2](../../../../articles/governance/policy/samples/nist-sp-800-171-r2.md)-->

有关此符合性标准的详细信息，请参阅 [NIST SP 800-171 R2](https://csrc.nist.gov/publications/detail/sp/800-171/rev-2/final)。

|域 |控制 ID |控制标题 |策略<br /><sub>（Azure 门户）</sub> |Policy 版本<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|系统和信息完整性 |3.14.1 |及时识别、报告和更正系统缺陷。 |[Kubernetes 服务应升级到不易受攻击的 Kubernetes 版本](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffb893a29-21bb-418c-a157-e99480ec364c) |[1.0.1-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_UpgradeVersion_KubernetesService_Audit.json) |

<!-- Update_Description: new article about microsoft.containerservice -->
<!--NEW.date: 08/10/2020-->
