---
title: 移动 Azure 应用服务资源
description: 使用 Azure 资源管理器将应用服务资源移到新的资源组或订阅。
ms.topic: conceptual
origin.date: 08/10/2020
author: rockboyfor
ms.date: 10/12/2020
ms.testscope: yes
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: e04e64dc7f998accc177b10360db595fffa12cdf
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937194"
---
# <a name="move-guidance-for-app-service-resources"></a>针对应用服务资源的移动指南

本文介绍移动应用服务资源的步骤。 将应用服务资源移到新订阅有特定要求。

## <a name="move-across-subscriptions"></a>跨订阅移动

在订阅之间移动 Web 应用时，应遵循以下指导：

- 目标资源组中不能有任何现有的应用服务资源。 应用服务资源包括：
    - Web 应用
    - 应用服务计划
    - 上传或导入的 TLS/SSL 证书
    - 应用服务环境
- 资源组中的所有应用服务资源必须一起移动。
- 应用服务环境不能移到新资源组，也不能移到新订阅。 但是，可以将某个 Web 应用和应用服务计划转移到新订阅，而无需移动应用服务环境。 移动后，Web 应用不再在应用服务环境中托管。
- 只要将证书与资源组中的所有其他资源一起移动，就可以将绑定的证书移到 Web，而无需删除 TLS 绑定。
- 只能从最初创建应用服务资源的资源组中移动它们。 如果应用服务资源不再位于其原始资源组中，请将其移回其原始资源组。 然后，在订阅之间移动资源。

<!--MOONCAKE: Not Available on **Diagnose and solve problems**-->
<!--Not Available on Select **Migration Options**.-->
<!--Not Available on Select the option for recommended steps to move the web app.-->
<!--Not Available on You see the recommended actions to take before moving the resources.-->

## <a name="move-support"></a>移动支持

若要确定可以移动哪些应用服务资源，请查看以下项的移动支持状态：

<!--Not Available on - [Microsoft.AppService](../move-support-resources.md#microsoftappservice)-->
<!--Not Available on - [Microsoft.CertificateRegistration](../move-support-resources.md#microsoftcertificateregistration)-->
<!--Not Available on - [Microsoft.DomainRegistration](../move-support-resources.md#microsoftdomainregistration)-->

- [Microsoft.Web](../move-support-resources.md#microsoftweb)

## <a name="next-steps"></a>后续步骤

有关用于移动资源的命令，请参阅[将资源移到新资源组或订阅](../move-resource-group-and-subscription.md)。

<!-- Update_Description: update meta properties, wording update, update link -->