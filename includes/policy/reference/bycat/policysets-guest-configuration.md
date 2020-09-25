---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
origin.date: 09/10/2020
ms.date: 09/15/2020
ms.author: v-tawe
ms.custom: generated
ms.openlocfilehash: 93b59280d7b2fd6286ed799d5a51af44795e135a
ms.sourcegitcommit: f5d53d42d58c76bb41da4ea1ff71e204e92ab1a7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/15/2020
ms.locfileid: "90523618"
---
|名称 |说明 |策略 |版本 |
|---|---|---|---|
|[审核具有不安全密码安全设置的计算机](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_WindowsPasswordSettingsAINE.json) |此计划部署策略要求，并审核具有不安全密码安全设置的计算机。 有关 Guest Configuration 策略的详细信息，请访问 [https://docs.azure.cn/governance/policy/concepts/guest-configuration](https://docs.azure.cn/governance/policy/concepts/guest-configuration) |9 |1.0.0 |
|[部署先决条件以在虚拟机上启用来宾配置策略](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_Prerequisites.json) |此计划添加系统分配的托管标识，并向符合来宾配置策略监视条件的虚拟机部署平台相应的来宾配置扩展。 这是所有来宾配置策略的先决条件，在使用任何来宾配置策略之前必须被分配到策略分配范围。 有关来宾配置的详细信息，请访问 [https://docs.azure.cn/governance/policy/concepts/guest-configuration](https://docs.azure.cn/governance/policy/concepts/guest-configuration)。 |4 |1.0.0-preview |
|[Windows 计算机应符合 Azure 安全基线的要求](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_AzureBaseline.json) |此计划审核具有不符合 Azure 安全基线的设置的 Windows 计算机。 有关详细信息，请访问 [https://docs.azure.cn/governance/policy/concepts/guest-configuration](https://docs.azure.cn/governance/policy/concepts/guest-configuration) |29 |2.0.0-preview |
