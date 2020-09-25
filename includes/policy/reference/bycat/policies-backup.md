---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
origin.date: 09/10/2020
ms.date: 09/15/2020
ms.author: v-tawe
ms.custom: generated
ms.openlocfilehash: a9f9db1b6bf9b9b236ce899f720c733b875bbdf5
ms.sourcegitcommit: f5d53d42d58c76bb41da4ea1ff71e204e92ab1a7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/15/2020
ms.locfileid: "90524012"
---
|名称<br /><sub>（Azure 门户）</sub> |说明 |效果 |版本<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[应为虚拟机启用 Azure 备份](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F013e242c-8828-4970-87b3-ab247555486d) |通过启用 Azure 备份，确保对 Azure 虚拟机进行保护。 Azure 备份是一种安全且经济高效的数据保护解决方案，适用于 Azure。 |AuditIfNotExists、Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Backup/VirtualMachines_EnableAzureBackup_Audit.json) |

<!-- |[Configure backup on VMs of a location to an existing central Vault in the same location](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F09ce66bc-1220-4153-8104-e3f51c936913) |This policy configures Azure Backup protection on VMs in a given location to an existing central vault in the same location. It applies to only those VMs that are not already configured for backup. It is recommended that this policy is assigned to not more than 200 VMs. If the policy is assigned for more than 200 VMs, it can result in the backup getting triggered a few hours beyond the defined schedule. This policy will be enhanced to support more VM images. |deployIfNotExists, auditIfNotExists, disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Backup/VirtualMachineBackup_Backup_DeployIfNotExists.json) | -->
<!-- |[Deploy Diagnostic Settings for Recovery Services Vault to Log Analytics workspace for resource specific categories.](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc717fb0c-d118-4c43-ab3d-ece30ac81fb3) |Deploy Diagnostic Settings for Recovery Services Vault to stream to Log Analytics workspace for Resource specific categories. If any of the Resource specific categories are not enabled, a new diagnostic setting is created. |deployIfNotExists |[1.0.1-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Backup/EnableRecoveryServiceVaultDiagnosticSetting_Backup_DeployIfNotExist.json) | -->
